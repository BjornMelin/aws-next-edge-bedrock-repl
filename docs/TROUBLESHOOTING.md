# Troubleshooting Guide

## Common Issues and Solutions

### 1. AWS Bedrock Connection Issues

#### Issue: Authentication Errors
```bash
Error: AccessDeniedException: User is not authorized to perform bedrock:InvokeModel
```

**Solution:**
1. Check AWS credentials:
```bash
aws configure list
aws sts get-caller-identity
```

2. Verify IAM permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:ListFoundationModels"
      ],
      "Resource": "*"
    }
  ]
}
```

3. Confirm environment variables:
```bash
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_REGION
```

#### Issue: Region Availability
```bash
Error: Model is not available in the specified region
```

**Solution:**
1. Check model availability in your region
2. Update region configuration:
```typescript
const client = new BedrockRuntimeClient({
  region: 'us-east-1', // Use supported region
});
```

### 2. Development Environment Issues

#### Issue: Next.js Build Errors
```bash
Error: Failed to compile
```

**Solution:**
1. Clear Next.js cache:
```bash
rm -rf .next
npm run build
```

2. Check TypeScript errors:
```bash
npm run type-check
```

3. Verify dependencies:
```bash
npm ci
```

#### Issue: Environment Variables
```bash
Error: Missing environment variables
```

**Solution:**
1. Copy example environment file:
```bash
cp .env.example .env.local
```

2. Verify all required variables:
```typescript
// scripts/check-env.ts
import { z } from 'zod';

const requiredEnvVars = z.object({
  AWS_REGION: z.string(),
  AWS_ACCESS_KEY_ID: z.string(),
  AWS_SECRET_ACCESS_KEY: z.string(),
  // Add other required variables
});

export function checkEnv() {
  try {
    requiredEnvVars.parse(process.env);
  } catch (error) {
    console.error('Missing environment variables:', error.message);
    process.exit(1);
  }
}
```

### 3. Performance Issues

#### Issue: Slow Response Times
```typescript
// Debugging response times
const debugResponse = async (modelId: string, prompt: string) => {
  const start = performance.now();
  
  try {
    const response = await invokeModel(modelId, prompt);
    const duration = performance.now() - start;
    
    console.log({
      modelId,
      duration,
      promptLength: prompt.length,
      responseLength: response.length,
    });
    
    return response;
  } catch (error) {
    console.error('Response error:', error);
    throw error;
  }
};
```

**Solution:**
1. Implement caching:
```typescript
const cache = new Map();

export async function getCachedResponse(key: string) {
  if (cache.has(key)) {
    return cache.get(key);
  }
  
  const response = await generateResponse(key);
  cache.set(key, response);
  return response;
}
```

2. Optimize request parameters:
```typescript
const optimizedConfig = {
  maxTokens: 1000, // Limit response size
  temperature: 0.7,
  topP: 0.9,
};
```

### 4. Model-Specific Issues

#### Issue: Token Limit Exceeded
```bash
Error: Token limit exceeded for model
```

**Solution:**
1. Implement token counting:
```typescript
function estimateTokens(text: string): number {
  // Rough estimation: 4 chars per token
  return Math.ceil(text.length / 4);
}

function checkTokenLimit(prompt: string, maxTokens: number): boolean {
  const estimatedTokens = estimateTokens(prompt);
  return estimatedTokens <= maxTokens;
}
```

2. Split large requests:
```typescript
async function processByChunks(text: string, chunkSize: number) {
  const chunks = text.match(new RegExp(`.{1,${chunkSize}}`, 'g')) || [];
  const results = await Promise.all(
    chunks.map(chunk => processChunk(chunk))
  );
  return results.join('');
}
```

### 5. API and Network Issues

#### Issue: Rate Limiting
```typescript
class RateLimiter {
  private requests: number = 0;
  private resetTime: number = Date.now();

