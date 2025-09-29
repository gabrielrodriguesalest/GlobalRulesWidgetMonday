# Development Standards - Monday Dashboard Widgets

## TypeScript Standards

### Strict Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### Type Definitions
```typescript
// types/widget.types.ts
export interface WidgetSettings {
  title: string;
  chartType: ChartType;
  refreshInterval: number;
  showLegend: boolean;
  colorScheme: ColorScheme;
}

export interface WidgetData {
  id: string;
  boardId: string;
  items: WidgetItem[];
  lastUpdated: Date;
}

export interface WidgetItem {
  id: string;
  name: string;
  values: Record<string, ColumnValue>;
}

export type ChartType = 'bar' | 'line' | 'pie' | 'metrics';
export type WidgetSize = 'small' | 'medium' | 'large';
export type ColorScheme = 'default' | 'vibrant' | 'monochrome';
```

## Code Style Standards

### ESLint Configuration
```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "react-hooks/recommended",
    "prettier"
  ],
  "rules": {
    "no-console": "error",
    "prefer-const": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "react-hooks/exhaustive-deps": "error"
  }
}
```

### Naming Conventions
```typescript
// ✅ Good - PascalCase for components
const MetricsWidget = () => {};
const ChartContainer = () => {};

// ✅ Good - camelCase for functions and variables
const fetchWidgetData = () => {};
const isLoading = true;

// ✅ Good - UPPER_SNAKE_CASE for constants
const API_ENDPOINTS = {
  BOARDS: '/boards',
  ITEMS: '/items'
};

// ✅ Good - kebab-case for files
// metrics-widget.tsx
// chart-container.tsx
```

## Component Standards

### Functional Components with Hooks
```typescript
interface MetricsWidgetProps {
  boardIds: string[];
  settings: WidgetSettings;
  size: WidgetSize;
  onSettingsChange?: (settings: WidgetSettings) => void;
}

const MetricsWidget: React.FC<MetricsWidgetProps> = ({
  boardIds,
  settings,
  size,
  onSettingsChange
}) => {
  // 1. State hooks
  const [data, setData] = useState<WidgetData[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  // 2. Custom hooks
  const { theme } = useTheme();
  const { executeQuery } = useMonday();
  
  // 3. Memoized values
  const processedData = useMemo(() => 
    processMetricsData(data, settings), 
    [data, settings]
  );
  
  // 4. Callbacks
  const handleRefresh = useCallback(async () => {
    setLoading(true);
    try {
      const newData = await fetchWidgetData(boardIds);
      setData(newData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [boardIds]);
  
  // 5. Effects
  useEffect(() => {
    handleRefresh();
  }, [handleRefresh]);
  
  // 6. Early returns
  if (loading) return <WidgetSkeleton size={size} />;
  if (error) return <ErrorMessage error={error} onRetry={handleRefresh} />;
  if (!data.length) return <EmptyState />;
  
  // 7. Main render
  return (
    <WidgetContainer size={size} theme={theme}>
      <WidgetHeader 
        title={settings.title}
        onRefresh={handleRefresh}
      />
      <MetricsDisplay data={processedData} />
    </WidgetContainer>
  );
};

export default MetricsWidget;
```

### Props Validation
```typescript
import { z } from 'zod';

const WidgetSettingsSchema = z.object({
  title: z.string().min(1).max(100),
  chartType: z.enum(['bar', 'line', 'pie', 'metrics']),
  refreshInterval: z.number().min(30).max(3600),
  showLegend: z.boolean(),
  colorScheme: z.enum(['default', 'vibrant', 'monochrome'])
});

// Runtime validation
const validateSettings = (settings: unknown): WidgetSettings => {
  return WidgetSettingsSchema.parse(settings);
};
```

## Custom Hooks Standards

### Data Fetching Hook
```typescript
interface UseWidgetDataOptions {
  refreshInterval?: number;
  enabled?: boolean;
}

export const useWidgetData = (
  boardIds: string[],
  options: UseWidgetDataOptions = {}
) => {
  const { refreshInterval = 300000, enabled = true } = options;
  
  const [data, setData] = useState<WidgetData[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    if (!enabled || boardIds.length === 0) return;
    
    setLoading(true);
    setError(null);
    
    try {
      const result = await fetchBoardsData(boardIds);
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [boardIds, enabled]);
  
  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);
  
  // Auto-refresh
  useEffect(() => {
    if (!enabled) return;
    
    fetchData();
    
    const interval = setInterval(fetchData, refreshInterval);
    return () => clearInterval(interval);
  }, [fetchData, refreshInterval, enabled]);
  
  return {
    data,
    loading,
    error,
    refetch
  };
};
```

### Settings Hook
```typescript
export const useWidgetSettings = (
  initialSettings: WidgetSettings,
  onSettingsChange?: (settings: WidgetSettings) => void
) => {
  const [settings, setSettings] = useState(initialSettings);
  
  const updateSettings = useCallback((
    updates: Partial<WidgetSettings>
  ) => {
    const newSettings = { ...settings, ...updates };
    
    // Validate settings
    try {
      validateSettings(newSettings);
      setSettings(newSettings);
      onSettingsChange?.(newSettings);
    } catch (error) {
      console.error('Invalid settings:', error);
    }
  }, [settings, onSettingsChange]);
  
  const resetSettings = useCallback(() => {
    setSettings(initialSettings);
    onSettingsChange?.(initialSettings);
  }, [initialSettings, onSettingsChange]);
  
  return {
    settings,
    updateSettings,
    resetSettings
  };
};
```

