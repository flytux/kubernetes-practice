# Install nginx ingress controller on kubeadm cluster

```bash
# Untaint master node
kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-

# Install ingress-controller
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml | kubectl apply -f -  

# Patch hostnetwork  
$ cat <<EOF > patch.yml
spec:
  template:
    spec:
      hostNetwork: true
EOF  

kubectl patch deployment ingress-nginx-controller --patch-file atch.yml -n ingress-nginx
```
