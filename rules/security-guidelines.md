# Security Guidelines - Monday Dashboard Widgets

## Input Validation & Sanitization

### Data Validation Schema
```typescript
import Joi from 'joi';
import DOMPurify from 'dompurify';

// Widget settings validation
const WidgetSettingsSchema = Joi.object({
  title: Joi.string().min(1).max(100).required(),
  boardIds: Joi.array().items(Joi.string().pattern(/^\d+$/)).max(10).required(),
  chartType: Joi.string().valid('bar', 'line', 'pie', 'metrics').required(),
  refreshInterval: Joi.number().min(30).max(3600).required(),
  showLegend: Joi.boolean().required(),
  colorScheme: Joi.string().valid('default', 'vibrant', 'monochrome').required()
});

// Board data validation
const BoardDataSchema = Joi.object({
  id: Joi.string().required(),
  name: Joi.string().max(255).required(),
  items: Joi.array().items(Joi.object({
    id: Joi.string().required(),
    name: Joi.string().max(255).required(),
    column_values: Joi.array().items(Joi.object({
      id: Joi.string().required(),
      value: Joi.alternatives().try(Joi.string(), Joi.number(), Joi.object()).allow(null),
      text: Joi.string().allow('').optional()
    }))
  })).max(1000)
});

// Validation functions
export const validateWidgetSettings = (settings: unknown): WidgetSettings => {
  const { error, value } = WidgetSettingsSchema.validate(settings);
  if (error) {
    throw new ValidationError(`Invalid widget settings: ${error.details[0].message}`);
  }
  return value;
};

export const validateBoardData = (data: unknown): BoardData => {
  const { error, value } = BoardDataSchema.validate(data);
  if (error) {
    throw new ValidationError(`Invalid board data: ${error.details[0].message}`);
  }
  return value;
};
```

### HTML Sanitization
```typescript
// Strict HTML sanitization
export const sanitizeHtml = (html: string): string => {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'span'],
    ALLOWED_ATTR: ['class'],
    FORBID_SCRIPTS: true,
    FORBID_TAGS: ['script', 'object', 'embed', 'iframe'],
    FORBID_ATTR: ['onerror', 'onload', 'onclick', 'onmouseover']
  });
};

// Text-only sanitization
export const sanitizeText = (text: string): string => {
  return DOMPurify.sanitize(text, {
    ALLOWED_TAGS: [],
    ALLOWED_ATTR: []
  });
};

// URL sanitization
export const sanitizeUrl = (url: string): string => {
  try {
    const parsed = new URL(url);
    
    // Only allow safe protocols
    if (!['http:', 'https:', 'mailto:'].includes(parsed.protocol)) {
      throw new Error('Invalid protocol');
    }
    
    return parsed.toString();
  } catch (error) {
    throw new ValidationError('Invalid URL format');
  }
};
```

### Input Escaping
```typescript
// Escape special characters
export const escapeHtml = (text: string): string => {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
};

// Escape for JSON
export const escapeJson = (obj: any): string => {
  return JSON.stringify(obj).replace(/</g, '\\u003c').replace(/>/g, '\\u003e');
};

// Escape SQL-like strings (for GraphQL)
export const escapeGraphQL = (str: string): string => {
  return str.replace(/[\\"']/g, '\\$&').replace(/\n/g, '\\n').replace(/\r/g, '\\r');
};
```

## Authentication & Authorization

### Token Management
```typescript
class SecureTokenManager {
  private static readonly TOKEN_KEY = 'monday_widget_token';
  private static readonly REFRESH_THRESHOLD = 300000; // 5 minutes
  
  static setToken(token: string, expiresIn: number): void {
    const tokenData = {
      token,
      expiresAt: Date.now() + (expiresIn * 1000),
      createdAt: Date.now()
    };
    
    // Store in memory only, never in localStorage for security
    sessionStorage.setItem(this.TOKEN_KEY, JSON.stringify(tokenData));
  }
  
  static getToken(): string | null {
    try {
      const stored = sessionStorage.getItem(this.TOKEN_KEY);
      if (!stored) return null;
      
      const tokenData = JSON.parse(stored);
      
      // Check if token is expired
      if (Date.now() >= tokenData.expiresAt) {
        this.clearToken();
        return null;
      }
      
      // Check if token needs refresh
      if (Date.now() >= tokenData.expiresAt - this.REFRESH_THRESHOLD) {
        this.refreshToken();
      }
      
      return tokenData.token;
    } catch (error) {
      console.error('Token retrieval error:', error);
      this.clearToken();
      return null;
    }
  }
  
  static clearToken(): void {
    sessionStorage.removeItem(this.TOKEN_KEY);
  }
  
  private static async refreshToken(): Promise<void> {
    try {
      // Implement token refresh logic with Monday API
      const newToken = await monday.get('sessionToken');
      if (newToken) {
        this.setToken(newToken, 3600); // 1 hour
      }
    } catch (error) {
      console.error('Token refresh failed:', error);
      this.clearToken();
    }
  }
}
```

