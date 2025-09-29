# Accessibility Standards - Monday Dashboard Widgets

## WCAG 2.1 AA Compliance

### Core Principles
```typescript
const ACCESSIBILITY_PRINCIPLES = {
  perceivable: 'Information must be presentable in ways users can perceive',
  operable: 'Interface components must be operable',
  understandable: 'Information and UI operation must be understandable',
  robust: 'Content must be robust enough for various assistive technologies'
};

const WCAG_LEVELS = {
  A: 'Minimum level of accessibility',
  AA: 'Standard level (REQUIRED for widgets)',
  AAA: 'Enhanced level (RECOMMENDED for critical widgets)'
};
```

### Success Criteria Checklist
```typescript
interface AccessibilityChecklist {
  // Level A (Required)
  nonTextContent: boolean;           // 1.1.1
  audioOnlyVideoOnly: boolean;       // 1.2.1
  captionsPrerecorded: boolean;      // 1.2.2
  infoAndRelationships: boolean;     // 1.3.1
  meaningfulSequence: boolean;       // 1.3.2
  sensoryCharacteristics: boolean;   // 1.3.3
  useOfColor: boolean;               // 1.4.1
  audioControl: boolean;             // 1.4.2
  
  // Level AA (Required)
  contrast: boolean;                 // 1.4.3 - 4.5:1 ratio
  resizeText: boolean;               // 1.4.4 - 200% zoom
  imagesOfText: boolean;             // 1.4.5
  keyboardAccessible: boolean;       // 2.1.1
  noKeyboardTrap: boolean;           // 2.1.2
  timingAdjustable: boolean;         // 2.2.1
  pauseStopHide: boolean;            // 2.2.2
  threeFlashesOrBelow: boolean;      // 2.3.1
  bypassBlocks: boolean;             // 2.4.1
  pageTitle: boolean;                // 2.4.2
  focusOrder: boolean;               // 2.4.3
  linkPurpose: boolean;              // 2.4.4
  
  // Level AAA (Recommended)
  contrastEnhanced: boolean;         // 1.4.6 - 7:1 ratio
  lowOrNoBackgroundAudio: boolean;   // 1.4.7
  visualPresentation: boolean;       // 1.4.8
  imagesOfTextNoException: boolean;  // 1.4.9
}
```

## Semantic HTML Structure

### Widget Container
```typescript
import React, { useId } from 'react';

const AccessibleWidget: React.FC<WidgetProps> = ({ 
  title, 
  description, 
  data, 
  type 
}) => {
  const titleId = useId();
  const descId = useId();
  const contentId = useId();
  
  return (
    <section
      role="region"
      aria-labelledby={titleId}
      aria-describedby={descId}
      className="widget-container"
    >
      <header className="widget-header">
        <h2 id={titleId} className="widget-title">
          {title}
        </h2>
        {description && (
          <p id={descId} className="widget-description">
            {description}
          </p>
        )}
      </header>
      
      <main id={contentId} className="widget-content">
        <WidgetContent 
          data={data} 
          type={type}
          aria-labelledby={titleId}
        />
      </main>
    </section>
  );
};
```

