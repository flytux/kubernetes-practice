### 1) Install Airflow helm chart

- DAG 폴더를 PVC로 설정하여 NAS 디렉토리를 통해서 update
- helm upgrade -i airflow -f values.yaml airflow-1.15.0.tgz -n airflow --create-namespace

```
# values.yaml

ingress:
  web:
    enabled: true
    host: "airflow.kw01"
    ingressClassName: "nginx"

dags:
  persistence:
    enabled: true
    size: 1Gi
    storageClassName: nfs-csi
    accessMode: ReadWriteMany
```

### 2) Config DAG persistent volume (NFS-CSI)

- NAS 스토리지 클래스 설치

### 3) Add Kubectl RBAC 

- Provide create, get, list, delete pod/deploy/statefulsets to default serviceaccount of airflow namespace
- apply on airflow namespace
```
# sa.yaml
# k apply -f sa.yaml

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: airflow
  name: clusterrole-airflow-kubectl
rules:
  - apiGroups: [""]
    resources:
      - pods
    verbs:
      - create
      - get
      - list
      - delete
      - watch
  - apiGroups: ["apps"]
    resources:
      - deployments
      - deployments/scale
      - replicasets
      - replicasets/scale
      - statefulsets
      - statefulsets/scale
    verbs:
      - create
      - get
      - list
      - patch
      - update
      - watch
      - scale
      - delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: clusterrole-bidning-airflow-kubectl
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: clusterrole-airflow-kubectl
subjects:
  - kind: ServiceAccount
    name: default
    namespace: airflow
```

### 4) Create DAG for kubectl command 

- create two kubernetes clusters
- mount KUBECONFIG as secret in kubernetesPodOperator

```
# airflow 네임스페이스에 secret 생성
# 2nd cluster용 kubeconfig
# kubeconfig 설정 없으면, 자체 클러스터로 접속
$ k create secret generic kubeconfig-svc -n airflow --from-file=node-02.config

```

- execute kubectl from kubectl pod

```
from pendulum import datetime
from airflow import DAG
# Secret 사용 설정
from airflow.kubernetes.secret import Secret
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import (
    KubernetesPodOperator,
)

with DAG(
    dag_id="kubectl-exectutor",
) as dag:

    # 자체 클러스터에 ngnix 디플로이 삭제
    delete_nginx = KubernetesPodOperator(
        image="bitnami/kubectl:1.30.4",
        name="delete-nginx",
        task_id="delete-nginx",
        cmds=["/bin/sh"],
        arguments=["-c", "kubectl delete deployment nginx-v2 --namespace=airflow"],
        labels={"foo": "bar"},
        is_delete_operator_pod=True,
        get_logs=True,
    )

    # 자체 클러스터에 ngnix 디플로이 생성
    create_nginx = KubernetesPodOperator(
        image="bitnami/kubectl:1.30.4",
        name="create-nginx",
        task_id="create-nginx",
        cmds=["/bin/sh"],
        arguments=["-c", "kubectl create deployment nginx-v2 --image=nginx --namespace=airflow"],
        labels={"foo": "bar"},
        is_delete_operator_pod=True,
        get_logs=True,
    )

    # 자체 클러스터에 ngnix 디플로이 스케일 아웃 
    scaleout_nginx = KubernetesPodOperator(
        image="bitnami/kubectl:1.30.4",
        name="scaleout-nginx",
        task_id="scaleout-nginx",
        cmds=["/bin/sh"],
        arguments=["-c", "kubectl scale deployment nginx-v2 --replicas=2  --namespace=airflow"],
        labels={"foo": "bar"},
        is_delete_operator_pod=True,
        get_logs=True,
    )

    # 파드 내 node-02.config 파일을 시크릿으로 마운트
    # kubectl 명령어 실행 시 참조
    secret = Secret('volume', '/config/', 'kubeconfig-svc')

    # 2nd cluster 노드 조회
    check_svc = KubernetesPodOperator(
        image="bitnami/kubectl:1.30.4",
        name="check-svc",
        task_id="check-svc",
        cmds=["/bin/sh"],
        arguments=["-c", "kubectl get nodes -o wide --kubeconfig /config/node-02.config"],
        labels={"foo": "bar"},
        is_delete_operator_pod=False,
        get_logs=True,
        secrets=[secret, ],
    )

    delete_nginx >> create_nginx >> scaleout_nginx

```
