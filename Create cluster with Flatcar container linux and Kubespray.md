# Create cluster with Flatcar container linux and Kubespray

- references : https://computingforgeeks.com/deploy-kubernetes-on-kvm-using-flatcar-container-linux/?expand_article=1

```bash

# create flatcar linux image for qemu-libvirt
mkdir -p /var/lib/libvirt/images/flatcar-linux
cd /var/lib/libvirt/images/flatcar-linux
wget https://stable.release.flatcar-linux.net/amd64-usr/current/flatcar_production_qemu_image.img.bz2{,.sig}
gpg --verify flatcar_production_qemu_image.img.bz2.sig
bunzip2 flatcar_production_qemu_image.img.bz2

cd /var/lib/libvirt/images/flatcar-linux
qemu-img create -f qcow2 -F qcow2 -b flatcar_production_qemu_image.img flatcar-linux1.qcow2

mkdir -p /var/lib/libvirt/flatcar-linux/flatcar-linux1/
echo '{"ignition":{"version":"2.0.0"}}' > /var/lib/libvirt/flatcar-linux/flatcar-linux1/provision.ign

#SElinux
semanage fcontext -a -t virt_content_t "/var/lib/libvirt/flatcar-linux/flatcar-linux1"
restorecon -R "/var/lib/libvirt/flatcar-linux/flatcar-linux1"

#Apparmor
echo "  # For ignition files" >> /etc/apparmor.d/abstractions/libvirt-qemu
echo "  /var/lib/libvirt/flatcar-linux/** r," >> /etc/apparmor.d/abstractions/libvirt-qemu

# edit VM config 
example.yaml

variant: flatcar
version: 1.0.0
storage:
  files:
  - path: /etc/hostname
    contents:
      inline: "flatcar-linux1"

passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0g+ZTxC7weoIJLUafOgrm+h..."

# Create VM config
cat example.yaml | docker run --rm -i quay.io/coreos/butane:release > /var/lib/libvirt/flatcar-linux/flatcar-linux1/provision.ign

# Install vm
virt-install --connect qemu:///system \
             --import \
             --name flatcar-linux1 \
             --ram 1024 --vcpus 1 \
             --os-info=generic \
             --disk path=/var/lib/libvirt/images/flatcar-linux/flatcar-linux1.qcow2,format=qcow2,bus=virtio \
             --vnc --noautoconsole \
             --qemu-commandline='-fw_cfg name=opt/org.flatcar-linux/config,file=/var/lib/libvirt/flatcar-linux/flatcar-linux1/provision.ign'

# Check IP
virsh net-dhcp-leases default

# Access VM
ssh core@192.168.122.184

# Install ansible and kubespray


```
