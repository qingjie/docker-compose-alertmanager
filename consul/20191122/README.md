
![](img/Consul_Across_K8S.jpg)

```
kubectl create -f pv.yml

helm install -f prometheus-values.yml -n promethues stable/prometheus

watch -n 1 kubectl get pods


helm install -f consul-values.yml -n consul ./consul-helm
```
