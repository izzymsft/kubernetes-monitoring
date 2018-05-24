# Installation Steps

## Disclaimer
This setup uses EmptyDir ephemeral storage to house the ElasticSearch data. It also does not have X-Pack security enabled.
It is only recommended for monitoring development environments and not for Production.
For production, you need to use Kibana and ElasticSearch images that utilize X-Pack security to enable Authentication and Authorization to gain access to the data in ElasticSearch as well as the Kibana UI. You will also need to use Persistent storage large enough to support the amount of data needed for the ElasticSearch stateful set so that the data from the ElasticSearch data nodes can survive pod restarts or relocation to other k8s nodes. Experience managing ElasticSearch clusters is necessary for production scenarios. The daily data volume and retention policies influence the amount of storage needed.

#### Docker Images for Kibana and ElasticSearch

Here are a list of Docker images for Kibana and ElasticSearch

https://www.docker.elastic.co/#

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/docker.html#_image_types

https://www.elastic.co/guide/en/kibana/6.2/docker.html#image-type


#### Subscription Options for ElasticSearch and Kibana with XPack Security

X-Pack Security is enabled in the platinum image. The platinum image includes a trial license for 30 days. After that, you can obtain one of the available subscriptions or revert to a Basic licence. To access your cluster, it’s necessary to set an initial password for the elastic user. The initial password can be set at start up time via the ELASTIC_PASSWORD environment variable.

https://www.elastic.co/subscriptions

## Check Out the Kubernetes Resources YAML Files

Get the codebase and then switch to the v2 branch

```shell

git clone git@github.com:izzyacademy/kubernetes-monitoring.git

cd kubernetes-monitoring

git checkout v2

git pull

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

#### Using AzureDisk for Persistent Storage
In production scenarios, you want the data to survive pod relocation and restarts. If you are running on Azure you can use Azure Disk to set up a storage class that can be used to dynamically provision data volumes for the stateful sets. If you go that route, you can use the following two steps instead to 
(a) Set up the Storage Class for Azure Disk
(b) Set up the Statefulset that utilizes Azure Disk Storage Class for the Persistent VolumeClaimTemplate

```shell
# Set up the Azure Disk Storage Class for Dynamic Volume Provisioning
kubectl create -f storage/azuredisk-storage-class.yaml

# Create the ElasticSearch Cluster with the Statefulset that utilizes the dynamic volume provisioning
 kubectl create -f elasticsearch/elasticsearch-statefulset-persistent-volume.yaml
 
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

#### Label the Nodes for Fluentd Daemonsets

You will also have to label all the kubernetes nodes with the following label before deploying the config maps and fluentd daemonsets

```shell
beta.kubernetes.io/fluentd-ds-ready=true
```

Get the names of your nodes

```shell
$ kubectl get nodes

NAME                        STATUS    ROLES     AGE       VERSION
k8s-agentpool1-30584733-0   Ready     agent     6d        v1.9.7
k8s-agentpool1-30584733-1   Ready     agent     6d        v1.9.7
k8s-agentpool1-30584733-2   Ready     agent     6d        v1.9.7
k8s-master-30584733-0       Ready     master    6d        v1.9.7
```

Use the following syntax to label each node 

```shell
$ kubectl label nodes <node-name> <label-key>=<label-value>

$ kubectl label nodes k8s-agentpool1-30584733-0 beta.kubernetes.io/fluentd-ds-ready=true
$ kubectl label nodes k8s-agentpool1-30584733-1 beta.kubernetes.io/fluentd-ds-ready=true
$ kubectl label nodes k8s-agentpool1-30584733-2 beta.kubernetes.io/fluentd-ds-ready=true
$ kubectl label nodes k8s-master-30584733-0 beta.kubernetes.io/fluentd-ds-ready=true
```

Once the nodes have been labelled you can now set up the Fluentd objects

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