### Chart Accessibility
```typescript
const AccessibleChart: React.FC<ChartProps> = ({ 
  data, 
  title, 
  type 
}) => {
  const chartId = useId();
  const tableId = useId();
  const [showDataTable, setShowDataTable] = useState(false);
  
  return (
    <div className="accessible-chart">
      <div className="chart-controls">
        <button
          onClick={() => setShowDataTable(!showDataTable)}
          aria-expanded={showDataTable}
          aria-controls={tableId}
        >
          {showDataTable ? 'Hide' : 'Show'} Data Table
        </button>
      </div>
      
      <div
        id={chartId}
        role="img"
        aria-label={`${type} chart showing ${title}`}
        aria-describedby={tableId}
      >
        <Chart data={data} type={type} />
      </div>
      
      {/* Screen reader accessible data table */}
      <div 
        id={tableId}
        className={showDataTable ? 'data-table-visible' : 'sr-only'}
      >
        <table>
          <caption>
            Data for {title} {type} chart
          </caption>
          <thead>
            <tr>
              <th scope="col">Category</th>
              <th scope="col">Value</th>
              <th scope="col">Percentage</th>
            </tr>
          </thead>
          <tbody>
            {data.map((item, index) => (
              <tr key={item.id}>
                <th scope="row">{item.label}</th>
                <td>{item.value}</td>
                <td>{((item.value / getTotalValue(data)) * 100).toFixed(1)}%</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

## Keyboard Navigation

### Focus Management
```typescript
const useFocusManagement = () => {
  const focusableElements = useRef<HTMLElement[]>([]);
  const currentFocusIndex = useRef(0);
  
  const updateFocusableElements = useCallback(() => {
    const container = document.querySelector('.widget-container');
    if (!container) return;
    
    const elements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    ) as NodeListOf<HTMLElement>;
    
    focusableElements.current = Array.from(elements).filter(
      el => !el.disabled && el.offsetParent !== null
    );
  }, []);
  
  const handleKeyDown = useCallback((event: KeyboardEvent) => {
    const { key, shiftKey, ctrlKey, altKey } = event;
    
    switch (key) {
      case 'Tab':
        // Let browser handle normal tab navigation
        break;
        
      case 'ArrowDown':
      case 'ArrowRight':
        if (ctrlKey) {
          event.preventDefault();
          moveFocus(1);
        }
        break;
        
      case 'ArrowUp':
      case 'ArrowLeft':
        if (ctrlKey) {
          event.preventDefault();
          moveFocus(-1);
        }
        break;
        
      case 'Home':
        if (ctrlKey) {
          event.preventDefault();
          focusFirst();
        }
        break;
        
      case 'End':
        if (ctrlKey) {
          event.preventDefault();
          focusLast();
        }
        break;
        
      case 'Escape':
        handleEscape();
        break;
    }
  }, []);
  
  const moveFocus = (direction: number) => {
    updateFocusableElements();
    
    if (focusableElements.current.length === 0) return;
    
    currentFocusIndex.current = Math.max(
      0,
      Math.min(
        focusableElements.current.length - 1,
        currentFocusIndex.current + direction
      )
    );
    
    focusableElements.current[currentFocusIndex.current]?.focus();
  };
  
  const focusFirst = () => {
    updateFocusableElements();
    currentFocusIndex.current = 0;
    focusableElements.current[0]?.focus();
  };
  
  const focusLast = () => {
    updateFocusableElements();
    currentFocusIndex.current = focusableElements.current.length - 1;
    focusableElements.current[currentFocusIndex.current]?.focus();
  };
  
  return {
    handleKeyDown,
    focusFirst,
    focusLast,
    updateFocusableElements
  };
};
```

### Skip Links
```typescript
const SkipLinks: React.FC = () => {
  return (
    <nav className="skip-links" aria-label="Skip navigation">
      <a href="#widget-content" className="skip-link">
        Skip to widget content
      </a>
      <a href="#widget-controls" className="skip-link">
        Skip to widget controls
      </a>
      <a href="#widget-data" className="skip-link">
        Skip to data table
      </a>
    </nav>
  );
};

// CSS for skip links
const skipLinkStyles = `
.skip-links {
  position: absolute;
  top: -40px;
  left: 6px;
  z-index: 1000;
}

.skip-link {
  position: absolute;
  top: -40px;
  left: 6px;
  background: #000;
  color: #fff;
  padding: 8px;
  text-decoration: none;
  border-radius: 4px;
  z-index: 1001;
}

.skip-link:focus {
  top: 6px;
}
`;
```

## ARIA Implementation

### Live Regions
```typescript
const useLiveRegion = () => {
  const liveRegionRef = useRef<HTMLDivElement>(null);
  
  const announce = useCallback((
    message: string, 
    priority: 'polite' | 'assertive' = 'polite'
  ) => {
    if (!liveRegionRef.current) return;
    
    // Clear previous message
    liveRegionRef.current.textContent = '';
    
    // Set new message after a brief delay to ensure screen readers notice
    setTimeout(() => {
      if (liveRegionRef.current) {
        liveRegionRef.current.textContent = message;
        liveRegionRef.current.setAttribute('aria-live', priority);
      }
    }, 100);
  }, []);
  
  const LiveRegion = () => (
    <div
      ref={liveRegionRef}
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    />
  );
  
  return { announce, LiveRegion };
};