  async checkLimit(limit: number, windowMs: number): Promise<boolean> {
    if (Date.now() > this.resetTime + windowMs) {
      this.requests = 0;
      this.resetTime = Date.now();
    }

    if (this.requests >= limit) {
      throw new Error('Rate limit exceeded');
    }

    this.requests++;
    return true;
  }
}
```

**Solution:**
1. Implement retry logic:
```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (error.message.includes('Rate limit')) {
        await new Promise(resolve => setTimeout(resolve, delay * Math.pow(2, i)));
        continue;
      }
      throw error;
    }
  }
  
  throw lastError!;
}
```

### 6. Edge Function Issues

#### Issue: Edge Function Deployment
```bash
Error: Edge Function deployment failed
```

**Solution:**
1. Check edge function configuration:
```typescript
// next.config.js
module.exports = {
  experimental: {
    runtime: 'edge',
  },
};
```

2. Verify supported features:
```typescript
// Check Edge compatibility
const checkEdgeCompatibility = () => {
  const features = {
    supportsDynamicImports: true,
    supportsWebCrypto: typeof crypto !== 'undefined',
    supportsStreaming: typeof ReadableStream !== 'undefined',
  };
  
  console.log('Edge Compatibility:', features);
};
```

### 7. Debugging Tools

#### Console Logger
```typescript
// utils/debug.ts
export const debug = {
  log: (...args: any[]) => {
    if (process.env.DEBUG === 'true') {
      console.log('[DEBUG]', ...args);
    }
  },
  error: (...args: any[]) => {
    if (process.env.DEBUG === 'true') {
      console.error('[ERROR]', ...args);
    }
  },
  time: (label: string) => {
    if (process.env.DEBUG === 'true') {
      console.time(`[${label}]`);
    }
  },
  timeEnd: (label: string) => {
    if (process.env.DEBUG === 'true') {
      console.timeEnd(`[${label}]`);
    }
  },
};
```

#### Performance Monitoring
```typescript
// utils/performance.ts
export const performanceMonitor = {
  startTime: 0,
  
  start() {
    this.startTime = performance.now();
  },
  
  end(label: string) {
    const duration = performance.now() - this.startTime;
    debug.log(`${label}: ${duration}ms`);
    return duration;
  },
  
  async measure<T>(label: string, fn: () => Promise<T>): Promise<T> {
    this.start();
    const result = await fn();
    this.end(label);
    return result;
  },
};
```

### 8. Common Error Codes

```typescript
// types/errors.ts
export enum ErrorCode {
  AWS_AUTH_ERROR = 'AWS_AUTH_ERROR',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED',
  MODEL_NOT_AVAILABLE = 'MODEL_NOT_AVAILABLE',
  TOKEN_LIMIT_EXCEEDED = 'TOKEN_LIMIT_EXCEEDED',
  INVALID_INPUT = 'INVALID_INPUT',
  NETWORK_ERROR = 'NETWORK_ERROR',
  EDGE_FUNCTION_ERROR = 'EDGE_FUNCTION_ERROR',
}

export class AppError extends Error {
  constructor(
    message: string,
    public code: ErrorCode,
    public details?: any
  ) {
    super(message);
    this.name = 'AppError';
  }
}
```

### 9. Error Recovery

```typescript
// utils/recovery.ts
export class ErrorRecovery {
  static async recover(error: AppError): Promise<void> {
    switch (error.code) {
      case ErrorCode.AWS_AUTH_ERROR:
        await this.refreshCredentials();
        break;
      case ErrorCode.RATE_LIMIT_EXCEEDED:
        await this.handleRateLimit();
        break;
      case ErrorCode.MODEL_NOT_AVAILABLE:
        await this.switchToFallbackModel();
        break;
      // Handle other error types
    }
  }

  private static async refreshCredentials(): Promise<void> {
    // Implement credential refresh logic
  }

  private static async handleRateLimit(): Promise<void> {
    // Implement rate limit handling
  }

  private static async switchToFallbackModel(): Promise<void> {
    // Implement model fallback logic
  }
}
```

### 10. Health Checks

```typescript
// utils/health.ts
export class HealthCheck {
  static async runChecks(): Promise<HealthStatus> {
    const checks = await Promise.all([
      this.checkAWS(),
      this.checkAPI(),
      this.checkDatabase(),
      this.checkCache(),
    ]);

    return {
      healthy: checks.every(check => check.healthy),
      checks,
      timestamp: new Date().toISOString(),
    };
  }

  private static async checkAWS(): Promise<Check> {
    try {
      // Implement AWS health check
      return { name: 'aws', healthy: true };
    } catch (error) {
      return { name: 'aws', healthy: false, error };
    }
  }
}
```

### 11. Monitoring and Alerts

```typescript
// utils/monitoring.ts
export class Monitor {
  static async track(metric: string, value: number): Promise<void> {
    // Implement metric tracking
    console.log(`[METRIC] ${metric}: ${value}`);
  }

  static async alert(severity: 'low' | 'medium' | 'high', message: string): Promise<void> {
    // Implement alerting
    console.error(`[ALERT][${severity}] ${message}`);
  }
}
```

Remember to check the [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/) and [Next.js Documentation](https://nextjs.org/docs) for additional troubleshooting information.
