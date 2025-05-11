Here is the content formatted as a `README.md` file with a **single-click copy button** (assuming you are viewing this on GitHub or in a markdown renderer that supports code block copy buttons):

````markdown
# Create Amazon EKS Cluster using `eksctl` with IAM Roles

This guide helps you set up an Amazon EKS (Elastic Kubernetes Service) cluster using `eksctl`, including setting up IAM roles for the cluster and the node group.

---

## ✅ Prerequisites

- Install `eksctl`
```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
````

* Install AWS CLI and configure:

```bash
aws configure
```

Ensure your AWS IAM user has permissions to create IAM roles, EC2, and EKS resources.

---

## ✅ Step 1: Create IAM Role for EKS Cluster

### 1. Create trust policy for EKS control plane (trust-policy.json)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 2. Create and attach the cluster role:

```bash
aws iam create-role --role-name EKSClusterRole \
  --assume-role-policy-document file://trust-policy.json

aws iam attach-role-policy \
  --role-name EKSClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

---

## ✅ Step 2: Create IAM Role for Node Group

### 1. Create trust policy for EC2 worker nodes (trust-node-policy.json)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 2. Create and attach the node role:

```bash
aws iam create-role --role-name EKSNodeRole \
  --assume-role-policy-document file://trust-node-policy.json

aws iam attach-role-policy --role-name EKSNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy --role-name EKSNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam attach-role-policy --role-name EKSNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
```

---

## ✅ Step 3: Create EKS Cluster using `eksctl`

### Create a file named `eks-cluster.yaml`

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-eks-cluster
  region: us-west-2

iam:
  withOIDC: true
  serviceRoleARN: arn:aws:iam::<account-id>:role/EKSClusterRole

nodeGroups:
  - name: ng-1
    instanceType: t3.medium
    desiredCapacity: 2
    iam:
      instanceRoleARN: arn:aws:iam::<account-id>:role/EKSNodeRole
```

> 🔁 Replace `<account-id>` with your actual AWS account ID.

### Run the following command to create the cluster:

```bash
eksctl create cluster -f eks-cluster.yaml
```

---

✅ Your EKS cluster will be up and running in a few minutes.

---

```

Would you like me to zip this README with the two trust policy files (`trust-policy.json` and `trust-node-policy.json`) for download?
```
