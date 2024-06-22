### Load test web page with busybox

```
K8S Horizontal Autoscaler Test

디플로이먼트는 컨테이너를 실행시키는 기본 단위로, 원하는 POD를 선언적으로 생성, 관리할 수 있습니다.
디플로이먼트에 서비스를 연결하여 동적인 서비스 확장 및 축소 시 서비스 영향도를 최소화 할수 있으며,
History 및 롤백 기능을 이용하여 버전 관리가 가능합니다.

클러스터 > 워크로드 > Create > Deployment

Namespace > Create New > "apps" 

Name > "httpd" 
Image >  "httpd"
Networking > Add Port or Service > Service Type > "Cluster IP"
Name > http > Private port > "80"

$ cat << EOF >> httpd.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: httpd
  name: httpd
  namespace: apps
spec:
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
      namespace: apps
    spec:
      containers:
        - image: httpd
          name: container-0
          ports:
            - containerPort: 80
              name: http
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: httpd
  name: httpd
  namespace: apps
spec:
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 80
  selector:
    app: httpd
  type: ClusterIP

EOF

$ cat << EOF >> patch.yml
spec:
  template:
    spec:
      containers:
        - name: container-0
          resources:
            limits:
              cpu: '1'
              memory: 1000Mi
            requests:
              cpu: 100m
              memory: 100Mi
EOF

$ kubectl patch deploy httpd --patch-file patch.yml -n apps

$ kubectl autoscale deployment httpd --cpu-percent=10 --min=1 --max=10 -n apps

$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://httpd.apps:8080; done"

```
