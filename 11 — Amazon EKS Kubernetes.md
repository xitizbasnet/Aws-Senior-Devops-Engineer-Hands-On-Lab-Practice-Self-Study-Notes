# Amazon EKS Kubernetes

> **Objective**
>
> Create an Amazon Elastic Kubernetes Service (EKS) cluster, configure `kubectl`, deploy a containerized application, and use Helm for Kubernetes package management.

---

# Learning Objectives

After completing this lab, you will be able to:

* Create an Amazon EKS Kubernetes cluster.
* Configure Kubernetes command-line access using `kubectl`.
* Create and manage EKS worker node groups.
* Deploy containerized applications to Kubernetes.
* Expose applications using Kubernetes Services.
* Install and manage applications using Helm charts.
* Apply Kubernetes and EKS production best practices.

---

# Prerequisites

Before starting this lab, ensure that you have:

* An active AWS account.
* Administrative AWS permissions.
* AWS CLI installed and configured.
* An existing Amazon ECR container image.

Example:

```text id="c7w1oa"
ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1
```

* An existing VPC:

```text id="j9yx2b"
devops-vpc
```

> **Important**
>
> Replace `ACCOUNT-ID` with your AWS Account ID.

---

# Task 1 — Create an Amazon EKS Cluster

## Step 1 — Create Cluster Using AWS Console

1. Search for:

```text id="q6h8az"
EKS
```

2. Navigate to:

```text id="x6l3oe"
Add cluster → Create
```

Configure the cluster.

| Setting              | Value                                  |
| -------------------- | -------------------------------------- |
| Cluster Name         | `devops-eks-cluster`                   |
| Kubernetes Version   | `1.29`                                 |
| Cluster Service Role | IAM Role with `AmazonEKSClusterPolicy` |

Create the required IAM role if it does not already exist.

---

# Step 2 — Configure Networking

Configure the cluster networking.

| Setting                 | Value                          |
| ----------------------- | ------------------------------ |
| VPC                     | `devops-vpc`                   |
| Subnets                 | All subnets (public + private) |
| Security Groups         | Default                        |
| Cluster Endpoint Access | Public and Private             |

Click:

```text id="d3v6q1"
Next → Create
```

Wait for the EKS cluster status to become:

```text id="0d7vny"
ACTIVE
```

---

# Step 3 — Add EKS Node Group

Navigate to:

```text id="m2s1ri"
EKS → devops-eks-cluster → Compute → Add node group
```

Configure the node group.

| Setting         | Value                                                                                 |
| --------------- | ------------------------------------------------------------------------------------- |
| Node Group Name | `devops-ng`                                                                           |
| IAM Role        | AmazonEKSWorkerNodePolicy + AmazonEC2ContainerRegistryReadOnly + AmazonEKS_CNI_Policy |
| AMI             | Amazon Linux 2 EKS Optimized                                                          |
| Instance Type   | `t3.medium`                                                                           |
| Minimum Nodes   | 1                                                                                     |
| Desired Nodes   | 2                                                                                     |
| Maximum Nodes   | 3                                                                                     |
| Subnets         | Private subnets                                                                       |

Click:

```text id="q5y8b0"
Create
```

---

# Task 2 — Configure kubectl & Deploy Application

## Step 1 — Install kubectl

Install Kubernetes command-line tools on Ubuntu EC2.

```bash id="p6b1dd"
curl -LO 'https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl'

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify installation.

```bash id="wq2k9m"
kubectl version --client
```

---

# Step 2 — Install eksctl

Install the Amazon EKS CLI tool.

```bash id="n6s4tx"
curl --silent --location 'https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz' | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

Verify installation.

```bash id="d0x9qq"
eksctl version
```

---

# Step 3 — Configure Kubernetes Access

Update the Kubernetes configuration.

```bash id="7j4vna"
aws eks update-kubeconfig \
--name devops-eks-cluster \
--region ap-south-1
```

Verify cluster access.

```bash id="q9w2ac"
kubectl get nodes
```

List namespaces.

```bash id="0h2dpm"
kubectl get namespaces
```

---

# Step 4 — Create Kubernetes Deployment

Create a file:

```text id="5k3m8a"
nginx-deployment.yaml
```

Add the following Kubernetes manifest.

