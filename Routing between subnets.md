# Routing between subnets

1. Host VM - Multipass VMs 

- host : 192.168.45.98
- Multipass VM : 10.78.15.151/24
  
  
2. Multipass VMs - Libvirt VMs

- host : 192.168.45.98
- Multipass VM : 10.78.15.151/24
- Vagrant VMs : 192.168.121.181/24

Multipass VM --> Vagrant VM

ip route add 192.168.121.0/24 via 10.78.15.1 dev ens3 metric 1

  

