# Performance Optimization - Monday Dashboard Widgets

## Performance Targets

### Core Web Vitals
```typescript
const PERFORMANCE_TARGETS = {
  // Loading Performance
  firstContentfulPaint: 1500, // ms
  largestContentfulPaint: 2000, // ms
  timeToInteractive: 2500, // ms
  
  // Runtime Performance
  renderTime: 16, // ms (60fps)
  memoryUsage: 50, // MB max
  bundleSize: 100, // KB max
  
  // Network Performance
  apiResponseTime: 500, // ms
  cacheHitRate: 0.8, // 80%
  maxConcurrentRequests: 3
};
```

## Bundle Optimization

### Code Splitting
```typescript
// Lazy load heavy components
const ChartComponent = lazy(() => import('./components/ChartComponent'));
const DataTable = lazy(() => import('./components/DataTable'));
const AdvancedFilters = lazy(() => import('./components/AdvancedFilters'));

const Widget: React.FC = () => {
  const [activeTab, setActiveTab] = useState('chart');
  
  return (
    <Suspense fallback={<ComponentSkeleton />}>
      {activeTab === 'chart' && <ChartComponent />}
      {activeTab === 'table' && <DataTable />}
      {activeTab === 'filters' && <AdvancedFilters />}
    </Suspense>
  );
};
```

### Dynamic Imports
```typescript
// Load chart libraries on demand
const loadChartLibrary = async (type: ChartType) => {
  switch (type) {
    case 'bar':
      return import('recharts').then(module => module.BarChart);
    case 'line':
      return import('recharts').then(module => module.LineChart);
    case 'pie':
      return import('recharts').then(module => module.PieChart);
    default:
      throw new Error(`Unsupported chart type: ${type}`);
  }
};

const DynamicChart: React.FC<{ type: ChartType; data: any[] }> = ({ type, data }) => {
  const [ChartComponent, setChartComponent] = useState<React.ComponentType | null>(null);
  
  useEffect(() => {
    loadChartLibrary(type).then(setChartComponent);
  }, [type]);
  
  if (!ChartComponent) return <ChartSkeleton />;
  
  return <ChartComponent data={data} />;
};
```

### Webpack Configuration
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          maxSize: 200000 // 200KB
        },
        charts: {
          test: /[\\/]node_modules[\\/](recharts|chart\.js|d3)[\\/]/,
          name: 'charts',
          chunks: 'all',
          priority: 10
        },
        vibe: {
          test: /[\\/]node_modules[\\/]@vibe[\\/]/,
          name: 'vibe',
          chunks: 'all',
          priority: 15
        }
      }
    },
    usedExports: true,
    sideEffects: false
  }
};
```

## React Performance

### Memoization Strategies
```typescript
// Component memoization
const ExpensiveWidget = memo(({ data, settings }: WidgetProps) => {
  // Expensive calculations
  const processedData = useMemo(() => 
    processLargeDataset(data, settings), 
    [data, settings]
  );
  
  // Expensive renders
  const chartConfig = useMemo(() => 
    generateComplexChartConfig(processedData, settings.chartType),
    [processedData, settings.chartType]
  );
  
  return <Chart config={chartConfig} />;
}, (prevProps, nextProps) => {
  // Custom comparison for complex objects
  return (
    prevProps.data === nextProps.data &&
    JSON.stringify(prevProps.settings) === JSON.stringify(nextProps.settings)
  );
});

// Callback memoization
const useOptimizedCallbacks = (onDataChange: (data: any) => void) => {
  const handleDataUpdate = useCallback((newData: any) => {
    // Debounce updates
    const debouncedUpdate = debounce(onDataChange, 300);
    debouncedUpdate(newData);
  }, [onDataChange]);
  
  const handleRefresh = useCallback(async () => {
    // Throttle refresh requests
    const throttledRefresh = throttle(async () => {
      await fetchData();
    }, 1000);
    
    return throttledRefresh();
  }, []);
  
  return { handleDataUpdate, handleRefresh };
};
```

### Virtual Scrolling
```typescript
import { FixedSizeList as List } from 'react-window';

interface VirtualizedListProps {
  items: any[];
  itemHeight: number;
  containerHeight: number;
}

const VirtualizedList: React.FC<VirtualizedListProps> = ({
  items,
  itemHeight,
  containerHeight
}) => {
  const Row = memo(({ index, style }: { index: number; style: CSSProperties }) => (
    <div style={style}>
      <ListItem data={items[index]} />
    </div>
  ));
  
  return (
    <List
      height={containerHeight}
      itemCount={items.length}
      itemSize={itemHeight}
      width="100%"
      overscanCount={5} // Render 5 extra items for smooth scrolling
    >
      {Row}
    </List>
  );
};

