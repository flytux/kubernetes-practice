How to fix Terraform libvirt permission denied on /var/lib/libvirt/images

If when running Terraform Libvirtd driver you get a permission denied when trying to open a disk image file in /var/lib/libvirtd/images. Here is instructions on to fix that.

Error message will be similar to: “Could not open ‘/var/lib/libvirt/images/disk_image.raw’: Permission denied’)”

Fix:

Please add the following lines for *.qcow and *.raw in file /etc/apparmor.d/libvirt/TEMPLATE.qemu

contents of /etc/apparmor.d/libvirt/TEMPLATE.qemu
```
#
# This profile is for the domain whose UUID matches this file.
#

#include <tunables/global>

profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
  #include <abstractions/libvirt-qemu>
   /var/lib/libvirt/images/**.qcow2 rwk,
   /var/lib/libvirt/images/**.raw rwk,
}
```
After adding the config file line in /etc/apparmor.d/libvirt/TEMPLATE.qemu please restart the libvirtd systemd service.

Systemd restart libvirtd command

```
sudo systemctl restart libvirtd
```
