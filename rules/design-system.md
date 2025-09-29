# Design System Integration - Monday Dashboard Widgets

## Monday Vibe Design System

### Required Dependencies
```json
{
  "dependencies": {
    "@vibe/core": "^2.0.0",
    "@vibe/style": "^2.0.0",
    "@vibe/icons": "^2.0.0"
  }
}
```

### Core Components Usage
```typescript
import { 
  Button, 
  TextField, 
  Flex, 
  Box,
  Text,
  Loader,
  Icon,
  Tooltip,
  Modal,
  Dropdown
} from '@vibe/core';

import { 
  spacing, 
  colors, 
  typography, 
  shadows,
  borderRadius 
} from '@vibe/style';
```

## Theme Integration

### Theme Provider Setup
```typescript
import { VibeProvider, VibeComponentsProvider } from '@vibe/core';

const WidgetApp: React.FC = () => {
  return (
    <VibeProvider>
      <VibeComponentsProvider>
        <Widget />
      </VibeComponentsProvider>
    </VibeProvider>
  );
};
```

### Theme Hook
```typescript
import { useTheme } from '@vibe/core';

const useWidgetTheme = () => {
  const theme = useTheme();
  
  return {
    isDark: theme === 'dark',
    colors: {
      primary: theme === 'dark' ? colors.primary_dark : colors.primary_light,
      background: theme === 'dark' ? colors.dark_background : colors.light_background,
      text: theme === 'dark' ? colors.dark_text : colors.light_text,
      border: theme === 'dark' ? colors.dark_border : colors.light_border
    },
    spacing,
    typography,
    shadows: theme === 'dark' ? shadows.dark : shadows.light
  };
};
```

## Component Styling Standards

### Widget Container
```typescript
import styled from 'styled-components';
import { Box } from '@vibe/core';
import { colors, spacing, borderRadius, shadows } from '@vibe/style';

const WidgetContainer = styled(Box)<{ size: WidgetSize; theme: 'light' | 'dark' }>`
  width: ${({ size }) => WIDGET_DIMENSIONS[size].width}px;
  height: ${({ size }) => WIDGET_DIMENSIONS[size].height}px;
  background-color: ${({ theme }) => 
    theme === 'dark' ? colors.dark_background : colors.light_background
  };
  border: 1px solid ${({ theme }) => 
    theme === 'dark' ? colors.dark_border : colors.light_border
  };
  border-radius: ${borderRadius.medium};
  box-shadow: ${({ theme }) => 
    theme === 'dark' ? shadows.dark.small : shadows.light.small
  };
  padding: ${spacing.medium};
  display: flex;
  flex-direction: column;
  overflow: hidden;
`;

const WidgetHeader = styled(Flex)`
  justify-content: space-between;
  align-items: center;
  margin-bottom: ${spacing.medium};
  padding-bottom: ${spacing.small};
  border-bottom: 1px solid ${({ theme }) => 
    theme === 'dark' ? colors.dark_border : colors.light_border
  };
`;

const WidgetContent = styled(Box)`
  flex: 1;
  overflow: auto;
`;
```

### Typography Standards
```typescript
import { Text } from '@vibe/core';
import { typography } from '@vibe/style';

// Widget Title
const WidgetTitle = styled(Text)`
  font-size: ${typography.fontSize.h3};
  font-weight: ${typography.fontWeight.bold};
  line-height: ${typography.lineHeight.h3};
  margin: 0;
`;

// Widget Subtitle
const WidgetSubtitle = styled(Text)`
  font-size: ${typography.fontSize.body};
  font-weight: ${typography.fontWeight.normal};
  color: ${({ theme }) => 
    theme === 'dark' ? colors.dark_text_secondary : colors.light_text_secondary
  };
`;

// Metric Value
const MetricValue = styled(Text)`
  font-size: ${typography.fontSize.h1};
  font-weight: ${typography.fontWeight.bold};
  line-height: 1;
`;

// Metric Label
const MetricLabel = styled(Text)`
  font-size: ${typography.fontSize.small};
  font-weight: ${typography.fontWeight.medium};
  text-transform: uppercase;
  letter-spacing: 0.5px;
`;
```

## Color System

