# Sports API Management System  

## Overview  
This project demonstrates a containerized API management system for querying real-time sports data. It leverages **Amazon ECS (Fargate)** for containerized workloads, **Amazon API Gateway** for exposing RESTful endpoints, and integrates with an external **Sports API** for live updates. Designed with advanced cloud practices, the system emphasizes scalability, security, and efficiency. This project began during Week 2, Day 1 of the **DevOps All-Star Challenge** and is currently in development.  

## Features  
- RESTful API for real-time sports data queries.  
- Containerized backend running on **Amazon ECS with Fargate**.  
- Scalable, serverless architecture.  
- Secure API routing through **Amazon API Gateway**.  

## Prerequisites  
To set up and run this project, you’ll need:  
- **Sports API Key**: Obtain an API key from [serpapi.com](https://serpapi.com).  
- **AWS Account**: Ensure you have an AWS account with basic knowledge of ECS, API Gateway, Docker, and Python.  
- **AWS CLI**: Install and configure it for programmatic interaction with AWS services.  
- **Serpapi Library**: Install locally using: `pip install google-search-results`.  
- **Docker**: Ensure Docker CLI and Desktop are installed for building and pushing container images.  

## Technical Architecture  
- **Cloud Provider**: AWS  
- **Core Services**: Amazon ECS (Fargate), API Gateway, CloudWatch  
- **Programming Language**: Python 3.x  
- **Containerization**: Docker  
- **Security**: Custom IAM policies for secure integration between ECS and API Gateway  

## Setup Instructions  

### Step 1: Create an Amazon ECR Repository  
Run the following command to create an ECR repository:  
```sh
aws ecr create-repository --repository-name sports-api --region us-east-1
```  

### Step 2: Build and Push Docker Image  
Authenticate Docker with Amazon ECR, build the image, and push it:  
```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

docker build --platform linux/amd64 -t sports-api .  
docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest  
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest  
```  

### Step 3: Set Up ECS Cluster and Task Definition  

#### Create an ECS Cluster:  
1. Navigate to the **ECS Console** → Clusters → Create Cluster.  
2. Select **Fargate** and name the cluster `sports-api-cluster`.  

#### Create a Task Definition:  
1. Go to **Task Definitions** → Create New Task Definition.  
2. Select **Fargate**, and name the task definition `sports-api-task`.  
3. Add a container:  
   - **Name**: sports-api-container  
   - **Image URI**: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest`  
   - **Port Mappings**: 8080 (TCP)  
4. Add environment variables:  
   - **Key**: `SPORTS_API_KEY`  
   - **Value**: `<YOUR_SPORTSDATA.IO_API_KEY>`  

### Step 4: Deploy Service with Application Load Balancer (ALB)  

#### Create a Service:  
1. Go to the **ECS Cluster** → Select Cluster → Create Service.  
2. Choose **Fargate** and assign the task definition `sports-api-task`.  
3. Name the service `sports-api-service` and set the desired tasks to `2`.  
4. Create a new security group:  
   - **Type**: Allow all TCP traffic.  
   - **Source**: Anywhere.  

#### Configure the Load Balancer:  
1. Use an **Application Load Balancer (ALB)**.  
2. Name it `sports-api-alb`.  
3. Set the target group health check path to `/sports`.  

### Step 5: Configure API Gateway  

#### Create a REST API:  
1. Navigate to the **API Gateway Console** → Create API → REST API.  
2. Name it `Sports API Gateway`.  

#### Add a Resource and Method:  
1. Create a resource `/sports`.  
2. Add a `GET` method and select **HTTP Proxy** as the integration type.  
3. Enter the ALB DNS name:  
   `http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports`.  

#### Deploy the API:  
1. Deploy the API to a stage, e.g., `prod`.  
2. Note the endpoint URL.  

## Testing  

Once deployed, test the system using the following commands or a browser:  
- ALB Test:  
  ```sh
  curl http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.amazonaws.com/sports
  ```  
- API Gateway Test:  
  ```sh
  curl https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports
  ```  

## Current Status  
This project is a work in progress. Stay tuned for updates as new features and enhancements are added!  
