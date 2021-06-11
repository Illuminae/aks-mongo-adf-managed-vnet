# Connecting to a database hosted inside an AKS cluster from Azure Data Factory / Azure Synapse Pipelines using Azure IR

Recently, I have seen multiple scenarios where customers needed to integrate data from sources such as PostgreSQL and MongoDB hosted within an Azure Kubernetes Servic (AKS) cluster. As the cluster is commonly not accessible via public IP, until recently a self-hosted IR (SHIR) was required within the cluster VNET. With Managed VNET for Azure IR and Private Link Service, we can save ourselves the trouble of managing the SHIR. This write-up documents a the sample setup of Mongo DB within AKS & ADF with Azure IR Managed VNET. 

![Setup](https://user-images.githubusercontent.com/7138690/121501238-566ff580-c9df-11eb-8a14-ee1089ca3d07.png)


## 1. AKS setup

1. I set up a 2-node AKS cluster with standard load balancer (default).

```yaml
Basics:
  Subscription:             yourSubscription
  Resource Group:           yourResourceGroup
  Kubernetes Cluster Name:  yourAksClusterName
  Region:                   GermanyWestCentral
  Availability Zones:       1,2,3
  Kubernetes Version:       1.19.11 (default)
  Node Size:                Standard_DS_v2
  Node count:               2
Integrations:
  Container Monitoring:     Disabled
```

3. Setup Mongo DB

    ```shell
    #Adding bitnami chart repository
    helm repo add bitnami http://charts.bitnami.com/bitnami
    ```

We need to specify the required annoations and parameters defined in https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mongodb/values.yaml. 

```yaml
architecture: standalone
service:
    type : LoadBalancer
annotations : 
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```
<!-- tsk --> 

```shell
#Create mongodb namespace
kubectl create namespace mongodb
#Install bitnami chart with values.yaml defining params and annotations
helm install mongodb -f values.yaml bitnami/mongodb -n mongodb
#Verify the service is up and running
kubectl get service -n mongodb
```