// Usage in widget
const DataWidget: React.FC<{ data: any[] }> = ({ data }) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const [containerHeight, setContainerHeight] = useState(400);
  
  useEffect(() => {
    if (containerRef.current) {
      setContainerHeight(containerRef.current.clientHeight);
    }
  }, []);
  
  return (
    <div ref={containerRef} style={{ height: '100%' }}>
      {data.length > 100 ? (
        <VirtualizedList 
          items={data}
          itemHeight={50}
          containerHeight={containerHeight}
        />
      ) : (
        <RegularList items={data} />
      )}
    </div>
  );
};
```

## Data Management

### Efficient Data Fetching
```typescript
class DataManager {
  private cache = new Map<string, CacheEntry>();
  private requestQueue = new Map<string, Promise<any>>();
  
  async fetchBoardData(boardIds: string[]): Promise<BoardData[]> {
    // Batch requests for multiple boards
    const batchSize = 5;
    const batches = this.chunkArray(boardIds, batchSize);
    
    const results = await Promise.all(
      batches.map(batch => this.fetchBoardBatch(batch))
    );
    
    return results.flat();
  }
  
  private async fetchBoardBatch(boardIds: string[]): Promise<BoardData[]> {
    const cacheKey = `boards:${boardIds.join(',')}`;
    
    // Check cache first
    const cached = this.cache.get(cacheKey);
    if (cached && !this.isCacheExpired(cached)) {
      return cached.data;
    }
    
    // Deduplicate concurrent requests
    if (this.requestQueue.has(cacheKey)) {
      return this.requestQueue.get(cacheKey)!;
    }
    
    const request = this.executeBoardQuery(boardIds);
    this.requestQueue.set(cacheKey, request);
    
    try {
      const data = await request;
      this.cache.set(cacheKey, {
        data,
        timestamp: Date.now(),
        ttl: 300000 // 5 minutes
      });
      return data;
    } finally {
      this.requestQueue.delete(cacheKey);
    }
  }
  
  private chunkArray<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
}
```

### Data Normalization
```typescript
interface NormalizedData {
  entities: {
    boards: Record<string, Board>;
    items: Record<string, BoardItem>;
    columns: Record<string, Column>;
  };
  result: string[];
}

const normalizeData = (boards: Board[]): NormalizedData => {
  const normalized: NormalizedData = {
    entities: {
      boards: {},
      items: {},
      columns: {}
    },
    result: []
  };
  
  boards.forEach(board => {
    normalized.entities.boards[board.id] = {
      ...board,
      items: board.items.map(item => item.id),
      columns: board.columns.map(col => col.id)
    };
    
    board.items.forEach(item => {
      normalized.entities.items[item.id] = item;
    });
    
    board.columns.forEach(column => {
      normalized.entities.columns[column.id] = column;
    });
    
    normalized.result.push(board.id);
  });
  
  return normalized;
};

// Selector for efficient data access
const selectBoardWithItems = (
  normalizedData: NormalizedData,
  boardId: string
): Board | null => {
  const board = normalizedData.entities.boards[boardId];
  if (!board) return null;
  
  return {
    ...board,
    items: board.items.map(itemId => 
      normalizedData.entities.items[itemId]
    ),
    columns: board.columns.map(colId => 
      normalizedData.entities.columns[colId]
    )
  };
};
```

## Caching Strategies

### Multi-Level Caching
```typescript
class CacheManager {
  private memoryCache = new Map<string, CacheEntry>();
  private sessionCache = window.sessionStorage;
  private localCache = window.localStorage;
  
  async get<T>(key: string): Promise<T | null> {
    // Level 1: Memory cache (fastest)
    const memoryEntry = this.memoryCache.get(key);
    if (memoryEntry && !this.isExpired(memoryEntry)) {
      return memoryEntry.data;
    }
    
    // Level 2: Session cache
    try {
      const sessionData = this.sessionCache.getItem(key);
      if (sessionData) {
        const parsed = JSON.parse(sessionData);
        if (!this.isExpired(parsed)) {
          // Promote to memory cache
          this.memoryCache.set(key, parsed);
          return parsed.data;
        }
      }
    } catch (error) {
      console.warn('Session cache error:', error);
    }
    
    // Level 3: Local cache (persistent)
    try {
      const localData = this.localCache.getItem(key);
      if (localData) {
        const parsed = JSON.parse(localData);
        if (!this.isExpired(parsed)) {
          // Promote to higher levels
          this.memoryCache.set(key, parsed);
          this.sessionCache.setItem(key, localData);
          return parsed.data;
        }
      }
    } catch (error) {
      console.warn('Local cache error:', error);
    }
    
    return null;
  }
  
  async set<T>(
    key: string, 
    data: T, 
    ttl: number = 300000,
    persistent: boolean = false
  ): Promise<void> {
    const entry: CacheEntry = {
      data,
      timestamp: Date.now(),
      ttl
    };
    
    // Always store in memory
    this.memoryCache.set(key, entry);
    
    // Store in session cache
    try {
      this.sessionCache.setItem(key, JSON.stringify(entry));
    } catch (error) {
      console.warn('Session cache storage error:', error);
    }
    
    // Store in local cache if persistent
    if (persistent) {
      try {
        this.localCache.setItem(key, JSON.stringify(entry));
      } catch (error) {
        console.warn('Local cache storage error:', error);
      }
    }
  }
  