### Permission Validation
```typescript
interface UserPermissions {
  canViewBoard: (boardId: string) => boolean;
  canEditBoard: (boardId: string) => boolean;
  canCreateItems: (boardId: string) => boolean;
  canDeleteItems: (boardId: string) => boolean;
}

class PermissionValidator {
  constructor(private permissions: UserPermissions) {}
  
  validateBoardAccess(boardIds: string[], operation: 'read' | 'write'): void {
    for (const boardId of boardIds) {
      if (operation === 'read' && !this.permissions.canViewBoard(boardId)) {
        throw new AuthorizationError(`No read access to board ${boardId}`);
      }
      
      if (operation === 'write' && !this.permissions.canEditBoard(boardId)) {
        throw new AuthorizationError(`No write access to board ${boardId}`);
      }
    }
  }
  
  validateItemOperation(boardId: string, operation: 'create' | 'delete'): void {
    if (operation === 'create' && !this.permissions.canCreateItems(boardId)) {
      throw new AuthorizationError(`No create permission for board ${boardId}`);
    }
    
    if (operation === 'delete' && !this.permissions.canDeleteItems(boardId)) {
      throw new AuthorizationError(`No delete permission for board ${boardId}`);
    }
  }
}
```

## API Security

### Rate Limiting
```typescript
class RateLimiter {
  private requests = new Map<string, number[]>();
  
  constructor(
    private maxRequests: number,
    private windowMs: number
  ) {}
  
  async checkLimit(identifier: string): Promise<boolean> {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // Get existing requests for this identifier
    const userRequests = this.requests.get(identifier) || [];
    
    // Remove old requests outside the window
    const validRequests = userRequests.filter(time => time > windowStart);
    
    // Check if limit exceeded
    if (validRequests.length >= this.maxRequests) {
      return false;
    }
    
    // Add current request
    validRequests.push(now);
    this.requests.set(identifier, validRequests);
    
    return true;
  }
  
  getRemainingRequests(identifier: string): number {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    const userRequests = this.requests.get(identifier) || [];
    const validRequests = userRequests.filter(time => time > windowStart);
    
    return Math.max(0, this.maxRequests - validRequests.length);
  }
}

// Usage in API service
class SecureMondayApiService {
  private rateLimiter = new RateLimiter(100, 60000); // 100 requests per minute
  
  async executeQuery<T>(query: string, variables?: any): Promise<T> {
    const userId = this.getCurrentUserId();
    
    // Check rate limit
    if (!await this.rateLimiter.checkLimit(userId)) {
      throw new RateLimitError('Rate limit exceeded');
    }
    
    // Validate query for potential injection
    this.validateGraphQLQuery(query);
    
    try {
      const response = await monday.api(query, { variables });
      
      if (response.errors) {
        this.logSecurityEvent('graphql_error', { query, errors: response.errors });
        throw new ApiError('GraphQL errors', response.errors);
      }
      
      return response.data;
    } catch (error) {
      this.logSecurityEvent('api_error', { query, error: error.message });
      throw error;
    }
  }
  
  private validateGraphQLQuery(query: string): void {
    // Check for suspicious patterns
    const suspiciousPatterns = [
      /\b(union|fragment)\b/i,
      /\{.*\{.*\{/,  // Deep nesting
      /__schema|__type/i,  // Introspection
      /mutation.*delete/i  // Dangerous mutations
    ];
    
    for (const pattern of suspiciousPatterns) {
      if (pattern.test(query)) {
        throw new SecurityError('Potentially unsafe GraphQL query');
      }
    }
    
    // Check query complexity
    const depth = this.calculateQueryDepth(query);
    if (depth > 10) {
      throw new SecurityError('Query too complex');
    }
  }
  
  private calculateQueryDepth(query: string): number {
    const openBraces = (query.match(/\{/g) || []).length;
    const closeBraces = (query.match(/\}/g) || []).length;
    
    if (openBraces !== closeBraces) {
      throw new SecurityError('Malformed GraphQL query');
    }
    
    return openBraces;
  }
}
```

### Request Signing
```typescript
class RequestSigner {
  constructor(private secretKey: string) {}
  
  async signRequest(payload: any, timestamp: number): Promise<string> {
    const message = JSON.stringify(payload) + timestamp.toString();
    const encoder = new TextEncoder();
    const data = encoder.encode(message);
    
    const key = await crypto.subtle.importKey(
      'raw',
      encoder.encode(this.secretKey),
      { name: 'HMAC', hash: 'SHA-256' },
      false,
      ['sign']
    );
    
    const signature = await crypto.subtle.sign('HMAC', key, data);
    return Array.from(new Uint8Array(signature))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
  }
  
  async verifySignature(
    payload: any, 
    timestamp: number, 
    signature: string
  ): Promise<boolean> {
    const expectedSignature = await this.signRequest(payload, timestamp);
    return this.constantTimeCompare(signature, expectedSignature);
  }
  
  private constantTimeCompare(a: string, b: string): boolean {
    if (a.length !== b.length) return false;
    
    let result = 0;
    for (let i = 0; i < a.length; i++) {
      result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    }
    
    return result === 0;
  }
}
```

