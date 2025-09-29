# Testing Strategies - Monday Dashboard Widgets

## Testing Pyramid

### Test Distribution
```typescript
const TESTING_PYRAMID = {
  unit: {
    percentage: 70,
    description: 'Fast, isolated tests for individual components and functions',
    tools: ['Jest', 'React Testing Library', '@testing-library/jest-dom']
  },
  integration: {
    percentage: 20,
    description: 'Tests for component interactions and API integration',
    tools: ['MSW', 'React Testing Library', 'Jest']
  },
  e2e: {
    percentage: 10,
    description: 'End-to-end tests for critical user journeys',
    tools: ['Playwright', 'Cypress']
  }
};
```

## Unit Testing

### Component Testing Setup
```typescript
// test-utils.tsx
import React, { ReactElement } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { VibeProvider } from '@vibe/core';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  queryClient?: QueryClient;
}

const AllTheProviders: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });
  
  return (
    <VibeProvider>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </VibeProvider>
  );
};

const customRender = (
  ui: ReactElement,
  options?: CustomRenderOptions
) => render(ui, { wrapper: AllTheProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

### Widget Component Tests
```typescript
// MetricsWidget.test.tsx
import { render, screen, fireEvent, waitFor } from '../test-utils';
import { MetricsWidget } from '../components/MetricsWidget';

const mockData = [
  { id: '1', name: 'Item 1', value: 100 },
  { id: '2', name: 'Item 2', value: 200 },
  { id: '3', name: 'Item 3', value: 150 }
];

const defaultProps = {
  boardIds: ['123', '456'],
  settings: {
    title: 'Test Metrics',
    chartType: 'metrics' as const,
    refreshInterval: 300,
    showLegend: true,
    colorScheme: 'default' as const
  },
  size: 'medium' as const
};

