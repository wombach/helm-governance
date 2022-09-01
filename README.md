Getting started
===============
Welcome to the Aurelius Atlas solution powered by Apache Atlas! Aurelius Atlas is an open-source Data Governance solution, based on a selection of open-source tools to facilitate business users to access governance information in an easy consumable way and meet the data governance demands of the distributed data world.

Google Setup Instructions
=========================
Install Gcloud
https://cloud.google.com/sdk/docs/install#deb

Install kubectl and configure to work with gloud
https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#gcloud

follow steps in Installation Instructions

Installation Instructions
=========================

This installation assumes that you have:
- a kubernetes cluster running
- kubectl installed
- Chosen cloud Cli installed (gcloud or az)
- A Domain Name Space 
  - Not nessisary for Azure

If you do not please look at the Section Above for links on how to install.

## Required Packages
The deployment requires the following packages:
- Certificate Manager
  - To handel and manage the creation of certificates
  - Used in demo: cert-manager
- Ingress Controller
  - Used to create an entry point to the cluster through an external IP.
  - Used in demo: Nginx Controller
- Elastic
  - Used to deploy elastic on the kubernetes cluster
- Reflector
  - Used to reflect secrets across namespaces
  - Used in demo to share the DNS certificate to different namespace

### The steps on how to install the required packages

##### 1. Install Certificate manager
The certificate manager here is cert-manager.
https://cert-manager.io/docs/installation/helm/
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install  cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.9.1 
```
##### 2. Install Ingress Nginx Controller
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```
##### 3. Install Elastic
```bash
kubectl create -f https://download.elastic.co/downloads/eck/2.3.0/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.3.0/operator.yaml
```
##### 4. Install Reflector
```bash
helm repo add emberstack https://emberstack.github.io/helm-charts
helm repo update
helm upgrade --install reflector emberstack/reflector
```

## Get Ingress Controller External IP to link to DNS 

##### Get External IP to link to DNS
```bash
kubectl get service/nginx-ingress-ingress-nginx-controller
```
Take the external-IP of the ingress controller
Link your DNS to this external IP.

In Azure it is possible to apply a dns label to the ingress controller.
#### Azure DNS Label
https://hovermind.com/azure-kubernetes-service/applying-dns-label-to-the-service.html
Edit the ingress controller deployment 
```bash
kubectl edit deployment.apps/nginx-ingress-ingress-nginx-controller
```
Under Annotations add the following providing your desire label <label>:

```
service.beta.kubernetes.io/azure-dns-label-name: <label>
```
Save and exit.
Resulting DSN will be ``<label>.westeurope.cloudapp.azure.com``


## Certify DNS to Secret

##### Define a cluster issuer
Here we define a CLusterIssuer using letsencrypt on the cert-manager namespace
- move to the directory of the chart helm-governance
* uncomment prod_issuer.yaml in templates
* update the email on line 11 ``{{ EMAIL ADDRESS }}``
* Create the clusterIssuer with the following command
```bash
kubectl apply -f templates/prod_issuer.yaml
```
comment out prod_issuer.yaml in templates
Check that it is running:
```bash
kubectl get clusterissuer -n cert-manager 
```
It is running when Ready is True
![img.png](img.png)

##### Create certificate for DSN
- Assumes you have a DNS linked to the external IP of the ingress controller
- move to the directory of the chart helm-governance
* uncomment prod_issuer.yaml in templates
* update the dnsNames on line 12 to your DNS name ``{{ DNS URL }}``
* Create the certificate with the following command
```bash
kubectl apply -f templates/certificate.yaml
```
comment out certificate.yaml in templates
Check that it is approved.
```bash
kubectl get certificate -n cert-manager 
```
It is running when Ready is True
![img_1.png](img_1.png)


Deploy Aurelius Atlas
============
- Create the namespace
- Update the Values file 
  - DNS name
  - external IP
deploy the services

```bash
kubectl create namespace <namespace>
cd helm-governance
helm install --generate-name -n <namespace>  -f values.yaml .
./get_passwords.sh <namespace>
```

Check that all pods are running
``` bash
kubectl -n <namespace> get all # check that all pods are running
```
Initialize the Atlas flink tasks and optionally load sample data
```bash ${1}
kubectl -n <namespace> exec -it <pod/flink-jobmanager-f795bd969-f4lpk> -- bash
cd init
./init_jobs.sh
## To Load the Sample Demo Data 
./load_sample_data.sh
```



Elastic:
===================
- In order to deploy elastic, ``Elastic Cluster on Kubernetes (ECK)`` must be installed on the cluster. To install ECK on the cluster, please follow the instructions provided on https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-deploy-eck.html
- For more details about this elastic helm chart look at [elastic readme](./charts/elastic/README.md)

Flink:
===================
- For more details about this flink helm chart look at [flink readme](./charts/flink/README.md)


Atlas is now accessible via reverse proxy at
https://aureliusdev.westeurope.cloudapp.azure.com/anwo/atlas2/login.jsp


