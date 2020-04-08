
Instructions:

* Replace the broker urls in `04-knative-kafka.yaml` and `D/02-kafkasource.yaml`
* Replace `foo-bar` in `99-openshift-proxy.yaml` with the hostname of your proxy
* Replace `THE_SECRET` in `G-github-source/02-secret.yaml` with the secret you got from Github 

```
k apply -f 01-catalogsource.yaml
# TODO: go to the console, create a subscription for the operator

k apply -f 02-serving.yaml
k apply -f 03-eventing.yaml

# TODO: install Knative Kafka Operator over operatorHub 
k apply -f 04-knative-kafka.yaml

# TODO: install KamelK Operator over operatorHub 
# TODO: create an IntegrationPlatform CR for CamelK
# TODO: apply https://github.com/knative/eventing-contrib/releases/download/v0.13.3/camel.yaml
k apply -f 98-camel-hack.yaml

kubectl apply -n knative-sources -f https://github.com/knative/eventing-contrib/releases/download/v0.11.2/github.yaml


# --- enable proxy
k apply -f 99-openshift-proxy.yaml
# wait until following looks good
k get nodes

# --- start tests

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

k apply -f D-kafka-source/
stern -n default .
k delete -f D-kafka-source/

k apply -f E-kafka-channel/
stern -n default .
k delete -f E-kafka-channel/
  
# NO external services - stopped the work
# k apply -f F-camel-source/
# stern -n default .
# k delete -f F-camel-source/

k apply -f G-github-source/
stern -n default .
k delete -f G-github-source/

```  
