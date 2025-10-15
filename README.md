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


If we set the environment variables by prepending them, it would look like the following:
```bash
DB_USERNAME=username_here DB_PASSWORD=password_here python app.py
```
The benefit here is that it's explicitly set. However, note that the `DB_PASSWORD` value is now recorded in the session's history in plaintext. There are several ways to work around this including setting environment variables in a file and sourcing them in a terminal session.

3. Verifying The Application
* Generate report for check-ins grouped by dates
`curl <BASE_URL>/api/reports/daily_usage`

* Generate report for check-ins grouped by users
`curl <BASE_URL>/api/reports/user_visits`

## Project Instructions
1. Set up a Postgres database with a Helm Chart
2. Create a `Dockerfile` for the Python application. Use a base image that is Python-based.
3. Write a simple build pipeline with AWS CodeBuild to build and push a Docker image into AWS ECR
4. Create a service and deployment using Kubernetes configuration files to deploy the application
5. Check AWS CloudWatch for application logs

### Deliverables
1. `Dockerfile`
2. Screenshot of AWS CodeBuild pipeline
3. Screenshot of AWS ECR repository for the application's repository
4. Screenshot of `kubectl get svc`
5. Screenshot of `kubectl get pods`
6. Screenshot of `kubectl describe svc <DATABASE_SERVICE_NAME>`
7. Screenshot of `kubectl describe deployment <SERVICE_NAME>`
8. All Kubernetes config files used for deployment (ie YAML files)
9. Screenshot of AWS CloudWatch logs for the application
10. `README.md` file in your solution that serves as documentation for your user to detail how your deployment process works and how the user can deploy changes. The details should not simply rehash what you have done on a step by step basis. Instead, it should help an experienced software developer understand the technologies and tools in the build and deploy process as well as provide them insight into how they would release new builds.


### Stand Out Suggestions
Please provide up to 3 sentences for each suggestion. Additional content in your submission from the standout suggestions do _not_ impact the length of your total submission.
1. Specify reasonable Memory and CPU allocation in the Kubernetes deployment configuration
2. In your README, specify what AWS instance type would be best used for the application? Why?
3. In your README, provide your thoughts on how we can save on costs?

### Best Practices
* Dockerfile uses an appropriate base image for the application being deployed. Complex commands in the Dockerfile include a comment describing what it is doing.
* The Docker images use semantic versioning with three numbers separated by dots, e.g. `1.2.1` and  versioning is visible in the  screenshot. See [Semantic Versioning](https://semver.org/) for more details.# webhook test
# Webhook test trigger
