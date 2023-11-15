---
title: prometheus + grafana with docker 구축
date: 2023-11-16 01:36:36
tags: 
  - 모니터링
  - grafana
  - prometheus
  - docker
categories:
  - 인프라
---



prometheus + grafana with docker

### 사전 준비
* docker
* docker-compose

# step 1 docker-compose.yml

```yml
version: '2.23.0'

services:
  influxdb:
    image: bitnami/influxdb:1.8.5
    container_name: influxdb
    ports:
      - "8086:8086"
      - "8085:8088"
    environment:
      - INFLUXDB_ADMIN_USER_PASSWORD=bitnami123
      - INFLUXDB_ADMIN_USER_TOKEN=admintoken123
      - INFLUXDB_HTTP_AUTH_ENABLED=false
      - INFLUXDB_DB=myk6db
  granafa:
    image: bitnami/grafana:latest
    ports:
      - "4000:4000"

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - "./prometheus.yml:/prometheus/prometheus.yml"
    ports:
      - "9090:9090"
    command: 
      - '--web.enable-lifecycle'
    restart: always
  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
```

# Step 2 prometheus.yml
```yml
global:
  scrape_interval: 10s
scrape_configs:
 - job_name: prometheus
   static_configs:
    - targets:
       - prometheus:9090
 - job_name: node
   static_configs:
    - targets:
       - node-exporter:9100
```

# Step docker up
```bash
sudo docker-compose up 
```
출처: https://mxulises.medium.com/simple-prometheus-setup-on-docker-compose-f702d5f98579