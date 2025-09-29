# Deployment Guidelines - Monday Dashboard Widgets

## Deployment Pipeline

### CI/CD Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy Widget

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  MONDAY_API_TOKEN: ${{ secrets.MONDAY_API_TOKEN }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run type checking
        run: npm run type-check
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NODE_ENV: production
      
      - name: Analyze bundle size
        run: npm run analyze-bundle
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-files
          path: dist/
  
  e2e:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-files
          path: dist/
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload E2E results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
  
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: [test, build, e2e]
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-files
          path: dist/
      
      - name: Deploy to staging
        run: |
          npm run deploy:staging
        env:
          MONDAY_API_TOKEN: ${{ secrets.MONDAY_API_TOKEN_STAGING }}
          DEPLOY_ENV: staging
  
  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: [test, build, e2e]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-files
          path: dist/
      
      - name: Deploy to production
        run: |
          npm run deploy:production
        env:
          MONDAY_API_TOKEN: ${{ secrets.MONDAY_API_TOKEN_PROD }}
          DEPLOY_ENV: production
      
      - name: Create release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release v${{ github.run_number }}
          draft: false
          prerelease: false
```

## Build Optimization

### Webpack Configuration
```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

const isProduction = process.env.NODE_ENV === 'production';
const shouldAnalyze = process.env.ANALYZE_BUNDLE === 'true';

module.exports = {
  mode: isProduction ? 'production' : 'development',
  
  entry: {
    main: './src/index.tsx'
  },
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isProduction 
      ? '[name].[contenthash:8].js' 
      : '[name].js',
    chunkFilename: isProduction 
      ? '[name].[contenthash:8].chunk.js' 
      : '[name].chunk.js',
    clean: true,
    publicPath: '/'
  },
  
  optimization: {
    minimize: isProduction,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: isProduction,
            drop_debugger: isProduction
          }
        }
      })
    ],
    
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          maxSize: 200000 // 200KB
        },
        vibe: {
          test: /[\\/]node_modules[\\/]@vibe[\\/]/,
          name: 'vibe',
          chunks: 'all',
          priority: 10
        },
        charts: {
          test: /[\\/]node_modules[\\/](recharts|chart\.js|d3)[\\/]/,
          name: 'charts',
          chunks: 'all',
          priority: 15
        }
      }
    },
    
    runtimeChunk: {
      name: 'runtime'
    }
  },
  
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              transpileOnly: true,
              configFile: 'tsconfig.json'
            }
          }
        ],
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /\.(png|jpg|jpeg|gif|svg)$/,
        type: 'asset/resource',
        generator: {
          filename: 'images/[name].[contenthash:8][ext]'
        }
      }
    ]
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: isProduction ? {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      } : false
    }),
    
    ...(isProduction ? [
      new MiniCssExtractPlugin({
        filename: '[name].[contenthash:8].css',
        chunkFilename: '[name].[contenthash:8].chunk.css'
      })
    ] : []),
    
    ...(shouldAnalyze ? [
      new BundleAnalyzerPlugin({
        analyzerMode: 'static',
        openAnalyzer: false,
        reportFilename: 'bundle-report.html'
      })
    ] : [])
  ],
  
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@services': path.resolve(__dirname, 'src/services'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@types': path.resolve(__dirname, 'src/types')
    }
  },
  
  devServer: {
    port: 3000,
    hot: true,
    historyApiFallback: true,
    compress: true
  },
  
  performance: {
    maxAssetSize: 100000, // 100KB
    maxEntrypointSize: 200000, // 200KB
    hints: isProduction ? 'error' : 'warning'
  }
};
```

### Package.json Scripts
```json
{
  "scripts": {
    "dev": "webpack serve --mode development",
    "build": "webpack --mode production",
    "build:analyze": "ANALYZE_BUNDLE=true npm run build",
    "type-check": "tsc --noEmit",
    "lint": "eslint src --ext .ts,.tsx --max-warnings 0",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "test:unit": "jest",
    "test:integration": "jest --config jest.integration.config.js",
    "test:e2e": "playwright test",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "deploy:staging": "monday-cli deploy --env staging",
    "deploy:production": "monday-cli deploy --env production",
    "version:patch": "npm version patch && git push --tags",
    "version:minor": "npm version minor && git push --tags",
    "version:major": "npm version major && git push --tags"
  }
}
```

## Environment Configuration

### Environment Variables
```typescript
// src/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  apiToken: string;
  environment: 'development' | 'staging' | 'production';
  debug: boolean;
  logLevel: 'debug' | 'info' | 'warn' | 'error';
  cacheEnabled: boolean;
  cacheTtl: number;
  maxRetries: number;
  requestTimeout: number;
}