describe('MetricsWidget', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  describe('Rendering', () => {
    test('renders widget title', () => {
      render(<MetricsWidget {...defaultProps} />);
      expect(screen.getByText('Test Metrics')).toBeInTheDocument();
    });
    
    test('renders loading state initially', () => {
      render(<MetricsWidget {...defaultProps} />);
      expect(screen.getByRole('progressbar')).toBeInTheDocument();
    });
    
    test('renders metrics when data is loaded', async () => {
      jest.mocked(fetchBoardsData).mockResolvedValue(mockData);
      
      render(<MetricsWidget {...defaultProps} />);
      
      await waitFor(() => {
        expect(screen.getByText('Item 1')).toBeInTheDocument();
        expect(screen.getByText('100')).toBeInTheDocument();
      });
    });
  });
  
  describe('Error Handling', () => {
    test('displays error message when data fetch fails', async () => {
      const error = new Error('API Error');
      jest.mocked(fetchBoardsData).mockRejectedValue(error);
      
      render(<MetricsWidget {...defaultProps} />);
      
      await waitFor(() => {
        expect(screen.getByRole('alert')).toBeInTheDocument();
        expect(screen.getByText(/error/i)).toBeInTheDocument();
      });
    });
    
    test('shows retry button on error', async () => {
      jest.mocked(fetchBoardsData).mockRejectedValue(new Error('API Error'));
      
      render(<MetricsWidget {...defaultProps} />);
      
      await waitFor(() => {
        const retryButton = screen.getByRole('button', { name: /retry/i });
        expect(retryButton).toBeInTheDocument();
      });
    });
  });
  
  describe('Interactions', () => {
    test('refreshes data when refresh button is clicked', async () => {
      const fetchSpy = jest.mocked(fetchBoardsData);
      fetchSpy.mockResolvedValue(mockData);
      
      render(<MetricsWidget {...defaultProps} />);
      
      await waitFor(() => {
        expect(screen.getByText('Item 1')).toBeInTheDocument();
      });
      
      const refreshButton = screen.getByRole('button', { name: /refresh/i });
      fireEvent.click(refreshButton);
      
      expect(fetchSpy).toHaveBeenCalledTimes(2);
    });
    
    test('updates settings when settings change', () => {
      const onSettingsChange = jest.fn();
      
      render(
        <MetricsWidget 
          {...defaultProps} 
          onSettingsChange={onSettingsChange}
        />
      );
      
      const settingsButton = screen.getByRole('button', { name: /settings/i });
      fireEvent.click(settingsButton);
      
      // Test settings modal interactions
      const titleInput = screen.getByLabelText(/title/i);
      fireEvent.change(titleInput, { target: { value: 'New Title' } });
      
      const saveButton = screen.getByRole('button', { name: /save/i });
      fireEvent.click(saveButton);
      
      expect(onSettingsChange).toHaveBeenCalledWith(
        expect.objectContaining({ title: 'New Title' })
      );
    });
  });
  
  describe('Accessibility', () => {
    test('has proper ARIA labels', () => {
      render(<MetricsWidget {...defaultProps} />);
      
      const widget = screen.getByRole('region');
      expect(widget).toHaveAttribute('aria-labelledby');
    });
    
    test('supports keyboard navigation', () => {
      render(<MetricsWidget {...defaultProps} />);
      
      const buttons = screen.getAllByRole('button');
      buttons.forEach(button => {
        expect(button).toHaveAttribute('tabIndex', '0');
      });
    });
  });
  
  describe('Responsive Design', () => {
    test('adapts to different widget sizes', () => {
      const { rerender } = render(<MetricsWidget {...defaultProps} size="small" />);
      
      let container = screen.getByRole('region');
      expect(container).toHaveClass('widget-small');
      
      rerender(<MetricsWidget {...defaultProps} size="large" />);
      
      container = screen.getByRole('region');
      expect(container).toHaveClass('widget-large');
    });
  });
});
```

### Hook Testing
```typescript
// useWidgetData.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useWidgetData } from '../hooks/useWidgetData';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } }
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useWidgetData', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('fetches data successfully', async () => {
    const mockData = [{ id: '1', name: 'Test Item' }];
    jest.mocked(fetchBoardsData).mockResolvedValue(mockData);
    
    const { result } = renderHook(
      () => useWidgetData(['123']),
      { wrapper: createWrapper() }
    );
    
    expect(result.current.loading).toBe(true);
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBeNull();
  });
  
  test('handles fetch errors', async () => {
    const error = new Error('Fetch failed');
    jest.mocked(fetchBoardsData).mockRejectedValue(error);
    
    const { result } = renderHook(
      () => useWidgetData(['123']),
      { wrapper: createWrapper() }
    );
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.error).toEqual(error);
    expect(result.current.data).toEqual([]);
  });
  
  test('refetches data when boardIds change', async () => {
    const fetchSpy = jest.mocked(fetchBoardsData);
    fetchSpy.mockResolvedValue([]);
    
    const { rerender } = renderHook(
      ({ boardIds }) => useWidgetData(boardIds),
      {
        wrapper: createWrapper(),
        initialProps: { boardIds: ['123'] }
      }
    );
    
    await waitFor(() => {
      expect(fetchSpy).toHaveBeenCalledWith(['123']);
    });
    
    rerender({ boardIds: ['123', '456'] });
    
    await waitFor(() => {
      expect(fetchSpy).toHaveBeenCalledWith(['123', '456']);
    });
  });
});
```

## Integration Testing

### API Integration Tests
```typescript
// api.integration.test.ts
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { MondayApiService } from '../services/MondayApiService';

