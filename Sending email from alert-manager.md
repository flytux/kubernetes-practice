# Sending email from alert-manager

- Install jenkins helm chart with Prometheus Plugin installed
- Install Kube-Prometheus-Stack
- Setup email SMTP
- Create ServiceMonitor / PromehtuesRule / AlertmanagerConfig / Email credential secrets

---

```bash
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jenkins
  namespace: jenkins
spec:
  endpoints:
    - interval: 30s
      path: /prometheus
      port: http
  jobLabel: jenkins
  namespaceSelector:
    matchNames:
      - jenkins
  selector:
    matchLabels:
      app.kubernetes.io/component: jenkins-controller
      app.kubernetes.io/instance: jenkins
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: jenkins-down
  namespace: jenkins
spec:
  groups:
    - interval: 30s
      name: jenkins
      rules:
        - alert: jenkins-down
          annotations:
            message: ''
            summary: >-
              Jenkins {{ $labels.pod }} is down - number of running jenkins pods
              is less than 3
          expr: sum(default_jenkins_up) < 3
          for: 30s
          labels:
            cluster_id: local
            cluster_name: local
            namespace: jenkins
            severity: critical
---
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: jenkins-down
  namespace: jenkins
spec:
  receivers:
    - emailConfigs:
        - authPassword:
            key: password
            name: mail-credential
          authUsername: flytux
          from: flytux@naver.com
          requireTLS: true
          sendResolved: false
          smarthost: smtp.naver.com:587
          tlsConfig: {}
          to: hooney.jung@gmail.com
      name: naver
  route:
    groupBy:
      - namespace
    groupInterval: 1m
    groupWait: 30s
    matchers:
      - matchType: '='
        name: alertname
        value: jenkins-down
    receiver: naver
    repeatInterval: 2m
---
apiVersion: v1
data:
  password: XXXXXXXXXXXXXXXX # base64 decode password string of yours
kind: Secret
metadata:
  name: mail-credential
  namespace: jenkins
```
