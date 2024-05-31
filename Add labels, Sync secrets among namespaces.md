## Add labels, Sync secrets among namespaces using Kyverno


```
# Install kyverno
$ kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.12.0/install.yaml
```

### 1. Add labels

```
# Create ClusterPolicy add labels on pod team: bravo
$ kubectl create -f- << EOF
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-labels
spec:
  rules:
  - name: add-team
    match:
      any:
      - resources:
          kinds:
          - Pod
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            +(team): bravo
EOF

$ kubectl run redis --image redis

$ kubectl get pods --show-labels
```

### 2. Sync secrets

```
$ kubectl -n default create secret docker-registry regcred \
  --docker-server=myinternalreg.corp.com \
  --docker-username=john.doe \
  --docker-password=Passw0rd123! \
  --docker-email=john.doe@corp.com

# Sync secret regcred from default namespace

$ kubectl create -f- << EOF
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: sync-secrets
spec:
  rules:
  - name: sync-image-pull-secret
    match:
      any:
      - resources:
          kinds:
          - Namespace
    generate:
      apiVersion: v1
      kind: Secret
      name: regcred
      namespace: "{{request.object.metadata.name}}"
      synchronize: true
      clone:
        namespace: default
        name: regcred
EOF

$ kubectl create ns mytestns

$ kubectl get secret -n mytestns
```
```
