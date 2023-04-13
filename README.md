# k3s-example-project

This project is for developers who want to try and play with Kubernetes themselves but never had an opportunity to do so. 
I'm using here the lightweight Kubernetes distribution called **k3s** https://k3s.io. So, you can run it on your laptop easily.
All the commands are tested on Ubuntu 22.04. k3d and k3s versions I've used are here:
```bash
k3d version v5.4.6
k3s version v1.24.4-k3s1
```

The project includes/uses the following components:
* k3s, k3d
* kubectl
* ArgoCD
* Prometheus
* Grafana
* Example NodeJS app deployed to Kubernetes from here https://github.com/madlopt/simple-nodejs-app

After installation, you will be able to see metrics of the NodeJS app in Grafana dashboard, see logs of it in ArgoCD, see what Prometheus is collecting and so on.

### Prerequisites
Before proceeding, make sure you have the following software installed:

* kubectl
* k3d

### k3d and kubectl installation
Run the following commands to install k3d and kubectl:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo apt-get install -y apt-transport-https
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
k3d --version
```

Then clone the repository and run the following commands from the root of the project.

### Cluster Creation
Create a k3d cluster by running the following command:

```bash
k3d cluster create dev-cluster --port 8080:80@loadbalancer --port 8443:443@loadbalancer
```
To delete the cluster, run:

```bash
k3d cluster delete --all
```
###  ArgoCD Installation
Create the ArgoCD namespace and deploy it:

```bash
kubectl create namespace argocd
kubectl create -n argocd -f argocd/install.yaml
```
You can create the ArgoCD ingress by running:

```bash
kubectl apply -f ingress/ingress.yaml -n argocd
```
Get the initial admin password by running:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```
You can access the ArgoCD dashboard by visiting http://localhost:8080/argocd.

If you need to create a development namespace, run:

```bash
kubectl create namespace dev
```
To create a new project in the default namespace, run:

```bash
kubectl create -n argocd -f project/development-project.yaml
```
###  Prometheus
Deploy Prometheus using the following command:

```bash
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
```
Create the Prometheus service account and role binding:

```bash
kubectl apply -f prometheus/prom-rbac.yaml
```
Deploy the Prometheus instance:

```bash
kubectl apply -f prometheus/prometheus.yaml
```
You can check if the Prometheus instance is running by running:

```bash
kubectl get prometheus
```
Create the Prometheus service:

```bash
kubectl apply -f prometheus/prometheus-service.yaml
```
To access the Prometheus dashboard, run:

```bash
kubectl port-forward svc/prometheus 9090
```
Visit http://localhost:9090 to view the dashboard.

Create the Prometheus ServiceMonitor:

```bash
kubectl apply -f prometheus/prometheus-servicemonitor.yaml
```

### Grafana

To install Grafana, follow these steps:

Create the necessary Kubernetes resources using the following commands:
```bash
kubectl create -f grafana/datasources-config.yaml
kubectl create -f grafana/dashboard-providers-config.yaml
kubectl create -f grafana/dashboard-config.yaml
```
Apply the Grafana YAML file:
```bash
kubectl apply -f grafana/grafana.yaml
```
Wait a few minutes for Grafana to be deployed.

Forward the Grafana port to your local machine using the following command:

```bash
kubectl port-forward service/grafana 3000:3000
```
Open your web browser and go to http://127.0.0.1:3000.

Log in to Grafana using the default username and password (admin/admin), then change the password to a secure one.

Check the Prometheus datasource by navigating to Data Sources and adding a new one.

Get the IP for the URL by running the following command:

```bash
kubectl get service
```
Look for the prometheus service and note the ClusterIP address.

Import the Prometheus dashboard by going to https://grafana.com/grafana/dashboards/3662-prometheus-2-0-overview/ and using the dashboard ID `3662`.

### Node.js Application
To install the Node.js application, follow these steps:

Create the necessary Kubernetes resources using the following command:
```bash
kubectl create -n argocd -f nodejs-application.yaml
```
Wait a few minutes for the application to be deployed.

To delete the application, run the following command:

```bash
kubectl delete app nodejsapp -n argocd
```

That's it! You can now access the Node.js application by visiting http://localhost:8080.

## Usage
### Grafana
To use Grafana, follow these steps:

Forward the Grafana port to your local machine using the following command:
```bash
kubectl port-forward service/grafana 3000:3000
```
Open your web browser and go to http://127.0.0.1:3000.

Log in to Grafana using your username and password.

You can now create and manage your dashboards.

### Node.js Application
To use the Node.js application, follow these steps:

Forward the application port to your local machine using the following command:
```bash
kubectl port-forward service/nodejsapp --namespace=default 80:80
```
Open your web browser and go to http://localhost/metrics.

You can now view the application metrics.

### Important Paths
Here are some important paths that you may need to access:

* /var/lib/grafana/dashboards: Contains Grafana dashboards
* /etc/grafana/provisioning/dashboards: Contains dashboard provider YAML files
* /etc/grafana/provisioning/datasources: Contains datasource YAML files

### Restarting Deployments
To restart a deployment, use the following command:

```bash
kubectl rollout restart deployment <deployment_name> -n <namespace>
```
### Deleting Resources
To delete all resources in the default namespace, use the following command:
```bash
kubectl delete all --all --namespace default
```
Remember this command! It will help you clean up your cluster. k3s it's a lightweight alternative to k8s, but it's still a huge tool which can use the whole of your resources and your machine will stuck.



### Just FYI

If you use the prom-client library https://github.com/siimon/prom-client, please look at the memory leak fix I've made here https://github.com/madlopt/simple-nodejs-app


