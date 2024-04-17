# Deploy Next.js App in AWS Elastic Kubernetes Cluster (EKS)

Before we begin refer to [Pre-requisites](https://github.com/famasboy888/AWS_EKS_cluster/blob/main/Pre-requisite.md)

## 1) Create cluster
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
...
...
...
2024-04-16 22:45:09 [ℹ]  nodegroup "linux-node" has 2 node(s)
2024-04-16 22:45:09 [ℹ]  node "ip-192-168-39-36.ap-northeast-1.compute.internal" is ready
2024-04-16 22:45:09 [ℹ]  node "ip-192-168-71-90.ap-northeast-1.compute.internal" is ready
2024-04-16 22:45:09 [✔]  EKS cluster "test-cluster" in "ap-northeast-1" region is ready
```
## 2) We can start deploying our Application

I created a Docker Image for my previous [Next.js Project](https://github.com/famasboy888/promptopia_nextjs)

I did not use HELM for this since it was a small test deployment.

**Deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-webapp-frontend
spec:
  selector:
    matchLabels:
      app: react-webapp
      tier: frontend
      track: stable
  replicas: 1
  revisionHistoryLimit: 0
  template:
    metadata:
      labels:
        app: react-webapp
        tier: frontend
        track: stable
    spec:
      containers:
      - name: react-webapp
        image: famasboy888/next-js-app:1.0.2.RELEASE
        imagePullPolicy: Always
        ports:
        - containerPort: 80
```

**Service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: react-webapp-frontend
spec:
  selector:
    app: react-webapp
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**Ingress.yaml**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web-app
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: react-webapp-frontend
              port:
                number: 80
```

For now, the ingress will not function yet.

We need to deploy `AWS Load Balancer Controller`.

## 3) We will now add AWS Load Balancer Controller to our EKS cluster
### 3.1 Create and configure OIDC provider.
We need this to enable an application to access/modify AWS resources.

```bash
eksctl utils associate-iam-oidc-provider --cluster <cluster_name> --approve
```
To verify:
```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
### 3.2 We create a IAM Policy and Role first

This file is IAM policy is widely used:
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy:
```bash
aws iam create-policy \
 --policy-name AWSLoadBalancerControllerIAMPolicy \
 --policy-document file://iam_policy.json
```

Create IAM Role:
```bash
eksctl create iamserviceaccount \
 --cluster=<your-cluster-name> \
 --namespace=kube-system \
 --name=aws-load-balancer-controller \
 --role-name AmazonEKSLoadBalancerControllerRole \
 --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
 --approve
```

### 3.3 We deploy AWS Load Balancer Controller using HEML
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
 -n kube-system \
 --set clusterName=<your-cluster-name> \
 --set serviceAccount.create=false \
 --set serviceAccount.name=aws-load-balancer-controller \
 --set region=<region> \
 --set vpcId=<your-vpc-id>
```

Check if all pods are running:
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
## 4) We will see Application Load Balancer magically appear (well, it is not really a magic...)

<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/AWS_EKS_cluster/assets/23441168/f9a9cb1f-8d42-48de-9f90-45a63de1fa54">
</p>

You will also notice that our ingress now has an `ADDRESS`
<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/AWS_EKS_cluster/assets/23441168/8d2b7ca6-bfd2-41eb-90d5-b2a77bc6fb9d">
</p>

## 5) Congrats! Our webapp can now be accessed using the generated DNS provided by Application Load Balancer

<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/AWS_EKS_cluster/assets/23441168/108159e8-78a1-462f-bbe8-0c253f81ffaa">
</p>
