# Security Guide

## Security Best Practices

### 1. Authentication & Authorization

#### API Authentication
```typescript
// middleware/auth.ts
import { NextRequest } from 'next/server';
import { verifyToken } from '@/lib/auth';

export async function authMiddleware(req: NextRequest) {
  const token = req.headers.get('Authorization')?.split(' ')[1];
  if (!token) {
    return new Response('Unauthorized', { status: 401 });
  }

  try {
    const verified = await verifyToken(token);
    return verified;
  } catch (error) {
    return new Response('Invalid token', { status: 403 });
  }
}
```

#### JWT Implementation
```typescript
// lib/auth.ts
import { sign, verify } from 'jsonwebtoken';

export const createToken = (payload: object) => {
  return sign(payload, process.env.JWT_SECRET!, {
    expiresIn: '24h',
  });
};

export const verifyToken = (token: string) => {
  return verify(token, process.env.JWT_SECRET!);
};
```

### 2. Rate Limiting

#### Basic Rate Limiter
```typescript
// lib/rate-limit.ts
import { RateLimiter } from '@/lib/utils';

const limiter = new RateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
});

export const rateLimitMiddleware = async (req: NextRequest) => {
  try {
    await limiter.check(req); // throws error if limit reached
    return true;
  } catch {
    return new Response('Too Many Requests', { status: 429 });
  }
};
```

#### Advanced Rate Limiting
```typescript
// config/rate-limit.ts
export const rateLimitConfig = {
  tiers: {
    free: {
      requestsPerMinute: 60,
      tokensPerDay: 100000,
    },
    pro: {
      requestsPerMinute: 300,
      tokensPerDay: 1000000,
    },
  },
  global: {
    maxRequestsPerIp: 1000,
    windowSize: '1h',
  },
};
```

### 3. Input Validation

#### Request Validation
```typescript
// lib/validation.ts
import { z } from 'zod';

const ExecutionRequestSchema = z.object({
  code: z.string().max(10000),
  modelId: z.string().regex(/^[\w\-\.]+$/),
  parameters: z.object({
    temperature: z.number().min(0).max(1),
    maxTokens: z.number().min(1).max(4096),
  }),
});

export const validateExecutionRequest = (data: unknown) => {
  return ExecutionRequestSchema.parse(data);
};
```

#### Content Sanitization
```typescript
// lib/sanitize.ts
import { sanitizeHtml } from 'sanitize-html';

export const sanitizeOutput = (content: string) => {
  return sanitizeHtml(content, {
    allowedTags: ['b', 'i', 'em', 'strong', 'p'],
    allowedAttributes: {},
  });
};
```

### 4. AWS Security

#### IAM Configuration
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
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}
```

#### Secure AWS Client
```typescript
// lib/aws.ts
import { BedrockRuntimeClient } from '@aws-sdk/client-bedrock-runtime';

export const createSecureClient = () => {
  return new BedrockRuntimeClient({
    region: process.env.AWS_REGION,
    credentials: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
    },
    maxAttempts: 3,
  });
};
```

### 5. Data Protection

#### Environment Variables
```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  JWT_SECRET: z.string().min(32),
  AWS_ACCESS_KEY_ID: z.string(),
  AWS_SECRET_ACCESS_KEY: z.string(),
  AWS_REGION: z.string(),
});

export const validateEnv = () => {
  const parsed = envSchema.safeParse(process.env);
  if (!parsed.success) {
    console.error('Invalid environment variables:', parsed.error.format());
    process.exit(1);
  }
  return parsed.data;
};
```

#### Secure Headers
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'origin-when-cross-origin'
  }
];
```

### 6. CORS Configuration

#### CORS Middleware
```typescript
// middleware/cors.ts
export const corsMiddleware = (req: NextRequest) => {
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization',
        'Access-Control-Allow-Origin': process.env.ALLOWED_ORIGIN || '*',
      },
    });
  }
};
```

### 7. Error Handling

#### Secure Error Handler
```typescript
// lib/error.ts
export class SecurityError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number
  ) {
    super(message);
    this.name = 'SecurityError';
  }
}

export const handleSecurityError = (error: SecurityError) => {
  // Log error securely
  console.error('Security Error:', {
    code: error.code,
    message: error.message,
    timestamp: new Date().toISOString(),
  });

  // Return safe error response
  return new Response(
    JSON.stringify({
      error: 'Security Error',
      message: 'An error occurred',
    }),
    { status: error.statusCode }
  );
};
```

### 8. Monitoring & Logging

#### Security Monitoring
```typescript
// lib/monitoring.ts
interface SecurityEvent {
  type: 'auth' | 'rate-limit' | 'validation';
  severity: 'low' | 'medium' | 'high';
  details: object;
}

export class SecurityMonitor {
  static async logEvent(event: SecurityEvent) {
    // Log security event
    console.log('Security Event:', {
      ...event,
      timestamp: new Date().toISOString(),
    });
  }

  static async checkThreshold(type: string): Promise<boolean> {
    // Check security thresholds
    return true;
  }
}
```

### 9. Security Best Practices Checklist

1. **Authentication**
   - [ ] Implement JWT authentication
   - [ ] Use secure session management
   - [ ] Implement proper password hashing
   - [ ] Enable MFA where applicable

2. **Authorization**
   - [ ] Implement role-based access control
   - [ ] Use principle of least privilege
   - [ ] Validate user permissions
   - [ ] Audit access logs

3. **Data Security**
   - [ ] Encrypt sensitive data
   - [ ] Secure environment variables
   - [ ] Implement proper data sanitization
   - [ ] Regular security audits

4. **API Security**
   - [ ] Implement rate limiting
   - [ ] Validate all inputs
   - [ ] Sanitize all outputs
   - [ ] Use HTTPS only

5. **AWS Security**
   - [ ] Use IAM roles and policies
   - [ ] Enable CloudTrail logging
   - [ ] Regular credential rotation
   - [ ] Enable VPC endpoints

### 10. Security Updates

```typescript
// lib/security-updates.ts
interface SecurityUpdate {
  version: string;
  date: string;
  changes: string[];
  critical: boolean;
}

export class SecurityUpdates {
  static async checkUpdates(): Promise<SecurityUpdate[]> {
    // Check for security updates
    return [];
  }

  static async applyUpdates(): Promise<void> {
    // Apply security updates
  }
}
```

### 11. Dependency Security

```typescript
// scripts/check-dependencies.ts
import { exec } from 'child_process';

export const checkDependencies = async () => {
  // Run npm audit
  exec('npm audit', (error, stdout, stderr) => {
    if (error) {
      console.error(`Security audit failed: ${error}`);
      return;
    }
    console.log('Security audit results:', stdout);
  });
};
```

### 12. Security Documentation

- Keep this security documentation up to date
- Document all security-related changes
- Maintain security changelog
- Document incident response procedures
- Regular security reviews
