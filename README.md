# Udagram Microservices on AWS Kubernetes Project

## Project Overview
This project deploys a microservices-based application using **Amazon EKS (Elastic Kubernetes Service)**.  
It includes two main services:
- **Analytics Service** (Python Flask app)
- **PostgreSQL Database**

The infrastructure is fully automated using **AWS CloudFormation**, with the container images stored in **Amazon ECR (Elastic Container Registry)** and deployed to **EKS** via Kubernetes manifests.

---

## 1. How to Create the Infrastructure

### Step 1: Create the EKS Cluster and Node Group
Run the following CloudFormation templates in AWS Console or AWS CLI:
```bash
aws cloudformation create-stack --stack-name eksctl-maha-eks-cluster --template-body file://eks-cluster.yaml --capabilities CAPABILITY_IAM
aws cloudformation create-stack --stack-name eksctl-maha-eks-nodegroup-maha-nodes --template-body file://eks-nodegroup.yaml --capabilities CAPABILITY_IAM

### Step 2: Configure AWS CLI and Connect to EKS

source aws_creds.sh
aws sts get-caller-identity
aws eks --region us-east-1 update-kubeconfig --name maha-eks
kubectl config current-context
kubectl get nodes

## 2. Build and Push Docker Image to ECR
### Step 1: Authenticate Docker to ECR

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 436713843511.dkr.ecr.us-east-1.amazonaws.com

### Step 2: Build, Tag, and Push the Image
From inside the /deployments/app folder:

docker build -t mahaanalytics:v1 .
docker tag mahaanalytics:v1 436713843511.dkr.ecr.us-east-1.amazonaws.com/mahaanalytics:v1
docker push 436713843511.dkr.ecr.us-east-1.amazonaws.com/mahaanalytics:v1

## 3. Deploy the Application to Kubernetes

### Step 1: Deploy the Database

cd deployments/db
kubectl apply -f .
kubectl get pods,svc,pv,pvc

### Step 2: Deploy the Analytics App

cd ../app
kubectl apply -f analytics-deployment.yaml
kubectl rollout restart deployment analytics
kubectl rollout status deployment analytics
kubectl get pods
kubectl get svc analytics-service
When the rollout completes successfully, note the EXTERNAL-IP of the LoadBalancer — this is your app’s public URL.

## 4. Verify Deployment
Open the following in the browser:

http://<EXTERNAL-IP>:5153/health_check

In this case:
http://a0e54a56683af43518dc0f860fbc8ee6-1232120243.us-east-1.elb.amazonaws.com:5153/health_check

If successful, you should see:
ok

## 5. Delete the Infrastructure (Cleanup)
To avoid charges, delete the stacks and resources after validation:

aws cloudformation delete-stack --stack-name eksctl-maha-eks-nodegroup-maha-nodes
aws cloudformation delete-stack --stack-name eksctl-maha-eks-cluster

## 6. Evidence 
Since resources were deleted after deployment, below are screenshots proving successful setup and deployment.

#	Screenshot	Description
1️⃣	1_CloudFormation_Outputs.png	Shows CloudFormation stacks for the EKS Cluster and Node Group with CREATE_COMPLETE status and Outputs tab open.
2️⃣	2_Kubectl_Get_Nodes.png	Shows successful EKS cluster connection with node in Ready state.
3️⃣	3_Postgres_Resources.png	Output of kubectl get pods,svc,pv,pvc showing PostgreSQL pod running and PVC bound.
4️⃣	4_Analytics_Rollout_Status.png	Output of rollout commands showing successful deployment of Analytics app.
5️⃣	5_Kubectl_Get_Pods.png	Both Analytics and Postgres pods running, with LoadBalancer EXTERNAL-IP.
6️⃣	6_Health_Check_OK.png	Browser screenshot showing /health_check endpoint returns “ok”.
7a️⃣	7a_Docker_Push_Console.png	Terminal output showing successful Docker image build and push to ECR.
7b️⃣	7b_ECR_Repository.png	AWS Console screenshot showing ECR repository mahaanalytics with image tag v1.

## 7. Notes
Region used: us-east-1 (N. Virginia)

ECR Repository: 436713843511.dkr.ecr.us-east-1.amazonaws.com/mahaanalytics

Kubernetes Cluster: maha-eks

Load Balancer verified successfully via browser access.

✅ Project Status: DEPLOYMENT SUCCESSFUL
All services were built, deployed, and verified successfully.
Final output confirmed via /health_check endpoint.


