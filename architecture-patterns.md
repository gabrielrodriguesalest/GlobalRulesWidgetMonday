# Architecture Patterns - Monday Dashboard Widgets

## Clean Architecture for Widgets

### Layer Structure
```
┌─────────────────────────────────────────┐
│           UI Layer (React)              │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │   Widgets   │  │   Components    │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│         Application Layer               │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │    Hooks    │  │    Services     │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│          Domain Layer                   │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │  Entities   │  │ Business Rules  │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│       Infrastructure Layer             │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │ Monday API  │  │     Cache       │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘
```

### Implementation Example
```typescript
// Domain Layer - Widget Entity
export class Widget {
  constructor(
    private id: string,
    private settings: WidgetSettings,
    private boardIds: string[]
  ) {}
  
  validateSettings(): void {
    if (this.boardIds.length === 0) {
      throw new Error('At least one board is required');
    }
  }
}

// Application Layer - Widget Service
export class WidgetService {
  constructor(
    private dataRepository: IDataRepository,
    private cacheService: ICacheService
  ) {}
  
  async getWidgetData(widget: Widget): Promise<WidgetData> {
    widget.validateSettings();
    
    const cacheKey = `widget:${widget.id}`;
    const cached = await this.cacheService.get(cacheKey);
    
    if (cached) return cached;
    
    const data = await this.dataRepository.fetchData(widget.boardIds);
    await this.cacheService.set(cacheKey, data, 300);
    
    return data;
  }
}

// Infrastructure Layer - Monday Repository
export class MondayDataRepository implements IDataRepository {
  async fetchData(boardIds: string[]): Promise<WidgetData> {
    const query = `
      query GetBoards($ids: [Int!]!) {
        boards(ids: $ids) {
          id
          name
          items {
            id
            name
            column_values {
              id
              text
              value
            }
          }
        }
      }
    `;
    
    return monday.api(query, { variables: { ids: boardIds } });
  }
}
```

## Component Architecture

### Base Widget Structure
```typescript
interface BaseWidgetProps {
  settings: WidgetSettings;
  boardIds: string[];
  size: WidgetSize;
  onError?: (error: Error) => void;
}

const BaseWidget: React.FC<BaseWidgetProps> = ({
  settings,
  boardIds,
  size,
  onError
}) => {
  const { data, loading, error } = useWidgetData(boardIds);
  const theme = useTheme();
  
  if (error) {
    onError?.(error);
    return <ErrorFallback error={error} />;
  }
  
  return (
    <WidgetContainer size={size} theme={theme}>
      <WidgetHeader title={settings.title} />
      <WidgetContent loading={loading}>
        {data && <WidgetBody data={data} settings={settings} />}
      </WidgetContent>
    </WidgetContainer>
  );
};
```

### Factory Pattern for Charts
```typescript
abstract class ChartFactory {
  abstract createChart(type: ChartType, data: ChartData): React.ComponentType;
  
  protected validateData(data: ChartData): void {
    if (!data || data.length === 0) {
      throw new Error('Chart data is required');
    }
  }
}

class VibeChartFactory extends ChartFactory {
  createChart(type: ChartType, data: ChartData): React.ComponentType {
    this.validateData(data);
    
    switch (type) {
      case 'bar':
        return () => <BarChart data={data} />;
      case 'line':
        return () => <LineChart data={data} />;
      case 'pie':
        return () => <PieChart data={data} />;
      default:
        throw new Error(`Unsupported chart type: ${type}`);
    }
  }
}
```

## State Management Patterns

### Context + Reducer Pattern
```typescript
interface WidgetState {
  data: WidgetData | null;
  loading: boolean;
  error: string | null;
  settings: WidgetSettings;
}

type WidgetAction = 
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: WidgetData }
  | { type: 'FETCH_ERROR'; payload: string }
  | { type: 'UPDATE_SETTINGS'; payload: WidgetSettings };

const widgetReducer = (state: WidgetState, action: WidgetAction): WidgetState => {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, data: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    case 'UPDATE_SETTINGS':
      return { ...state, settings: action.payload };
    default:
      return state;
  }
};

const WidgetProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(widgetReducer, initialState);
  
  return (
    <WidgetContext.Provider value={{ state, dispatch }}>
      {children}
    </WidgetContext.Provider>
  );
};
```

## Performance Patterns

### Virtualization for Large Datasets
```typescript
import { FixedSizeList as List } from 'react-window';

const VirtualizedList: React.FC<{ items: any[] }> = ({ items }) => {
  const Row = ({ index, style }: { index: number; style: CSSProperties }) => (
    <div style={style}>
      <ListItem data={items[index]} />
    </div>
  );
  
  return (
    <List
      height={400}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </List>
  );
};
```

### Memoization Patterns
```typescript
const OptimizedWidget = memo(({ data, settings }: WidgetProps) => {
  const processedData = useMemo(() => 
    processWidgetData(data, settings), 
    [data, settings]
  );
  
  const chartConfig = useMemo(() => 
    generateChartConfig(processedData, settings.chartType),
    [processedData, settings.chartType]
  );
  
  return <Chart config={chartConfig} />;
});
```

## Error Handling Patterns

### Error Boundary with Fallback
```typescript
class WidgetErrorBoundary extends React.Component<
  { children: ReactNode; fallback?: React.ComponentType<{ error: Error }> },
  { hasError: boolean; error: Error | null }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Widget Error:', error, errorInfo);
    // Send to monitoring service
  }
  
  render() {
    if (this.state.hasError) {
      const Fallback = this.props.fallback || DefaultErrorFallback;
      return <Fallback error={this.state.error!} />;
    }
    
    return this.props.children;
  }
}
```

## Testing Patterns

### Widget Testing Strategy
```typescript
describe('MetricsWidget', () => {
  const mockData = [
    { id: '1', name: 'Item 1', value: 100 },
    { id: '2', name: 'Item 2', value: 200 }
  ];
  
  test('renders metrics correctly', () => {
    render(
      <MetricsWidget 
        data={mockData} 
        settings={{ showTotal: true }} 
      />
    );
    
    expect(screen.getByText('Total: 300')).toBeInTheDocument();
  });
  
  test('handles empty data', () => {
    render(<MetricsWidget data={[]} settings={{}} />);
    expect(screen.getByText('No data available')).toBeInTheDocument();
  });
});
```
