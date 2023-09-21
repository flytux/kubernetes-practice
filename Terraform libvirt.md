# Terraform libvirt vm

- Libvirt Provider => Config variables => Cloud init (SSH Key) => OS volume > Network => VM Domain
- Cloud init config [cloud_init.cfg](./cloud_init.cfg)
```bash

# Provider
terraform {
  required_version = ">= 0.14"
  required_providers {
    libvirt = {
      source  = "dmacvicar/libvirt"
      version = "0.6.10"
    }
  }
}

# instance of the provider
provider "libvirt" {
  # ensures system connection, not userspace qemu:///session
  uri = "qemu:///system"
}

# Variables
variable "network_name" { default = "nat100" }
variable "prefix_ip" { default = "100.100.100" }
variable "master_ip" { default = "100.100.100.101" }
variable "network_domain_name" { default = "kubeworks.net" }
# OS qcow2 images
#variable "cloud_image_name" { default = "focal-server-cloudimg-amd64.img" }
variable "cloud_image_name" { default = "Rocky-8-GenericCloud.latest.x86_64.qcow2" }
variable "disk_pool" { default = "default" }
variable "qemu_connect" { default = "qemu:///system" }
variable "join_cmd" { default = "$(ssh -i $HOME/.ssh/id_rsa.key -o StrictHostKeyChecking=no 100.100.100.101 -- cat join_cmd)" }
variable "kubeadm_nodes" {
  type = map(object({ role = string, octetIP = string , vcpu = number, memoryMB = number, incGB = number}))
  default = {
              kb-master-1 = { role = "master-init",   octetIP = "101" , vcpu = 2, memoryMB = 1024 * 8, incGB = 30},
              kb-worker-1 = { role = "worker",        octetIP = "201" , vcpu = 2, memoryMB = 1024 * 8, incGB = 30}
              kb-master-2 = { role = "master-member", octetIP = "102" , vcpu = 2, memoryMB = 1024 * 8, incGB = 30}
  }
}

# SSH key
resource "tls_private_key" "generic-ssh-key" {
  algorithm = "RSA"
  rsa_bits  = 4096

  provisioner "local-exec" {
    command = <<EOF
      mkdir -p .ssh-${terraform.workspace}/
      echo "${tls_private_key.generic-ssh-key.private_key_openssh}" > .ssh-${terraform.workspace}/id_rsa.key
      echo "${tls_private_key.generic-ssh-key.public_key_openssh}" > .ssh-${terraform.workspace}/id_rsa.pub
      chmod 400 .ssh-${terraform.workspace}/id_rsa.key
      chmod 400 .ssh-${terraform.workspace}/id_rsa.key
    EOF
  }

  provisioner "local-exec" {
    when    = destroy
    command = <<EOF
      rm -rvf .ssh-${terraform.workspace}/
    EOF
  }
}

# Cloud-init config 
data "template_file" "cloud_inits" {
  for_each = var.kubeadm_nodes
  template = file("${path.module}/artifacts/config/cloud_init.cfg")
  vars = {
    hostname = each.key
    fqdn     = "${each.key}.${var.network_domain_name}"
    password = var.password
    public_key = "${tls_private_key.generic-ssh-key.public_key_openssh}"
  }
}

# Create cloud init disk
resource "libvirt_cloudinit_disk" "cloudinit_disks" {
  for_each = var.kubeadm_nodes
  name           = "${each.key}-cloudinit.iso"
  pool           = var.disk_pool
  user_data      = data.template_file.cloud_inits[each.key].rendered
}

# OS images for libvirt VMs
resource "libvirt_volume" "os_images" {
  for_each = var.kubeadm_nodes
  name   = "${each.key}.qcow2"
  pool   = var.disk_pool
  source = "artifacts/images/${var.cloud_image_name}"
  format = "qcow2"

# Extend libvirt primary volume
  provisioner "local-exec" {
    command = <<EOF
      echo "Increment disk size by ${each.value.incGB}GB"
      sleep 10
      poolPath=$(virsh --connect ${var.qemu_connect} pool-dumpxml ${var.disk_pool} | grep -Po '<path>\K[^<]+')
      sudo qemu-img resize $poolPath/${each.key}.qcow2 +${each.value.incGB}G
      sudo chgrp libvirt $poolPath/${each.key}.qcow2
      sudo chmod g+w $poolPath/${each.key}.qcow2
    EOF
  }
}

# Network config
resource "libvirt_network" "nat" {
  name = "${var.network_name}"
  mode = "nat"
  addresses = [ "${var.prefix_ip}.0/24" ]
  domain = "${var.network_domain_name}"
  dns {
    enabled = true
    local_only = true
  }
  dhcp { enabled = false }
  autostart = true
}

# Create the machine
resource "libvirt_domain" "kubeadm_nodes" {
  depends_on = [libvirt_volume.os_images]
  for_each = var.kubeadm_nodes
  name   = "${each.key}-${var.prefix_ip}.${each.value.octetIP}"
  memory = each.value.memoryMB
  vcpu   = each.value.vcpu
  disk { volume_id = libvirt_volume.os_images[each.key].id }
  # Set static IP
  network_interface {
    network_name = "${var.network_name}"
    addresses = [ "${var.prefix_ip}.${each.value.octetIP}" ]
  }
  cloudinit = libvirt_cloudinit_disk.cloudinit_disks[each.key].id
  console {
    type        = "pty"
    target_port = "0"
    target_type = "serial"
  }
  graphics {
    type        = "spice"
    listen_type = "address"
    autoport    = "true"
  }
}

