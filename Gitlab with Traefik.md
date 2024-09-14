## Gitlab , Argocd with Traefik Ingress


#### Option 1) Helm install with traefik ingress

```
$ helm upgrade -i gitlab gitlab-8.3.2.tgz \
--set global.edition=ce \
--set global.hosts.domain=amc.seoul.kr \ 
--set global.ingress.configureCertmanager=false \
--set global.ingress.provider=traefik \
--set global.ingress.class=traefik \
--set certmanager.install=false \
--set nginx-ingress.enabled=false \
--set gitlab-runner.install=false \
--set prometheus.install=false \
--set registry.enabled=false \
-n gitlab --create-namespace

$ add Ingress annotations, delete nginx annotations

traefik.ingress.kubernetes.io/router.tls: "true"
traefik.ingress.kubernetes.io/router.entrypoints: websecure

```
#### Option 2) Create IngressRoute

```
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: gitlab-shell
  namespace: gitlab
spec:
  entryPoints:
    - gitlab-shell
  routes:
  - match: HostSNI(`*`)
    services:
    - name: gitlab-gitlab-shell # Put the gitlab-shell service name
      namespace: gitlab
      port: 2232 # Put gitlab.shell.port
      proxyProtocol: # Only if global.shell.tcp.proxyProtocol
        version: 2  # Only if global.shell.tcp.proxyProtocol
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: gitlab-https-redirect
  namespace: gitlab
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: gitlab-security
  namespace: gitlab
spec:
  headers:
    frameDeny: true
    sslRedirect: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 31536000
---
apiVersion: traefik.io/v1alpha1
kind: ServersTransport
metadata:
  name: gitlab-transport
  namespace: gitlab
spec:
  serverName: gitlab
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: gitlab-tlsoptions
  namespace: gitlab
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_AES_128_GCM_SHA256
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_FALLBACK_SCSV
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: false
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: gitlab-websecure
  namespace: gitlab
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`gitlab.amc.seoul.kr`)
      services:
        - name: gitlab-webservice-default
          port: 8181
          serversTransport: gitlab-transport
      middlewares:
        - name: gitlab-security
  tls:
    secretName: gitlab-urbaman
    options:
      name: gitlab-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: gitlab-web
  namespace: gitlab
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`gitlab.amc.seoul.kr`)
      services:
        - name: gitlab-webservice-default
          port: 8181
      middlewares:
        - name: gitlab-https-redirect
```

#### ArgoCD IngressRoute

```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: argocd-server
  namespace: argocd
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`argocd.amc.seoul.kr`)
      priority: 10
      services:
        - name: argocd-server
          port: 80
    - kind: Rule
      match: Host(`argocd.amc.seoul.kr`) && Headers(`Content-Type`, `application/grpc`)
      priority: 11
      services:
        - name: argocd-server
          port: 80
          scheme: h2c
  tls: {}
```
