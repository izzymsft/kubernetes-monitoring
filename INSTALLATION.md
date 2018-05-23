# Installation Steps

## Setting up ElasticSearch
Use the following steps to setup the ElasticSearch cluster

```shell

# Set up the ServiceAccount, ClusterRole and ClusterRoleBinding
kubectl create -f elasticsearch/elasticsearch-roles.yaml

# Set up the Internal and External Services
kubectl create -f elasticsearch/elasticsearch-services.yaml 
```

The LoadBalancer services take about 3 to 5 minutes to come up. Once the services are up, you can grab the LoadBalancer ingress IP that will be used to route public traffic to the ElasticSearch cluster via Kibana from the following command:

```shell
kubectl describe service elasticsearch-loadbalancer
```

#### Setting up the ElasticSearch StatefulSet

This command sets up 3 data nodes for the ElasticSearch cluster using Ephemeral Storage.

```shell
# Set up the ElasticSearch cluster (3 ElasticSearch data nodes)
kubectl create -f elasticsearch/elasticsearch-statefulset.yaml
```

#### Setting up the ElasticSearch Environment ConfigMap
Once the external service is up and running, please grab the Public IP address from the LoadBalancer Ingress IP and update the ConfigMap definition values for the elasticsearch.external.url and elasticsearch.external.host config map values.

```shell
vim elasticsearch/elasticsearch-environment-configmap.yaml
```

Once the value has been placed in the elasticsearch-environment-configmap.yaml file you can create the ConfigMap object

```shell
kubectl create -f elasticsearch/elasticsearch-environment-configmap.yaml
```