## Data Protection

### Encryption Utilities
```typescript
class DataEncryption {
  private static readonly ALGORITHM = 'AES-GCM';
  private static readonly KEY_LENGTH = 256;
  
  static async generateKey(): Promise<CryptoKey> {
    return await crypto.subtle.generateKey(
      {
        name: this.ALGORITHM,
        length: this.KEY_LENGTH
      },
      true,
      ['encrypt', 'decrypt']
    );
  }
  
  static async encrypt(data: string, key: CryptoKey): Promise<EncryptedData> {
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(data);
    
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    const encrypted = await crypto.subtle.encrypt(
      {
        name: this.ALGORITHM,
        iv: iv
      },
      key,
      dataBuffer
    );
    
    return {
      data: Array.from(new Uint8Array(encrypted)),
      iv: Array.from(iv)
    };
  }
  
  static async decrypt(
    encryptedData: EncryptedData, 
    key: CryptoKey
  ): Promise<string> {
    const decrypted = await crypto.subtle.decrypt(
      {
        name: this.ALGORITHM,
        iv: new Uint8Array(encryptedData.iv)
      },
      key,
      new Uint8Array(encryptedData.data)
    );
    
    const decoder = new TextDecoder();
    return decoder.decode(decrypted);
  }
}

interface EncryptedData {
  data: number[];
  iv: number[];
}
```

### Secure Storage
```typescript
class SecureStorage {
  private encryptionKey: CryptoKey | null = null;
  
  async initialize(): Promise<void> {
    // Generate or retrieve encryption key
    const keyData = sessionStorage.getItem('widget_key');
    
    if (keyData) {
      this.encryptionKey = await crypto.subtle.importKey(
        'raw',
        new Uint8Array(JSON.parse(keyData)),
        { name: 'AES-GCM' },
        false,
        ['encrypt', 'decrypt']
      );
    } else {
      this.encryptionKey = await DataEncryption.generateKey();
      
      const exportedKey = await crypto.subtle.exportKey('raw', this.encryptionKey);
      sessionStorage.setItem('widget_key', JSON.stringify(Array.from(new Uint8Array(exportedKey))));
    }
  }
  
  async setSecureItem(key: string, value: any): Promise<void> {
    if (!this.encryptionKey) {
      throw new Error('Storage not initialized');
    }
    
    const serialized = JSON.stringify(value);
    const encrypted = await DataEncryption.encrypt(serialized, this.encryptionKey);
    
    sessionStorage.setItem(`secure_${key}`, JSON.stringify(encrypted));
  }
  
  async getSecureItem<T>(key: string): Promise<T | null> {
    if (!this.encryptionKey) {
      throw new Error('Storage not initialized');
    }
    
    const stored = sessionStorage.getItem(`secure_${key}`);
    if (!stored) return null;
    
    try {
      const encrypted = JSON.parse(stored);
      const decrypted = await DataEncryption.decrypt(encrypted, this.encryptionKey);
      return JSON.parse(decrypted);
    } catch (error) {
      console.error('Decryption failed:', error);
      return null;
    }
  }
  
  removeSecureItem(key: string): void {
    sessionStorage.removeItem(`secure_${key}`);
  }
}
```

## Security Headers & CSP

### Content Security Policy
```typescript
const CSP_DIRECTIVES = {
  'default-src': ["'self'"],
  'script-src': [
    "'self'",
    "'unsafe-inline'", // Required for React
    "https://cdn.monday.com",
    "https://dapulse-res.cloudinary.com"
  ],
  'style-src': [
    "'self'",
    "'unsafe-inline'", // Required for styled-components
    "https://fonts.googleapis.com"
  ],
  'font-src': [
    "'self'",
    "https://fonts.gstatic.com"
  ],
  'img-src': [
    "'self'",
    "data:",
    "https://dapulse-res.cloudinary.com",
    "https://files.monday.com"
  ],
  'connect-src': [
    "'self'",
    "https://api.monday.com",
    "https://auth.monday.com"
  ],
  'frame-ancestors': [
    "https://*.monday.com"
  ],
  'object-src': ["'none'"],
  'base-uri': ["'self'"]
};

// Generate CSP header
const generateCSPHeader = (): string => {
  return Object.entries(CSP_DIRECTIVES)
    .map(([directive, sources]) => `${directive} ${sources.join(' ')}`)
    .join('; ');
};
```

