### https://juejin.im/post/5c9dc0b06fb9a070ae3da6e7#heading-5

```
#启动容器：
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml up -d
#删除容器：
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml down
```

```
[root@ip-172-31-18-94 config]# cat alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'zhaoqingjie@gmail.com'
  smtp_auth_username: 'zhaoqingjie@gmail.com'
  smtp_auth_password: 'rwsqpsqxtbmhmwat'
  smtp_require_tls: yes

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10m
  receiver: live-monitoring
  routes:
  - receiver: alertmanager2es

receivers:
- name: 'live-monitoring'
  email_configs:
  - to: 'elvewyn@gmail.com'
  - to: 'zhaoqingjie@gmail.com'
- name: alertmanager2es
  webhook_configs:
  - url: 'http://172.31.41.120:9097/webhook'
```

```
[root@ip-172-31-18-94 config]# cat docker-compose-monitor.yml
version: '2'

networks:
    monitor:
        driver: bridge

services:
    prometheus:
        image: prom/prometheus
        container_name: prometheus
        hostname: prometheus
        restart: always
        volumes:
            - /usr/local/src/config/prometheus.yml:/etc/prometheus/prometheus.yml
            - /usr/local/src/config/node_down.yml:/etc/prometheus/node_down.yml
        ports:
            - "9090:9090"
        networks:
            - monitor

    alertmanager:
        image: prom/alertmanager
        container_name: alertmanager
        hostname: alertmanager
        restart: always
        volumes:
            - /usr/local/src/config/alertmanager.yml:/etc/alertmanager/alertmanager.yml
        ports:
            - "9093:9093"
        networks:
            - monitor

    grafana:
        image: grafana/grafana
        container_name: grafana
        hostname: grafana
        restart: always
        ports:
            - "3000:3000"
        networks:
            - monitor

    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: always
        ports:
            - "9100:9100"
        networks:
            - monitor

    cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        hostname: cadvisor
        restart: always
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:rw
            - /sys:/sys:ro
            - /var/lib/docker/:/var/lib/docker:ro
        ports:
            - "8080:8080"
        networks:
            - monitor
    es01:
        image: docker.elastic.co/elasticsearch/elasticsearch:7.3.2
        container_name: es01
        environment:
         - node.name=es01
             - discovery.seed_hosts=es02
             - cluster.initial_master_nodes=es01,es02
             - cluster.name=docker-cluster
             - bootstrap.memory_lock=true
        ulimits:
           memlock:
               soft: -1
               hard: -1
        volumes:
             - esdata01:/usr/share/elasticsearch/data
        ports:
             - 9200:9200


volumes:
    esdata01:
        driver: local
```
```
[root@ip-172-31-18-94 config]# cat node_down.yml
groups:
- name: node_down
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      user: test
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

```

```
[root@ip-172-31-18-94 config]# cat prometheus.yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['172.31.18.94:9093']
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
    - targets: ['172.31.18.94:9090']

  - job_name: 'cadvisor'
    static_configs:
    - targets: ['172.31.21.46:8080']

  - job_name: 'node'
    scrape_interval: 8s
    static_configs:
    - targets: ['172.31.21.46:9100']
```

### https://github.com/cloudflare/alertmanager2es

```
./alertmanager2es -help
./alertmanager2es -addr 0.0.0.0:9097 -esURL https://search-test51-vzzwj46x7abo6gvtjsalygnfbi.us-east-1.es.amazonaws.com
```


