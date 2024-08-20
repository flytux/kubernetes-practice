### Tekton Pipeline prunner

---
- tkn cli를 실행할 수 있는 dev-tools 컨테이너를 이용하여 tkn prune 명령어를 cronjob으로 실행
- 'tkn pr delete --keep-since 10080 -n build --force ' # 60 분 x 24 시간 x 7 일
---

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-rbac
subjects:
  - kind: ServiceAccount
    name: default
    namespace: dev
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: tkn-prunner
  namespace: dev
spec:
  jobTemplate:
    metadata:
      namespace: dev
    spec:
      template:
        spec:
          containers:
            - command:
                - /bin/sh
                - '-c'
                - 'tkn pr delete --keep-since 10080 -n build --force '
              image: 10.42.186.95:5000/dev-tools:v1
              imagePullPolicy: Always
              name: tkn-prunner
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          terminationGracePeriodSeconds: 30
  schedule: 0 1 * * 6
  successfulJobsHistoryLimit: 3
  suspend: false
```
