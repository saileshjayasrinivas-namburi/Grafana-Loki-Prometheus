
# Logging Setup with Grafana, Prometheus, and Loki on AWS/GCP

This repository contains configurations and instructions for setting up logging infrastructure using Grafana, Prometheus, and Loki on AWS.

#### Access files based on the service name in the repository. 

## Prometheus Setup
Add Helm repository and update:
  ``` 
helm repo add Prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
  ``` 
### Update Prometheus.yaml file as needed. Modify configurations such as jobs, retention, and volume size.

#### yaml
# Example configuration for jobs and retention
  ``` 
scrape_configs:
  - job_name: Port_PlutoMetrics
    scheme: http
    static_configs:
      - targets: ['IP']
    basic_auth:
      username: "admin"
      password: "password"

retention: "90d"

persistentVolume:
  size: <size_as_required>

  ``` 
  
Install Prometheus using Helm:
  ```
helm install prometheus Prometheus-community/prometheus --namespace prometheus --values prometheus.yaml
  ``` 

## Grafana Setup
Create Grafana namespace:
  ``` 
kubectl create namespace grafana
Add Grafana Helm repository:
helm repo add grafana https://grafana.github.io/helm-charts
  ``` 
## Install Grafana using Helm:

  ```
helm install grafana grafana/grafana \
  --namespace grafana \
  --set persistence.storageClassName="gp2" \
  --set persistence.enabled=true \
  --set adminPassword='password' \
  --set service.type=LoadBalancer
  ``` 
  
## Loki Setup
Create logging namespace:
  ``` 
kubectl create ns logging'
  ``` 
## Loki Helm repository:
  ``` 
helm repo add grafana https://grafana.github.io/helm-charts
  ```
### Update loki.yaml file with necessary changes, including bucket name, access key, secret key, and node selector.

### Example configuration for Loki
loki:
  bucketname: <your_bucket_name>
  accesskey: <your_access_key>
  secretkey: <your_secret_key>
  nodeselector: <your_node_selector>

Update s3 values as well 
#Repace values as per your Env  
s3://accesskey:secret-key@ap-south-1/s3-bucketname 


### Install Loki using Helm:
  ``` 
helm upgrade --install loki grafana/loki-simple-scalable -n logging -f loki.yaml
  ```
## Install Promtail:

#### Update the lokiAddress in promtail

ex:- http://loki-write.name-space.svc.cluster.local:3100/loki/api/v1/push 

  ``` 
helm upgrade --install promtail grafana/promtail -f promtail.yaml -n logging
  ``` 

## Apply Loki Ingress:
### Loki ingress helps to Connect globally, EX: Add as a DataSource for LogCli

#### ADD host-Path (DNS) In Ingress for Read & Write Service

  ```

kubectl apply -f loki_ingress.yaml
  ``` 

### Once the setup is complete, add the Loki datasource in Grafana using the provided URL.

#### Example URL: http://loki-read.name-space.svc.cluster.local:3100

## Grafana Ingress Configurations 

#### To access Grafana globally

1.obtain the LoadBalancer URL Doing Describe Ingress,and either associate it with a DNS entry 
2.configure an Ingress that is accessible under whitelisted or private networks.


#### Update Below Values In Grafana/Ingress1.yaml

1.certificate-arn
2.security-groups
3.whitelist-source-range
4.ssl-policy

##### Update Below Values In Grafana/Ingress2.yaml
It helps to Access Grafana in HTTPS (Basic Redirections from 80 to 443)

#### Update Below Values In Grafana/Ingress2.yaml

1.ssl-policy 

# GCP Loki Setup Will Change For GCP

## Introduction
This example gives you an example or getting started overrides value file for deploying Loki using the Simple Scalable architecture in GKE and using GCS.

## Installation of Helm Chart
These instructions assume you have already have access to a Kubernetes cluster, GCS Bucket and GCP Service Account which has read/write permissions to that GCS Bucket.

### Populate Secret Values

Populate the examples/enterprise/enterprise-secrets.yaml so that:
- The gcp_service_account.json secret has the contents of your GCP Service Account JSON key
- Update the Values inside Loki-secrets.yaml 

Deploy the secrets file to your k8s cluster.

```
 kubectl apply -f loki-secrets.yaml
```



### Configure the Helm Chart
lokivalues-gcs.yaml and replace  ```{YOUR_GCS_BUCKET} ``` with the name of your GCS bucket. If there are other things you'd like to configure, view the core Values.yaml file and override anything else you need to within the lokivalues-gcs.yaml file.

### Add Repo to Loki 

 ```
 helm repo add grafana https://grafana.github.io/helm-charts
 helm repo update
```

### Install the Helm chart

 ```
 helm upgrade --install --values {PATH_TO_YOUR_OVERRIDES_YAML_FILE} {YOUR_RELEASE_NAME} grafana/loki-simple-scalable --namespace {KUBERNETES_NAMESPACE}
  ```

### Once the setup is complete, add the Loki datasource in Grafana using the provided URL.

##### Example URL: http://loki-read.<name-space>.svc.cluster.local:3100






















