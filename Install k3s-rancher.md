### Install k3s rancher

```bash
# Install k3s 
$ curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.27.8+k3s2 sh -s - --token k3s-token --write-kubeconfig-mode 644 --cluster-init
$ cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# Add k3s master
$ curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.27.8+k3s2 sh -s - --token k3s-token --write-kubeconfig-mode 644 --server https://10.10.10.11:6443

# Add k3s worker
$ curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.27.8+k3s2 K3S_URL=https://10.10.10.11:6443 K3S_TOKEN=k3s-token sh -s -

# Install cert-manager
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml

# Install helm
$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add rancher repo
$ helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

# Install rancher
$ helm upgrade -i rancher rancher-latest/rancher \
--set hostname=rancher.kw01 --set bootstrapPassword=admin \
--set replicas=1 --set global.cattle.psp.enabled=false \
--create-namespace -n cattle-system

cat << EOF >> ~/.bashrc
# k8s alias

source <(kubectl completion bash)
complete -o default -F __start_kubectl k

alias k=kubectl
alias vi=vim
alias kn='kubectl config set-context --current --namespace'
alias kc='kubectl config use-context'
alias kcg='kubectl config get-context
EOF

source ~/.bashrc
source /etc/bash_completion


# Disable Rancher Feature CAPI
cat <<EOF | kubectl apply -f -
apiVersion: management.cattle.io/v3
kind: Feature
metadata:
  name: embedded-cluster-api
spec:
  value: false
EOF
```
