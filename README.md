# Deploy App in EKS Cluster

Before we begin refer to [Pre-requisites](https://github.com/famasboy888/AWS_EKS_cluster/blob/main/Pre-requisite.md)

## Create cluster
```bash
eksctl create cluster \
 --name test-cluster \
 --region ap-northeast-1 \
 --nodegroup-name linux-node \
 --node-type t2.micro \
 --nodes 2
```

This will take a looooong time, so be patient. Also, watch out for errors

```bash
Output:
2024-04-16 22:31:03 [ℹ]  eksctl version 0.175.0
2024-04-16 22:31:03 [ℹ]  using region ap-northeast-1
2024-04-16 22:31:04 [ℹ]  setting availability zones to [ap-northeast-1d ap-northeast-1c ap-northeast-1a]
2024-04-16 22:31:04 [ℹ]  subnets for ap-northeast-1d - public:192.168.0.0/19 private:192.168.96.0/19
2024-04-16 22:31:04 [ℹ]  subnets for ap-northeast-1c - public:192.168.32.0/19 private:192.168.128.0/19
2024-04-16 22:31:04 [ℹ]  subnets for ap-northeast-1a - public:192.168.64.0/19 private:192.168.160.0/19
2024-04-16 22:31:04 [ℹ]  nodegroup "linux-node" will use "" [AmazonLinux2/1.29]
2024-04-16 22:31:04 [ℹ]  using Kubernetes version 1.29
....................
```
