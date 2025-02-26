### Docker Registry Proxy Cache


##### 1. docker distribution 설치
```
k create ns registry

# 사설 인증서 생성
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout docker.key -out docker.crt -subj '/CN=docker.local' -addext 'subjectAltName=DNS:docker.local'

# TLS 인증서 secret 생성
kubectl create secret tls docker-registry-tls --key docker.key --cert docker.crt -n registry


# 스토리지 클래스 생성 및 기본 스토리지 클래스 설정
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.31/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# 프록시 설정, ingress 사설 인증서 설정
cat << EOF >> values.yaml
replicaCount: 1

proxy:
  enabled: true
  remoteurl: https://registry-1.docker.io

ingress:
  enabled: true
  className: traefik
  path: /
  hosts:
    - docker.local
  tls:
    - secretName: docker-registry-tls
      hosts:
        - docker.local
persistence:
  accessMode: 'ReadWriteOnce'
  enabled: true
  size: 10Gi
EOF

# registry 설치
helm upgrade -i docker-registry docker-registry-2.2.3.tgz -f values.yaml -n registry


# proxy cache 설정 확인
k logs -n registry $(k get pod -n registry -o name) | grep proxy
```

##### 2. Docker / Containerd Mirror 설정 

```
# 도커 데몬에 registry-mirror 설정

/etc/docker/daemon.json

{
  "registry-mirrors" : [ "https://docker.local" ]
}

systemctl restart docker

# 이미지 pull
docker pull nginx

# proxy cache 이용 pull 확인
k logs -n registry $(k get pod -n registry -o name) | grep nginx
```
<br>

```
# 클러스터 containerd에 미러 설정
   
version = 2
[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
      [plugins."io.containerd.grpc.v1.cri".registry]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
          [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
            endpoint = ["https://docker.local"]
            [plugins."io.containerd.grpc.v1.cri".registry.configs."docker.local".tls]
              ca_file = "/etc/pki/ca-trust/source/anchors/docker.crt"
              cert_file = "/etc/pki/ca-trust/source/anchors/docker.crt"
              key_file = "/etc/pki/ca-trust/source/anchors/docker.key"

# 각 노드에 인증서 복사
scp docker.crt node-02.local:/etc/pki/ca-trust/source/anchors/docker.crt
scp docker.key node-02.local:/etc/pki/ca-trust/source/anchors/docker.key

# ca 인증서 추가
update-ca-trust
```

##### 3. 이미지 Pull Test 
```
# Deploy 생성
k create deploy httpd --image httpd 

# Proxy cache 로그 확인
k logs -n registry $(k get pod -n registry -o name) | grep httpd
```
