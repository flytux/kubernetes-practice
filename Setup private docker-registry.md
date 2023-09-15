# Setup private docker-registry

```bash

# Create Private Cert and Key 
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout registry.key -out registry.crt \
  -subj "/CN=docker.kw01" -addext "subjectAltName=DNS:docker.kw01,DNS:*.kw01,IP:192.168.100.101"

# Register CA cerfificate chain
cp registry.* /usr/local/share/ca-certificates/
update-ca-certificates

# Create secret
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

helm repo add twuni https://helm.twun.io
helm upgrade -i docker-registry -f values.yaml twuni/docker-registry
   
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
EOF

# Restart k3s 
systemctl restart k3s
cat /var/lib/rancher/k3s/agent/etc/containerd/config.toml

wget https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz

# Login registry
nerdctl login docker.kw01

# Apply to other workloads
