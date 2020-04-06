
Instructions:

* Replace `foo-bar` in `99-openshift-proxy.yaml` with the hostname of your proxy

```
# PHASE 1: try things once to cache images

k apply -f 01-catalogsource.yaml
# go to the console, create a subscription for the operator
k apply -f 02-serving.yaml
k apply -f 03-eventing.yaml

k apply -f A-source-to-sink/
stern -n default .
k delete -f A-source-to-sink/

k apply -f B-channel-and-subscription/
stern -n default .
k delete -f B-channel-and-subscription/

k apply -f C-broker-trigger/
stern -n default .
k delete -f C-broker-trigger/
k label namespace default knative-eventing-injection=disabled --overwrite
k delete broker -n default default
```

```
# PHASE 2: enable proxy settings (to a proxy that blocks everything except google.com and quay.io)

k apply -f 99-openshift-proxy.yaml
# wait until following looks good
k get nodes

```


```
# PHASE 3: try again with no internet connection

k apply -f A-source-to-sink/
stern -n default .
k delete -f A-source-to-sink/

k apply -f B-channel-and-subscription/
stern -n default .
k delete -f B-channel-and-subscription/

k apply -f C-broker-trigger/
stern -n default .
k delete -f C-broker-trigger/
k label namespace default knative-eventing-injection=disabled --overwrite
k delete broker -n default default
```  