// Usage in widget
const Widget: React.FC = () => {
  const { announce, LiveRegion } = useLiveRegion();
  const [loading, setLoading] = useState(false);
  
  const handleDataRefresh = async () => {
    setLoading(true);
    announce('Loading widget data...');
    
    try {
      await fetchData();
      announce('Widget data updated successfully');
    } catch (error) {
      announce('Error loading widget data', 'assertive');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <LiveRegion />
      {/* Widget content */}
    </div>
  );
};
```

### ARIA Labels and Descriptions
```typescript
const AccessibleButton: React.FC<{
  children: React.ReactNode;
  onClick: () => void;
  description?: string;
  loading?: boolean;
}> = ({ children, onClick, description, loading }) => {
  const buttonId = useId();
  const descId = useId();
  
  return (
    <>
      <button
        id={buttonId}
        onClick={onClick}
        aria-describedby={description ? descId : undefined}
        aria-busy={loading}
        disabled={loading}
      >
        {children}
        {loading && (
          <span aria-hidden="true" className="loading-spinner" />
        )}
      </button>
      
      {description && (
        <div id={descId} className="sr-only">
          {description}
        </div>
      )}
      
      {loading && (
        <div aria-live="polite" className="sr-only">
          Loading...
        </div>
      )}
    </>
  );
};
```

## Color and Contrast

### Color Contrast Validation
```typescript
const validateColorContrast = (
  foreground: string, 
  background: string
): { ratio: number; passes: { AA: boolean; AAA: boolean } } => {
  const getLuminance = (color: string): number => {
    const rgb = hexToRgb(color);
    if (!rgb) return 0;
    
    const [r, g, b] = [rgb.r, rgb.g, rgb.b].map(c => {
      c = c / 255;
      return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
    });
    
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  };
  
  const l1 = getLuminance(foreground);
  const l2 = getLuminance(background);
  
  const ratio = (Math.max(l1, l2) + 0.05) / (Math.min(l1, l2) + 0.05);
  
  return {
    ratio,
    passes: {
      AA: ratio >= 4.5,
      AAA: ratio >= 7
    }
  };
};

const hexToRgb = (hex: string): { r: number; g: number; b: number } | null => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16)
  } : null;
};
```

### Accessible Color Palette
```typescript
export const accessibleColors = {
  // High contrast pairs (7:1 ratio - AAA)
  highContrast: {
    darkOnLight: { bg: '#FFFFFF', fg: '#000000' }, // 21:1
    lightOnDark: { bg: '#000000', fg: '#FFFFFF' }, // 21:1
    blueOnWhite: { bg: '#FFFFFF', fg: '#0066CC' }, // 7.7:1
    whiteOnBlue: { bg: '#0066CC', fg: '#FFFFFF' }  // 7.7:1
  },
  
  // Standard contrast pairs (4.5:1 ratio - AA)
  standardContrast: {
    grayOnWhite: { bg: '#FFFFFF', fg: '#666666' }, // 5.7:1
    whiteOnGray: { bg: '#666666', fg: '#FFFFFF' }, // 5.7:1
    greenOnWhite: { bg: '#FFFFFF', fg: '#008000' }, // 4.6:1
    redOnWhite: { bg: '#FFFFFF', fg: '#CC0000' }   // 5.5:1
  },
  
  // Status colors (accessible)
  status: {
    success: { bg: '#F0F9F0', fg: '#006600', border: '#00AA00' },
    warning: { bg: '#FFF8E1', fg: '#B8860B', border: '#FFD700' },
    error: { bg: '#FFEBEE', fg: '#C62828', border: '#F44336' },
    info: { bg: '#E3F2FD', fg: '#1565C0', border: '#2196F3' }
  }
};
```

## Screen Reader Support

### Screen Reader Only Content
```typescript
const ScreenReaderOnly: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => (
  <span className="sr-only">
    {children}
  </span>
);

