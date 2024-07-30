### Install RKE2 Airgapped, Cilium with Load Balancer and Ingress Controller

---
- Cilium을 이용하여 인그레스 컨트롤러, 로드밸런서를 별도의 솔루션 없이 적용이 가능하다
---


#### 1. Download RKE2 Air-gapped, Start with no ingress controller, cni

```
$ mkdir -p /root/rke2-artifacts && cd /root/rke2-artifacts
$ curl -OLs https://github.com/rancher/rke2/releases/download/v1.30.2%2Brke2r1/rke2-images.linux-amd64.tar.zst
$ curl -OLs https://github.com/rancher/rke2/releases/download/v1.30.2%2Brke2r1/rke2.linux-amd64.tar.gz
$ curl -OLs https://github.com/rancher/rke2/releases/download/v1.30.2%2Brke2r1/sha256sum-amd64.txt
$ curl -sfL https://get.rke2.io --output install.sh

$ INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh

$ mkdir -p /etc/rancher/rke2
$ cat << EOF >> /etc/rancher/rke2/config.yaml
cni: none
disable: rke2-ingress-nginx
EOF

$ systemctl enable rke2-server --now
```


#### 2. Install Cillium with ingress-controller, load-balancer

```
# announce.yaml
apiVersion: cilium.io/v2alpha1
kind: CiliumL2AnnouncementPolicy
metadata:
  name: default-l2-announcement-policy
  namespace: kube-system
spec:
  externalIPs: true
  loadBalancerIPs: true


# ip-pool.yaml
apiVersion: "cilium.io/v2alpha1"
kind: CiliumLoadBalancerIPPool
metadata:
  name: "first-pool"
spec:
  blocks:
    - start: "192.168.1.200"
      stop: "192.168.1.249"
```

```
# values.yaml - Cillium Helm Chart
k8sServiceHost: 192.168.122.11
k8sServicePort: 6443

kubeProxyReplacement: true

l2announcements:
  enabled: true

externalIPs:
  enabled: true

k8sClientRateLimit:
  qps: 50
  burst: 200

operator:
  replicas: 1  # Uncomment this if you only have one node
  rollOutPods: true

rollOutCiliumPods: true

ingressController:
  enabled: true
  default: true
  loadbalancerMode: shared
  service:
    annotations:
      io.cilium/lb-ipam-ips: 192.168.1.200

# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - announce.yaml
  - ip-pool.yaml

helmCharts:
  - name: cilium
    repo: https://helm.cilium.io
    version: 1.15.5
    releaseName: "cilium"
    includeCRDs: true
    namespace: kube-system
    valuesFile: values.yaml
```

```
$ kubectl kustomize --enable-helm . | kubectl apply -f -
```

#### 3. Test Sample

```
# smoke-test.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: whoami
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  namespace: whoami
spec:
  replicas: 1
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - image: containous/whoami
          imagePullPolicy: Always
          name: whoami
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
  namespace: whoami
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
  selector:
    app: whoami
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
  namespace: whoami
spec:
  rules:
    - host: whoami.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: whoami
                port:
                  number: 80

$ kubectl apply -f whoami.yaml

$ curl --header 'Host: whoami.local' 192.168.1.200

$ curl -v 192.168.1.201
```
