```
consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' -server --client=0.0.0.0

consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' --server=false --client=0.0.0.0 --join 172.31.41.232


consul client:
private IP: 172.31.47.127
public IP: 18.206.201.62
consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' --server=false --client=0.0.0.0 --join 172.31.41.232 --config-dir=/opt/consul


[root@ip-172-31-47-127 consul]# cat services.json
{
  "services": [
    {
      "id": "hello-id",
      "name": "hello-name",
      "tags": [
        "primary"
      ],
      "address": "172.31.47.127",
      "port": 9090,
      "checks": [
        {
        "http": "http://localhost:9090/",
        "tls_skip_verify": false,
        "method": "GET",
        "interval": "10s",
        "timeout": "1s"
        }
      ]
    }
  ]
}

consul server: 
public IP:54.89.154.123:
private IP: 172.31.41.232
consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' -server --client=0.0.0.0

http://54.89.154.123:8500
```
![](img/1.png)
![](img/2.png)
![](img/3.png)
---
python command to start port
![](img/11.png)
![](img/22.png)

---
yum install bind-utils
dig @127.0.0.1 -p 8600 helloname.service.dc1.consul. ANY
curl http://127.0.0.1:8500/v1/health/service/helloname?passing=false



[root@ip-172-31-41-232 centos]# cat in.tpl
{{ range service "helloname" }}
server {{ .Name }} {{ .Address }}:{{ .Port }}{{ end }}

./consul-template -template "in.tpl:out.txt" -once

https://github.com/hashicorp/consul-template
https://github.com/hashicorp/consul-template#service
https://github.com/hashicorp/consul-template#services