```yaml id="9q7v3k"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: devops-nginx
  namespace: default

spec:
  replicas: 2

  selector:
    matchLabels:
      app: devops-nginx

  template:
    metadata:
      labels:
        app: devops-nginx

    spec:
      containers:
        - name: nginx
          image: ACCOUNT-ID.dkr.ecr.ap-south-1.amazonaws.com/devops-nginx:v1

          ports:
            - containerPort: 80

          resources:
            requests:
              cpu: 100m
              memory: 128Mi

            limits:
              cpu: 250m
              memory: 256Mi

---

apiVersion: v1
kind: Service

metadata:
  name: devops-nginx-svc

spec:
  type: LoadBalancer

  selector:
    app: devops-nginx

  ports:
    - port: 80
      targetPort: 80
```

---

## Step 5 — Deploy Application

Apply the Kubernetes manifest.

```bash id="1g4p4k"
kubectl apply -f nginx-deployment.yaml
```

Verify pods.

```bash id="p3z4fw"
kubectl get pods
```

Get the external Load Balancer address.

```bash id="f8x1mc"
kubectl get svc devops-nginx-svc
```

---

# Task 3 — Install Helm & Deploy Applications

## Step 1 — Install Helm

Install Helm 3.

```bash id="6z0s4x"
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify installation.

```bash id="9w0xnm"
helm version
```

---

# Step 2 — Add Helm Repository

Add the Bitnami Helm repository.

```bash id="8h6rkm"
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update repositories.

```bash id="t1j4kl"
helm repo update
```

---

# Step 3 — Search Available Charts

Search for Nginx charts.

```bash id="3f7kq2"
helm search repo bitnami/nginx
```

---

# Step 4 — Install Nginx Using Helm

Deploy Nginx.

```bash id="4z8xw0"
helm install my-nginx bitnami/nginx \
--namespace devops-ns \
--create-namespace
```

---

# Step 5 — Manage Helm Releases

List installed Helm releases.

```bash id="9r6mnp"
helm list -n devops-ns
```

Upgrade the release.

```bash id="0p8q4x"
helm upgrade my-nginx bitnami/nginx \
--set replicaCount=3 \
-n devops-ns
```

Remove the Helm deployment.

```bash id="7m4wq9"
helm uninstall my-nginx -n devops-ns
```

---

# Best Practice Tips

> **Tip**
>
> Follow these Amazon EKS and Kubernetes best practices for production environments.

* Use **private subnets** for EKS node groups.

Recommended architecture:

```text
Internet
   |
Load Balancer / Ingress
   |
Kubernetes Workloads
   |
Private EKS Nodes
```

* Use the **AWS Load Balancer Controller** Helm chart for native AWS ALB/NLB integration with Kubernetes Ingress.
* Use **Karpenter** instead of Cluster Autoscaler for faster and cost-efficient node scaling.
* Configure resource requests and limits on **all containers**.

Example:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

* Use Kubernetes namespaces to isolate environments:

```text
dev
staging
prod
```

* Use **IAM Roles for Service Accounts (IRSA)** to provide AWS permissions directly to Kubernetes pods without using node IAM roles.

---

# Alternative Tool — eksctl CLI

> **Note**
>
> `eksctl` provides a faster and repeatable method for creating EKS clusters through the command line.

Create an EKS cluster using:

```bash id="z5x1rb"
eksctl create cluster \
--name devops-cluster \
--region ap-south-1 \
--nodegroup-name devops-ng \
--node-type t3.medium \
--nodes 2
```

Benefits:

* Creates the complete EKS cluster.
* Creates node groups automatically.
* Configures:

  * VPC
  * IAM roles
  * Kubernetes configuration

`eksctl` is faster than the AWS Console for repeatable cluster provisioning.

---

# Lab Summary

In this lab, you completed the following tasks:

* ✅ Created an Amazon EKS cluster.
* ✅ Configured Kubernetes access using `kubectl`.
* ✅ Created an EKS managed node group.
* ✅ Installed `kubectl` and `eksctl`.
* ✅ Deployed an Nginx containerized application.
* ✅ Exposed the application using a Kubernetes LoadBalancer service.
* ✅ Installed Helm.
* ✅ Deployed and managed applications using Helm charts.
* ✅ Reviewed Kubernetes and EKS production best practices.
* ✅ Explored `eksctl` for automated EKS provisioning.
