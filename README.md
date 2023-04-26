# Repository for CS5296 Course Project (Group 9)

This repository aims to reproduce and deploy the Kubernetes cluster with **AWS EKS** and **kOps**. 

Most of the code is borrowed from 
[prometheus-operator](https://github.com/prometheus-operator/kube-prometheus) and [Google Cloud Platform](https://github.com/GoogleCloudPlatform/microservices-demo). 

## Configuration
Before the deployment for EKS, please remember to set up the configuration of AWS with the access key, private key, region and set the default output to **Json**.

## Installation
- Kubernetes CLI
- AWS CLI
- Eksctl

## Deployment
> All source code are inside the `k8s-config` folder, which contains **microservices-app-demo** and **monitoring**.

>The web application can be opened with the `external IP`.

### Amazon Elastic Kubernetes Service(EKS)
To deploy the demo application with the Kubernetes cluster on AWS EKS, we need to move to our application folder and execute the following command in the command prompt:
```
eksctl create cluster --name our-cluster-name --region our-region --nodegroup-name our-node-group-name --node-type t2.medium --managed

kubectl create ns boutique

kubectl -n boutique apply -f ./release/kubernetes-manifests.yaml

kubectl -n boutique get pods

kubectl -n boutique get svc/frontend-external
```

### Kubernetes Operations(kOps)
Similarly, to deploy the demo application with kOps, we need to move to our application folder and execute the following command in the command prompt:
```
kops create cluster --name cs5296.k8s.local --state s3://cs5296-k8s-kops-store --zones us-east-1a --master-size t2.large --node-size t2.medium --node-count 3

kubectl create ns boutique

kubectl -n boutique apply -f ./release/kubernetes-manifests.yaml

kubectl -n boutique get svc/frontend-external
```

### Monitoring with Grafana dashboard
We can move on to **monitoring** folder and execute the following code for activing **Prometheus** and **Grafana**:
```
kubectl apply --server-side -f monitoring/prometheus-grafana/setup

kubectl wait \
    --for condition=Established \
    --all CustomResourceDefinition \
    --namespace=monitoring

kubectl apply -f manifests/

kubectl port-forward -n monitoring grafana-application-name 3000
```

The pod name should be looks like **grafana-xxx-xxx**. Lastly, we can go to localhost:3000 to the Grafana dashboard. And the default username and password are both `admin`. 