### Color Palette
```typescript
export const widgetColors = {
  // Primary colors
  primary: {
    50: '#E6F3FF',
    100: '#BAE3FF',
    200: '#7CC4FA',
    300: '#47A3F3',
    400: '#2186EB',
    500: '#0073E6', // Main primary
    600: '#0062CC',
    700: '#004FB3',
    800: '#003D99',
    900: '#002266'
  },
  
  // Status colors
  success: {
    light: '#00C875',
    main: '#00A86B',
    dark: '#008B5A'
  },
  
  warning: {
    light: '#FDAB3D',
    main: '#FF9F40',
    dark: '#E8890B'
  },
  
  error: {
    light: '#FF6B6B',
    main: '#E74C3C',
    dark: '#C0392B'
  },
  
  // Chart colors
  chart: {
    series1: '#0073E6',
    series2: '#00C875',
    series3: '#FDAB3D',
    series4: '#9D50DD',
    series5: '#FF6B6B',
    series6: '#579BFC'
  }
};
```

### Color Usage Guidelines
```typescript
const useWidgetColors = (theme: 'light' | 'dark') => {
  return {
    // Background colors
    background: {
      primary: theme === 'dark' ? '#1F2937' : '#FFFFFF',
      secondary: theme === 'dark' ? '#374151' : '#F9FAFB',
      accent: theme === 'dark' ? '#4B5563' : '#F3F4F6'
    },
    
    // Text colors
    text: {
      primary: theme === 'dark' ? '#F9FAFB' : '#111827',
      secondary: theme === 'dark' ? '#D1D5DB' : '#6B7280',
      disabled: theme === 'dark' ? '#9CA3AF' : '#9CA3AF'
    },
    
    // Border colors
    border: {
      light: theme === 'dark' ? '#374151' : '#E5E7EB',
      medium: theme === 'dark' ? '#4B5563' : '#D1D5DB',
      strong: theme === 'dark' ? '#6B7280' : '#9CA3AF'
    }
  };
};
```

## Responsive Design

### Breakpoints
```typescript
export const breakpoints = {
  small: '320px',
  medium: '768px',
  large: '1024px',
  xlarge: '1440px'
};

export const mediaQueries = {
  small: `@media (min-width: ${breakpoints.small})`,
  medium: `@media (min-width: ${breakpoints.medium})`,
  large: `@media (min-width: ${breakpoints.large})`,
  xlarge: `@media (min-width: ${breakpoints.xlarge})`
};
```

### Widget Size Adaptation
```typescript
const ResponsiveWidget = styled(WidgetContainer)<{ size: WidgetSize }>`
  ${({ size }) => {
    switch (size) {
      case 'small':
        return css`
          .widget-title {
            font-size: ${typography.fontSize.h4};
          }
          .metric-value {
            font-size: ${typography.fontSize.h2};
          }
          .chart-container {
            height: 120px;
          }
        `;
      case 'medium':
        return css`
          .widget-title {
            font-size: ${typography.fontSize.h3};
          }
          .metric-value {
            font-size: ${typography.fontSize.h1};
          }
          .chart-container {
            height: 250px;
          }
        `;
      case 'large':
        return css`
          .widget-title {
            font-size: ${typography.fontSize.h2};
          }
          .metric-value {
            font-size: ${typography.fontSize.display};
          }
          .chart-container {
            height: 400px;
          }
        `;
    }
  }}
`;
```

## Icon System

### Icon Usage
```typescript
import { Icon } from '@vibe/core';
import { 
  Refresh, 
  Settings, 
  Download, 
  Filter,
  ChevronDown,
  Info,
  Alert,
  Check
} from '@vibe/icons';

const WidgetActions: React.FC = () => {
  return (
    <Flex gap={spacing.small}>
      <Button kind="tertiary" size="small">
        <Icon icon={Refresh} size="small" />
      </Button>
      <Button kind="tertiary" size="small">
        <Icon icon={Settings} size="small" />
      </Button>
      <Button kind="tertiary" size="small">
        <Icon icon={Download} size="small" />
      </Button>
    </Flex>
  );
};
```

### Custom Icon Components
```typescript
const StatusIcon: React.FC<{ status: 'success' | 'warning' | 'error' }> = ({ status }) => {
  const iconMap = {
    success: Check,
    warning: Alert,
    error: Alert
  };
  
  const colorMap = {
    success: widgetColors.success.main,
    warning: widgetColors.warning.main,
    error: widgetColors.error.main
  };
  
  return (
    <Icon 
      icon={iconMap[status]} 
      size="small" 
      color={colorMap[status]}
    />
  );
};
```