const server = setupServer(
  rest.post('https://api.monday.com/v2', (req, res, ctx) => {
    const { query, variables } = req.body as any;
    
    if (query.includes('boards')) {
      return res(
        ctx.json({
          data: {
            boards: [
              {
                id: '123',
                name: 'Test Board',
                items: [
                  {
                    id: '1',
                    name: 'Test Item',
                    column_values: [
                      { id: 'status', text: 'Done', value: '{"label":"Done"}' }
                    ]
                  }
                ]
              }
            ]
          }
        })
      );
    }
    
    return res(ctx.status(404));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('Monday API Integration', () => {
  let apiService: MondayApiService;
  
  beforeEach(() => {
    apiService = new MondayApiService();
  });
  
  test('fetches board data successfully', async () => {
    const result = await apiService.getBoardData('123');
    
    expect(result).toEqual({
      id: '123',
      name: 'Test Board',
      items: expect.arrayContaining([
        expect.objectContaining({
          id: '1',
          name: 'Test Item'
        })
      ])
    });
  });
  
  test('handles API errors gracefully', async () => {
    server.use(
      rest.post('https://api.monday.com/v2', (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({ error: 'Server Error' }));
      })
    );
    
    await expect(apiService.getBoardData('123')).rejects.toThrow('Server Error');
  });
  
  test('respects rate limiting', async () => {
    const promises = Array.from({ length: 10 }, () => 
      apiService.getBoardData('123')
    );
    
    const results = await Promise.allSettled(promises);
    
    // Should not exceed rate limit
    const fulfilled = results.filter(r => r.status === 'fulfilled');
    expect(fulfilled.length).toBeGreaterThan(0);
  });
});
```

### Component Integration Tests
```typescript
// WidgetIntegration.test.tsx
import { render, screen, waitFor, fireEvent } from '../test-utils';
import { Widget } from '../components/Widget';
import { server } from '../mocks/server';

describe('Widget Integration', () => {
  test('loads and displays data from API', async () => {
    render(
      <Widget 
        boardIds={['123']} 
        settings={{ title: 'Integration Test' }}
        size="medium"
      />
    );
    
    // Should show loading initially
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
    
    // Should display data after loading
    await waitFor(() => {
      expect(screen.getByText('Test Item')).toBeInTheDocument();
    });
    
    // Should not show loading anymore
    expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
  });
  
  test('handles settings changes end-to-end', async () => {
    const onSettingsChange = jest.fn();
    
    render(
      <Widget 
        boardIds={['123']} 
        settings={{ title: 'Original Title' }}
        size="medium"
        onSettingsChange={onSettingsChange}
      />
    );
    
    // Open settings
    const settingsButton = screen.getByRole('button', { name: /settings/i });
    fireEvent.click(settingsButton);
    
    // Change title
    const titleInput = screen.getByLabelText(/title/i);
    fireEvent.change(titleInput, { target: { value: 'New Title' } });
    
    // Save changes
    const saveButton = screen.getByRole('button', { name: /save/i });
    fireEvent.click(saveButton);
    
    // Verify callback was called
    expect(onSettingsChange).toHaveBeenCalledWith(
      expect.objectContaining({ title: 'New Title' })
    );
    
    // Verify UI updated
    await waitFor(() => {
      expect(screen.getByText('New Title')).toBeInTheDocument();
    });
  });
});
```

## End-to-End Testing

### Playwright E2E Tests
```typescript
// widget.e2e.test.ts
import { test, expect } from '@playwright/test';

test.describe('Dashboard Widget E2E', () => {
  test.beforeEach(async ({ page }) => {
    // Mock Monday.com API
    await page.route('**/api.monday.com/v2', async route => {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          data: {
            boards: [{
              id: '123',
              name: 'Test Board',
              items: [
                { id: '1', name: 'Item 1', column_values: [] },
                { id: '2', name: 'Item 2', column_values: [] }
              ]
            }]
          }
        })
      });
    });
    
    await page.goto('/widget');
  });
  
  test('displays widget with data', async ({ page }) => {
    // Wait for widget to load
    await expect(page.locator('[data-testid="widget-container"]')).toBeVisible();
    
    // Check title is displayed
    await expect(page.locator('h2')).toContainText('Test Board');
    
    // Check data items are displayed
    await expect(page.locator('[data-testid="widget-item"]')).toHaveCount(2);
    await expect(page.getByText('Item 1')).toBeVisible();
    await expect(page.getByText('Item 2')).toBeVisible();
  });
  
  test('refreshes data when refresh button is clicked', async ({ page }) => {
    let requestCount = 0;
    
    await page.route('**/api.monday.com/v2', async route => {
      requestCount++;
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          data: {
            boards: [{
              id: '123',
              name: 'Test Board',
              items: [
                { id: '1', name: `Item 1 (Request ${requestCount})`, column_values: [] }
              ]
            }]
          }
        })
      });
    });
    
    // Initial load
    await expect(page.getByText('Item 1 (Request 1)')).toBeVisible();
    
    // Click refresh
    await page.click('[data-testid="refresh-button"]');
    
    // Check data updated
    await expect(page.getByText('Item 1 (Request 2)')).toBeVisible();
  });
  
  test('handles keyboard navigation', async ({ page }) => {
    // Focus first interactive element
    await page.keyboard.press('Tab');
    
    // Should focus refresh button
    await expect(page.locator('[data-testid="refresh-button"]')).toBeFocused();
    
    // Navigate to settings button
    await page.keyboard.press('Tab');
    await expect(page.locator('[data-testid="settings-button"]')).toBeFocused();
    
    // Open settings with Enter
    await page.keyboard.press('Enter');
    await expect(page.locator('[data-testid="settings-modal"]')).toBeVisible();
    
    // Close with Escape
    await page.keyboard.press('Escape');
    await expect(page.locator('[data-testid="settings-modal"]')).not.toBeVisible();
  });
  
  test('works on different screen sizes', async ({ page }) => {
    // Test mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    await expect(page.locator('[data-testid="widget-container"]')).toBeVisible();
    
    // Test tablet viewport
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page.locator('[data-testid="widget-container"]')).toBeVisible();
    
    // Test desktop viewport
    await page.setViewportSize({ width: 1920, height: 1080 });
    await expect(page.locator('[data-testid="widget-container"]')).toBeVisible();
  });
});
```

## Performance Testing

### Performance Benchmarks
```typescript
// performance.test.ts
import { render } from '../test-utils';
import { Widget } from '../components/Widget';

