# Create cluster using Kubespray

```
# create 3 VMs with multipass

multipass launch -n kw04 -c 2 -m 4G -d 100G
multipass launch -n kw05 -c 2 -m 4G -d 100G
multipass launch -n kw06 -c 2 -m 4G -d 100G

# create ssh key add each VMs
ssh kw04

ssh-keygen
ssh-copy-id 10.122.203.72
ssh-copy-id 10.122.203.254
ssh-copy-id 10.122.203.140

# install ansible
sudo apt install pip -y
python3 -m pip install --user ansible-core==2.14.5

# clone kubespray git repo
git clone https://github.com/kubernetes-sigs/kubespray.git

# install dependencies
cd kubespray
sudo pip install -r requirements.txt

# prepare cluster install modules
cp -rfp inventory/sample inventory/mycluster

# add VM IPs to install 
declare -a IPS=(10.122.203.72 10.122.203.254 10.122.203.140)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# reset cluster
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml

# create cluster
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```