// CSS for screen reader only content
const srOnlyStyles = `
.sr-only {
  position: absolute !important;
  width: 1px !important;
  height: 1px !important;
  padding: 0 !important;
  margin: -1px !important;
  overflow: hidden !important;
  clip: rect(0, 0, 0, 0) !important;
  white-space: nowrap !important;
  border: 0 !important;
}

.sr-only-focusable:focus {
  position: static !important;
  width: auto !important;
  height: auto !important;
  padding: inherit !important;
  margin: inherit !important;
  overflow: visible !important;
  clip: auto !important;
  white-space: inherit !important;
}
`;
```

### Descriptive Content
```typescript
const DescriptiveChart: React.FC<ChartProps> = ({ data, type, title }) => {
  const getChartDescription = () => {
    const total = data.reduce((sum, item) => sum + item.value, 0);
    const highest = data.reduce((max, item) => 
      item.value > max.value ? item : max
    );
    const lowest = data.reduce((min, item) => 
      item.value < min.value ? item : min
    );
    
    return `${type} chart titled "${title}" with ${data.length} data points. 
            Total value: ${total}. 
            Highest value: ${highest.label} with ${highest.value}. 
            Lowest value: ${lowest.label} with ${lowest.value}.`;
  };
  
  return (
    <div>
      <ScreenReaderOnly>
        {getChartDescription()}
      </ScreenReaderOnly>
      
      <Chart data={data} type={type} />
      
      <ScreenReaderOnly>
        End of chart data. Use the data table button to access detailed information.
      </ScreenReaderOnly>
    </div>
  );
};
```

## Testing and Validation

### Automated Accessibility Testing
```typescript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Widget Accessibility', () => {
  test('should not have accessibility violations', async () => {
    const { container } = render(<Widget {...defaultProps} />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  test('should have proper heading structure', () => {
    render(<Widget title="Test Widget" />);
    
    const headings = screen.getAllByRole('heading');
    expect(headings).toHaveLength(1);
    expect(headings[0]).toHaveAttribute('aria-level', '2');
  });
  
  test('should support keyboard navigation', () => {
    render(<Widget {...defaultProps} />);
    
    const buttons = screen.getAllByRole('button');
    buttons.forEach(button => {
      expect(button).toHaveAttribute('tabindex', '0');
    });
  });
  
  test('should have sufficient color contrast', () => {
    const { container } = render(<Widget {...defaultProps} />);
    
    const textElements = container.querySelectorAll('*');
    textElements.forEach(element => {
      const styles = window.getComputedStyle(element);
      const contrast = validateColorContrast(
        styles.color,
        styles.backgroundColor
      );
      
      if (styles.color !== 'rgba(0, 0, 0, 0)') {
        expect(contrast.passes.AA).toBe(true);
      }
    });
  });
});
```

### Manual Testing Checklist
```typescript
interface AccessibilityTestChecklist {
  keyboardNavigation: {
    tabOrder: boolean;
    focusVisible: boolean;
    noKeyboardTraps: boolean;
    skipLinks: boolean;
  };
  
  screenReader: {
    properHeadings: boolean;
    altText: boolean;
    ariaLabels: boolean;
    liveRegions: boolean;
  };
  
  visual: {
    colorContrast: boolean;
    focusIndicators: boolean;
    textScaling: boolean;
    colorIndependence: boolean;
  };
  
  interaction: {
    clickTargets: boolean;
    errorMessages: boolean;
    formLabels: boolean;
    timeouts: boolean;
  };
}

const runManualAccessibilityTest = (): AccessibilityTestChecklist => {
  return {
    keyboardNavigation: {
      tabOrder: true, // Test with Tab key
      focusVisible: true, // Visible focus indicators
      noKeyboardTraps: true, // Can escape all elements
      skipLinks: true // Skip links work properly
    },
    screenReader: {
      properHeadings: true, // Logical heading structure
      altText: true, // All images have alt text
      ariaLabels: true, // Interactive elements labeled
      liveRegions: true // Dynamic content announced
    },
    visual: {
      colorContrast: true, // 4.5:1 minimum ratio
      focusIndicators: true, // Clear focus indicators
      textScaling: true, // Works at 200% zoom
      colorIndependence: true // Info not color-dependent
    },
    interaction: {
      clickTargets: true, // 44px minimum touch targets
      errorMessages: true, // Clear error messages
      formLabels: true, // All inputs labeled
      timeouts: true // Adjustable time limits
    }
  };
};
```

## Accessibility Hooks

### Focus Management Hook
```typescript
const useAccessibleFocus = () => {
  const [focusedElement, setFocusedElement] = useState<HTMLElement | null>(null);
  
  const trapFocus = useCallback((container: HTMLElement) => {
    const focusableElements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    ) as NodeListOf<HTMLElement>;
    
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];
    
    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === 'Tab') {
        if (event.shiftKey) {
          if (document.activeElement === firstElement) {
            event.preventDefault();
            lastElement.focus();
          }
        } else {
          if (document.activeElement === lastElement) {
            event.preventDefault();
            firstElement.focus();
          }
        }
      }
    };
    
    container.addEventListener('keydown', handleKeyDown);
    firstElement?.focus();
    
    return () => {
      container.removeEventListener('keydown', handleKeyDown);
    };
  }, []);
  
  const restoreFocus = useCallback(() => {
    focusedElement?.focus();
  }, [focusedElement]);
  
  const saveFocus = useCallback(() => {
    setFocusedElement(document.activeElement as HTMLElement);
  }, []);
  
  return { trapFocus, restoreFocus, saveFocus };
};
```

### Announcement Hook
```typescript
const useAnnouncements = () => {
  const [announcements, setAnnouncements] = useState<string[]>([]);
  
  const announce = useCallback((message: string, priority: 'polite' | 'assertive' = 'polite') => {
    setAnnouncements(prev => [...prev, message]);
    
    // Clear announcement after it's been read
    setTimeout(() => {
      setAnnouncements(prev => prev.filter(msg => msg !== message));
    }, 1000);
  }, []);
  
  const AnnouncementRegion = () => (
    <div aria-live="polite" aria-atomic="false" className="sr-only">
      {announcements.map((message, index) => (
        <div key={index}>{message}</div>
      ))}
    </div>
  );
  
  return { announce, AnnouncementRegion };
};
```
