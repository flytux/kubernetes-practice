# Alert Manager to gitlab issues with AM executor

- AM executor from https://github.com/imgix/prometheus-am-executor
- Alertmanger alerts values passed to am-exectutor script with ENVs

```bash
- AMX_RECEIVER: name of receiver in the AM triggering the alert
- AMX_STATUS: alert status
- AMX_EXTERNAL_URL: URL to reach alertmanager
- AMX_ALERT_LEN: Number of alerts; for iterating through AMX_ALERT_<n>.. vars
- AMX_LABEL_<label>: alert label pairs
- AMX_GLABEL_<label>: label pairs used to group alert
- AMX_ANNOTATION_<key>: alert annotation key/value pairs
- AMX_ALERT_<n>_STATUS: status of alert
- AMX_ALERT_<n>_START: start of alert in seconds since epoch
- AMX_ALERT_<n>_END: end of alert, 0 for firing alerts
- AMX_ALERT_<n>_URL: URL to metric in prometheus
- AMX_ALERT_<n>_FINGERPRINT: Message Fingerprint
- AMX_ALERT_<n>_LABEL_<label>: alert label pairs
- AMX_ALERT_<n>_ANNOTATION_<key>: alert annotation key/value pairs
```

---

- create running script configmap and mount on am-executor volume
- script getting alertname and alert description from the alerts
- create single alert description message from alert lists
- create json payload to send to gitlab issue registration formats

---

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: am-executor
  namespace: cattle-monitoring-system
spec:
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-cattle-monitoring-system-am-executor
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-cattle-monitoring-system-am-executor
      namespace: cattle-monitoring-system
    spec:
      containers:
      - args:
        - /etc/config/gitlab-alert.sh
        command:
        - prometheus-am-executor
        image: tbd5d1uh.private-ncr.fin-ntruss.com/k8s/dev/devops/prometheus-am-executor:hero
        imagePullPolicy: Always
        name: am-executor
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        securityContext:
          runAsNonRoot: false
          runAsUser: 0
        volumeMounts:
        - mountPath: /etc/config/gitlab-alert.sh
          name: gitlab-alert
          subPath: gitlab-alert.sh
      imagePullSecrets:
      - name: hero-reg
      volumes:
      - configMap:
          defaultMode: 493
          name: gitlab-alert
        name: gitlab-alert
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-alert
  namespace: cattle-monitoring-system
data:
  gitlab-alert.sh: |+
    #!/bin/bash
  
    ALERT="================================================================<p>"
    for (( i=1; i < $AMX_ALERT_LEN+1; i++))
     do
         ALERT="${ALERT} [$(printenv | grep AMX_ALERT_${i}_LABEL_alertname | cut -d "=" -f 2)]<p>"
         ALERT="${ALERT} $(printenv | grep AMX_ALERT_${i}_ANNOTATION_description | cut -d "=" -f 2)<p>"
     done
  
     echo "$ALERT"
     echo "{\"title\": \"$AMX_ALERT_1_LABEL_alertname\", \"description\":\"$ALERT\"}" > test.json
  
     curl -L --data @test.json -H "Content-Type: application/json" -X POST http://gitlab.herosonsa.co.kr/api/v4/projects/44/issues -H "PRIVATE-TOKEN: glpat-yzNTyQsxr8nvzh7Q2ZGQ"
```
