# VM management with virsh

```bash
# check kvm
apt install cpu-checker

kvm-ok

# install libvirt
apt install -y qemu-kvm virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils

# vm install
virt-install --virt-type kvm --name kw01-demo --memory 2048 \
--disk path=/var/lib/libvirt/images/kw01-demo.qcow2,size=20 \
--location https://debian.osuosl.org/debian/dists/stable/main/installer-amd64/ \
--os-variant debian10 --graphics none --extra-args='console tty0 console=ttyS0,115200n8 serial'

# list networks
virsh net-list --all

# check vm ip address
virsh domifaddr --domain kb-master-2-100.100.100.102

# check ip assign
virsh net-dhcp-leases nat100

# forward nat subnets

1) From host set ip forward
sysctl -w net.ipv4.ip_forward=1

2) check permanent 
vi /etc/sysctl.conf
net.ipv4.ip_forward=1

3) iptables accept
virbr1 : 10.10.10.0/24 < = > virbr2 : 100.100.100.0/24

iptables -I FORWARD -i virbr1 -o virbr2 -s 10.10.10.0/24 -d 100.100.100.0/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -I FORWARD -i virbr2 -o virbr1 -s 100.100.100.0/24 -d 10.10.10.0/24 -m conntrack --ctstate NEW -j ACCEPT

ping 10.10.10.101 from 100.100.100.101
```

