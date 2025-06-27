```
1) 인증서 SAN 확인
$ openssl x509 -text -in /etc/kubernetes/pki/apiserver.crt -noout | grep DNS

2) Kubeadm Config 확인
$ kubectl -n kube-system get configmap kubeadm-config -o jsonpath='{.data.ClusterConfiguration}' > kubeadm.yaml

3) kubeadm.yaml에 SAN 추가

apiServer:
  certSANs:
  - 192.168.100.1
  - lb.local

4) 인증서 백업
mv /etc/kubernetes/pki/apiserver.{crt,key} ~

5) 인증서 생성
kubeadm init phase certs apiserver --config kubeadm.yaml 

6) kubeapi server 재기동
nerdctl kill $(nerdctl ps | grep api | grep -v pause | cut -d ' ' -f 1)

7) kube-config 업로드
kubeadm init phase upload-config kubeadm --config kubeadm.yaml
```
