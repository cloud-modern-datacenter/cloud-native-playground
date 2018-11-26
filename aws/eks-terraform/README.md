# aws-eks-terraform

In this getting started guide we are going to create an AWS EKS cluster with Terraform.

### Terraform

Linux:
```bash
curl -o terraform_0.11.10_linux_amd64.zip \
https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip

unzip terraform_0.11.10_linux_amd64.zip -d /usr/local/bin/
```

macOS:
```bash
curl -o terraform_0.11.10_darwin_amd64.zip \
https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_darwin_amd64.zip

unzip terraform_0.11.10_linux_amd64.zip -d /usr/local/bin/
```

Alternatively using the Homebrew package manager on macOS:

```bash
brew install terraform
```

Also see
* https://www.terraform.io/intro/getting-started/install.html

## Create IAM account for deployment

We need IAM credentials with are suitable to create AutoScaling, EC2, EKS, and IAM resources.

First configure AWS CLI and provide your access key and secret.
```bash
aws configure
```

Create a new terraform user and attach to an IAM policy.
```bash
aws iam create-user --user-name terraform
aws iam attach-user-policy --user-name terraform --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

> NOTE: The terraform user will get administrator access. For production you may restrict access for the terraform user. 

Create access key.
```bash
aws iam create-access-key --user-name terraform
```

> NOTE: This access key will be used by terraform to create the infrastructure.

The AWS provider offers a flexible means of providing credentials for authentication. See https://www.terraform.io/docs/providers/aws/index.html#authentication

For now we will put them in `terraform.tfvars`

```bash
cat terraform.tfvars                                                                                                    access_key  = "XXX"
secret_key  = "XXX"
```

## Create Kubernetes cluster on AWS EKS

We are using the getting started guide from Terraform:
https://www.terraform.io/docs/providers/aws/guides/eks-getting-started.html

Terraform will create
* EKS Cluster: AWS managed Kubernetes cluster of master servers
* AutoScaling Group containing 2 m4.large instances based on the latest EKS Amazon Linux 2 AMI: Operator managed Kubernetes worker nodes for running Kubernetes service deployments
* Associated VPC, Internet Gateway, Security Groups, and Subnets: Operator managed networking resources for the EKS Cluster and worker node instances
* Associated IAM Roles and Policies: Operator managed access resources for EKS and worker node instances

Initialize
```bash
terraform init
```

View plan
```bash
terraform plan
```

Apply plan
```bash
terraform apply
```

> NOTE: Creating the infrastructure can take more than 10 minutes.

Lets see if EKS Cluster and EC2 instances are created
```bash
aws eks describe-cluster --name cloud-native-eks-playground
aws ec2 describe-instances
```


## Configure kubernetes-client

You have to create a kubeconfig file for your cluster. 
See https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

Using AWS CLI

```bash
aws eks update-kubeconfig 
```

Or using Terraform

```bash
terraform output kubeconfig > ~/.kube/config
```

## Kubernetes Configuration to Join Worker Nodes

The EKS service does not provide a cluster-level API parameter or resource to automatically configure the underlying Kubernetes cluster to allow worker nodes to join the cluster via AWS IAM role authentication.

See https://www.terraform.io/docs/providers/aws/guides/eks-getting-started.html#configuring-kubectl-for-eks

Get the Config Map AWS Auth and save the configuration into a file, e.g. `config_map_aws_auth.yml`

```bash
terraform output config_map_aws_auth > config_map_aws_auth.yml
``` 

Apply config to Kubernetes

```bash
kubectl apply -f config_map_aws_auth.yml 
```

You can verify the worker nodes are joining the cluster via: 

```bash
kubectl get nodes --watch
```

## Destroy infrastructure

Tear down all associated resources which are created above

```bash
terraform destroy
```