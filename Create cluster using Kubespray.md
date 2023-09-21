# Create cluster using Kubespray

```
#Rocky8.8 kubespray  설치

1) Terraform kubespray 모듈로 VM 생성 후

2) kubespray 실행
scp -i .ssh-default/id_rsa.key -o StrictHostKeyChecking=no .ssh-default/id_rsa.key 101.101.101.101:/root/.ssh

ssh -i .ssh-default/id_rsa.key -o StrictHostKeyChecking=no 101.101.101.101

dnf update -y
dnf install python39 git -y
python3 -m pip install --user ansible-core==2.14.5

git clone https://github.com/kubernetes-sigs/kubespray.git

cd kubespray
pip3 install -r requirements.txt
cp -rfp inventory/sample inventory/mycluster

# Node IP 정보 설정

declare -a IPS=(101.101.101.101 101.101.101.102 101.101.101.201)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}


kubernetes 버전 1.28.2 => 1.26.8
containerd 설치 => docker

# 클러스터 설치
ansible-playbook -i inventory/mycluster/hosts.yaml  -e kube_version=v1.26.8 -e container_manager=docker --private-key=~/.ssh/id_rsa.key --become --become-user=root cluster.yml

# 클러스터 리셋
ansible-playbook -i inventory/mycluster/hosts.yaml  --private-key=~/.ssh/id_rsa.key --become --become-user=root reset.yml