## Service Layer Standards

### API Service
```typescript
class MondayApiService {
  private rateLimiter = new RateLimiter(100, 'minute');
  
  async executeQuery<T>(
    query: string, 
    variables?: Record<string, any>
  ): Promise<T> {
    await this.rateLimiter.acquire();
    
    try {
      const response = await monday.api(query, { variables });
      
      if (response.errors) {
        throw new ApiError('GraphQL errors', response.errors);
      }
      
      return response.data;
    } catch (error) {
      this.handleApiError(error);
      throw error;
    }
  }
  
  private handleApiError(error: any): void {
    if (error.status === 429) {
      console.warn('Rate limit exceeded, backing off...');
    } else if (error.status >= 500) {
      console.error('Server error:', error);
    }
  }
}
```

### Cache Service
```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  ttl: number;
}

class CacheService {
  private cache = new Map<string, CacheEntry<any>>();
  
  set<T>(key: string, data: T, ttlSeconds: number): void {
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
      ttl: ttlSeconds * 1000
    });
  }
  
  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) return null;
    
    if (Date.now() - entry.timestamp > entry.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  invalidate(pattern: string): void {
    for (const key of this.cache.keys()) {
      if (key.includes(pattern)) {
        this.cache.delete(key);
      }
    }
  }
  
  clear(): void {
    this.cache.clear();
  }
}
```

## Error Handling Standards

### Error Types
```typescript
export class WidgetError extends Error {
  constructor(
    message: string,
    public code: string,
    public context?: Record<string, any>
  ) {
    super(message);
    this.name = 'WidgetError';
  }
}

export class ApiError extends WidgetError {
  constructor(message: string, public errors: any[]) {
    super(message, 'API_ERROR', { errors });
  }
}

export class ValidationError extends WidgetError {
  constructor(message: string, public field: string) {
    super(message, 'VALIDATION_ERROR', { field });
  }
}
```

### Error Handling Hook
```typescript
export const useErrorHandler = () => {
  const [error, setError] = useState<Error | null>(null);
  
  const handleError = useCallback((error: Error) => {
    console.error('Widget error:', error);
    
    // Log to monitoring service
    if (error instanceof WidgetError) {
      logError(error.code, error.message, error.context);
    } else {
      logError('UNKNOWN_ERROR', error.message);
    }
    
    setError(error);
  }, []);
  
  const clearError = useCallback(() => {
    setError(null);
  }, []);
  
  return {
    error,
    handleError,
    clearError
  };
};
```

## Testing Standards

### Unit Test Structure
```typescript
describe('MetricsWidget', () => {
  const defaultProps = {
    boardIds: ['123', '456'],
    settings: {
      title: 'Test Widget',
      chartType: 'metrics' as const,
      refreshInterval: 300,
      showLegend: true,
      colorScheme: 'default' as const
    },
    size: 'medium' as const
  };
  
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('renders loading state initially', () => {
    render(<MetricsWidget {...defaultProps} />);
    expect(screen.getByTestId('widget-skeleton')).toBeInTheDocument();
  });
  
  test('renders data when loaded', async () => {
    const mockData = [
      { id: '1', name: 'Item 1', value: 100 }
    ];
    
    jest.mocked(fetchBoardsData).mockResolvedValue(mockData);
    
    render(<MetricsWidget {...defaultProps} />);
    
    await waitFor(() => {
      expect(screen.getByText('Item 1')).toBeInTheDocument();
    });
  });
  
  test('handles error state', async () => {
    const error = new Error('API Error');
    jest.mocked(fetchBoardsData).mockRejectedValue(error);
    
    render(<MetricsWidget {...defaultProps} />);
    
    await waitFor(() => {
      expect(screen.getByText('API Error')).toBeInTheDocument();
    });
  });
});
```

### Integration Test Example
```typescript
describe('Widget Integration', () => {
  test('fetches and displays real data', async () => {
    const server = setupServer(
      rest.post('/api/graphql', (req, res, ctx) => {
        return res(
          ctx.json({
            data: {
              boards: [
                {
                  id: '123',
                  items: [
                    { id: '1', name: 'Test Item', column_values: [] }
                  ]
                }
              ]
            }
          })
        );
      })
    );
    
    server.listen();
    
    render(<MetricsWidget {...defaultProps} />);
    
    await waitFor(() => {
      expect(screen.getByText('Test Item')).toBeInTheDocument();
    });
    
    server.close();
  });
});
```

## Performance Standards

### Bundle Size Limits
```json
{
  "bundlesize": [
    {
      "path": "./dist/widget.js",
      "maxSize": "100kb"
    },
    {
      "path": "./dist/vendor.js",
      "maxSize": "200kb"
    }
  ]
}
```

### Performance Monitoring
```typescript
const usePerformanceMonitor = (componentName: string) => {
  useEffect(() => {
    const startTime = performance.now();
    
    return () => {
      const duration = performance.now() - startTime;
      
      if (duration > 100) { // Log slow renders
        console.warn(`Slow render: ${componentName} took ${duration}ms`);
      }
    };
  });
};
```
