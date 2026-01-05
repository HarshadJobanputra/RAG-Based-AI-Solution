# Deployment Guide

Complete guide to deploying your own instance of this RAG system.

---

## Prerequisites

- AWS Account with Bedrock access
- AWS CLI configured
- Python 3.11+
- Basic understanding of AWS services

---

## Step 1: Set Up AWS Credentials
```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
# Default output format: json
```

---

## Step 2: Request Bedrock Model Access

1. Go to AWS Console → Bedrock → Model access
2. Request access to: `Meta Llama 3 70B Instruct`
3. Wait for approval (usually instant)

---

## Step 3: Create S3 Bucket (Optional - for FAISS future)
```bash
aws s3 mb s3://your-bucket-name --region us-east-1
```

---

## Step 4: Create IAM Role
```bash
# Create trust policy
cat > lambda-trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
  --role-name RAGLambdaRole \
  --assume-role-policy-document file://lambda-trust-policy.json

# Attach policies
aws iam attach-role-policy \
  --role-name RAGLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

aws iam attach-role-policy \
  --role-name RAGLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess
```

---

## Step 5: Deploy Lambda Function

### Option A: AWS Console

1. **Go to Lambda Console** → Create function
2. **Configuration:**
   - Name: `YourRAGFunction`
   - Runtime: Python 3.11
   - Role: Use existing role → `RAGLambdaRole`
3. **Copy code** from `lambda_deployment/lambda_function.py`
4. **Configuration:**
   - Memory: 512 MB
   - Timeout: 30 seconds
5. **Deploy**

### Option B: AWS CLI
```bash
# Create deployment package
cd lambda_deployment
zip -r lambda-code.zip lambda_function.py

# Create function
aws lambda create-function \
  --function-name YourRAGFunction \
  --runtime python3.11 \
  --role arn:aws:iam::YOUR_ACCOUNT_ID:role/RAGLambdaRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://lambda-code.zip \
  --timeout 30 \
  --memory-size 512 \
  --region us-east-1
```

---

## Step 6: Create API Gateway

1. **Go to API Gateway** → Create API → REST API
2. **Create Resource:**
   - Name: `query`
   - Path: `/query`
   - Enable CORS: ✓
3. **Create Method:**
   - Select: POST
   - Integration: Lambda Function
   - Lambda: `YourRAGFunction`
   - Use Lambda Proxy: ✓
4. **Enable CORS:**
   - Actions → Enable CORS → Confirm
5. **Deploy API:**
   - Actions → Deploy API
   - Stage: `prod`
6. **Copy Invoke URL**

---

## Step 7: Test Deployment
```bash
curl -X POST https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/prod/query \
  -H "Content-Type: application/json" \
  -d '{"question": "Was sagt die Reisekostenrichtlinie?"}'
```

Expected response:
```json
{
  "answer": "Die Reisekostenrichtlinie regelt...",
  "sources": [...],
  "department": "Finance"
}
```

---

## Step 8: Monitor & Optimize

### CloudWatch Logs
```bash
aws logs tail /aws/lambda/YourRAGFunction --follow
```

### Cost Monitoring
- Go to AWS Cost Explorer
- Filter by service: Lambda, API Gateway, Bedrock
- Set up billing alerts

---

## Customization

### Add Your Own Policies

Edit `lambda_function.py` → `DOCUMENT_CONTEXTS` dictionary:
```python
DOCUMENT_CONTEXTS = {
    "YourDepartment": """Your policy content here..."""
}
```

### Change Model

Edit `lambda_function.py` → `call_bedrock_llm` function:
```python
modelId='anthropic.claude-3-sonnet-20240229-v1:0'  # Use Claude instead
```

### Adjust Keywords

Edit `lambda_function.py` → `detect_department` function:
```python
if any(w in q for w in ['your', 'keywords', 'here']):
    return "YourDepartment"
```

---

## Troubleshooting

### Error: "AccessDeniedException"
- Check IAM role has Bedrock permissions
- Verify model access is approved in Bedrock console

### Error: "Timeout"
- Increase Lambda timeout (Configuration → General)
- Check CloudWatch logs for bottlenecks

### Error: "429 Too Many Requests"
- Bedrock has rate limits
- Implement exponential backoff
- Request quota increase

---

## Security Best Practices

1. **Never commit AWS credentials**
   - Use IAM roles

2. **Enable API Gateway authentication**
   - Add API keys
   - Use AWS IAM authorization
   - Implement usage plans

3. **Monitor costs**
   - Set up billing alerts
   - Use AWS Cost Explorer
   - Tag resources for tracking

4. **Regular updates**
   - Keep Python runtime updated
   - Review IAM permissions
   - Monitor security advisories

---

## Cost Optimization

1. **Right-size Lambda memory** (512 MB is good balance)
2. **Enable CloudWatch Logs retention** (7 days instead of infinite)
3. **Use API Gateway caching** for common queries
4. **Implement rate limiting** to prevent abuse
5. **Consider Reserved Capacity** for Bedrock if high volume

---

## Next Steps

- Add authentication (API keys)
- Implement FAISS vector search
- Build web UI
- Add monitoring dashboard
- Set up CI/CD pipeline
- Enable multi-region deployment

---

## Support

For issues or questions:
- Check CloudWatch Logs first
- Review AWS service quotas
- Consult AWS documentation

---

**Estimated Monthly Cost:** €5-10 (for testing/demo usage)
