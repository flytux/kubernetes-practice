### Install metallb


```
# 메탈LB 설치
$ curl -LO https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml

# 메탈LB 대역대 설정

$ cat << EOF > ipaddress-pool.yml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.10.1.1-10.10.1.250
EOF

# l2 라우팅 설정

$ cat << EOF > l2advertisement.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF

# kubectl apply

$ kubectl apply -f ./

# ip route 설정

클러스터 노드 네트워클 통해 metallb IP로 라우팅

ip route add 10.10.0.0/16 via 192.168.122.1 dev virbr0

# 샘플 서비스 테스트

$ kubectl create deployment nginx --image=nginx -n default
$ kubectl expose deployment nginx --type=LoadBalancer --port=80 -n default

$ kubectl get svc로  external IP 확인

$ curl -v %external IP%
```
