# Sports API Management System

## **Project Overview**
This project demonstrates building a containerized API management system for querying sports data. It leverages **Amazon ECS (Fargate)** for running containers, **Amazon API Gateway** for exposing REST endpoints, and an external **Sports API** for real-time sports data. The project showcases advanced cloud computing practices, including API management, container orchestration, and secure AWS integrations.

---

## **Features**
- Exposes a REST API for querying real-time sports data
- Runs a containerized backend using Amazon ECS with Fargate
- Scalable and serverless architecture
- API management and routing using Amazon API Gateway
 
---

## **Prerequisites**
- **Sports API Key**: Sign up for a free account and subscription & obtain your API Key at `serpapi.com`
- **AWS Account**: Create an AWS Account & have basic understanding of `ECS`, `API Gateway`, `Docker` & `Python`
- **AWS CLI Installed and Configured**: Install & configure AWS CLI to programatically interact with AWS
- **Serpapi Library**: Install Serpapi library in local environment "pip install google-search-results"
- **Docker CLI and Desktop Installed**: To build & push container images

---

## **Technical Architecture**
![Brown Minimalist Lifestyle Daily Vlog YouTube Thumbnail (2)](https://github.com/user-attachments/assets/32e49fe6-df16-40cb-b262-af1478cf01d5)

---

## **Technologies**
- **Cloud Provider**: AWS
- **Core Services**: Amazon ECS (Fargate), API Gateway, CloudWatch
- **Programming Language**: Python 3.x
- **Containerization**: Docker
- **IAM Security**: Custom least privilege policies for ECS task execution and API Gateway

---

## **Project Structure**

```bash
sports-api-management/
├── app.py # Flask application for querying sports data
├── Dockerfile # Dockerfile to containerize the Flask app
├── requirements.txt # Python dependencies
├── .gitignore
└── README.md # Project documentation
```

---

## **Setup Instructions**

### **Clone the Repository**
```bash
git clone https://github.com/ifeanyiro9/containerized-sports-api.git
cd containerized-sports-api
```
### **Create ECR Repo**
```bash
aws ecr create-repository --repository-name sports-api --region us-east-1
```

### **Authenticate Build and Push the Docker Image**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

docker build --platform linux/amd64 -t sports-api .
docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
```

<img width="743" height="180" alt="build-docker image" src="https://github.com/user-attachments/assets/f7f3c58b-e3cd-4bc8-abde-aa97e79bc0b5" />


<img width="728" height="164" alt="push-image" src="https://github.com/user-attachments/assets/27888237-2e8c-4420-bfa3-df740ce4e324" />


### **Set Up ECS Cluster with Fargate**
### Atach an IAM policy to your user or role that grants the necessary permissions to work with ECS clusters.
-Go to the `IAM Console`: Log in to your AWS Management Console and navigate to the `IAM service.`
-Find your `user`: In the left navigation pane, click on `Users`, then select your `user`
-Attach a policy: In the Permissions tab for your user, click `Add permissions` > Attach policies directly.
-Attach `AmazonECS_FullAccess`: In the search bar, type `AmazonECS_FullAccess`. This is a managed policy that gives you comprehensive permissions to manage all aspects of Amazon ECS. Select it and click `Add permissions`.

<img width="891" height="165" alt="schedule from api gateway" src="https://github.com/user-attachments/assets/9c417fa1-6c43-467b-ae31-25d7f25489e1" />


1. Create an ECS Cluster: `# infrastructure that is going to hold our container`
- Go to the ECS Console → Clusters → Create Cluster
- Name your Cluster (sports-api-cluster)
- For Infrastructure, select Fargate, then create Cluster

2. Create a Task Definition: `#a single unit or container that perform a specific job`
- Go to Task Definitions → Create New Task Definition
- Name your task definition (sports-api-task)
- For Infrastructure, select Fargate

  <img width="647" height="338" alt="creating cluster" src="https://github.com/user-attachments/assets/b08c3a9a-757a-496b-ae4d-7bc568d29c4c" />

- Add the container:
  - Name your container (sports-api-container)
  - Image URI: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
  - Container Port: 8080
  - Protocol: TCP
  - Port Name: Leave Blank
  - App Protocol: HTTP
- Define Environment Eariables:
  - Key: SPORTS_API_KEY
  - Value: <YOUR_SPORTSDATA.IO_API_KEY>
  - Create task definition
3. Run the Service with an ALB
- Go to Clusters → Select Cluster → Service → Create.
- Capacity provider: Fargate
- Select Deployment configuration family (sports-api-task)
- Name your service (sports-api-service)
- Desired tasks: 2
- Networking: Create new security group
- Networking Configuration:
  - Type: All TCP
  - Source: Anywhere
 
- Load Balancing: Select Application Load Balancer (ALB).
- ALB Configuration:
 - Create a new ALB:
 - Name: sports-api-alb
 - Target Group health check path: "/sports"
 - Create service
   
<img width="914" height="425" alt="laod-balacer" src="https://github.com/user-attachments/assets/1fe978d1-fd04-466b-9bcf-55c53c3e0c60" />

   
4. Test the ALB:
- After deploying the ECS service, note the DNS name of the ALB (e.g., sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com)
- Confirm the API is accessible by visiting the ALB DNS name in your browser and adding /sports at end (e.g, http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports)
  
<img width="495" height="476" alt="sport-schedule" src="https://github.com/user-attachments/assets/cdf1aa05-b8a9-4ac6-ba88-feb47bb08411" />

  

### **Configure API Gateway**
1. Create a New REST API:
- Go to API Gateway Console → Create API → REST API
- Name the API (e.g., Sports API Gateway)
  
<img width="628" height="287" alt="Create API gate way" src="https://github.com/user-attachments/assets/2ef2ef86-60ff-464a-a528-6d18e79844d4" />

  

2. Set Up Integration:
- Create a resource /sports
- Create a GET method
- Choose HTTP Proxy as the integration type
- Enter the DNS name of the ALB that includes "/sports" (e.g. http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports
  
<img width="715" height="353" alt="resources" src="https://github.com/user-attachments/assets/b2907a16-c64a-4725-b75d-955bec7a4d08" />



3. Deploy the API:
- Deploy the API to a stage (e.g., prod)
- Note the endpoint URL

### **Test the System**
- Use curl or a browser to test:
```bash
curl https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports
```

<img width="891" height="165" alt="schedule from api gateway" src="https://github.com/user-attachments/assets/afee6963-cda5-47d6-ac5f-216a8c6bf963" />

### **What We Learned**
Setting up a scalable, containerized application with ECS
Creating public APIs using API Gateway.

### **Future Enhancements**
Add caching for frequent API requests using Amazon ElastiCache
Add DynamoDB to store user-specific queries and preferences
Secure the API Gateway using an API key or IAM-based authentication
Implement CI/CD for automating container deployments


