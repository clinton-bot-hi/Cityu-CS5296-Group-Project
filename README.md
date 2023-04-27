# Artifact Repository for CS5296 Course Project (Group 9)

This repository aims at reproducing and deploying the Kubernetes cluster with **AWS EKS** and **kOps**. 

Most of the codes are borrowed from 
 - [prometheus-operator / kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)
 - [GoogleCloudPlatform / microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
 - [kubernetes-sigs / metrics-server](https://github.com/kubernetes-sigs/metrics-server/)

## Configuration
Before using EKS and kOps for the deployment, please remember to set up the configuration of AWS with the `Access key ID`, `Secret access key`, default AWS zone and set the default output to **Json**.

## Installation
- AWS CLI [[Reference]](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
- Kubernetes [[Reference]](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
- kOps [[Reference]](https://kops.sigs.k8s.io/getting_started/install/)
```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
```
- eksctl [[Reference]](https://eksctl.io/introduction/?h=install#installation)
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

## Deployment
> All source code are inside the `k8s-config` folder, which contains **microservices-app-demo** and **monitoring**.

>The web application can be opened with the `external IP`.

### Amazon Elastic Kubernetes Service (EKS)
To deploy the demo application with the Kubernetes cluster on AWS EKS, we need to move to our application folder **microservices-app-demo** and execute the following command in the command prompt:
```
eksctl create cluster --name <our-cluster-name> --region <our-region> --nodegroup-name <our-node-group-name> --node-type t2.medium --managed

kubectl create ns boutique
kubectl -n boutique apply -f ./release/kubernetes-manifests.yaml
kubectl -n boutique get pods
kubectl -n boutique get svc/frontend-external
```

### Kubernetes Operations (kOps)
Similarly, to deploy the demo application with kOps, we need to move to our application folder **microservices-app-demo** and execute the following command in the command prompt:
```
kops create cluster --name <our-cluster-name> --state <our-s3-bucket-location> --zones <our-availability-zone> --master-size t2.large --node-size t2.medium --node-count 3

kops update cluster --name <our-cluster-name> --yes --admin

kubectl create ns boutique
kubectl -n boutique apply -f ./release/kubernetes-manifests.yaml
kubectl -n boutique get svc/frontend-external
```
Then, validate the created cluster by running following code
```
kops vaidate cluster
```

### Monitoring with metric-server API
We can move to **monitoring** folder and execute the following code for activating the **metric-server** and access its API
```
kubectl apply -f metric-server.yaml
kubectl top nodes
kubectl top pods -n <namespace>
```

### Monitoring with Kubernetes Dashboard
To activate **Kubernetes Dashboard**, we will need to create an `admin-user` and create a token for `admin-user` to access the dashboard UI. We can move to **monitoring** folder and execute the following code to create the **ClusterRole** - `admin-user`.
```
kubectl apply -f k8sdashboardrole.yaml
```
Then, execute the following code to activate the dashboard and create a bearer token for access. 
```
kubectl apply -f k8sdashboard.yaml
kubectl -n kubernetes-dashboard create token admin-user
```
The bearer token should look like
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```
Finally, run the following code to access the **Kubernetes Dashboard** [here](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/). Paste the bearer token at `Token`.
```
kubectl proxy
```

### Monitoring with Prometheus & Grafana Dashboard
We can move to **monitoring** folder and execute the following code for activating **Prometheus** and **Grafana**:
```
kubectl apply --server-side -f ./prometheus-grafana/setup
kubectl wait \
    --for condition=Established \
    --all CustomResourceDefinition \
    --namespace=monitoring
kubectl apply -f ./prometheus-grafana/
kubectl port-forward -n monitoring grafana-application-name 3000
```

The pod name should be looks like **grafana-xxx-xxx**. Lastly, we can go to localhost:3000 to view the dashboard. The default username and password are both `admin`. 