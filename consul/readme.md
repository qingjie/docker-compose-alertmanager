```
consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' -server --client=0.0.0.0

consul agent -ui --data-dir /tmp/consul --dev -bind '{{GetInterfaceIP "eth0"}}' --server=false --client=0.0.0.0 --join 172.31.41.232 --config-dir=/opt/consul

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

* https://github.com/hashicorp/consul-template
* https://github.com/hashicorp/consul-template#service
* https://github.com/hashicorp/consul-template#services

---
* https://github.com/luksa/kubernetes-in-action/blob/master/Chapter05/kubia-svc-nodeport.yaml
* https://www.consul.io/docs/platform/k8s/run.html

```
(base) Qingjies-MBP-2:test qingjiezhao$ cat test-nodeport.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia

```

```
(base) Qingjies-MBP-2:test qingjiezhao$ cat test-nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30123
  selector:
    app: kubia

```

```
global:
  datacenter: hashidc1

ui:
  service:
    type: "LoadBalancer"

connectInject:
  enabled: true

client:
  enabled: true
  grpc: true

server:
  replicas: 1
  bootstrapExpect: 1
  disruptionBudget:
    enabled: true
    maxUnavailable: 0
syncCatalog:
  enabled: true
  toConsul: true
  toK8S: false
```

* helm upgrade hedgehog /Users/qingjiezhao/Downloads/demo-consul-101/k8s/consul-helm-0.12.0 -f helm-consul-values.yaml
---
```
helm install -f helm-consul-values.yaml --name qingjie --namespace kube-public ./consul-helm

helm delete qingjie --purge
minikube service list
minikube service qingjie-consul-ui -n kube-public

```
---
11/11/2019

outside consul(IP:192.168.1.105)

./consul agent -data-dir=/tmp/data -server -bind '{{ GetInterfaceIP "en0" }}' -bootstrap-expect 1 -ui






helm install -f consul-values.yaml --name consul ./consul-helm-0.12.0

(base) Qingjies-MBP-2:consul qingjiezhao$ cat consul-values.yaml
global:
    datacenter: dc1
    enabled: false

client:
    enabled: true
    join: ["192.168.1.105:8301"]

syncCatalog:
    enabled: true
    toConsul: true
    toK8S: false





----
Note: outside consul IP:192.168.1.105
http://localhost:8500/ui/dc1/services
http://192.168.99.102:30123/
https://sysdig.com/blog/kubernetes-monitoring-prometheus/