  private isExpired(entry: CacheEntry): boolean {
    return Date.now() - entry.timestamp > entry.ttl;
  }
}
```

### Cache Invalidation
```typescript
class SmartCacheManager extends CacheManager {
  private dependencies = new Map<string, Set<string>>();
  
  setDependency(key: string, dependsOn: string[]): void {
    dependsOn.forEach(dep => {
      if (!this.dependencies.has(dep)) {
        this.dependencies.set(dep, new Set());
      }
      this.dependencies.get(dep)!.add(key);
    });
  }
  
  invalidate(pattern: string): void {
    // Direct pattern matching
    for (const key of this.memoryCache.keys()) {
      if (key.includes(pattern)) {
        this.memoryCache.delete(key);
        this.sessionCache.removeItem(key);
        this.localCache.removeItem(key);
      }
    }
    
    // Dependency-based invalidation
    const dependentKeys = this.dependencies.get(pattern);
    if (dependentKeys) {
      dependentKeys.forEach(key => {
        this.memoryCache.delete(key);
        this.sessionCache.removeItem(key);
        this.localCache.removeItem(key);
      });
    }
  }
}

// Usage with Monday API
const useSmartCache = () => {
  const cacheManager = useMemo(() => new SmartCacheManager(), []);
  
  const fetchBoardData = useCallback(async (boardId: string) => {
    const cacheKey = `board:${boardId}`;
    
    // Set up dependencies
    cacheManager.setDependency(cacheKey, [`board:${boardId}:items`, `board:${boardId}:columns`]);
    
    let data = await cacheManager.get(cacheKey);
    if (!data) {
      data = await mondayApi.getBoard(boardId);
      await cacheManager.set(cacheKey, data, 300000); // 5 minutes
    }
    
    return data;
  }, [cacheManager]);
  
  return { fetchBoardData, invalidateCache: cacheManager.invalidate.bind(cacheManager) };
};
```

## Network Optimization

### Request Batching
```typescript
class RequestBatcher {
  private batchQueue = new Map<string, BatchRequest>();
  private batchTimeout = 50; // ms
  
  async batchRequest<T>(
    key: string,
    request: () => Promise<T>
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      if (!this.batchQueue.has(key)) {
        this.batchQueue.set(key, {
          requests: [],
          timeout: setTimeout(() => this.executeBatch(key), this.batchTimeout)
        });
      }
      
      const batch = this.batchQueue.get(key)!;
      batch.requests.push({ resolve, reject, request });
    });
  }
  
  private async executeBatch(key: string): Promise<void> {
    const batch = this.batchQueue.get(key);
    if (!batch) return;
    
    this.batchQueue.delete(key);
    clearTimeout(batch.timeout);
    
    try {
      // Execute all requests in parallel
      const results = await Promise.allSettled(
        batch.requests.map(req => req.request())
      );
      
      results.forEach((result, index) => {
        const request = batch.requests[index];
        if (result.status === 'fulfilled') {
          request.resolve(result.value);
        } else {
          request.reject(result.reason);
        }
      });
    } catch (error) {
      batch.requests.forEach(req => req.reject(error));
    }
  }
}
```

### Connection Pooling
```typescript
class ConnectionPool {
  private activeConnections = 0;
  private maxConnections = 6;
  private queue: Array<() => void> = [];
  
  async execute<T>(request: () => Promise<T>): Promise<T> {
    if (this.activeConnections >= this.maxConnections) {
      await this.waitForSlot();
    }
    
    this.activeConnections++;
    
    try {
      return await request();
    } finally {
      this.activeConnections--;
      this.processQueue();
    }
  }
  
  private waitForSlot(): Promise<void> {
    return new Promise(resolve => {
      this.queue.push(resolve);
    });
  }
  
  private processQueue(): void {
    if (this.queue.length > 0 && this.activeConnections < this.maxConnections) {
      const next = this.queue.shift();
      next?.();
    }
  }
}
```

## Performance Monitoring

### Performance Metrics Hook
```typescript
const usePerformanceMonitor = (componentName: string) => {
  const startTime = useRef<number>();
  const [metrics, setMetrics] = useState<PerformanceMetrics>({});
  
  useEffect(() => {
    startTime.current = performance.now();
    
    return () => {
      if (startTime.current) {
        const duration = performance.now() - startTime.current;
        
        setMetrics(prev => ({
          ...prev,
          [componentName]: {
            renderTime: duration,
            timestamp: Date.now()
          }
        }));
        
        // Log slow renders
        if (duration > 16) { // 60fps threshold
          console.warn(`Slow render: ${componentName} took ${duration.toFixed(2)}ms`);
        }
      }
    };
  });
  
  return metrics;
};
```

### Bundle Analysis
```typescript
// webpack-bundle-analyzer integration
const analyzeBundleSize = () => {
  if (process.env.ANALYZE_BUNDLE) {
    const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
    
    return new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    });
  }
  
  return null;
};

// Performance budget
const performanceBudget = {
  maxAssetSize: 100000, // 100KB
  maxEntrypointSize: 200000, // 200KB
  hints: 'warning'
};
```
