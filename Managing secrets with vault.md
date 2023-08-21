# Managing static secrets with vault

- Install vault in cluster
- Install valut secret
- Enable kubernetes auth & kvv2 secret engine
- Configure policy / role  to sync vault with secret
- Create Vault Auth / Vault Static Secret in the cluster

```bash
# install vault
helm repo add hashicorp https://helm.releases.hashicorp.com

cat << EOF >> vault-values.yaml
server:
  dev:
    enabled: true
    devRootToken: "root"
  logLevel: debug

injector:
  enabled: "false"
EOF

helm install vault hashicorp/vault -n vault --create-namespace --values vault/vault-values.yaml

kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh

# Enable kubernetes auth
vault auth enable -path demo-auth-mount kubernetes

# Config kubernetes auth method
vault write auth/demo-auth-mount/config \
   kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

# Enable kv-v2 secret engine
vault secrets enable -path=kvv2 kv-v2

# Config read only policy
vault policy write dev - <<EOF
path "kvv2/*" {
   capabilities = ["read"]
}
EOF

# Create vault role1 to create secret
vault write auth/demo-auth-mount/role/role1 \
   bound_service_account_names=default \
   bound_service_account_namespaces=app \
   policies=dev \
   audience=vault \
   ttl=24h


# Create secret in vault
vault kv put kvv2/webapp/config username="static-user" password="static-password"


# Install value secret operator
cat << EOF >> vault-operator-values.yaml
defaultVaultConnection:
  # toggles the deployment of the VaultAuthMethod CR
  enabled: true

  # Address of the Vault Server
  # Example: http://vault.default.svc.cluster.local:8200
  address: "http://vault.vault.svc.cluster.local:8200"
  skipTLSVerify: false
EOF

# Install vault-operator
helm upgrade -i vault-secrets-operator hashicorp/vault-secrets-operator -n vault-secrets-operator-system --create-namespace --values vault-operator-values.yaml

# Create vault secret
kubectl create ns app

# Configure vault auth for static secret
cat << EOF >> vault-auth-static.yml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultAuth
metadata:
  name: static-auth
  namespace: app
spec:
  method: kubernetes
  mount: demo-auth-mount
  kubernetes:
    role: role1
    serviceAccount: default
    audiences:
      - vault
EOF

# Create static secret
cat << EOF >> vault-static-secret.yml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: vault-kv-app
  namespace: app
spec:
  type: kv-v2

  # mount path
  mount: kvv2

  # path of the secret
  path: webapp/config

  # dest k8s secret
  destination:
    name: secretkv
    create: true

  # static secret refresh interval
  refreshAfter: 30s

  # Name of the CRD to authenticate to Vault
  vaultAuthRef: static-auth

kubectl apply -f vault-static-secret.yml

# Check secret values
kubectl get secret -n app secretkv -ojsonpath='{.data.password}' | base64 -d

# Update secret
kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh
vault kv put kvv2/webapp/config username="static-user2" password="static-password2"  

# Check secret values
```
