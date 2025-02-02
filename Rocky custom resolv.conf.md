### Rocky custom resolv.conf

- virsh로 rocky vm을 만드는 경우 resov.conf에 virbr0 가 설정되고, kubernetes 클러스터에서 외부 DNS로 통신이 안되는 경우가 발생 ex) google.com
- 이 경우 vm의 resolv.conf에 커스텀 엔트리를 추가하고, netowkrmanager에서 관리 제외하도록 설정 필요

```
$ vi /etc/NetworkManager/conf.d/90-dns-none.conf

[main]
dns=none

$ systemctl reload NetworkManager

$ vi /etc/resolv.conf

nameserver 8.8.8.8

$ systemctl reload NetworkManager

```
