# EKS Setup

## EKS Clusters Creation and Setup

eksctl create cluster --name EKS-1 --region us-east-1


aws eks update-kubeconfig --name EKS-1 --region us-east-1

## EKS Clusters Deletion

eksctl delete cluster --name EKS-1 --region us-east-1
