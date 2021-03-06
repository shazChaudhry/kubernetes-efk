# Elastic Cloud on Kubernetes
https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html

**NOTE**: Namespace in the elasticstack has been changed to `logging`

In this repository, we will be setting up Elasticsearch, Kibana and then Fluentd

## Elastic
In this section, we will set up elastitc operator along with Elasticsearch and Kibana
### Deploy ECK
Install custom resource definitions and the operator with its RBAC rules: 
- `kubectl apply -f elastic/all-in-one.yaml`
- `kubectl -n logging logs -f statefulset.apps/elastic-operator` - Monitor the operator logs: 

### Deploy an Elasticsearch cluster
Apply a simple Elasticsearch cluster specification, with three Elasticsearch node:
- Deploy elasticsearch: `kubectl apply -f elastic/elasticsearch.yaml`
- Monitor cluster health and creation progress: 
  - `kubectl get elasticsearch -n logging`
  - `kubectl get pods -n logging --selector='elasticsearch.k8s.elastic.co/cluster-name=elasticsearch'` 

### Deploy a Kibana instance
Specify a Kibana instance and associate it with your Elasticsearch cluster: 
- `kubectl apply -f elastic/kibana.yaml`
- `kubectl get kibana -n logging`

## Get elastic credentials:
Once elasticsearch has been deployed and is showing healthy, use the following command to retrieve password. These credentials will be used by both 
fluentd and beats:
- `export PASSWORD=$(kubectl get secret elasticsearch-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n logging)`
- `echo $PASSWORD; echo`

## Deploy Fluentd
In this section, we will deploy fluentd _(as a DaemonSet)_ which will also set up and index lifecycle management in Elasticsearch:
- `kubectl apply -f fluentd/fluentd.yaml`

## Index patterns
Before you can view logs in Kibana: 
- `kubectl port-forward service/kibana-kb-http 5601 -n logging`
- Visit `http://localhost:5601/` in your web browser
- you will need to create a timestemp based index pattern `logstash-*`
- Log in with user = `elastic` and password that you retrieved above

### References
- https://github.com/fluent/fluentd-kubernetes-daemonset
- https://github.com/uken/fluent-plugin-elasticsearch
- https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/templates/conf/fluent.conf.erb

## Test shipping of contaier logs
- `kubectl run pingpong --image alpine ping 8.8.8.8`
