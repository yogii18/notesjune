# EKS Setup and Configuration Guide using cloudformation

This document details the steps required to create an Amazon EKS cluster, configure networking using an AWS CloudFormation template, and deploy the AWS Load Balancer Controller.

## Table of Contents

1. [Pre-requisites](#1-pre-requisites)
2. [Configure AWS CLI](#2-configure-aws-cli)
3. [Attach Policies to user group](#3-attach-policies-to-user-group)
4. [Resources Created](#4-resources-created)
5. [Parameters](#5-parameters)
6. [CloudFormation Template and deployment](#6-cloudformation-template-and-deployment)
7. [Create AWS Load Balancer Controller and Associate IAM OIDC Provider](#7-create-aws-load-balancer-controller-and-associate-iam-oidc-provider)
8. [Sample Nginx Deployment](#8-sample-nginx-deployment)
9. [Sample Ingress](#9-sample-ingress)
10. [Clean Up Resources](#10-clean-up-resources)

## 1. Pre-requisites 
Before starting, ensure the following: 
- An AWS account with sufficient permissions. 
- The AWS CLI installed and configured. 
- The kubectl CLI tool installed. 
- eksctl CLI tool installed. 
- IAM roles set up for EKS, ALB, and Node Groups (ensure correct policies are attached). 

## 2. Configure AWS CLI 
First, configure your AWS CLI with your IAM user credentials: 

- aws configure 
 
You’ll be prompted to enter: 
- AWS Access Key ID 
- AWS Secret Access Key 
- Default region name (e.g., <region-name>) 
- Default output format (e.g., json) 

## 3. Attach Policies to user group
3.1 Create a user group
- aws iam create-group --group-name <GroupName>

3.2 To create eks cluster the below policies are required:

- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/IAMFullAccess
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/AWSCertificateManagerFullAccess
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/AWSCloudFormationFullAccess
- aws iam attach-group-policy --group-name SMTC-EKS-group --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

3.3 Add a User to the Group
Once the group is set up with policies, add a user to it using the aws iam add-user-to-group command:

aws iam add-user-to-group \
  --group-name <GroupName> \
  --user-name <UserName>
  
## 4. Resources Created
4.1. VPC and Networking
- VPC: Primary networking environment.
- Subnets: Public and private subnets for the cluster.
- Internet Gateway: To allow internet access.

4.2. EKS Cluster
- EKS cluster for managing Kubernetes resources.

4.3. Node Group
- A managed node group for the EKS cluster.

4.4. IAM Roles
- IAM roles and policies required for the EKS control plane and worker nodes.

## 5. Parameters
The default parametes are mentioned below:

| Parameter            | Description                          | Default         |
|:--------------------:|:------------------------------------:|:---------------:|
| **ClusterName**      | Name of the EKS Cluster             | MyEKSCluster    |
| **NodeInstanceType** | EC2 instance type for the node group | t3.medium       |
| **VpcCIDR**          | CIDR block for the VPC              | 10.0.0.0/16     |
| **PublicSubnetCIDR1**| CIDR block for the first public subnet | 10.0.1.0/24     |
| **PrivateSubnetCIDR1**| CIDR block for the first private subnet | 10.0.2.0/24     |

## 6. CloudFormation Template and deployment
6.1. Create a cloudformation template using cloudformation-template.yaml file

6.2. Validate the CloudFormation template before creating the stack

- aws cloudformation validate-template --template-body file://<file-name>

6.3. Deploy the stack using the following command

- aws cloudformation create-stack --stack-name <stack-name> --template-body file://<file-name> --capabilities CAPABILITY_IAM

6.4. Check the status of the stack

- aws cloudformation describe-stacks --stack-name <stack-name>

6.5. List the cluster:
- aws eks list-clusters

6.6. Update the kubeconfig.
- aws eks update-kubeconfig --region <region-name> --name <cluster-name>

6.7. Fetch the contexts and use the required cluster context.
- kubectl config get-contexts
- kubectl config use-context <name>


## setup nginx-ingress controller

helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace


## grafana loki setup:
helm repo add grafana https://grafana.github.io/helm-charts 
Kubectl create ns monitoring 
Kubectl apply -n monitoring pvc.yaml
helm install loki grafana/loki-stack --namespace monitoring --create-namespace --values loki-stack-values.yaml  
kubectl get secret --namespace monitoring loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo 
kubectl apply -f monitor-ingress.yaml


kubectl -n monitoring get ingress
NAME              CLASS   HOSTS                             ADDRESS                                                                   PORTS   AGE
monitor-ingress   nginx   monitor-test-aws.skill-mine.com   a2ae8e02c34f2435292641c84a08ecd6-397307865.ap-south-1.elb.amazonaws.com   80      14m


take the address and map to cloudflare: 


create the CNAME, map url to aws-ingress address.

pods are pending, describe them: 


kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

Kubectl -n monitoring apply -f  pvc.yaml
