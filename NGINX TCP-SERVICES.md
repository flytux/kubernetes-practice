## Redis TCP port 인그레스 설정


- 1) TCP 서비스 컨피그맵 설정 추가

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  6379: "redis/redis-master:6379"
```

- 2) nginx 서비스에 추가

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
    - name: proxied-tcp-9000
      port: 9000
      targetPort: 9000
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

- 3) Deployment에 실행 Arg 추가

```
 args:
    - /nginx-ingress-controller
    - --tcp-services-configmap=ingress-nginx/tcp-services  

```

- 4) Ingress 생성

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: '0'
  name: redis
  namespace: redis
spec:
  ingressClassName: nginx
  rules:
    - host: dev-redis.hanwhaaerospace.co.kr
      http:
        paths:
          - backend:
              service:
                name: redis-master
                port:
                  number: 6379
            path: /
            pathType: Prefix

```

- 5) Ingress명:Port로 접속

```
TCP endpoint: dev-redis.hanwhaaerospace.co.kr:6379
```
