# Install k3s on airgapped RHEL8 VM

```bash

cat <<EOF >/etc/yum.repos.d/rancher.repo
[rancher]
name=Rancher Repository
baseurl=https://rpm.rancher.io/
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF

yum install k3s-selinux --downloadonly --downloaddir=.

wget https://github.com/k3s-io/k3s/releases/download/v1.27.4%2Bk3s1/k3s ; chmod +x k3s ; mv k3s /usr/local/bin
wget https://github.com/k3s-io/k3s/releases/download/v1.27.4%2Bk3s1/k3s-airgap-images-amd64.tar.gz

sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp ./k3s-airgap-images-amd64.tar.gz /var/lib/rancher/k3s/agent/images/
curl -sfL https://get.k3s.io > install.sh ; chmod +x install.sh

INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh

```
