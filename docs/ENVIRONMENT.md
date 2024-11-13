# Environment Configuration Guide

## Environment Variables

### Required Variables

```env
# AWS Configuration
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
BEDROCK_ENDPOINT=https://bedrock.us-east-1.amazonaws.com

# Application Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Optional Variables

```env
# Feature Flags
ENABLE_DEBUG_MODE=true
ENABLE_TELEMETRY=false

# Performance
EDGE_CACHE_TTL=3600
MAX_REQUESTS_PER_MINUTE=60

# Development
DEBUG=true
VERBOSE_LOGGING=true
```

## Configuration Files

1. **next.config.js**
   ```javascript
   /** @type {import('next').NextConfig} */
   const nextConfig = {
     reactStrictMode: true,
     experimental: {
       serverActions: true,
     },
     images: {
       domains: ['your-domain.com'],
     },
   };

   module.exports = nextConfig;
   ```

2. **.env.example**
   ```env
   # AWS Configuration
   AWS_REGION=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   BEDROCK_ENDPOINT=

   # Application Configuration
   NEXT_PUBLIC_APP_URL=
   NODE_ENV=development
   ```

## Environment Setup

### Development Environment

1. **Local Setup**
   ```bash
   # Copy environment template
   cp .env.example .env.local

   # Edit environment variables
   nano .env.local
   ```

2. **Development Mode**
   ```bash
   # Start development server
   npm run dev
   ```

### Production Environment

1. **Vercel Deployment**
   - Add environment variables in Vercel Dashboard
   - Enable production optimization
   - Configure edge functions

2. **AWS Deployment**
   - Set up AWS Parameter Store
   - Configure CloudFormation
   - Enable CloudWatch logging

## Environment Types

### Development
```env
NODE_ENV=development
DEBUG=true
VERBOSE_LOGGING=true
EDGE_CACHE_TTL=0
```

### Staging
```env
NODE_ENV=production
DEBUG=true
VERBOSE_LOGGING=true
EDGE_CACHE_TTL=60
```

### Production
```env
NODE_ENV=production
DEBUG=false
VERBOSE_LOGGING=false
EDGE_CACHE_TTL=3600
```

## Feature Flags

```typescript
// config/features.ts
export const featureFlags = {
  debugMode: process.env.ENABLE_DEBUG_MODE === 'true',
  telemetry: process.env.ENABLE_TELEMETRY !== 'false',
  experimentalFeatures: process.env.ENABLE_EXPERIMENTAL === 'true',
};
```

## AWS Configuration

### Bedrock Settings
```typescript
// config/bedrock.ts
export const bedrockConfig = {
  region: process.env.AWS_REGION,
  endpoint: process.env.BEDROCK_ENDPOINT,
  maxTokens: parseInt(process.env.MAX_TOKENS || '2048', 10),
  temperature: parseFloat(process.env.TEMPERATURE || '0.7'),
};
```

## Security Configuration

### CORS Settings
```typescript
// next.config.js
{
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Origin', value: process.env.CORS_ORIGIN || '*' },
          { key: 'Access-Control-Allow-Methods', value: 'GET,POST,OPTIONS' },
        ],
      },
    ];
  },
}
```

## Rate Limiting

```typescript
// config/rate-limit.ts
export const rateLimitConfig = {
  maxRequests: parseInt(process.env.MAX_REQUESTS_PER_MINUTE || '60', 10),
  windowMs: 60 * 1000, // 1 minute
};
```

## Cache Configuration

```typescript
// config/cache.ts
export const cacheConfig = {
  ttl: parseInt(process.env.EDGE_CACHE_TTL || '3600', 10),
  staleWhileRevalidate: parseInt(process.env.SWR_TTL || '60', 10),
};
```

## Logging Configuration

```typescript
// config/logging.ts
export const loggingConfig = {
  level: process.env.LOG_LEVEL || 'info',
  verbose: process.env.VERBOSE_LOGGING === 'true',
  destination: process.env.LOG_DESTINATION || 'console',
};
```

## Development Tools

### Debug Configuration
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Next.js: Debug Server",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 9229,
      "console": "integratedTerminal"
    }
  ]
}
```

## Environment Validation

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  AWS_REGION: z.string(),
  AWS_ACCESS_KEY_ID: z.string(),
  AWS_SECRET_ACCESS_KEY: z.string(),
  BEDROCK_ENDPOINT: z.string().url(),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

export function validateEnv() {
  const parsed = envSchema.safeParse(process.env);
  if (!parsed.success) {
    console.error('❌ Invalid environment variables:', parsed.error.format());
    process.exit(1);
  }
  return parsed.data;
}
```

## Multi-environment Setup

```bash
# Development
.env.development

# Staging
.env.staging

# Production
.env.production
```

## Environment Best Practices

1. Never commit sensitive environment variables
2. Use different values per environment
3. Validate environment variables at startup
4. Document all environment variables
5. Use type-safe environment variables
6. Keep .env.example updated
7. Use environment-specific configurations
8. Implement secure secret management

## Troubleshooting

### Common Issues
1. Missing environment variables
2. Invalid AWS credentials
3. Wrong endpoint configuration
4. Rate limit issues
5. Cache configuration problems

### Validation
```bash
# Validate environment
npm run validate:env

# Check AWS configuration
npm run check:aws
```

### Debug Mode
```bash
# Enable debug mode
DEBUG=true npm run dev

# Enable verbose logging
VERBOSE_LOGGING=true npm run dev
```
