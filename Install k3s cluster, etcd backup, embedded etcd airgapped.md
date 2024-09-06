# Install k3s on airgapped RHEL8 VM

```bash

$ cat <<EOF >/etc/yum.repos.d/rancher.repo
[rancher]
name=Rancher Repository
baseurl=https://rpm.rancher.io/
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF

$ yum install k3s-selinux --downloadonly --downloaddir=.

$ wget https://github.com/k3s-io/k3s/releases/download/v1.30.4%2Bk3s1/k3s ; chmod +x k3s ; mv k3s /usr/local/bin
$ wget https://github.com/k3s-io/k3s/releases/download/v1.30.4%2Bk3s1/k3s-airgap-images-amd64.tar

$ sudo mkdir -p /var/lib/rancher/k3s/agent/images/
$ sudo cp ./k3s-airgap-images-amd64.tar.gz /var/lib/rancher/k3s/agent/images/
$ curl -sfL https://get.k3s.io > install.sh ; chmod +x install.sh

$ INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='server --cluster-init' ./install.sh 

```

# Install k3s on airgapped SLE 15 VM

```bash

https://github.com/k3s-io/k3s-selinux # download k3s-selinux policy

$ zypper in restorecond policycoreutils setools-console 
$ zypper ar -f https://download.opensuse.org/repositories/security:/SELinux_legacy/15.5/ SELinux-Legacy
$ zypper in selinux-policy-targeted selinux-policy-devel


$ wget https://github.com/k3s-io/k3s/releases/download/v1.30.4%2Bk3s1/k3s ; chmod +x k3s ; mv k3s /usr/local/bin
$ wget https://github.com/k3s-io/k3s/releases/download/v1.30.4%2Bk3s1/k3s-airgap-images-amd64.tar

$ sudo mkdir -p /var/lib/rancher/k3s/agent/images/
$ sudo cp ./k3s-airgap-images-amd64.tar.gz /var/lib/rancher/k3s/agent/images/
$ curl -sfL https://get.k3s.io > install.sh ; chmod +x install.sh

# Start Master Node
$ INSTALL_K3S_EXEC='server --cluster-init' INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh

$ cat /var/lib/rancher/k3s/server/token 

# Join Server
$ K3S_URL=https://192.168.122.11:6443 K3S_TOKEN=$TOKEN INSTALL_K3S_EXEC='server --cluster-init' INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh

# Join Agent
$ K3S_URL=https://192.168.122.11:6443 K3S_TOKEN=$TOKEN INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh 

```

# Create ETCD Snapshots

```
$ k3s etcd-snapshot save

$ k3s etcd-snapshot save --s3 --s3-endpoint minio.kw01 --s3-skip-ssl-verify --s3-bucket etcd-snapshot --s3-access-key minio --s3-secret-key minio123

# Backup server token
cat /var/lib/rancher/k3s/server/token 

```

# Config Nerdctl

```
$ mkdir -p /etc/nerdctl
$ vi /etc/nerdctl/nerdctl.toml

address     = "unix:///run/k3s/containerd/containerd.sock"
namespace   = "k8s.io"

$ nerdctl images

