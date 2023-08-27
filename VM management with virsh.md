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
```
