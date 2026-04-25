# 🚀 Cloud Native Capstone Project

A hybrid cloud-native application built on AWS using:
- **Amazon ECS (Fargate)** — Containerized web application
- **AWS Lambda + API Gateway** — Serverless REST API
- **AWS CloudFormation** — Infrastructure as Code
- **Amazon CloudWatch** — Observability & monitoring

---

## Architecture

```
Browser → API Gateway → Lambda → CloudWatch Logs
Browser → ECS Fargate Container (Nginx) ← deployed via CloudFormation
```

---

## Project Structure

```
capstone/
├── infrastructure.yml      # CloudFormation template (ECS + VPC + Networking)
├── lambda-api/
│   └── index.js            # Lambda function (serverless backend)
├── webapp/
│   ├── Dockerfile          # Container definition (Nginx + HTML)
│   └── index.html          # Web application UI
└── README.md
```

---

## Deployment Steps

### Step 1 — Build & Push Docker Image to ECR

```bash
# 1. Create an ECR repository
aws ecr create-repository --repository-name capstone-webapp --region us-east-1

# 2. Authenticate Docker with ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <YOUR_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

# 3. Build the Docker image
cd webapp
docker build -t capstone-webapp .

# 4. Tag the image for ECR
docker tag capstone-webapp:latest \
  <YOUR_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/capstone-webapp:latest

# 5. Push to ECR
docker push \
  <YOUR_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/capstone-webapp:latest
```

### Step 2 — Deploy Infrastructure with CloudFormation

```bash
aws cloudformation deploy \
  --template-file infrastructure.yml \
  --stack-name capstone-stack \
  --parameter-overrides \
    ECRImageURI=<YOUR_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/capstone-webapp:latest \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Step 3 — Deploy the Lambda Function

```bash
# Zip the Lambda code
cd lambda-api
zip function.zip index.js

# Create the Lambda function
aws lambda create-function \
  --function-name capstone-receiver \
  --runtime nodejs18.x \
  --role arn:aws:iam::<YOUR_ACCOUNT_ID>:role/lambda-basic-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --region us-east-1
```

### Step 4 — Create API Gateway

1. Go to **AWS Console → API Gateway**
2. Create a new **REST API**
3. Create a resource `/submit` with a **POST** method
4. Integrate it with your `capstone-receiver` Lambda function
5. Enable **CORS**
6. **Deploy** the API to a stage called `prod`
7. Copy the **Invoke URL** — update `API_URL` in `webapp/index.html`

### Step 5 — Set up CloudWatch Dashboard

1. Go to **AWS Console → CloudWatch → Dashboards**
2. Create a new dashboard called `capstone-dashboard`
3. Add a widget → **Line graph** → Select metric:
   - `Lambda → By Function Name → capstone-receiver → Invocations`
4. Add another widget for `Errors`
5. Save the dashboard

---

## Screenshots

> *(Add your screenshots here after deployment)*

**1. Live ECS Web Application**
<!-- Screenshot of your app running in the browser -->

**2. Successful API Gateway Test**
<!-- Screenshot of Postman/curl/AWS Console showing a 200 response -->

**3. CloudWatch Dashboard**
<!-- Screenshot of your dashboard showing Invocations and Errors metrics -->

---

## Live URLs

| Resource | URL |
|----------|-----|
| ECS Web App | `http://<ECS-PUBLIC-IP>` |
| API Gateway | `https://<API-ID>.execute-api.us-east-1.amazonaws.com/prod/submit` |
| CloudWatch Dashboard | AWS Console → CloudWatch → Dashboards → capstone-dashboard |
