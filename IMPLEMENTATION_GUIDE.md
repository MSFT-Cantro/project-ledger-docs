# Technical Debt Implementation Guide

This guide explains how to use the newly implemented technical debt solutions.

## 1. Error Boundaries

### Usage in Components

```tsx
import { ErrorBoundary } from '../components/common/ErrorBoundary';

// Page-level error boundary
<ErrorBoundary level="page">
  <YourPageComponent />
</ErrorBoundary>

// Component-level error boundary
<ErrorBoundary level="component" onError={(error, info) => console.log('Error:', error)}>
  <YourComponent />
</ErrorBoundary>

// Critical error boundary (for app-breaking errors)
<ErrorBoundary level="critical">
  <CriticalComponent />
</ErrorBoundary>
```

### Custom Fallback UI

```tsx
<ErrorBoundary fallback={<CustomErrorComponent />}>
  <YourComponent />
</ErrorBoundary>
```

## 2. Logging Service

### Basic Usage

```tsx
import { useLogger } from '../hooks/useLogger';

const MyComponent = () => {
  const { logInfo, logError, logUserAction } = useLogger();

  const handleButtonClick = () => {
    logUserAction('button_click', { buttonId: 'submit', page: 'invoice-create' });
    // Your logic here
  };

  const handleApiError = (error: Error) => {
    logError('API call failed', { endpoint: '/api/invoices' }, error);
  };

  return <button onClick={handleButtonClick}>Submit</button>;
};
```

### Direct Logger Usage

```tsx
import logger from '../services/logger';

// In service files or utilities
logger.info('User logged in', { userId: 123 });
logger.warn('Slow API response', { duration: 2000, endpoint: '/api/data' });
logger.error('Database connection failed', { host: 'localhost' }, error);
```

### API Call Logging

```tsx
import { useAPIMonitoring } from '../hooks/useMonitoring';

const MyComponent = () => {
  const { trackAPICall } = useAPIMonitoring();

  const fetchData = async () => {
    return trackAPICall('GET', '/api/invoices', async () => {
      const response = await fetch('/api/invoices');
      return response.json();
    });
  };
};
```

## 3. Monitoring and Performance

### Component Render Time

```tsx
import { useRenderTime } from '../hooks/useMonitoring';

const MyComponent = () => {
  useRenderTime('MyComponent'); // Automatically tracks render time
  
  return <div>Component content</div>;
};
```

### Custom Performance Metrics

```tsx
import { useMonitoring } from '../hooks/useMonitoring';

const MyComponent = () => {
  const { measureStart, measureEnd, reportCustomMetric } = useMonitoring();

  const handleComplexOperation = async () => {
    measureStart('complex_operation');
    
    // Your complex operation
    await doComplexStuff();
    
    const duration = measureEnd('complex_operation');
    if (duration && duration > 1000) {
      reportCustomMetric('slow_operation_count', 1);
    }
  };
};
```

### Health Checks

```tsx
import monitoring from '../services/monitoring';

// Add health check for external service
monitoring.addHealthCheck('api', async () => {
  try {
    const response = await fetch('/api/health');
    return response.ok;
  } catch {
    return false;
  }
});

// Get current health status
const healthStatus = monitoring.getHealthStatus();
```

## 4. Testing Guidelines

### Testing Components with Error Boundaries

```tsx
import { render, screen } from '@testing-library/react';
import { ErrorBoundary } from './ErrorBoundary';

const ThrowError = ({ shouldThrow = true }) => {
  if (shouldThrow) throw new Error('Test error');
  return <div>No error</div>;
};

test('handles errors gracefully', () => {
  render(
    <ErrorBoundary>
      <ThrowError />
    </ErrorBoundary>
  );
  
  expect(screen.getByText('Something went wrong')).toBeInTheDocument();
});
```

### Testing Logging

```tsx
import { renderHook } from '@testing-library/react';
import { useLogger } from './useLogger';

test('logs user actions', () => {
  const consoleSpy = jest.spyOn(console, 'info');
  const { result } = renderHook(() => useLogger());
  
  result.current.logUserAction('test_action');
  
  expect(consoleSpy).toHaveBeenCalledWith(
    expect.stringContaining('[INFO]'),
    'User Action',
    expect.objectContaining({ action: 'test_action' }),
    undefined
  );
});
```

## 5. Environment Configuration

### Development
- All log levels enabled (DEBUG, INFO, WARN, ERROR)
- Console transport active
- Error details shown in UI
- Performance monitoring active

### Production
- Only WARN and ERROR logs
- Remote transport for log aggregation
- Error details hidden from users
- Performance monitoring optimized

### Test
- All logging goes to console for debugging
- Mocked performance APIs
- Error boundaries testable

## 6. Best Practices

### Error Handling
1. Always wrap page-level components with error boundaries
2. Use component-level boundaries for complex components
3. Provide meaningful error context when logging
4. Use proper error types instead of generic errors

### Logging
1. Use structured logging with context objects
2. Log user actions for analytics and debugging
3. Log API calls with timing information
4. Avoid logging sensitive data (passwords, tokens)

### Performance
1. Monitor critical user flows
2. Set performance budgets and alert on violations
3. Track custom metrics for business-critical operations
4. Use health checks for external dependencies

### Testing
1. Test error boundary behavior
2. Test logging calls in important flows
3. Mock external dependencies consistently
4. Use performance mocks in tests

## 7. Migration Guide

### Replacing Console Statements

**Before:**
```tsx
console.log('User logged in:', user);
console.error('API failed:', error);
```

**After:**
```tsx
import { useLogger } from '../hooks/useLogger';

const { logInfo, logError } = useLogger();
logInfo('User logged in', { userId: user.id });
logError('API failed', { endpoint: '/api/login' }, error);
```

### Adding Error Boundaries to Existing Pages

**Before:**
```tsx
export const MyPage = () => {
  return <div>Page content</div>;
};
```

**After:**
```tsx
export const MyPage = () => {
  return (
    <ErrorBoundary level="page">
      <div>Page content</div>
    </ErrorBoundary>
  );
};
```

## 8. Monitoring Dashboard

The monitoring service automatically tracks:
- Page load times
- API response times
- Error rates
- User actions
- Component render times
- Health check status

Access health status programmatically:
```tsx
import monitoring from '../services/monitoring';

const healthStatus = monitoring.getHealthStatus();
console.log('Service health:', healthStatus);
```

This infrastructure provides a solid foundation for maintaining code quality and operational visibility as the application scales.
