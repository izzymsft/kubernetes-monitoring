

## Create the Dashboard Service Account and Cluster Role Binding

This is based on the steps here https://github.com/kubernetes/dashboard/wiki/Creating-sample-user

```shell

kubectl create -f dashboard-admin.yaml
```

## Setting Up Default Namespace

To avoid having to specify the namespace each type we need to look up a resource, lets set the default namespace as "kube-system" in the configs



```shell

### This is the syntax

kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>

kubectl config set-context $(kubectl config current-context) --namespace=kube-system
```

## Validate The Configuration Change

```shell
kubectl config view | grep namespace
```

## Let's verify if the service account was created

```shell
kubectl get secret | grep admin-user
```

## Retrieve the Access Token

Let's retrieve the access token, using the secret name retrieved from the previous step

```shell
kubectl describe secret admin-user-token-6mc8b
```

## Connect to the Dashboard

Use the token to log into the dashboard when prompted for a valid Bearer Token

Open a different console window and run the following command to bring up the proxy necessary for accessing the dashboard

```shell

kubectl proxy

```

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
