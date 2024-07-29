### Install RKE2 Cillium


1. Download RKE2 Server

```
$ curl -sfL https://get.rke2.io | sh -
```

2. CNI config

```
$ mkdir -p /etc/rancher/rke2/

$ cat << EOF > /etc/rancher/rke2/config.yaml
  cni: none
  EOF
```

3. Start RKE2 Server

```
$ systemctl enable rke2-server --now
```

4. Download Kubectl, Helm, Kubeconfig

```
$ curl -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl

$ chmod +x kubectl && mv kubectl /usr/local/bin

$ mkdir -p $HOME/.kube

$ cp /etc/rancher/rke2/rke2.yaml $HOME/.kube/config

$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

5. Install Cillium with helm

```
$ helm repo add cilium https://helm.cilium.io/

$ helm install cilium cilium/cilium --version 1.16.0 --namespace kube-system
```
