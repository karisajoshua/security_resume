
# AWS Resume API Challenge Documentation

Welcome to my AWS Lambda Resume API Challenge journey. Here's a step-by-step narrative of how I built and deployed a serverless API using AWS Lambda and DynamoDB, integrated with GitHub Actions, secured with IAM roles, API keys, and JWT tokens, and containerized with Docker and Kubernetes.

## Challenge Objective 🎯

My task was to create a Lambda function that fetches resume data stored in DynamoDB and returns it in JSON format. To level up the challenge, I integrated GitHub Actions to automatically deploy updates to my Lambda function whenever I push to my repository and added security measures to protect the API. Additionally, I containerized the application and deployed it on a Kubernetes cluster.

## Getting Started 🚀

### 1. Set Up AWS

#### Sign Up for AWS

First, I signed up for an AWS account. I went to the [AWS Management Console](https://aws.amazon.com/console/) and created an AWS account since I didn't have one already.

#### Set Up AWS CLI

Next, I installed the AWS CLI on my local machine. I followed the [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) to get it up and running. Then, I configured the AWS CLI with my credentials by running:

```sh
aws configure
```

I entered my AWS Access Key ID, Secret Access Key, region, and output format.

### 2. Create a JSON Resume

I created a JSON file that follows this schema to use as sample data for the API:

```json
{
  "id": "1",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "phone": "123-456-7890",
  "education": [
    {
      "degree": "B.Sc. Computer Science",
      "institution": "XYZ University",
      "year": "2020"
    }
  ],
  "experience": [
    {
      "company": "ABC Corp",
      "role": "Software Engineer",
      "years": "2021-2023"
    }
  ]
}
```

### 3. Create AWS Resources Using Terraform

#### Install Terraform

First, I installed Terraform by following the [official installation guide](https://learn.hashicorp.com/tutorials/terraform/install-cli).

#### Write Terraform Configuration

I created a `main.tf` file with the following configuration to define the DynamoDB table, Lambda function, and security measures:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_dynamodb_table" "resumes" {
  name         = "Resumes"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

resource "aws_iam_role" "lambda_role" {
  name = "lambda_execution_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })

  inline_policy {
    name = "lambda_dynamodb_policy"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Action = [
            "dynamodb:GetItem",
            "dynamodb:Scan"
          ]
          Effect   = "Allow"
          Resource = "*"
        }
      ]
    })
  }
}

resource "aws_lambda_function" "resume_fetcher" {
  function_name = "resumeFetcher"
  role          = aws_iam_role.lambda_role.arn
  handler       = "lambda_function.lambda_handler"
  runtime       = "python3.8"
  filename      = "lambda_function_payload.zip"
  source_code_hash = filebase64sha256("lambda_function_payload.zip")
  environment {
    variables = {
      API_KEY = "your-api-key"
    }
  }
}

resource "aws_api_gateway_rest_api" "resume_api" {
  name        = "Resume API"
  description = "API for fetching resumes"
}

resource "aws_api_gateway_resource" "resumes_resource" {
  rest_api_id = aws_api_gateway_rest_api.resume_api.id
  parent_id   = aws_api_gateway_rest_api.resume_api.root_resource_id
  path_part   = "resumes"
}

resource "aws_api_gateway_method" "get_resumes_method" {
  rest_api_id   = aws_api_gateway_rest_api.resume_api.id
  resource_id   = aws_api_gateway_resource.resumes_resource.id
  http_method   = "GET"
  authorization = "AWS_IAM"
}

resource "aws_api_gateway_integration" "lambda_integration" {
  rest_api_id = aws_api_gateway_rest_api.resume_api.id
  resource_id = aws_api_gateway_resource.resumes_resource.id
  http_method = aws_api_gateway_method.get_resumes_method.http_method
  integration_http_method = "POST"
  type                     = "AWS_PROXY"
  uri                      = aws_lambda_function.resume_fetcher.invoke_arn
}

resource "aws_api_gateway_usage_plan" "usage_plan" {
  name = "Basic"
  api_stages {
    api_id = aws_api_gateway_rest_api.resume_api.id
    stage  = aws_api_gateway_deployment.resume_api_stage.stage_name
  }
}

resource "aws_api_gateway_api_key" "api_key" {
  name      = "my-api-key"
  enabled   = true
  stage_key {
    rest_api_id = aws_api_gateway_rest_api.resume_api.id
    stage_name  = aws_api_gateway_deployment.resume_api_stage.stage_name
  }
}

