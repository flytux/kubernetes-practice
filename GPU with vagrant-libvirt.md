### GPU with vagrant libvirt

```bash
#!/bin/bash

apt-get update
apt-get install -y build-essential
apt-get remove --purge nvidia*
apt-get autoremove

BASE="/vagrant"

CUDA="https://developer.nvidia.com/compute/cuda/9.1/Prod/local_installers/cuda_9.1.85_387.26_linux"
CUDA_INSTALL="$BASE/provision/cuda.run"
RSTUDIO_SERVER="https://download2.rstudio.org/rstudio-server-1.1.383-amd64.deb"
RSTUDIO_SERVER_INSTALL="$BASE/provision/rstudio-server.deb"
WGET_OPTS="-4 -q"

[[ -f $CUDA_INSTALL ]] || wget $WGET_OPTS $CUDA -O $CUDA_INSTALL

chmod u+x $CUDA_INSTALL
$CUDA_INSTALL --silent --driver --toolkit --verbose

grep cuda /etc/profile || cat >> /etc/profile <<EOF
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
EOF
source /etc/profile

grep nouveau /etc/modprobe.d/blacklist.conf || cat >> /etc/modprobe.d/blacklist.conf <<EOF
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist nvidiafb
blacklist rivatv
EOF

update-initramfs -u

rmmod -f nvidia_uvm
rmmod nvidia
rmmod nouveau

nvidia-smi

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
#reboot
```
---
### Install Vagrant
---

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

LIBVIRT_POOL = 'fast'

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu1604"
  config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_udp: false
  #config.vm.synced_folder "../../datastore/spindle/ML/datasets/", "/mnt", type: "nfs", nfs_udp: false
  config.vm.network "private_network", :dev => "br0", :mode => 'bridge', :ip => "192.168.17.25"
  config.vm.provider "libvirt" do |v|
    v.storage_pool_name = LIBVIRT_POOL
    v.memory = 16384
    v.cpus = 4
    v.machine_type = "q35"
    v.cpu_mode = "host-passthrough"
    v.kvm_hidden = true
    v.pci :bus => '0x01', :slot => '0x00', :function => '0x0'
  end
  config.vm.provision "docker", images: ["nvidia/cuda", "tensorflow/tensorflow:latest-gpu"]
  config.vm.provision "shell", path: "install.sh"
end
```
