### Tekton Dashboard Extensions ###

- 테크톤 대시보드에 extension을 추가하여 클러스터 내 리소스를 조회할 수 있다

```
# Tekton Dashboard Extension 추가
kubectl apply -n tekton-pipelines -f - <<EOF
apiVersion: dashboard.tekton.dev/v1alpha1
kind: Extension
metadata:
  name: cronjobs
spec:
  apiVersion: batch/v1
  name: cronjobs
  displayName: k8s cronjobs
EOF

# Tekton Service Account에 권한 추가
kubectl apply -f - <<EOF
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tekton-dashboard-daemonset-extension
  labels:
    rbac.dashboard.tekton.dev/aggregate-to-dashboard: "true"
rules:
  - apiGroups: ["batch"]
    resources: ["cronjobs"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["daemonsets"]
    verbs: ["get", "list", "watch"]
EOF


# 리소스 명을 정확하게 쓰지 않으면 권한 또는 목록 조회 오류 발생 가능
ex) daemonset > daemonsets

kubectl api-resources | grep deamonsets
 
```
