![image](https://github.com/flytux/kubernetes-practice/ip-route.png)


---
```
$ ip route add 192.168.122.0/24 via 192.168.45.76 dev enp3s0
$ iptables -I FORWARD -i enp2s0 -o virbr0 -s 192.168.45.0/24 -d 192.168.122.0/24 -m conntrack --ctstate NEW -j ACCEPT
```
