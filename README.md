# Cloud Native Playground

This playground contains a couple of getting started guides for running cloud native workloads using Kubernetes.

## Launch Kubernetes

With [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (EKS), [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS), [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE) you can run workloads on Kubernetes without operating the Kubernetes management infrastructure yourself.

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

Get the external ip-address to see if NGINX is running and exposed using a AWS Classic Load Balancer:

```bash
kubectl get svc -n demo mywebserver-nginx
```

## Ingress 

Ingress is the built‑in Kubernetes load‑balancing framework for HTTP traffic. With Ingress, you control the routing of external traffic. When running on public clouds like AWS or GKE the load-balancing feature is available out of the box.

**Why Ingress?**
For each service with LoadBalancer type, AWS will create the new ELB (which comes with costs if you have a lot of services). With Kubernetes ingress you will need only one for one IP address. There are several Ingress controllers like NGINX/NGINX Plus, Traefik, Voyager (HAProxy) or Contour (Envoy) but also Amazon and Google offer Ingress implementations (AWS Aplication Load Balancer or Google Cloud Load Balancer)

Ingress is the most useful if you want to expose multiple services under the same IP address, and these services all use the same L7 protocol (typically HTTP). You can get a lot of features out of the box (like SSL, Auth, Routing, etc) depending on the ingress implementation.

Read more about Load Balancer and Ingress Controllers here:
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

We will use the Nginx ingress controller which is probably the most used at the moment.

```bash
helm install --name ingress --namespace ingress -f ingress/nginx-ingress-values.yml stable/nginx-ingress
```

If you check for your ingress pods you will see two services, controller, and the default backend:

```bash
kubectl get pod -n ingress --selector=app=nginx-ingress
```

An example Ingress that makes use of the controller:

```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls
```

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

Automatic DNS. Create a DNS A record (e.g. *.demo.kpn-cloudnative.com) for:

```bash
kubectl get svc ingress-nginx-ingress-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' -n ingress
```

It is also possible to synchronize Kubernetes Services and Ingresses with DNS Providers. See https://github.com/kubernetes-incubator/external-dns

## Prometheus and Grafana

For Prometheus installation use the official Helm chart [prometheus-operator](https://github.com/helm/charts/tree/master/stable/prometheus-operator). This chart has a lot of options, you can have  take a look at [default values](https://github.com/helm/charts/blob/master/stable/prometheus-operator/values.yaml) file and override some values if needed. This chart installs Grafana and exporters ready to monitor your cluster.

First create storage class for creating persistent volumes using AWS EBS:

```bash
kubectl create -f aws/prometheus-storageclass.yaml
```

Install Prometheus:

```bash
helm install -f monitoring/prometheus-values.yaml stable/prometheus-operator --name prometheus --namespace monitoring
```

The Prometheus Operator has been installed. Check its status by running:

```bash
kubectl --namespace monitoring get pods -l "release=prometheus"
```

Visit https://coreos.com/operators/prometheus/docs/latest/ for instructions on how
to create & configure Alertmanager and Prometheus instances using the Operator.

If you want to access Prometheus, Alertmanager or Grafana you can forward the port to localhost.

Alert manager:

```bash
kubectl port-forward -n monitoring alertmanager-prom-prometheus-operator-alertmanager-0 9093
```

Prometheus server:

```bash
kubectl port-forward -n monitoring prometheus-prometheus-prometheus-oper-prometheus-0 9090
```

Grafana:

```bash
kubectl port-forward -n monitoring prometheus-grafana-7f7458f55d-dtpx2 3000
```

> NOTE: Replace `prometheus-grafana-7f7458f55d-dtpx2` with the correct pod name.

The helm chart prometheus-operator will install a couple of very useful Grafana dashboards for monitoring your cluster, nodes, namespaces, pods, statefulsets, etc.

## Roadmap
* Logging using Elasticsearch, Fluentd and Kibana
* Service Mesh using Consul or Istio
* Secrets management using Vault