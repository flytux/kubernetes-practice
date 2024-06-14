### Delete and add NEW Control-plane with KUBEADM

- Create master node with dns not with IP

```

$ kubeadm init --pod-network-cidr=192.168.100.0/24 --upload-certs \
  --control-plane-endpoint=k8smaster:6443 --kubernetes-version v1.28.8 | \
  sed -e '/kubeadm join/,/--certificate-key/!d' | head -n 3 > join_cmd

```

- Delete Etcd member from the cluster

```

$ alias etcdctl='ETCDCTL_API=3 etcdctl \
--endpoints=https://192.168.122.21:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key'
$ etcdctl member list
$ etcdctl member remove $MEMBER-ID

```

- Delete node from the cluster

```

$ kubectl get nodes
$ kubectl delete nodes

```

- Check /etc/hosts which IP -> k8smaster 

- Print join command from alive control plane

```

$ kubeadm token create --print-join-command # print join-command

$ kubeadm init phase upload-certs --upload-certs # print certificate-key 

# run from new control plane node
$ kubeadm join k8smaster:6443 --token 8cp3uh.1gzm54957cfqsp3k --control-plane \
  --discovery-token-ca-cert-hash sha256:a3ef2d5eae94d6f811fbbabd66921a6e014f097f53b529ce488f1cc6ea7c7a14 
  --certificate-key 80a318f884a26bb3646d205d49981935c034a8e52adb2c3179d0b74fba92ad03

```
