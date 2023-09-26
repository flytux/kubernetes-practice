# VM management with virsh

```bash
# check kvm
apt install cpu-checker

kvm-ok

# install libvirt
apt install -y qemu-kvm virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils

reboot

# vm install
virt-install --virt-type kvm --name kw01-demo --memory 2048 \
--disk path=/var/lib/libvirt/images/kw01-demo.qcow2,size=20 \
--location https://debian.osuosl.org/debian/dists/stable/main/installer-amd64/ \
--os-variant debian10 --graphics none --extra-args='console tty0 console=ttyS0,115200n8 serial'

# storage pool define

virsh pool-define-as --name default --type dir --target /var/lib/libvirt/images
virsh pool-autostart default
virsh pool-start default

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

---

```bash

# Premission denied for open a disk image file in /var/lib/libvirtd/images “Could not open ‘/var/lib/libvirt/images/disk_image.raw’: Permission denied’)”
# Add qcow2,raw lines to the file /etc/apparmor.d/libvirt/TEMPLATE.qemu
#include <tunables/global>

profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
  #include <abstractions/libvirt-qemu>
   /var/lib/libvirt/images/**.qcow2 rwk,
   /var/lib/libvirt/images/**.raw rwk,
}

systemd restart libvirtd command
```

---

```bash

virsh snapshot-create-as --domain kubespray-master-1-101.101.101.101 \
> --name kubespray-master-1 \
> --description "Snapshot kubespray"


virsh snapshot-create-as --domain kubespray-master-2-101.101.101.102 \
> --name kubespray-master-2 \
> --description "Snapshot kubespray"


virsh snapshot-create-as --domain kubespray-worker-1-101.101.101.201 \
> --name kubespray-worker-1 \
> --description "Snapshot kubespray"
```