const getEnvironmentConfig = (): EnvironmentConfig => {
  const env = process.env.NODE_ENV || 'development';
  
  const baseConfig = {
    apiUrl: process.env.MONDAY_API_URL || 'https://api.monday.com/v2',
    apiToken: process.env.MONDAY_API_TOKEN || '',
    environment: env as EnvironmentConfig['environment'],
    debug: env === 'development',
    cacheEnabled: env !== 'development',
    maxRetries: 3,
    requestTimeout: 10000
  };
  
  switch (env) {
    case 'development':
      return {
        ...baseConfig,
        logLevel: 'debug',
        cacheTtl: 60000, // 1 minute
        debug: true
      };
      
    case 'staging':
      return {
        ...baseConfig,
        logLevel: 'info',
        cacheTtl: 300000, // 5 minutes
        debug: false
      };
      
    case 'production':
      return {
        ...baseConfig,
        logLevel: 'error',
        cacheTtl: 600000, // 10 minutes
        debug: false
      };
      
    default:
      throw new Error(`Unknown environment: ${env}`);
  }
};

export const config = getEnvironmentConfig();

// Validate required environment variables
const validateEnvironment = () => {
  const required = ['MONDAY_API_TOKEN'];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
};

if (config.environment !== 'development') {
  validateEnvironment();
}
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production image
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

## Monitoring and Observability

### Application Monitoring
```typescript
// src/monitoring/monitoring.ts
interface MetricData {
  name: string;
  value: number;
  timestamp: number;
  tags?: Record<string, string>;
}

interface LogEntry {
  level: 'debug' | 'info' | 'warn' | 'error';
  message: string;
  timestamp: string;
  context?: Record<string, any>;
  userId?: string;
  sessionId?: string;
}

class MonitoringService {
  private metrics: MetricData[] = [];
  private logs: LogEntry[] = [];
  
  // Performance monitoring
  trackPerformance(name: string, startTime: number): void {
    const duration = performance.now() - startTime;
    
    this.trackMetric('performance', duration, {
      operation: name,
      environment: config.environment
    });
    
    // Log slow operations
    if (duration > 1000) {
      this.log('warn', `Slow operation: ${name} took ${duration}ms`);
    }
  }
  
  // Error tracking
  trackError(error: Error, context?: Record<string, any>): void {
    const errorData = {
      message: error.message,
      stack: error.stack,
      name: error.name,
      context,
      timestamp: new Date().toISOString(),
      userId: this.getCurrentUserId(),
      sessionId: this.getSessionId(),
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    this.log('error', 'Application error', errorData);
    
    // Send to external monitoring service
    if (config.environment === 'production') {
      this.sendToSentry(errorData);
    }
  }
  
  // Custom metrics
  trackMetric(name: string, value: number, tags?: Record<string, string>): void {
    const metric: MetricData = {
      name,
      value,
      timestamp: Date.now(),
      tags: {
        environment: config.environment,
        version: process.env.APP_VERSION || 'unknown',
        ...tags
      }
    };
    
    this.metrics.push(metric);
    
    // Send to monitoring service
    if (config.environment !== 'development') {
      this.sendMetric(metric);
    }
  }
  
  // Structured logging
  log(level: LogEntry['level'], message: string, context?: Record<string, any>): void {
    const logEntry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context,
      userId: this.getCurrentUserId(),
      sessionId: this.getSessionId()
    };
    
    this.logs.push(logEntry);
    
    // Console output for development
    if (config.debug) {
      console[level](message, context);
    }
    
    // Send to logging service
    if (config.environment !== 'development') {
      this.sendLog(logEntry);
    }
  }
  
  // Health check
  getHealthStatus(): HealthStatus {
    const now = Date.now();
    const recentErrors = this.logs.filter(
      log => log.level === 'error' && 
      new Date(log.timestamp).getTime() > now - 300000 // Last 5 minutes
    );
    
    const recentMetrics = this.metrics.filter(
      metric => metric.timestamp > now - 60000 // Last minute
    );
    
    return {
      status: recentErrors.length > 10 ? 'unhealthy' : 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.APP_VERSION || 'unknown',
      uptime: performance.now(),
      errors: recentErrors.length,
      metrics: recentMetrics.length
    };
  }
  
  private getCurrentUserId(): string | undefined {
    // Implementation to get current user ID
    return undefined;
  }
  
  private getSessionId(): string {
    // Implementation to get session ID
    return 'session-id';
  }
  
  private async sendToSentry(errorData: any): Promise<void> {
    // Implementation for Sentry integration
  }
  
  private async sendMetric(metric: MetricData): Promise<void> {
    // Implementation for metrics service
  }
  
  private async sendLog(logEntry: LogEntry): Promise<void> {
    // Implementation for logging service
  }
}

interface HealthStatus {
  status: 'healthy' | 'unhealthy';
  timestamp: string;
  version: string;
  uptime: number;
  errors: number;
  metrics: number;
}

export const monitoring = new MonitoringService();
```

