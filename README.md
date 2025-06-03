# Serverless App with CloudFormation

This project deploys a **production-ready serverless application** using AWS CloudFormation.

---

## 🧱 Architecture Overview

This serverless app includes:

- **Amazon CloudFront (HTTPS CDN)** – Delivers the frontend securely over HTTPS  
- **Amazon S3 (Static Website Hosting)** – Hosts the frontend with a dynamic button  
- **API Gateway (HTTP API)** – Receives frontend requests and triggers Lambda  
- **AWS Lambda (Node.js)** – Serverless compute that writes to DynamoDB  
- **Amazon DynamoDB** – Stores unique ID + timestamp on each API call  
- **IAM Roles** – Secure access for Lambda to DynamoDB and CloudWatch  
- **Amazon CloudWatch** – Logs Lambda function executions for debugging and monitoring  

---

## 🖱️ Button-to-Database Flow (S3 → API Gateway → Lambda → DynamoDB)

1. The user visits the static frontend hosted on S3 (via CloudFront HTTPS).  
2. Clicking the **“Call API”** button triggers a `fetch()` call to the API Gateway endpoint.  
3. API Gateway invokes the **Lambda function**.  
4. Lambda writes a new item (`id` and `createdAt`) into the **DynamoDB table**.  
5. Lambda returns a success message, which is rendered in the frontend `<pre>` block.

✅ You can **verify the written record** inside the AWS Console:  
**DynamoDB → MyAppItemsTable → Explore items**

---

## 📁 Folder Structure

```
cloudformation-serverless-app/
├── api/                  # API Gateway with CORS + Lambda integration
├── database/             # DynamoDB table template
├── frontend/             # S3 + CloudFront static website hosting
├── iam/                  # IAM roles and permissions
├── lambda/               # Lambda function (inline Node.js)
├── monitoring/           # CloudWatch log configuration
├── parameters/           # Parameter inputs for deployment
├── index.html            # Frontend file (fetches data from API)
└── .gitignore
```

---

## 🚀 Deployment Steps (AWS CLI)

Deploy each component **in this exact order**:

```bash
# 1. DynamoDB
aws cloudformation deploy \
  --template-file database/dynamodb.yaml \
  --stack-name ServerlessDynamoDB \
  --parameter-overrides TableName=MyAppItemsTable

# 2. IAM
aws cloudformation deploy \
  --template-file iam/iam-roles.yaml \
  --stack-name ServerlessIAM \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides TableName=MyAppItemsTable

# 3. Lambda
aws cloudformation deploy \
  --template-file lambda/lambda-function.yaml \
  --stack-name ServerlessLambda \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    LambdaFunctionName=MyAppLambdaFunction \
    LambdaRuntime=nodejs18.x \
    LambdaHandler=index.handler \
    LambdaTimeout=10 \
    TableName=MyAppItemsTable \
    LambdaExecutionRoleArn=arn:aws:iam::<your-account-id>:role/ServerlessIAM-LambdaExecutionRole \
    HttpApiId=<your-api-gateway-id>

# 4. API Gateway
aws cloudformation deploy \
  --template-file api/api-gateway.yaml \
  --stack-name ServerlessApiGateway \
  --parameter-overrides \
    ApiName=MyAppApi \
    LambdaFunctionArn=arn:aws:lambda:<your-region>:<your-account-id>:function:MyAppLambdaFunction

# 5. CloudFront + S3
aws cloudformation deploy \
  --template-file frontend/s3-static-site.yaml \
  --stack-name ServerlessFrontend \
  --parameter-overrides FrontendBucketName=myapp-frontend-site

# 6. CloudWatch Logs
aws cloudformation deploy \
  --template-file monitoring/cloudwatch.yaml \
  --stack-name ServerlessMonitoring \
  --parameter-overrides LambdaFunctionName=MyAppLambdaFunction
```

---

## 🌐 Access the Frontend (CloudFront)

After deployment, open the HTTPS URL shown in the **ServerlessFrontend** stack outputs:

```
https://<CloudFrontDomainName>.cloudfront.net
```

You’ll see the message:

```
Hello from my serverless S3 site!
```

Clicking the “Call API” button on this page will:

- Call the API Gateway endpoint  
- Trigger the Lambda function to write an item to DynamoDB  
- Display the JSON success response below the button

---

## 🧪 Test the API Endpoint (Optional)

If you want to test the API directly (e.g., via curl, Postman, or browser), make sure to use the correct format:

✅ **Correct** (must include trailing slash `/`):

```bash
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod/
```

❌ **Incorrect** (missing slash — won't match ANY `/` route):

```bash
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod
```

---

## ✅ Example API Response

Whether you test using the frontend or directly, you should receive:

```json
{
  "message": "Item successfully written to DynamoDB!",
  "id": "id-1748988204925",
  "table": "MyAppItemsTable"
}
```

---

## 👤 Author

**Rami Alshaar**  
[GitHub Profile](https://github.com/your-github-username)
