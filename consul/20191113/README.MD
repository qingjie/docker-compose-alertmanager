master consul
* http://localhost:8500/ui/dc1/services/prometheus
```
./consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "en0"}}' -server --client=0.0.0.0


```
master prometheus
* http://localhost:9090/
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
      - '{job="prometheus"}'
      - '{__name__=~"job:.*"}'
      - '{__name__="up"}'

    consul_sd_configs:
    - server: localhost:8500
      services:
      - prometheus

```
prometheus in kubernetes
* http://192.168.99.102:30080/

```
helm upgrade --reuse-values -f prometheus-values.yml prometheus stable/prometheus
```
```
podSecurityPolicy:
  enabled: true

server:
  service:
    nodePort: 30080
    type: NodePort

  retention: 48h

  global:
    scrape_interval: 15s
    scrape_timeout: 15s

  resources:
    requests:
      cpu: 10m
      memory: 1Gi

kubeStateMetrics:
  enabled: false

nodeExporter:
  enabled: false

alertmanager:
  enabled: false

pushgateway:
  enabled: false

```

./consul services register service.json
(base) Qingjies-MBP-2:consul qingjiezhao$ cat service.json
{
	"service": {
		"name": "prometheus",
		"id": "prometheus",
		"port": 30080,
		"address": "192.168.99.102"
	}
}

