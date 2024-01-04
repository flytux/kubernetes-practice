### Rancher 2.8.0 모니터링 Airgapped 설치

- Rancher 모니터링 차트 : 103.0.0+up45.31.1 기준

---

```bash

1. 이미지 Private Registry에 저장

192.168.122.11:30005/rancher/kubectl:v1.20.2
192.168.122.11:30005/rancher/mirrored-grafana-grafana:9.1.5
192.168.122.11:30005/rancher/mirrored-ingress-nginx-kube-webhook-certgen:v20221220-controller-v1.5.1-58-g787ea74b6
192.168.122.11:30005/rancher/mirrored-kiwigrid-k8s-sidecar:1.24.6
192.168.122.11:30005/rancher/mirrored-kube-state-metrics-kube-state-metrics:v2.6.0
192.168.122.11:30005/rancher/mirrored-library-nginx:1.24.0-alpine
192.168.122.11:30005/rancher/mirrored-prometheus-adapter-prometheus-adapter:v0.10.0
192.168.122.11:30005/rancher/mirrored-prometheus-alertmanager:v0.25.0
192.168.122.11:30005/rancher/mirrored-prometheus-node-exporter:v1.3.1
192.168.122.11:30005/rancher/mirrored-prometheus-operator-prometheus-config-reloader:v0.65.1
192.168.122.11:30005/rancher/mirrored-prometheus-operator-prometheus-operator:v0.65.1
192.168.122.11:30005/rancher/mirrored-prometheus-prometheus:v2.42.0
192.168.122.11:30005/rancher/pushprox-client:v0.1.0-rancher2-client
192.168.122.11:30005/rancher/pushprox-proxy:v0.1.0-rancher2-proxy
192.168.122.11:30005/rancher/shell:v0.1.18

2. Cluster Tools에서 모니터링 선택

Container Registry 선택 후 주소 입력 : 192.168.122.11:30005

3. Next > Edit YAML

355 번째 줄에 imageRegistry: 192.168.122.11:30005 로 변경

4. Daemonset rancher-monitoring-prometheus-node-exporter 이미지 주소가 중복되어, 변경

192.168.122.11:30005/rancher/mirrored-prometheus-node-exporter:v1.3.1
```
