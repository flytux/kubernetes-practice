# Otel Log to loki and elasticsearch

- Loki - Grafana 설치
- Elastic - Kibana 설치
- Otel Collector 설치
- Spring Application 설치
- 로그 테스트

### 1. Loki - Grafana - lgtm 설치

```
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm

vi run-lgtm.sh

docker run \
        --name lgtm \
        -p 3000:3000 \
        -p 14317:4317 \ # 포트 변경
        -p 14318:4318 \ # 포트 변경
        --rm \
        -ti \
        -v "$PWD"/container/grafana:/data/grafana \
        -v "$PWD"/container/prometheus:/data/prometheus \
        -v "$PWD"/container/loki:/data/loki \
        -e GF_PATHS_DATA=/data/grafana \
        --env-file .env \
        docker.io/grafana/otel-lgtm:"${RELEASE}"

docker compose up -d
```

### 2. Elastic - Kibana 설치

```
git clone https://github.com/davidsilwal/observability-demo.git
cd observability-demo
git checkout otel-elk
docker compose up -d
```

### 3. Otel Collector 설치

- 각각 otel collector 별도 설치됨 (docker compose 내)

```
otel collector endpoint : http://localhost:4318 # elk
otel collector endpoint : http://localhost:14318 # loki
```

### 4. Spring Application 설치

```
vi docker-otel-lgtm/example/java/run.sh

# otel endpoint 설정 (로키)
java -Dotel.metric.export.interval=500 -Dotel.bsp.schedule.delay=500 -Dotel.exporter.otlp.endpoint=http://192.168.219.101:14318 -javaagent:${jar} -jar ./target/rolldice.jar

# spring application 실행
docker-otel-lgtm/example/java/run.sh

# otel endpoint 설정 (elk)
java -Dotel.metric.export.interval=500 -Dotel.bsp.schedule.delay=500 -Dotel.exporter.otlp.endpoint=http://192.168.219.101:4318 -javaagent:${jar} -jar ./target/rolldice.jar

# spring application 실행
docker-otel-lgtm/example/java/run.sh

# 호출
docker-otel-lgtm/generate-traffic.sh
```

### 5. 로그 테스트

```
# 로키 접속 http://localhost:3000
# kibana 접속 http://localhost:5601
```
