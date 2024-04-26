
### Create Additional Config 


```
prometheus-additional-yaml

- job_name: 'federate'  
  honor_labels: true
  metrics_path: /federate
  params:
    match[]:
      - '{job="prometheus"}'
      - '{__name__=~"job:.*"}'
  static_configs:
    - targets:
      - 192.168.122.21:30010
```

### Create secret

```
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml --dry-run=client -oyaml > additional-scrape-configs.yaml
```

### Edit Prometheus

```
spec:
  additionalScrapeConfigs:
    key: prometheus-additional.yaml
    name: additional-scrape-configs
```

### Restart Prometheus

### Check Prometheus Operator Logs --> CRD Validation

### Create AlertmanagerConfig

```
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  labels:
    alertmanagerConfig: example
  name: config-kafka
  namespace: cattle-monitoring-system
spec:
  receivers:
    - name: webhook
      webhookConfigs:
        - url: http://kaf-receiver.cattle-monitoring-system:9792/alert
  route:
    groupBy:
      - job
    groupInterval: 10s
    groupWait: 10s
    matchers:
      - matchType: '='
        name: severity
        value: critical
    receiver: webhook
    repeatInterval: 10s
```

### add alertmanager2kafka webhook receiver 

### send test Alert

- exec in rancher pod
- $ sh test.sh critical

```

#!/bin/bash

# Set default values
name=$RANDOM
url='http://alertmanager-operated.cattle-monitoring-system:9093/api/v2/alerts'
summary='Testing summary!'
instance="$name.example.net"
default_severity='warning'

# Function to send alert
send_alert() {
    local status=$1
    local custom_severity=$2
    local current_severity=${custom_severity:-$default_severity}

    curl -XPOST -H "Content-Type: application/json" $url -d "[
        {
            \"status\": \"$status\",
            \"labels\": {
                \"alertname\": \"$name\",
                \"service\": \"my-service\",
                \"severity\":\"$current_severity\",
                \"instance\": \"$instance\"
            },
            \"annotations\": {
                \"summary\": \"$summary\"
            },
            \"generatorURL\": \"https://prometheus.local/<generating_expression>\"
        }
    ]"
    echo ""
}

# Main script
echo "Firing up alert $name"
send_alert "firing" "$1"

read -p "Press enter to resolve alert"

echo "Sending resolve"
send_alert "resolved" "$1"


```