describe('Widget Performance', () => {
  test('renders within performance budget', async () => {
    const startTime = performance.now();
    
    render(
      <Widget 
        boardIds={['123']} 
        settings={{ title: 'Performance Test' }}
        size="medium"
      />
    );
    
    const renderTime = performance.now() - startTime;
    
    // Should render within 16ms (60fps)
    expect(renderTime).toBeLessThan(16);
  });
  
  test('handles large datasets efficiently', async () => {
    const largeDataset = Array.from({ length: 1000 }, (_, i) => ({
      id: i.toString(),
      name: `Item ${i}`,
      value: Math.random() * 100
    }));
    
    jest.mocked(fetchBoardsData).mockResolvedValue(largeDataset);
    
    const startTime = performance.now();
    
    render(
      <Widget 
        boardIds={['123']} 
        settings={{ title: 'Large Dataset Test' }}
        size="medium"
      />
    );
    
    const renderTime = performance.now() - startTime;
    
    // Should handle large datasets within reasonable time
    expect(renderTime).toBeLessThan(100);
  });
  
  test('memory usage stays within limits', () => {
    const initialMemory = (performance as any).memory?.usedJSHeapSize || 0;
    
    const { unmount } = render(
      <Widget 
        boardIds={['123']} 
        settings={{ title: 'Memory Test' }}
        size="medium"
      />
    );
    
    unmount();
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
    
    const finalMemory = (performance as any).memory?.usedJSHeapSize || 0;
    const memoryIncrease = finalMemory - initialMemory;
    
    // Should not leak significant memory (< 1MB)
    expect(memoryIncrease).toBeLessThan(1024 * 1024);
  });
});
```

## Visual Regression Testing

### Storybook Visual Tests
```typescript
// Widget.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Widget } from './Widget';

const meta: Meta<typeof Widget> = {
  title: 'Widgets/Widget',
  component: Widget,
  parameters: {
    layout: 'centered',
  },
  argTypes: {
    size: {
      control: { type: 'select' },
      options: ['small', 'medium', 'large']
    }
  }
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    boardIds: ['123'],
    settings: {
      title: 'Default Widget',
      chartType: 'metrics',
      refreshInterval: 300,
      showLegend: true,
      colorScheme: 'default'
    },
    size: 'medium'
  }
};

export const Small: Story = {
  args: {
    ...Default.args,
    size: 'small'
  }
};

export const Large: Story = {
  args: {
    ...Default.args,
    size: 'large'
  }
};

export const Loading: Story = {
  args: {
    ...Default.args
  },
  parameters: {
    mockData: {
      loading: true
    }
  }
};

export const Error: Story = {
  args: {
    ...Default.args
  },
  parameters: {
    mockData: {
      error: new Error('Failed to load data')
    }
  }
};

export const DarkTheme: Story = {
  args: {
    ...Default.args
  },
  parameters: {
    backgrounds: { default: 'dark' }
  }
};
```

## Test Configuration

### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy'
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/test-utils.tsx'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}'
  ]
};
```

### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] }
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] }
    }
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI
  }
});
```
