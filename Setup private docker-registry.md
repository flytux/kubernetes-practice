# Setup private docker-registry

```bash

# Create Private Cert and Key 
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout registry.key -out registry.crt \
  -subj "/CN=docker.kw01" -addext "subjectAltName=DNS:docker.kw01,DNS:*.kw01,IP:192.168.100.101"

# Register CA cerfificate chain
cp registry.* /usr/local/share/ca-certificates/
update-ca-certificates

# Create secret
kubectl create ns registry
kubectl create secret tls docker-tls -n registry --cert=registry.crt --key=registry.key

# Install Docker registry
cat << EOF >> values.yaml
ingress:
  enabled: true
  className: traefik
  path: /
  hosts:
    - docker.kw01
  tls:
    - secretName: docker-tls
      hosts:
        - docker.kw01
persistence:
  accessMode: 'ReadWriteOnce'
  enabled: false
  size: 5Gi
EOF

cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add twuni https://helm.twun.io
helm upgrade -i docker-registry -f values.yaml twuni/docker-registry -n registry
   
# Configure containerd certificate
cat << EOF >> /etc/rancher/k3s/registries.yaml
mirrors:
  docker.kw01:
    endpoint:
      - "https://docker.kw01"
configs:
  docker.kw01:
    tls:
      ca_file: "/usr/local/share/ca-certificates/registry.crt"
      key_file: "/usr/local/share/ca-certificates/registry.key"
      cert_file: "/usr/local/share/ca-certificates/registry.key"
EOF

# Restart k3s 
systemctl restart k3s
cat /var/lib/rancher/k3s/agent/etc/containerd/config.toml

wget https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz -O - | tar -xz -C /usr/local/bin

# Login registry
echo "192.168.100.101 docker.kw01" >> /etc/hosts
mkdir -p /etc/nerdctl
cat << EOF >> /etc/nerdctl/nerdctl.toml
debug          = false
debug_full     = false
address        = "unix:///run/k3s/containerd/containerd.sock"
namespace      = "k8s.io"
snapshotter    = "stargz"
cgroup_manager = "cgroupfs"
hosts_dir      = ["/etc/containerd/certs.d", "/etc/docker/certs.d"]
experimental   = true
EOF

nerdctl login docker.kw01 # admin / 1
nerdctl images
# Apply to other workloads
