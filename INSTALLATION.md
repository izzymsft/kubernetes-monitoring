# Installation Steps

## Check Out the Kubernetes Resources YAML Files

Get the codebase and then switch to the v2 branch

```shell

git clone git@github.com:izzyacademy/kubernetes-monitoring.git

git checkout v2

cd kubernetes-monitoring

```

## Setting up ElasticSearch 3-Node Cluster
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

Once the statefulset is up and the LoadBalancer Ingress IP is available, you should be able to reach the ElasticSearch REST API using that IP and port number as follows:

```shell

http://104.136.220.85:9200

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

## Setting Up Fluentd DaemonSets

These steps will install the objects necessary to aggregate log events from the k8s cluster and ship them to ElasticSearch

```shell

# Set up the ServiceAccount, ClusterRole and ClusterRoleBinding
kubectl create -f fluentd/fluentd-elasticsearch-roles.yaml

# Set up the Config Map for FluentD
kubectl create -f fluentd/fluentd-elasticsearch-configmap.yaml  

# Install the Daemonset that grabs log events from all K8s nodes within the cluster
kubectl create -f fluentd/fluentd-elasticsearch-daemonset.yaml  

```


## Setting up Kibana UI

These steps will set up the Kibana User Interface to access log events from ElasticSearch

```shell

kubectl create -f kibana/kibana-services.yaml

kubectl create -f kibana/kibana-deployment.yaml  

```

Once the Load Balancer Service for Kibana Is Ready you should be able to retrieve the IP by describing the service as follows:

```shell

kubectl describe service kibana-loadbalancer

```

The Kibana UI can be accessed on port 5601 on the Public IP. You should be able to accept "logstash-*" as the mapping pattern and specify @timestamp as the date field.

