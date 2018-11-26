# AWS Prerequisites

You will need an AWS account and to setup the following tools for this playground.

## AWS CLI

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

## Kubernetes Client

Linux RPM (CentOS, RHEL or Fedora):
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

Linux:
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

macOS:
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

Alternatively using the Homebrew package manager on macOS:

```bash
brew install kubernetes-cli
```

See:
* https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl

## AWS IAM Authenticator

If you are planning to locally use the standard Kubernetes client, kubectl, it must be at least version 1.10 to support exec authentication with usage of [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator). aws-iam-authenticator is a tool developed by Heptio Team and this tool will allow us to manage EKS by using kubectl.

Linux:
```bash
curl -o aws-iam-authenticator \
https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv aws-iam-authenticator /usr/local/bin
```

macOS:
```bash
curl -o aws-iam-authenticator \
https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/darwin/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo mv aws-iam-authenticator /usr/local/bin
```

See:
* https://docs.aws.amazon.com/eks/latest/userguide/configure-kubectl.html