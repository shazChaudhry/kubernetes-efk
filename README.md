## Elastic Cloud on Kubernetes
https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html

Once elasticsearch has been deployed and is showing healthy, use the following command to retrieve password:
- `export PASSWORD=$(kubectl get secret elasticsearch-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n logging)`
- `echo $PASSWORD`

Generate passwords for Elasticsearch default users:
- `kubectl exec -it elasticsearch-es-default-0 -n logging -- bin/elasticsearch-setup-passwords auto -b`

## Deploy Fluentd
- https://github.com/fluent/fluentd-kubernetes-daemonset
- https://github.com/uken/fluent-plugin-elasticsearch
- https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/templates/conf/fluent.conf.erb

Before deploying Fluentd, create a Kubernetes Secret. Elasticsearch will otherwise refuse the connection:
- `kubectl create secret generic elasticsearch-pw -n logging --from-literal password=$PASSWORD`

## Nginx Ingress controller
https://github.com/nginxinc/kubernetes-ingress

## Test shipping of contaier logs
kubectl run pingpong --image alpine ping 8.8.8.8

## Setting up a stream in Elasticsearch
```Bash
DELETE _ilm/policy/fluentd_policy
GET _ilm/policy/fluentd_policy
PUT _ilm/policy/fluentd_policy
{
  "policy": {
    "phases": {
      "hot": {                      
        "actions": {
          "rollover": {
            "max_size": "1mb",     
            "max_age": "2m"
          }
        }
      },
      "delete": {
        "min_age": "5m",           
        "actions": {
          "delete": {}              
        }
      }
    }
  }
}

DELETE _index_template/fluentd_template
GET _index_template/fluentd_template
PUT _index_template/fluentd_template
{
  "index_patterns": ["fluentd"],                   
  "data_stream": {},
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "fluentd_policy"     
    }
  }
}

DELETE fluentd/_doc
PUT /fluentd/_bulk?refresh
{"create":{ }}
{ "@timestamp": "2020-12-08T11:04:05.000Z", "user": { "id": "vlb44hny" }, "message": "Login attempt failed" }
{"create":{ }}
{ "@timestamp": "2020-12-08T11:06:07.000Z", "user": { "id": "8a4f500d" }, "message": "Login successful" }
{"create":{ }}
{ "@timestamp": "2020-12-09T11:07:08.000Z", "user": { "id": "l7gk7f82" }, "message": "Logout successful" }

GET .ds-fluentd-*/_ilm/explain
GET /_data_stream/fluentd
DELETE /_data_stream/fluentd
```