resource "aws_api_gateway_usage_plan_key" "usage_plan_key" {
  key_id        = aws_api_gateway_api_key.api_key.id
  key_type      = "API_KEY"
  usage_plan_id = aws_api_gateway_usage_plan.usage_plan.id
}

resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.resume_fetcher.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_api_gateway_rest_api.resume_api.execution_arn}/*/*"
}

resource "aws_api_gateway_deployment" "resume_api_stage" {
  rest_api_id = aws_api_gateway_rest_api.resume_api.id
  stage_name  = "prod"

  depends_on = [aws_api_gateway_method.get_resumes_method]
}
```

### 4. Containerize the Application

#### Create Dockerfile

I created a `Dockerfile` to containerize the Lambda function:

```Dockerfile
FROM public.ecr.aws/lambda/python:3.8

# Copy function code
COPY lambda_function.py ${LAMBDA_TASK_ROOT}

# Install dependencies
RUN pip install boto3

# Set the CMD to your handler
CMD ["lambda_function.lambda_handler"]
```

#### Build and Push Docker Image

1. **Build Docker Image:**
   ```sh
   docker build -t resume-fetcher .
   ```

2. **Push Docker Image to ECR:**
   - Create a repository in ECR.
   - Tag the Docker image:
     ```sh
     docker tag resume-fetcher:latest <your-ecr-repo-uri>:latest
     ```
   - Push the image:
     ```sh
     docker push <your-ecr-repo-uri>:latest
     ```

### 5. Deploy to Kubernetes

#### Set Up EKS

1. **Create an EKS Cluster:**
   Follow the [EKS getting started guide](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) to create an EKS cluster.

2. **Configure kubectl:**
   Configure kubectl to interact with your EKS cluster.

#### Create Kubernetes Deployment and Service

1. **Create Deployment:**
   Create a `deployment.yaml` file:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: resume-fetcher
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: resume-fetcher
     template:
       metadata:
         labels:
           app: resume-fetcher
       spec:
         containers:
         - name: resume-fetcher
           image: <your-ecr-repo-uri>:latest
           ports:
           - containerPort: 8080
           env:
           - name: API_KEY
             value: "your-api-key"
   ```

2. **Create Service:**
   Create a `service.yaml` file:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: resume-fetcher-service
   spec:
     type: LoadBalancer
     selector:
       app: resume-fetcher
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```

3. **Apply the Deployment and Service:**
   ```sh
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

### 6. Create Your Workflow

#### GitHub Actions

1. **Create a GitHub Repository:**
   I created a new repository on GitHub for my project.

2. **Set Up GitHub Actions:**
   I created a `.github/workflows/deploy.yml` file in my repository with the following content to automate the deployment process:

   ```yaml
   name: Deploy Docker Image to ECR and Update Kubernetes

   on:
     push:
       branches:
         - main

   jobs:
     build_and_push:
       runs
   -on: ubuntu-latest

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Set up QEMU
         uses: docker/setup-qemu-action@v1

       - name: Set up Docker Buildx
         uses: docker/setup-buildx-action@v1

       - name: Login to Amazon ECR
         id: login-ecr
         uses: aws-actions/amazon-ecr-login@v1

       - name: Build and push Docker image
         id: build-image
         run: |
           docker build -t resume-fetcher .
           docker tag resume-fetcher:latest <your-ecr-repo-uri>:latest
           docker push <your-ecr-repo-uri>:latest

     deploy:
       runs-on: ubuntu-latest
       needs: build_and_push

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Configure kubectl
         run: |
           aws eks update-kubeconfig --region us-east-1 --name <your-cluster-name>

       - name: Set Docker image in Kubernetes deployment
         run: |
           kubectl set image deployment/resume-fetcher resume-fetcher=<your-ecr-repo-uri>:latest
   ```

### 7. Test Everything

1. **Test Lambda Function:**
   I tested my Lambda function by triggering it using the API Gateway endpoint. I verified that it correctly returned the resume data based on the provided id and checked that the security measures were in place.

2. **Test GitHub Actions:**
   I pushed some code changes to my GitHub repository and verified that GitHub Actions automatically deployed the updates to my Lambda function. The workflow ran smoothly, packaging and deploying my Lambda function as expected.

3. **Test Docker and Kubernetes Deployment:**
   I verified that the Docker image was correctly built and pushed to ECR. I also confirmed that the Kubernetes deployment was updated with the new image and that the service was accessible.

---

That’s how I built and deployed a serverless API using AWS Lambda and DynamoDB, integrated with GitHub Actions, added security measures, and containerized the application with Docker and Kubernetes. The entire process was a great learning experience, and I’m happy with the outcome.
