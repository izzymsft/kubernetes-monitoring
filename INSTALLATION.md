# Installation Steps

## Setting up ElasticSearch
Use the following steps to setup the ElasticSearch cluster

```shell


kubectl create -f elasticsearch/elasticsearch-roles.yaml

kubectl create -f elasticsearch/elasticsearch-services.yaml 

kubectl create -f elasticsearch/elasticsearch-statefulset.yaml

kubectl create -f elasticsearch/elasticsearch-environment-configmap.yaml

```
