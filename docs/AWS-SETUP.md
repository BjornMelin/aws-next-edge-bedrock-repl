# AWS Setup Guide

This guide walks you through setting up AWS Bedrock for use with the REPL.

## Prerequisites

1. **AWS Account**
   - Active AWS account
   - Admin access or appropriate IAM permissions
   - Billing information configured

2. **AWS CLI**
   ```bash
   # Install AWS CLI
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```

3. **Required Permissions**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "bedrock:*"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

## AWS Configuration Steps

1. **Configure AWS Credentials**
   ```bash
   aws configure
   ```
   Enter your:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default region
   - Output format (json recommended)

2. **Enable AWS Bedrock**
   - Navigate to AWS Console
   - Search for "Bedrock"
   - Click "Enable Service"

3. **Model Access**
   - Request access to required models
   - Wait for approval (usually instant)
   - Verify model access

4. **Create IAM Role**
   ```bash
   # Create role
   aws iam create-role --role-name bedrock-repl-role --assume-role-policy-document file://trust-policy.json

   # Attach policy
   aws iam attach-role-policy --role-name bedrock-repl-role --policy-arn arn:aws:iam::aws:policy/AWSBedrockFullAccess
   ```

## Environment Configuration

1. **Local Environment**
   ```bash
   # Copy example env file
   cp .env.example .env.local

   # Edit environment variables
   nano .env.local
   ```

2. **Required Environment Variables**
   ```env
   AWS_REGION=your-region
   AWS_ACCESS_KEY_ID=your-access-key
   AWS_SECRET_ACCESS_KEY=your-secret-key
   BEDROCK_ENDPOINT=https://bedrock.your-region.amazonaws.com
   ```

## Verification Steps

1. **Test AWS Configuration**
   ```bash
   # Verify AWS CLI setup
   aws sts get-caller-identity

   # Test Bedrock access
   aws bedrock list-foundation-models
   ```

2. **Test Application Integration**
   ```bash
   # Start development server
   npm run dev

   # Test REPL functionality
   curl http://localhost:3000/api/health
   ```

## Security Best Practices

1. **IAM Configuration**
   - Use principle of least privilege
   - Regular key rotation
   - Enable MFA
   - Use IAM roles when possible

2. **Key Management**
   - Never commit credentials
   - Use environment variables
   - Consider AWS Secrets Manager
   - Rotate credentials regularly

3. **Network Security**
   - Configure VPC if needed
   - Set up security groups
   - Enable CloudTrail logging
   - Monitor API usage

## Troubleshooting

1. **Common Issues**
   - Permission denied
   - Invalid credentials
   - Model access pending
   - Region availability

2. **Debug Commands**
   ```bash
   # Check AWS identity
   aws sts get-caller-identity

   # List available models
   aws bedrock list-foundation-models

   # Validate IAM role
   aws iam get-role --role-name bedrock-repl-role
   ```

## Cost Management

1. **Monitoring Usage**
   - Enable AWS Cost Explorer
   - Set up budgets
   - Monitor API calls
   - Track model usage

2. **Optimization Tips**
   - Use appropriate model sizes
   - Implement caching
   - Set rate limits
   - Monitor token usage

## Additional Resources

1. **AWS Documentation**
   - [Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
   - [IAM Documentation](https://docs.aws.amazon.com/IAM/)
   - [Pricing Information](https://aws.amazon.com/bedrock/pricing/)

2. **Security Resources**
   - [AWS Security Best Practices](https://docs.aws.amazon.com/security/)
   - [IAM Security Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

3. **Support Channels**
   - AWS Support
   - GitHub Issues
   - Community Forums