### Security Headers Hook
```typescript
const useSecurityHeaders = () => {
  useEffect(() => {
    // Set security headers via meta tags (for client-side)
    const setMetaTag = (name: string, content: string) => {
      let meta = document.querySelector(`meta[name="${name}"]`);
      if (!meta) {
        meta = document.createElement('meta');
        meta.setAttribute('name', name);
        document.head.appendChild(meta);
      }
      meta.setAttribute('content', content);
    };
    
    // Referrer Policy
    setMetaTag('referrer', 'strict-origin-when-cross-origin');
    
    // X-Content-Type-Options
    setMetaTag('X-Content-Type-Options', 'nosniff');
    
    // X-Frame-Options
    setMetaTag('X-Frame-Options', 'SAMEORIGIN');
    
    // Permissions Policy
    setMetaTag('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  }, []);
};
```

## Error Handling & Logging

### Secure Error Handling
```typescript
class SecureErrorHandler {
  private static readonly SAFE_ERROR_MESSAGES = {
    VALIDATION_ERROR: 'Invalid input provided',
    AUTHORIZATION_ERROR: 'Access denied',
    RATE_LIMIT_ERROR: 'Too many requests',
    API_ERROR: 'Service temporarily unavailable',
    UNKNOWN_ERROR: 'An unexpected error occurred'
  };
  
  static handleError(error: Error): SafeError {
    // Log full error details securely
    this.logError(error);
    
    // Return sanitized error to client
    if (error instanceof ValidationError) {
      return new SafeError('VALIDATION_ERROR', this.SAFE_ERROR_MESSAGES.VALIDATION_ERROR);
    }
    
    if (error instanceof AuthorizationError) {
      return new SafeError('AUTHORIZATION_ERROR', this.SAFE_ERROR_MESSAGES.AUTHORIZATION_ERROR);
    }
    
    if (error instanceof RateLimitError) {
      return new SafeError('RATE_LIMIT_ERROR', this.SAFE_ERROR_MESSAGES.RATE_LIMIT_ERROR);
    }
    
    if (error instanceof ApiError) {
      return new SafeError('API_ERROR', this.SAFE_ERROR_MESSAGES.API_ERROR);
    }
    
    // Default to unknown error
    return new SafeError('UNKNOWN_ERROR', this.SAFE_ERROR_MESSAGES.UNKNOWN_ERROR);
  }
  
  private static logError(error: Error): void {
    const errorLog = {
      timestamp: new Date().toISOString(),
      message: error.message,
      stack: error.stack,
      type: error.constructor.name,
      userId: this.getCurrentUserId(),
      sessionId: this.getSessionId(),
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    // Send to secure logging service
    console.error('Security Error:', errorLog);
    
    // In production, send to monitoring service
    if (process.env.NODE_ENV === 'production') {
      this.sendToMonitoringService(errorLog);
    }
  }
  
  private static sendToMonitoringService(errorLog: any): void {
    // Implementation for sending to monitoring service
    // e.g., Sentry, LogRocket, etc.
  }
}

class SafeError extends Error {
  constructor(
    public readonly code: string,
    message: string
  ) {
    super(message);
    this.name = 'SafeError';
  }
}
```

### Audit Logging
```typescript
class AuditLogger {
  private static readonly EVENTS = {
    WIDGET_LOADED: 'widget_loaded',
    DATA_ACCESSED: 'data_accessed',
    SETTINGS_CHANGED: 'settings_changed',
    ERROR_OCCURRED: 'error_occurred',
    SECURITY_VIOLATION: 'security_violation'
  };
  
  static logEvent(
    event: string, 
    details: Record<string, any> = {}
  ): void {
    const auditEntry = {
      timestamp: new Date().toISOString(),
      event,
      userId: this.getCurrentUserId(),
      sessionId: this.getSessionId(),
      widgetId: this.getWidgetId(),
      details: this.sanitizeDetails(details)
    };
    
    // Store audit log securely
    this.storeAuditLog(auditEntry);
  }
  
  private static sanitizeDetails(details: Record<string, any>): Record<string, any> {
    const sanitized: Record<string, any> = {};
    
    for (const [key, value] of Object.entries(details)) {
      // Remove sensitive information
      if (this.isSensitiveField(key)) {
        sanitized[key] = '[REDACTED]';
      } else if (typeof value === 'string') {
        sanitized[key] = sanitizeText(value);
      } else {
        sanitized[key] = value;
      }
    }
    
    return sanitized;
  }
  
  private static isSensitiveField(fieldName: string): boolean {
    const sensitiveFields = ['password', 'token', 'secret', 'key', 'email'];
    return sensitiveFields.some(field => 
      fieldName.toLowerCase().includes(field)
    );
  }
}
```
