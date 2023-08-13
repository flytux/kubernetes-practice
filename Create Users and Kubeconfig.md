# Create cluster users and RBAC kubeconfig 

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
    certificate-authority-data:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

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
    token:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

---

 k describe secret kw-dev-token-z4pfj -n kw-dev | grep "^token.*" | sed 's/.*:\s*//'

---
# kubeconfig - kw-dev
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
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
    token:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

 k describe secret kw-dev-token-z4pfj -n kw-dev | grep "^token.*" | sed 's/.*:\s*//'


❯ k get pods --kubeconfig kw-dev.config
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kw-dev:kw-dev" cannot list resource "pods" in API group "" in the namespace "default"

❯ k get pods --kubeconfig kw-admin.config
No resources found in default namespace.
```
