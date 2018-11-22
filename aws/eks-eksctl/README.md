# aws-eks-eksctl

[eksctl](https://eksctl.io/) is a simple CLI tool for creating clusters on EKS - Amazon’s new managed Kubernetes service for EC2. It is written in Go, and uses official AWS CloudFormation templates.

## Prerequisites

In order to create this playground you will need an AWS account and your AWS credentials configured.

### AWS CLI

The primary distribution method for the AWS CLI on Linux, Windows, and macOS is pip, a package manager for Python that provides an easy way to install, upgrade, and remove Python packages and their dependencies.

If you already have pip and a supported version of Python, you can install the AWS CLI with the following command:
```bash
pip install awscli --upgrade --user
```

Alternatively using the Homebrew package manager on mac OS:

```bash
brew install awscli
```

See: 
* https://docs.aws.amazon.com/cli/latest/userguide/installing.html 
* https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html

## Usage
To download the latest release, run:

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

Alternatively, macOS users can use Homebrew:

```bash
brew install weaveworks/tap/eksctl
```

You will need to have AWS API credentials configured. What works for AWS CLI or any other tools (kops, Terraform etc), should be sufficient. You can use `~/.aws/credentials` file or environment variables. For more information read AWS documentation.

To create a basic cluster, run:

```bash
eksctl create cluster
```

A cluster will be created with default parameters
* exciting auto-generated name, e.g. “fabulous-mushroom-1527688624”
* 2x m5.large nodes (this instance type suits most common use-cases, and is good value for money)
* use official AWS EKS AMI
* us-west-2 region
* dedicated VPC (check your quotas)
* using static AMI resolver

Once you have created a cluster, you will find that cluster credentials were added in `~/.kube/config`. If you have kubectl v1.10.x as well as aws-iam-authenticator commands in your PATH, you should be able to use kubectl. You will need to make sure to use the same AWS API credentials for this also. Check EKS docs for instructions. If you installed eksctl via Homebrew, you should have all of these dependencies installed already.

For more info see https://eksctl.io/