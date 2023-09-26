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
If when running Terraform Libvirtd driver you get a permission denied when trying to open a disk image file in /var/lib/libvirtd/images. Here is instructions on to fix that.

Error message will be similar to: “Could not open ‘/var/lib/libvirt/images/disk_image.raw’: Permission denied’)”

Fix:

Please add the following lines for *.qcow and *.raw in file /etc/apparmor.d/libvirt/TEMPLATE.qemu

contents of /etc/apparmor.d/libvirt/TEMPLATE.qemu

#
# This profile is for the domain whose UUID matches this file.
#

#include <tunables/global>

profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
  #include <abstractions/libvirt-qemu>
   /var/lib/libvirt/images/**.qcow2 rwk,
   /var/lib/libvirt/images/**.raw rwk,
}
After adding the config file line in /etc/apparmor.d/libvirt/TEMPLATE.qemu please restart the libvirtd systemd service.

Systemd restart libvirtd command

sudo systemctl restart libvirtd
```

