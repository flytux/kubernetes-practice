### Kube scheduler Binpacking

- Check Kube-Scheduler version with match target cluster's kubernetes version
- Check kubescheduler api version refer (https://v1-27.docs.kubernetes.io/docs/concepts/scheduling-eviction/resource-bin-packing/)
- Define scoring strategy and resources
- Check kubeconfig path of volume of deployment --kubeconfig=/etc/kubernetes/scheduler.conf

```bash

# gpu-binpack-scheduler/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gpu-binpack-scheduler-config
  namespace: kube-system
data:
  gpu-binpack-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta3
    kind: KubeSchedulerConfiguration
    profiles:
    - pluginConfig:
      - args:
          scoringStrategy:
            resources:
            - name: cpu
              weight: 1
            - name: memory
              weight: 1
            type: MostAllocated
        name: NodeResourcesFit
---
# gpu-binpack-scheduler/serviceaccount.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: gpu-binpack-scheduler
  namespace: kube-system

---
# gpu-binpack-scheduler/rbac.yaml

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gpu-binpack-scheduler
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gpu-binpack-scheduler
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: gpu-binpack-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: gpu-binpack-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gpu-binpack-scheduler-as-kube-scheduler
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: gpu-binpack-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gpu-binpack-scheduler-as-volume-scheduler
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: gpu-binpack-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:volume-scheduler
  apiGroup: rbac.authorization.k8s.io
  
---
# gpu-binpack-scheduler/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    component: scheduler
    tier: control-plane
  name: gpu-binpack-scheduler
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  replicas: 1
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
      serviceAccountName: gpu-binpack-scheduler
      containers:
      - name: kube-scheduler
        command:
        - /usr/local/bin/kube-scheduler
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/gpu-binpack-scheduler/gpu-binpack-scheduler-config.yaml
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        image: registry.k8s.io/kube-scheduler:v1.24.13
        livenessProbe:
          failureThreshold: 8
          httpGet:
            path: /healthz
            port: 10259
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 15
        startupProbe:
          failureThreshold: 30
          httpGet:
            path: /healthz
            port: 10259
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 15
        resources:
          requests:
            cpu: 100m
        volumeMounts:
          - name: config-volume
            mountPath: /etc/kubernetes/gpu-binpack-scheduler
            readOnly: true
          - name: kubeconfig
            mountPath: /etc/kubernetes/scheduler.conf
            readOnly: true  
      priorityClassName: system-node-critical
      restartPolicy: Always
      hostNetwork: false
      hostPID: false
      volumes:
        - name: config-volume
          configMap:
            name: gpu-binpack-scheduler-config
        - name: kubeconfig
          hostPath:
            path: /etc/kubernetes/scheduler.conf
            type: FileOrCreate

```
