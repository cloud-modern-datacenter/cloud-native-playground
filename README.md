# Cloud Native Playground

This playground contains a couple of getting started guides for running cloud native workloads using Kubernetes.

## Launch Kubernetes

With [Amazon Elastic Kubernetes Service]((https://aws.amazon.com/eks/)) (EKS), [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS), [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE) you can run workloads on Kubernetes without operating the Kubernetes management infrastructure yourself.

In this playground we will make use of AWS EKS. In the future we may add other certified Kubernetes distributions such as AKS, GKE, Pivotal Kubernet Service (PKS) or Docker EE Enterpise.

### AWS

Make sure you have installed AWS cli, Kubectl and aws-iam-authenticator, see [AWS prerequisites](aws/prerequisites.md)

We Create a EKS cluster using [eksctl](https://eksctl.io/). `eksctl` is a simple CLI tool for creating clusters on EKS - Amazon’s new managed Kubernetes service for EC2. It is written in Go, and uses CloudFormation.

we need to download the eksctl binary:

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
````

Alternatively, macOS users can use Homebrew:

```bash
brew install weaveworks/tap/eksctl
```

```bash
eksctl create cluster --name=kpn-cloud-native-playground --nodes=3
```

> Launching EKS and all the dependencies will take approximately 15 minutes

Once you have created a cluster, you will find that cluster credentials were added in `~/.kube/config`. If you have kubectl v1.10.x as well as aws-iam-authenticator commands in your PATH, you should be able to use kubectl. If you installed eksctl via Homebrew, you should have all of these dependencies installed already.

Alternatively, it is also possible to create an EKS cluster using Terraform using [this guide](aws/eks-terraform/).

## Kubernetes Dashboard

The official Kubernetes dashboard is not deployed by default, but there are instructions in [the official documentation](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

We can deploy the dashboard with the following command:

```bash
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

Since this is deployed to our private cluster, we need to access it via a proxy:

```bash
kubectl proxy --port=8080 --address='0.0.0.0'
```

We need a token to login in the dashboard:

```bash
aws-iam-authenticator token -i kpn-cloud-native-playground
```

Get the value from the "token" field and use it to login the dashboard:

http://localhost:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

## Helm

[Helm](https://www.helm.sh/) helps you manage Kubernetes applications. Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.

Let 's install Helm by downloading a binary for Linux or macOS.

Linux:

```bash
https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
tar -zxvf helm-v2.0.0-linux-amd64.tgz
mv linux-amd64/helm /usr/local/bin/helm
```

macOS:

```bash
https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-darwin-amd64.tar.gz
tar -zxvf helm-v2.11.0-darwin-amd64.tar.gz
mv darwin-amd64/helm /usr/local/bin/helm
```

Alternatively, macOS users can use Homebrew:

```bash
brew install kubernetes-helm
```

When you have a Kubernetes cluster with RBAC. You first need a ServiceAccount for Tiller (server component of Helm which will be installed within your cluster).

```bash
kubectl create -f helm/tiller-rbac-config.yaml
```

Initialize helm client and server with:

```bash
helm init --service-account tiller
```

> NOTE: Read more about securing Helm and Tiller here https://docs.helm.sh/using_helm/#securing-your-helm-installation

Let's try to add an extra chart repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Run a NGINX webserver:

```bash
helm install --name mywebserver --namespace demo bitnami/nginx
```

Get the external ip-address to see if NGINX is running and exposed using a load-balancer:

```bash
kubectl get svc -n demo mywebserver-nginx
```

## Prometheus and Grafana

Create storage class for creating persistent volumes using EBS:

```bash
kubectl create -f aws/prometheus-storageclass.yaml
```

Install Prometheus:

```bash
helm install -f monitoring/prometheus-values.yaml stable/prometheus --name prometheus --namespace prometheus
```

```bash
kubectl get pods --namespace prometheus
```

Get the Prometheus server URL:

```bash
export POD_NAME=$(kubectl get pods --namespace prometheus -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace prometheus port-forward $POD_NAME 9090
```

Install Grafana:

```bash
helm install -f monitoring/grafana-values.yaml stable/grafana --name prometheus --namespace grafana
```

Get the Grafana URL to visit:

```bash
export POD_NAME=$(kubectl get pods --namespace prometheus -l "app=grafana,component=" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace prometheus port-forward $POD_NAME 3000
```

There are several Kubernetes dashboards available on the Grafana Store. You can import those in Grafana. Feel free to try these out:
* https://grafana.com/dashboards/6417
* https://grafana.com/dashboards/3131
* https://grafana.com/dashboards/1471
* https://grafana.com/dashboards/1621

## Roadmap
* Logging using Elasticsearch, Fluentd and Kibana
* Ingress using NGINX or Amazon Load-Balancer (ALB)
* Service Mesh using Consul or Istio
* Secrets management using Vault