## Loading States

### Skeleton Components
```typescript
import { Skeleton } from '@vibe/core';

const WidgetSkeleton: React.FC<{ size: WidgetSize }> = ({ size }) => {
  const height = WIDGET_DIMENSIONS[size].height - 32; // Account for padding
  
  return (
    <WidgetContainer size={size}>
      <Flex direction="column" gap={spacing.medium}>
        <Skeleton width="60%" height="24px" />
        <Skeleton width="100%" height={`${height - 60}px`} />
      </Flex>
    </WidgetContainer>
  );
};

const MetricsSkeleton: React.FC = () => {
  return (
    <Flex direction="column" gap={spacing.medium}>
      {[1, 2, 3].map(i => (
        <Flex key={i} justify="space-between" align="center">
          <Skeleton width="40%" height="16px" />
          <Skeleton width="20%" height="24px" />
        </Flex>
      ))}
    </Flex>
  );
};
```

### Loading Spinner
```typescript
import { Loader } from '@vibe/core';

const WidgetLoader: React.FC<{ message?: string }> = ({ message = 'Loading...' }) => {
  return (
    <Flex 
      direction="column" 
      align="center" 
      justify="center" 
      gap={spacing.medium}
      style={{ height: '200px' }}
    >
      <Loader size="medium" />
      <Text size="small" color="secondary">
        {message}
      </Text>
    </Flex>
  );
};
```

## Interactive States

### Button States
```typescript
const WidgetButton = styled(Button)<{ variant?: 'primary' | 'secondary' | 'ghost' }>`
  ${({ variant = 'secondary' }) => {
    switch (variant) {
      case 'primary':
        return css`
          background-color: ${widgetColors.primary[500]};
          color: white;
          
          &:hover {
            background-color: ${widgetColors.primary[600]};
          }
          
          &:active {
            background-color: ${widgetColors.primary[700]};
          }
        `;
      case 'ghost':
        return css`
          background-color: transparent;
          border: 1px solid ${widgetColors.primary[300]};
          color: ${widgetColors.primary[500]};
          
          &:hover {
            background-color: ${widgetColors.primary[50]};
          }
        `;
    }
  }}
`;
```

### Focus States
```typescript
const FocusableElement = styled.div`
  &:focus {
    outline: 2px solid ${widgetColors.primary[500]};
    outline-offset: 2px;
    border-radius: ${borderRadius.small};
  }
  
  &:focus:not(:focus-visible) {
    outline: none;
  }
`;
```

## Animation Standards

### Transition Timing
```typescript
export const transitions = {
  fast: '150ms ease-in-out',
  medium: '250ms ease-in-out',
  slow: '350ms ease-in-out'
};

const AnimatedWidget = styled(WidgetContainer)`
  transition: all ${transitions.medium};
  
  &:hover {
    box-shadow: ${shadows.light.medium};
    transform: translateY(-2px);
  }
`;
```

### Loading Animations
```typescript
const PulseAnimation = keyframes`
  0% { opacity: 1; }
  50% { opacity: 0.5; }
  100% { opacity: 1; }
`;

const LoadingElement = styled.div`
  animation: ${PulseAnimation} 2s infinite;
`;
```

## Accessibility Integration

### ARIA Labels with Vibe
```typescript
const AccessibleWidget: React.FC<WidgetProps> = ({ title, data }) => {
  const titleId = useId();
  const descId = useId();
  
  return (
    <Box
      role="region"
      aria-labelledby={titleId}
      aria-describedby={descId}
      tabIndex={0}
    >
      <Text id={titleId} size="h3">
        {title}
      </Text>
      <Text id={descId} className="sr-only">
        Widget showing {data.length} items
      </Text>
      {/* Widget content */}
    </Box>
  );
};
```

### Screen Reader Support
```typescript
const ScreenReaderOnly = styled.span`
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
`;

const ChartWithScreenReader: React.FC<{ data: ChartData[] }> = ({ data }) => {
  return (
    <>
      <Chart data={data} />
      <ScreenReaderOnly>
        <table>
          <caption>Chart data</caption>
          <thead>
            <tr>
              <th>Category</th>
              <th>Value</th>
            </tr>
          </thead>
          <tbody>
            {data.map(item => (
              <tr key={item.id}>
                <td>{item.label}</td>
                <td>{item.value}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </ScreenReaderOnly>
    </>
  );
};
```
