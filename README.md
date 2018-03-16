# custom-metrics-hpa

0. update configuration for ICP 
```
[root@kvm-014724 installer]# git diff playbook/roles/master/templates/pods/master.json.j2
diff --git a/playbook/roles/master/templates/pods/master.json.j2 b/playbook/roles/master/templates/pods/m
index 04c61eb..0d1b3a5 100644
--- a/playbook/roles/master/templates/pods/master.json.j2
+++ b/playbook/roles/master/templates/pods/master.json.j2
@@ -41,7 +41,7 @@
           "--cloud-config={{ cloud_provider_conf }}",
           {% endif -%}
           "--leader-elect=true",
-          "--horizontal-pod-autoscaler-use-rest-clients=false"
+          "--horizontal-pod-autoscaler-use-rest-clients=true"
         ],
         "volumeMounts": [
           {
@@ -90,7 +90,7 @@
           "--requestheader-client-ca-file=/etc/cfc/conf/front/front-proxy-ca.pem",
           "--requestheader-username-headers=X-Remote-User",
           "--requestheader-group-headers=X-Remote-Group",
-          "--requestheader-allowed-names={{ cluster_CA_domain }}",
+          "--requestheader-allowed-names={{ cluster_CA_domain }},aggregator",
           "--requestheader-extra-headers-prefix=X-Remote-Extra-",
           "--proxy-client-cert-file=/etc/cfc/conf/front/front-proxy-client.pem",
           "--proxy-client-key-file=/etc/cfc/conf/front/front-proxy-client-key.pem",
```

1. install ICP

2.  clone demo codes
``` 
git clone git@github.com:huxiaoliang/custom-metrics-hpa.git
```
3.  deploy the Prometheus custom metrics API adapter
```
kubectl create -f ./custom-metrics-api
```
4.  list the custom metrics provided by Prometheus on pod:
```
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .  |grep "pods/"
```
5.  create some pods and hpa policy base on cup usage
```
kubectl create -f ./pod-info
```
6.    auto scaling
```
for a in `seq 1 50`; do ab -rSqd -c 200 -n 20000 http://9.111.212.131:31198/;done
```
7.  base on memory usage (10485760=10m)
```
[root@kvm-014377 podinfo]# cat podinfo-hpa-custom2.yaml 
---
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: podinfo
spec:
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: podinfo
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: memory_usage_bytes
      targetAverageValue: 10485760
```
8.    auto scaling
```
for a in `seq 1 50`; do ab -rSqd -c 200 -n 20000 http://9.111.212.131:31198/;done
```


