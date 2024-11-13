# Deployment Guide

This guide covers deploying the AWS Next Edge Bedrock REPL to production environments.

## Deployment Options

1. **Vercel (Recommended)**
   - Native Next.js support
   - Edge function capabilities
   - Automatic deployments
   - Built-in monitoring

2. **AWS Deployment**
   - Full AWS stack
   - Custom domain control
   - Advanced networking
   - Regional deployment

3. **Custom Edge Deployment**
   - Manual edge configuration
   - Custom CDN setup
   - Advanced caching
   - Custom routing

## Vercel Deployment

1. **Setup**
   ```bash
   # Install Vercel CLI
   npm i -g vercel

   # Login to Vercel
   vercel login
   ```

2. **Configuration**
   ```bash
   # Initialize Vercel
   vercel init

   # Deploy
   vercel deploy
   ```

3. **Environment Variables**
   - Add in Vercel Dashboard:
     ```env
     AWS_REGION=
     AWS_ACCESS_KEY_ID=
     AWS_SECRET_ACCESS_KEY=
     BEDROCK_ENDPOINT=
     ```

4. **Domain Setup**
   - Add custom domain
   - Configure SSL
   - Set up redirects

## AWS Deployment

1. **Prerequisites**
   - AWS CLI configured
   - IAM roles setup
   - Domain in Route 53

2. **Infrastructure Setup**
   ```bash
   # Install AWS CDK
   npm install -g aws-cdk

   # Initialize CDK
   cdk init app --language typescript
   ```

3. **Deployment Stack**
   ```typescript
   import * as cdk from 'aws-cdk-lib';
   import * as lambda from 'aws-cdk-lib/aws-lambda';
   import * as apigateway from 'aws-cdk-lib/aws-apigateway';

   export class BedrockReplStack extends cdk.Stack {
     constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
       super(scope, id, props);

       // Define resources
       const handler = new lambda.Function(this, 'Handler', {
         runtime: lambda.Runtime.NODEJS_18_X,
         code: lambda.Code.fromAsset('dist'),
         handler: 'index.handler',
       });

       // API Gateway
       new apigateway.LambdaRestApi(this, 'Endpoint', {
         handler,
       });
     }
   }
   ```

4. **Deploy**
   ```bash
   # Build application
   npm run build

   # Deploy infrastructure
   cdk deploy
   ```

## Edge Function Configuration

1. **Edge Function Setup**
   ```typescript
   // pages/api/execute.ts
   import { NextRequest } from 'next/server'

   export const config = {
     runtime: 'edge'
   }

   export default async function handler(req: NextRequest) {
     // Handler implementation
   }
   ```

2. **Caching Configuration**
   ```typescript
   // next.config.js
   module.exports = {
     experimental: {
       runtime: 'edge',
     },
     async headers() {
       return [
         {
           source: '/api/:path*',
           headers: [
             {
               key: 'Cache-Control',
               value: 's-maxage=1, stale-while-revalidate',
             },
           ],
         },
       ]
     },
   }
   ```

## Production Checklist

1. **Performance**
   - [ ] Bundle optimization
   - [ ] Image optimization
   - [ ] Edge caching
   - [ ] API response time

2. **Security**
   - [ ] Environment variables
   - [ ] API authentication
   - [ ] Rate limiting
   - [ ] CORS configuration

3. **Monitoring**
   - [ ] Error tracking
   - [ ] Performance monitoring
   - [ ] Usage analytics
   - [ ] Cost monitoring

4. **Reliability**
   - [ ] Health checks
   - [ ] Auto-scaling
   - [ ] Backup strategy
   - [ ] Rollback plan

## CI/CD Pipeline

1. **GitHub Actions**
   ```yaml
   name: Deploy
   on:
     push:
       branches: [main]

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         - uses: actions/setup-node@v2
         - run: npm ci
         - run: npm run test
         - run: npm run build
         - uses: aws-actions/configure-aws-credentials@v1
         - uses: amondnet/vercel-action@v20
   ```

2. **Automated Testing**
   ```yaml
   name: Test
   on: [pull_request]

   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         - run: npm ci
         - run: npm run test
         - run: npm run lint
   ```

## Monitoring Setup

1. **Error Tracking**
   ```typescript
   // sentry.config.js
   import * as Sentry from '@sentry/nextjs';

   Sentry.init({
     dsn: process.env.SENTRY_DSN,
     tracesSampleRate: 1.0,
   });
   ```

2. **Performance Monitoring**
   ```typescript
   // middleware.ts
   export function middleware(request: NextRequest) {
     // Add performance headers
     request.headers.set('Server-Timing', 'edge;dur=100');
   }
   ```

## Rollback Procedure

1. **Version Control**
   ```bash
   # List deployments
   vercel ls

   # Rollback to previous
   vercel rollback
   ```

2. **Data Backup**
   ```bash
   # Backup configuration
   aws s3 cp config.json s3://backup-bucket/

   # Restore configuration
   aws s3 cp s3://backup-bucket/config.json .
   ```

## Cost Optimization

1. **Resource Management**
   - Edge function optimization
   - Caching strategy
   - API request optimization
   - Model selection

2. **Monitoring**
   - Cost alerts
   - Usage tracking
   - Performance metrics
   - Optimization opportunities

## Support and Maintenance

1. **Health Checks**
   ```typescript
   // pages/api/health.ts
   export default async function handler(req: NextRequest) {
     return new Response(JSON.stringify({
       status: 'healthy',
       timestamp: new Date().toISOString(),
     }))
   }
   ```

2. **Logging**
   ```typescript
   // lib/logger.ts
   const logger = {
     info: (message: string) => {
       console.log(`[INFO] ${message}`);
     },
     error: (error: Error) => {
       console.error(`[ERROR] ${error.message}`);
     },
   };
   ```

## Security Considerations

1. **API Security**
   - Rate limiting
   - Authentication
   - Input validation
   - Output sanitization

2. **Infrastructure Security**
   - IAM roles
   - Network security
   - Secrets management
   - Compliance monitoring