### Performance Monitoring Hook
```typescript
// src/hooks/usePerformanceMonitoring.ts
import { useEffect, useRef } from 'react';
import { monitoring } from '../monitoring/monitoring';

export const usePerformanceMonitoring = (componentName: string) => {
  const startTime = useRef<number>();
  const renderCount = useRef(0);
  
  useEffect(() => {
    startTime.current = performance.now();
    renderCount.current++;
    
    return () => {
      if (startTime.current) {
        const duration = performance.now() - startTime.current;
        
        monitoring.trackMetric('component_render_time', duration, {
          component: componentName,
          renderCount: renderCount.current.toString()
        });
        
        // Track slow renders
        if (duration > 16) { // 60fps threshold
          monitoring.log('warn', `Slow render: ${componentName}`, {
            duration,
            renderCount: renderCount.current
          });
        }
      }
    };
  });
  
  const trackUserAction = (action: string, metadata?: Record<string, any>) => {
    monitoring.trackMetric('user_action', 1, {
      component: componentName,
      action,
      ...metadata
    });
  };
  
  return { trackUserAction };
};
```

## Rollback Strategy

### Automated Rollback
```typescript
// scripts/rollback.ts
interface DeploymentInfo {
  version: string;
  timestamp: string;
  commitHash: string;
  environment: string;
  status: 'success' | 'failed' | 'rolled_back';
}

class RollbackManager {
  private deploymentHistory: DeploymentInfo[] = [];
  
  async rollback(environment: string, targetVersion?: string): Promise<void> {
    const currentDeployment = this.getCurrentDeployment(environment);
    
    if (!currentDeployment) {
      throw new Error(`No deployment found for environment: ${environment}`);
    }
    
    const targetDeployment = targetVersion 
      ? this.getDeploymentByVersion(environment, targetVersion)
      : this.getPreviousSuccessfulDeployment(environment);
    
    if (!targetDeployment) {
      throw new Error('No valid rollback target found');
    }
    
    console.log(`Rolling back from ${currentDeployment.version} to ${targetDeployment.version}`);
    
    try {
      // Perform rollback
      await this.performRollback(environment, targetDeployment);
      
      // Update deployment status
      currentDeployment.status = 'rolled_back';
      
      // Notify team
      await this.notifyRollback(environment, currentDeployment, targetDeployment);
      
      console.log('Rollback completed successfully');
    } catch (error) {
      console.error('Rollback failed:', error);
      throw error;
    }
  }
  
  private async performRollback(
    environment: string, 
    targetDeployment: DeploymentInfo
  ): Promise<void> {
    // Implementation for actual rollback process
    // This would typically involve:
    // 1. Switching to previous version
    // 2. Updating configuration
    // 3. Restarting services
    // 4. Running health checks
  }
  
  private getCurrentDeployment(environment: string): DeploymentInfo | null {
    return this.deploymentHistory
      .filter(d => d.environment === environment)
      .sort((a, b) => new Date(b.timestamp).getTime() - new Date(a.timestamp).getTime())[0] || null;
  }
  
  private getPreviousSuccessfulDeployment(environment: string): DeploymentInfo | null {
    return this.deploymentHistory
      .filter(d => d.environment === environment && d.status === 'success')
      .sort((a, b) => new Date(b.timestamp).getTime() - new Date(a.timestamp).getTime())[1] || null;
  }
  
  private getDeploymentByVersion(environment: string, version: string): DeploymentInfo | null {
    return this.deploymentHistory
      .find(d => d.environment === environment && d.version === version) || null;
  }
  
  private async notifyRollback(
    environment: string,
    from: DeploymentInfo,
    to: DeploymentInfo
  ): Promise<void> {
    // Implementation for team notification
    // Could use Slack, email, etc.
  }
}

export const rollbackManager = new RollbackManager();
```

