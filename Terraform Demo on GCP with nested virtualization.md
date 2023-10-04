# Terraform demo on GCP

---
### Create VM on GCP and install terraform

```bash
# Create virtualization enabled VM
gcloud compute instances create kw01-demo \
    --project=kw-poc-01 --zone=us-central1-a --machine-type=n1-standard-8 \
    --enable-nested-virtualization --min-cpu-platform="Intel Haswell" \
	--service-account=265842716061-compute@developer.gserviceaccount.com \
	--create-disk=auto-delete=yes,boot=yes,device-name=kw01-demo,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20230918,mode=rw,size=200,type=projects/kw-poc-01/zones/us-central1-a/diskTypes/pd-balanced

# Zsh setup & Terraform install

sudo apt install zsh -y

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.zsh/zsh-syntax-highlighting
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.zsh/powerlevel10k

echo 'source ~/.zsh/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
echo 'source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh' >>~/.zshrc
echo 'source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh' >>~/.zshrc

cat <<EOF >> ~/.zshrc
# k8s alias
source <(kubectl completion zsh)
alias k=kubectl
alias vi=vim
alias kn='kubectl config set-context --current --namespace'
alias kc='kubectl config use-context'
alias kcg='kubectl config get-contexts'
alias di='docker images --format "table {{.Repository}}:{{.Tag}}\t{{.ID}}\t{{.Size}}\t{{.CreatedSince}}"'
alias kge="kubectl get events  --sort-by='.metadata.creationTimestamp'  -o 'go-template={{range .items}}{{.involvedObject.name}}{{\"\t\"}}{{.involvedObject.kind}}{{\"\t\"}}{{.message}}{{\"\t\"}}{{.reason}}{{\"\t\"}}{{.type}}{{\"\t\"}}{{.firstTimestamp}}{{\"\n\"}}{{end}}'"
EOF

wget https://releases.hashicorp.com/terraform/1.5.7/terraform_1.5.7_linux_amd64.zip
unzip terraform_1.5.7_linux_amd64.zip && sudo mv terraform /usr/local/bin
```
---
### Install Libvirt and configure
```bash
# check kvm
sudo apt install cpu-checker
kvm-ok

# install libvirt packages
apt install -y qemu-kvm libvirt-daemon-system virtinst libvirt-clients bridge-utils

# install libvirt images config for permission denied
vi /etc/apparmor.d/libvirt/TEMPLATE.qemu

#include <tunables/global>

profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
  #include <abstractions/libvirt-qemu>
   /var/lib/libvirt/images/**.qcow2 rwk,
   /var/lib/libvirt/images/**.raw rwk,
}

systemd restart libvirtd
```

---
### Terraform apply
```bash
# Clone terraform repo 
git clone https://github.com/flytux/kubeadm-kb

# Install kubeadm single node cluster with registry
cd kubeadm-kb/clusters/registry

# Download vm image
mkdir -p artifacts/kubeadm/images
wget https://download.rockylinux.org/pub/rocky/8/images/x86_64/Rocky-8-GenericCloud.latest.x86_64.qcow2 -P artifacts/kubeadm/images

terraform init
terraform plan

terraform apply -auto-approve

# connect deploy k8s cluster to registy cluster
iptables -I FORWARD -i virbr1 -o virbr2 -s 10.10.10.0/24 -d 100.100.100.0/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -I FORWARD -i virbr2 -o virbr1 -s 100.100.100.0/24 -d 10.10.10.0/24 -m conntrack --ctstate NEW -j ACCEPT
```
