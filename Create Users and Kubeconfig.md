# Create Users and Kubeconfig 

- Create user service account and secrets (Admin / Dev user)
- Create ClusterRole / ClusterroleBingings / Role / Rolebindings
- Get Service Account Tokens
- Create Kubeconfig from custer-admin's kubeconfig as a template
- Add User / Token to Kubeconfig file

```bash
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-kw-admin
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "namespaces", "nodes"]
    verbs: ["create", "get", "update", "list", "watch", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployment"]
    verbs: ["create", "get", "update", "list", "delete", "watch", "patch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["clusterroles", "clusterrolebindings"]
    verbs: ["create", "get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: crb-kw-admin
subjects:
  - kind: ServiceAccount
    name: kw-admin
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cr-kw-admin
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: r-kw-dev
  namespace: kw-dev
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "namespaces", "nodes"]
    verbs: ["create", "get", "update", "list", "watch", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployment"]
    verbs: ["create", "get", "update", "list", "delete", "watch", "patch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["clusterroles", "clusterrolebindings"]
    verbs: ["create", "get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-kw-dev
  namespace: kw-dev
subjects:
  - kind: ServiceAccount
    name: kw-dev
    namespace: kw-dev
roleRef:
  kind: Role
  name: r-kw-dev
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kw-admin
  namespace: kube-system
secrets:
  - name: kw-admin-secret
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kw-dev
  namespace: kw-dev
secrets:
  - name: kw-dev-secret
---
apiVersion: v1
kind: Secret
metadata:
  name: kw-admin-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: kw-admin
type: kubernetes.io/service-account-token
---
apiVersion: v1
kind: Secret
metadata:
  name: kw-dev-secret
  namespace: kw-dev
  annotations:
    kubernetes.io/service-account.name: kw-dev
type: kubernetes.io/service-account-token
--- 
apiVersion: v1
kind: Namespace
metadata:
  name: kw-dev
---
 k describe secret kw-admin-token-4wt77 -n kube-system | grep "^token.*" | sed 's/.*:\s*//'

# kubeconfig - kw-admin
---
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyT1RFek56QXpNemt3SGhjTk1qTXdPREEzTURFd05UTTVXaGNOTXpNd09EQTBNREV3TlRNNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyT1RFek56QXpNemt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFRTkt0Wjl3VFlvS0JaL1Bubk5FNVVNY0dyTm9ROUxJMmd3SmZaaW40QzcKbThlWTZGWVFPV2FhbE5NYmVSNDN0ejRJNjhhcE1zRjRJZHdCNCtISXlNVE1vMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVS9CTzA0Mm1qYzdJNkRIdlpNa3dxCkhZWTlHOG93Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUlnYjkrbVlueTRDMWorbUk1bXZXc2JYUitIM2E2SmRFZlcKaTZiOUluNWE1V3NDSVFEeW83cFY5emNFWXVZQzRYcXhKc29iTzhSYXAyMHdsTlM3YTJtUm9PTlNTUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://127.0.0.1:6443
  name: local
contexts:
- context:
    cluster: local
    user: kw-admin
  name: kw-admin-local
current-context: kw-admin-local
kind: Config
preferences: {}
users:
- name: kw-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6InZBRllwMG43MDAzZ2I2Zjd1VDJIZzhBSkZrbEgyYmZlNF8wbS0zLW9WUm8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdy1hZG1pbi10b2tlbi00d3Q3NyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdy1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjcxZjVlNjVkLWEyMWMtNDQxNC1iNWU5LTNkYmY0NDk3NDczZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdy1hZG1pbiJ9.E_Bfy32UbvbEsQ4kypwZHQozLCC6_d-DHuRfX1TicdDvNPoHuYBmHgMqdHSdBkBxG-30vE2xr6nfgqDJm90FTdibxyMZdFmwMSGHP7yk_S-pjD7CKaAFW5X29PQazdIbo2iFvzLwzUsHrUsVpXnIYC4jX9fGDhd1rXnzFW70HTv8_dWy8TxJ0u8z7OSwYtYT6v4WWsFzbvveEdQ7kE1o9h3C9wnujcNropNKmp5Bx0sN8mB2HsiCfB9tEPKF99ZsbXWttpqZsQiohr7BXd6R1D8FzsK1VzyFYyR6PUJOnYo6ARnuaZyIJVI1rXzcAxdXVDck9ZSfn74c_AW5qCP2ow

---

 k describe secret kw-dev-token-z4pfj -n kw-dev | grep "^token.*" | sed 's/.*:\s*//'

---
# kubeconfig - kw-dev
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyT1RFek56QXpNemt3SGhjTk1qTXdPREEzTURFd05UTTVXaGNOTXpNd09EQTBNREV3TlRNNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyT1RFek56QXpNemt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFRTkt0Wjl3VFlvS0JaL1Bubk5FNVVNY0dyTm9ROUxJMmd3SmZaaW40QzcKbThlWTZGWVFPV2FhbE5NYmVSNDN0ejRJNjhhcE1zRjRJZHdCNCtISXlNVE1vMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVS9CTzA0Mm1qYzdJNkRIdlpNa3dxCkhZWTlHOG93Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUlnYjkrbVlueTRDMWorbUk1bXZXc2JYUitIM2E2SmRFZlcKaTZiOUluNWE1V3NDSVFEeW83cFY5emNFWXVZQzRYcXhKc29iTzhSYXAyMHdsTlM3YTJtUm9PTlNTUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://127.0.0.1:6443
  name: local
contexts:
- context:
    cluster: local
    user: kw-dev
  name: kw-dev-local
current-context: kw-dev-local
kind: Config
preferences: {}
users:
- name: kw-dev
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6InZBRllwMG43MDAzZ2I2Zjd1VDJIZzhBSkZrbEgyYmZlNF8wbS0zLW9WUm8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdy1kZXYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoia3ctZGV2LXRva2VuLXo0cGZqIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt3LWRldiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImMwZjYwMmY4LTAxMWYtNDQ4My1hM2I5LTAwZDEyMDc4NDg0YSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdy1kZXY6a3ctZGV2In0.BnGbOLrlO7AN4QZfYX-l3jxWbJBFZ480oJSaUHOO4BuNZdZ-VqkM1ZEMZwwMt2TRM3QyxVki8NCppT_WhCCl-suj9Oy5Chnj84AitISyTE2Xgc5NQJmPsb0LGhV5jCbT2xs9TYZU07zXz_yE1cdekJ_GR8OqXyQ31BpswpWNSgUuKk3mQtzpCgycdiJNQur4XvfQcWMcHWbIyN_b-vdjBSm-VKd22aRD73fdJPeQ3c9Kb5xY19ZWF1zB08Hv0DLASCvOIzi6s1msLvH0ncIJ32I1OxKOSTv5Agb_mL_O5eRgnKG-BNHHbdyeJTbqvLVeqiqUDcvLTaJ0zFDslxz0FQ

 k describe secret kw-dev-token-z4pfj -n kw-dev | grep "^token.*" | sed 's/.*:\s*//'


❯ k get pods --kubeconfig kw-dev.config
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kw-dev:kw-dev" cannot list resource "pods" in API group "" in the namespace "default"

❯ k get pods --kubeconfig kw-admin.config
No resources found in default namespace.
```