### Health Checks
```typescript
// src/health/healthCheck.ts
interface HealthCheckResult {
  name: string;
  status: 'healthy' | 'unhealthy' | 'degraded';
  message?: string;
  duration: number;
  timestamp: string;
}

class HealthChecker {
  private checks: Map<string, () => Promise<HealthCheckResult>> = new Map();
  
  registerCheck(name: string, checkFn: () => Promise<HealthCheckResult>): void {
    this.checks.set(name, checkFn);
  }
  
  async runAllChecks(): Promise<HealthCheckResult[]> {
    const results: HealthCheckResult[] = [];
    
    for (const [name, checkFn] of this.checks) {
      try {
        const result = await checkFn();
        results.push(result);
      } catch (error) {
        results.push({
          name,
          status: 'unhealthy',
          message: error instanceof Error ? error.message : 'Unknown error',
          duration: 0,
          timestamp: new Date().toISOString()
        });
      }
    }
    
    return results;
  }
  
  async getOverallHealth(): Promise<{
    status: 'healthy' | 'unhealthy' | 'degraded';
    checks: HealthCheckResult[];
  }> {
    const checks = await this.runAllChecks();
    
    const unhealthyCount = checks.filter(c => c.status === 'unhealthy').length;
    const degradedCount = checks.filter(c => c.status === 'degraded').length;
    
    let status: 'healthy' | 'unhealthy' | 'degraded';
    
    if (unhealthyCount > 0) {
      status = 'unhealthy';
    } else if (degradedCount > 0) {
      status = 'degraded';
    } else {
      status = 'healthy';
    }
    
    return { status, checks };
  }
}

// Register health checks
const healthChecker = new HealthChecker();

healthChecker.registerCheck('api', async () => {
  const startTime = performance.now();
  
  try {
    const response = await fetch('/api/health');
    const duration = performance.now() - startTime;
    
    if (response.ok) {
      return {
        name: 'api',
        status: 'healthy',
        duration,
        timestamp: new Date().toISOString()
      };
    } else {
      return {
        name: 'api',
        status: 'unhealthy',
        message: `API returned ${response.status}`,
        duration,
        timestamp: new Date().toISOString()
      };
    }
  } catch (error) {
    return {
      name: 'api',
      status: 'unhealthy',
      message: error instanceof Error ? error.message : 'Unknown error',
      duration: performance.now() - startTime,
      timestamp: new Date().toISOString()
    };
  }
});

healthChecker.registerCheck('memory', async () => {
  const startTime = performance.now();
  
  const memoryInfo = (performance as any).memory;
  if (!memoryInfo) {
    return {
      name: 'memory',
      status: 'degraded',
      message: 'Memory info not available',
      duration: performance.now() - startTime,
      timestamp: new Date().toISOString()
    };
  }
  
  const usedMemoryMB = memoryInfo.usedJSHeapSize / 1024 / 1024;
  const totalMemoryMB = memoryInfo.totalJSHeapSize / 1024 / 1024;
  const memoryUsage = usedMemoryMB / totalMemoryMB;
  
  let status: 'healthy' | 'unhealthy' | 'degraded';
  let message: string | undefined;
  
  if (memoryUsage > 0.9) {
    status = 'unhealthy';
    message = `High memory usage: ${(memoryUsage * 100).toFixed(1)}%`;
  } else if (memoryUsage > 0.7) {
    status = 'degraded';
    message = `Elevated memory usage: ${(memoryUsage * 100).toFixed(1)}%`;
  } else {
    status = 'healthy';
  }
  
  return {
    name: 'memory',
    status,
    message,
    duration: performance.now() - startTime,
    timestamp: new Date().toISOString()
  };
});

export { healthChecker };
```

## Deployment Checklist

### Pre-Deployment Checklist
```typescript
interface DeploymentChecklist {
  codeQuality: {
    testsPass: boolean;
    lintingPass: boolean;
    typeCheckPass: boolean;
    coverageThreshold: boolean;
  };
  
  security: {
    vulnerabilityScan: boolean;
    secretsSecured: boolean;
    dependenciesUpdated: boolean;
  };
  
  performance: {
    bundleSizeOptimal: boolean;
    loadTimeAcceptable: boolean;
    memoryUsageNormal: boolean;
  };
  
  documentation: {
    changelogUpdated: boolean;
    readmeUpdated: boolean;
    apiDocsUpdated: boolean;
  };
  
  infrastructure: {
    environmentConfigured: boolean;
    monitoringSetup: boolean;
    rollbackPlanReady: boolean;
  };
}

const validateDeploymentReadiness = async (): Promise<DeploymentChecklist> => {
  return {
    codeQuality: {
      testsPass: await runTests(),
      lintingPass: await runLinting(),
      typeCheckPass: await runTypeCheck(),
      coverageThreshold: await checkCoverage()
    },
    
    security: {
      vulnerabilityScan: await runSecurityScan(),
      secretsSecured: await validateSecrets(),
      dependenciesUpdated: await checkDependencies()
    },
    
    performance: {
      bundleSizeOptimal: await checkBundleSize(),
      loadTimeAcceptable: await checkLoadTime(),
      memoryUsageNormal: await checkMemoryUsage()
    },
    
    documentation: {
      changelogUpdated: await checkChangelog(),
      readmeUpdated: await checkReadme(),
      apiDocsUpdated: await checkApiDocs()
    },
    
    infrastructure: {
      environmentConfigured: await checkEnvironment(),
      monitoringSetup: await checkMonitoring(),
      rollbackPlanReady: await checkRollbackPlan()
    }
  };
};
```
