# Capstone Project 1: Serverless API & Containerized Web App

A hybrid cloud-native application built on AWS combining a containerized web application, a serverless REST API, and full observability using CloudWatch.

---

## Architecture

| Component | Service |
|-----------|---------|
| Infrastructure as Code | AWS CloudFormation |
| Containerized Web App | Amazon ECS (Fargate) |
| Container Registry | Amazon ECR |
| Serverless Backend | AWS Lambda |
| API Layer | Amazon API Gateway |
| Observability | Amazon CloudWatch |

---

## Project Structure

```
capstone/
├── webapp/
│   ├── Dockerfile          # Container definition (Nginx + HTML)
│   └── index.html          # Web application UI
├── lambda-api/
│   └── index.js            # Lambda function (serverless backend)
├── infrastructure.yml      # CloudFormation template (ECS Cluster + Service)
└── README.md
```

---

## Live URLs

| Resource | URL |
|----------|-----|
| ECS Web App | http://100.25.180.229 |
| API Gateway Endpoint | https://2q3bn0lgef.execute-api.us-east-1.amazonaws.com/dev/submit |

---

## Screenshot 1: Live ECS Web Application

The web application running live on Amazon ECS Fargate at `http://100.25.180.229`

![ECS Web App Running]

---<img width="1914" height="1018" alt="Screenshot 2026-04-25 092137" src="https://github.com/user-attachments/assets/ec2c05b8-5f46-4acc-95cb-b76ca17d94d6" />


## Screenshot 2: ECS Task Networking

ECS Fargate task showing Public IP `100.25.180.229` assigned via the default VPC subnet.

![ECS Task Networking]

---<img width="1538" height="813" alt="Screenshot 2026-04-25 092157" src="https://github.com/user-attachments/assets/51780660-dabf-4d3c-86cc-bcaa16bc2601" />


## Screenshot 3: CloudFormation Stack Deployed

CloudFormation successfully created the ECS Cluster, Task Definition, and Fargate Service.

![CloudFormation Deploy]

---<img width="974" height="353" alt="Screenshot 2026-04-25 090246" src="https://github.com/user-attachments/assets/affa6f27-b11e-4f29-a0f4-7dee3aa8979d" />


## Screenshot 4: Amazon ECR Repository

Docker image pushed to Amazon ECR private repository.

![ECR Repository]

---<img width="1904" height="831" alt="Screenshot 2026-04-24 154914" src="https://github.com/user-attachments/assets/4f5c1fed-b1c3-4943-ae50-2fa6c89147c5" />


## Screenshot 5: Successful API Gateway Test

POST request to `/submit` via curl returning `statusCode: 200` and a generated submission ID.

![API Gateway Test]

---<img width="1919" height="695" alt="Screenshot 2026-04-25 094457" src="https://github.com/user-attachments/assets/291690cb-606d-4819-a383-0361f7c475e9" />


## Screenshot 6: CloudWatch Logs

Lambda function logging incoming payloads to CloudWatch Logs (`/aws/lambda/capstone-receiver`).

![CloudWatch Logs]

---<img width="1919" height="843" alt="Screenshot 2026-04-25 094819" src="https://github.com/user-attachments/assets/e31aa1af-cf46-4c22-8d1f-ccf9a39d347a" />


## Screenshot 7: CloudWatch Dashboard

Custom CloudWatch Dashboard showing Lambda **Invocations** and **Errors** metrics in real time.

![CloudWatch Dashboard]

---<img width="1897" height="741" alt="Screenshot 2026-04-25 182617" src="https://github.com/user-attachments/assets/2f273774-1d76-436f-8d6e-ae356bc275be" />


## Deployment Steps

### Step 1 — Build & Push Docker Image to ECR

```bash
# Authenticate Docker with ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  430287290736.dkr.ecr.us-east-1.amazonaws.com

# Build the image
cd webapp
docker build -t capstone-project-1-serverless-api .

# Tag and push to ECR
docker tag capstone-project-1-serverless-api:latest \
  430287290736.dkr.ecr.us-east-1.amazonaws.com/capstone-project-1-serverless-api:latest

docker push \
  430287290736.dkr.ecr.us-east-1.amazonaws.com/capstone-project-1-serverless-api:latest
```

### Step 2 — Deploy Infrastructure with CloudFormation

```bash
aws cloudformation deploy \
  --template-file infrastructure.yml \
  --stack-name capstone-stack \
  --region us-east-1
```

### Step 3 — Test the API

```bash
curl -X POST https://2q3bn0lgef.execute-api.us-east-1.amazonaws.com/dev/submit \
  -H "Content-Type: application/json" \
  -d '{"name": "Test User", "message": "Hello from API Gateway!"}'
```

Expected response:
```json
{
  "statusCode": 200,
  "body": "{\"message\":\"Data successfully received and logged!\",\"id\":\"zgk31f3z\"}"
}
```

---

## Services Used

- **Amazon ECS (Fargate)** — Runs the containerized Nginx web app without managing servers
- **Amazon ECR** — Stores the Docker image privately
- **AWS CloudFormation** — Provisions all ECS infrastructure as code
- **AWS Lambda** — Serverless function that receives and logs POST requests
- **Amazon API Gateway** — REST API that exposes the Lambda function via HTTP
- **Amazon CloudWatch** — Logs all Lambda invocations and displays metrics on a dashboard
