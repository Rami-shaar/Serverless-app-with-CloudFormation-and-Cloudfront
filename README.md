# Serverless App with CloudFormation

This project deploys a complete **serverless architecture** using AWS CloudFormation.

---

## 🧱 Architecture

This serverless app includes:

- **API Gateway (HTTP)** – Entry point for API requests
- **AWS Lambda (Node.js)** – Stateless compute layer
- **DynamoDB** – NoSQL database for persistent storage
- **S3 (Static Website Hosting)** – Serves a simple frontend
- **IAM Roles** – Permission boundaries for Lambda and services
- **CloudWatch** – Alarms for Lambda error monitoring

---

## 📁 Folder Structure

```
cloudformation-serverless-app/
├── api/                 # API Gateway config
├── database/            # DynamoDB table
├── frontend/            # S3 static hosting
├── iam/                 # IAM roles/policies
├── lambda/              # Lambda function
├── monitoring/          # CloudWatch alarm
├── parameters/          # Parameter inputs used in deployments
├── index.html           # Frontend HTML file
└── .gitignore
```

---

## 🚀 Deployment Order

Each component is deployed as a separate stack:

1. `ServerlessDynamoDB`
2. `ServerlessIAM`
3. `ServerlessLambda`
4. `ServerlessAPI`
5. `ServerlessFrontend`
6. `ServerlessMonitoring`

---

## ⚙️ Deploy with AWS CLI

Run each command from the root folder:

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
    LambdaExecutionRoleArn=arn:aws:iam::123456789012:role/ServerlessIAM-LambdaExecutionRole \
    HttpApiId=33wpd7j210

# 4. API Gateway
aws cloudformation deploy \
  --template-file api/api-gateway.yaml \
  --stack-name ServerlessAPI \
  --parameter-overrides \
    ApiName=MyAppApi \
    LambdaFunctionArn=arn:aws:lambda:eu-west-2:123456789012:function:MyAppLambdaFunction

# 5. S3 Frontend
aws cloudformation deploy \
  --template-file frontend/s3-static-site.yaml \
  --stack-name ServerlessFrontend \
  --parameter-overrides FrontendBucketName=myapp-frontend-bucket-123456789012

# 6. CloudWatch Alarm
aws cloudformation deploy \
  --template-file monitoring/cloudwatch.yaml \
  --stack-name ServerlessMonitoring \
  --parameter-overrides LambdaFunctionName=MyAppLambdaFunction
```

---

## 🌐 Static Website

After the S3 bucket is created, upload the site:

```bash
aws s3 cp index.html s3://myapp-frontend-bucket-123456789012/
```

The URL is available in the `ServerlessFrontend` stack outputs:
```
http://myapp-frontend-bucket-123456789012.s3-website.<region>.amazonaws.com
```

---

## 🔍 Test the API Endpoint

You can test the deployed API by visiting the endpoint exposed by **API Gateway**.

### ✅ Working URL (with `/` at the end):

```bash
curl https://33wpd7j210.execute-api.eu-west-2.amazonaws.com/prod/
```

This works because your route is defined as `ANY /`.

- If you open the **Lambda → Configuration → Triggers** tab, the link already includes the `/` and works as-is.
- If you copy the invoke URL from **API Gateway**, you must manually add `/` to the end.

### ❌ This will not work:

```bash
curl https://33wpd7j210.execute-api.eu-west-2.amazonaws.com/prod
```

It fails because it doesn’t match your defined route (`/`).

### ✅ Expected Response:

```json
{
  "message": "It works!",
  "path": "/",
  "method": "GET"
}
```

---

## 🧠 Author

[@Rami-shaar](https://github.com/Rami-shaar)
