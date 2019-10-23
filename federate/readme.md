### alertmanager.yml
```
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

receivers:
- name: 'live-monitoring'
  email_configs:
  - to: 'zhaoqingjie@gmail.com'
```


### prometheus.yml
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
    - targets: ['172.31.41.232:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['172.31.47.127:9090']
```
### The following are private IP
```
172.31.47.127:9090 is q1 prometheus
172.31.41.232:9090 is master prometheus
172.31.41.232:9093 is alertmanager
```
