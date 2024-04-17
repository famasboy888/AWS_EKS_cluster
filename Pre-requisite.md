# Pre-requisites before we start

- [x] Install `AWS CLI` and configure
- [x] Install `eksctl`

## Installing AWS CLI

Refer to [AWS CLI Instructions](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)

```bash
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Configure and paste in Credentials
```bash
aws configure
```

## Installing EKSCTL

Refer to [EKSCTL Instructions](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html#setting-up-eksctl-linux)

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
