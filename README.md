# 🚀 Deploying 2048 Game App on AWS EKS with ALB Ingress Controller

This project demonstrates how to deploy a sample application (2048 Game) on an AWS-managed Kubernetes (EKS) cluster using serverless Fargate profiles and expose it to the internet through an AWS Application Load Balancer (ALB) Ingress Controller.

---

## 📌 Project Architecture

- **AWS EKS Cluster** — Kubernetes Control Plane
- **Fargate Profile** — Serverless compute (no manual EC2 node management)
- **Ingress + ALB Controller** — Public access to the deployed app
- **IAM Role and Policy** — Permissions for ALB Controller to interact with AWS resources

---

## ⚙️ Prerequisites

- AWS CLI installed
- `kubectl` installed
- `eksctl` installed
- `helm` installed
- AWS Account

---

## 🛠️ Steps Performed

1. **Create EKS Cluster with Fargate Profile**:
    ```bash
    eksctl create cluster --name cd-cluster --region us-east-1 --fargate
    ```
2. **Update kubeconfig**:
    ```bash
    aws eks update-kubeconfig --name cd-cluster --region us-east-1
    ```
3. **Create Fargate Profile for the App**:
    ```bash
    eksctl create fargateprofile --cluster cd-cluster --region us-east-1 --name alb-sample-app --namespace game-2048
    ```
4. **Deploy the 2048 Application**:
    ```bash
    kubectl apply -f deployment/2048_full.yaml
    ```
5. **Create and Configure OIDC Provider**:
    ```bash
    eksctl utils associate-iam-oidc-provider --cluster cd-cluster --approve
    ```
6. **Create IAM Policy for ALB Controller**:
    ```bash
    aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://deployment/iam_policy.json
    ```
7. **Create IAM Service Account**:
    ```bash
    eksctl create iamserviceaccount \
      --cluster=cd-cluster \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name AmazonEKSLoadBalancerControllerRole \
      --attach-policy-arn=arn:aws:iam::<your-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
      --approve
    ```
8. **Install ALB Controller Using Helm**:
    ```bash
    helm repo add eks https://aws.github.io/eks-charts
    helm repo update
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=cd-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=us-east-1 \
      --set vpcId=<your-vpc-id>
    ```
9. **Check the Ingress and Access App**:
    ```bash
    kubectl get ingress -n game-2048
    ```

---

| Resource | Screenshot |
|:---------|:------------|
| EKS Cluster Created | ![EKS Cluster](https://github.com/user-attachments/assets/6fc905e9-bcbc-4c00-84eb-8b67bdf04c43) |
| Fargate Profile Created | ![Fargate Profile](https://github.com/user-attachments/assets/8f1d5b4a-92c2-488e-93c7-3449ffb98d42) |
| Load Balancer Created | ![Load Balancer](https://github.com/user-attachments/assets/05f6235b-687d-4cee-8969-691e91e8728b) |
| 2048 Game Running | ![App Screenshot](https://github.com/user-attachments/assets/a445cb66-07b0-488b-8794-e4d670719bd8) |


## 🧹 How to Destroy Everything

1. Delete Fargate profile:
    ```bash
    eksctl delete fargateprofile --cluster cd-cluster --name alb-sample-app
    ```
2. Uninstall ALB Controller:
    ```bash
    helm uninstall aws-load-balancer-controller -n kube-system
    ```
3. Delete IAM Service Account:
    ```bash
    eksctl delete iamserviceaccount --cluster cd-cluster --namespace kube-system --name aws-load-balancer-controller
    ```
4. Delete IAM Policy:
    ```bash
    aws iam delete-policy --policy-arn arn:aws:iam::<your-account-id>:policy/AWSLoadBalancerControllerIAMPolicy
    ```
5. Delete EKS Cluster:
    ```bash
    eksctl delete cluster --name cd-cluster --region us-east-1
    ```

---

## 📚 Learnings from this Project

- Deploying apps on serverless Kubernetes (Fargate)
- Using Ingress controllers to expose apps publicly
- IAM roles for service accounts (IRSA) for secure permissions
- YAML manifest writing for Kubernetes
- Basic AWS policy writing (JSON)

---

## 🧠 Credits

- Based on tutorials from [iam-veeramalla](https://github.com/iam-veeramalla/aws-devops-zero-to-hero).
