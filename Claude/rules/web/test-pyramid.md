---
name: web-test-pyramid
description: >-
  Web-specific test pyramid implementation strategy. Defines the optimal balance of
  unit, integration, and end-to-end tests for web applications.
priority: CRITICAL
---

# Test Pyramid for Web Applications

> This file extends [common/test-pyramid.md](../common/test-pyramid.md) with web-specific guidelines.

## Pyramid Structure

```text
  E2E Tests (10%)  # Critical user flows, cross-system validation
     │
     │    Integration Tests (40%)  # API endpoints, component interactions
     │         │
     │         │    Unit Tests (50%)  # Components, utilities, hooks, functions
     │         │         │
     ▼         ▼         ▼
  More Time ─────────── Less Time
  Less Speed ────────── More Speed
  Higher Cost ───────── Lower Cost
```

- **Unit Tests (50%)**: Fast, isolated, reliable tests for individual units
- **Integration Tests (40%)**: Validate interactions between components and systems
- **E2E Tests (10%)**: Real user scenarios, browser automation, slower execution

## Test Distribution Strategy

### Unit Tests (50%)

Focus on:
- React components and their behavior
- Utility functions and data transformations
- Custom hooks and their state management
- Business logic and pure functions
- Type validation and edge cases

```tsx
// Component unit test example
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('displays loading state correctly', () => {
    render(<Button isLoading>Submit</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

```tsx
// Custom hook test example
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('increments count when increment is called', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

### Integration Tests (40%)

Focus on:
- API endpoints and route handlers
- Database interactions and queries
- Third-party service integrations
- Component composition and interactions
- State management across components
- Authentication and authorization flows

```tsx
// API integration test example
import { NextRequest } from 'next/server';
import { GET } from '@/app/api/users/route';

describe('GET /api/users', () => {
  it('returns users successfully', async () => {
    const request = new NextRequest('http://localhost/api/users');
    const response = await GET(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
    expect(Array.isArray(data.users)).toBe(true);
  });

  it('applies filters correctly', async () => {
    const request = new NextRequest('http://localhost/api/users?role=admin');
    const response = await GET(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    data.users.forEach(user => {
      expect(user.role).toBe('admin');
    });
  });
});
```

```tsx
// Component integration test
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserForm } from '@/components/UserForm';
import { UserList } from '@/components/UserList';

describe('UserForm and UserList integration', () => {
  it('adds new user to list when form is submitted', async () => {
    const user = userEvent.setup();
    
    render(\n      <>\n        <UserForm />\n        <UserList />\n      </>\n    );

    // Fill and submit form
    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // Verify user appears in list
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
});
```

### End-to-End Tests (10%)

Focus on:
- Critical user journeys (login, signup, checkout, etc.)
- Cross-browser compatibility
- Realistic user interactions
- Performance and loading behavior
- Accessibility requirements

```tsx
// E2E test with Playwright
import { test, expect } from '@playwright/test';

test('user can complete purchase flow', async ({ page }) => {
  // Navigate to home
  await page.goto('/');

  // Add item to cart
  await page.click('text=Add to Cart');
  await page.click('text=View Cart');

  // Verify cart contents
  await expect(page.locator('.cart-item')).toHaveCount(1);

  // Proceed to checkout
  await page.click('text=Checkout');

  // Fill shipping information
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="name"]', 'Test User');
  await page.fill('input[name="address"]', '123 Main St');
  await page.fill('input[name="city"]', 'Anytown');
  
  // Submit order
  await page.click('text=Place Order');

  // Verify success
  await expect(page.locator('text=Order confirmed')).toBeVisible();
  await expect(page).toHaveURL(/confirmation/);
});
```

```tsx
// Login flow E2E test
test('user can login and access dashboard', async ({ page }) => {
  // Navigate to login
  await page.goto('/login');

  // Fill login form
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('text=Log In');

  // Verify redirect to dashboard
  await expect(page).toHaveURL('/dashboard');
  
  // Verify dashboard elements
  await expect(page.locator('h1')).toContainText('Dashboard');
  await expect(page.getByRole('navigation')).toBeVisible();
});
```

## Testing Guidelines

### Test Selection Patterns

| Scenario | Test Type |
|---------|-----------|
| Component UI and interactions | Unit test |
| Data fetching and caching | Integration test |
| Business logic and validation | Unit test |
| API endpoint behavior | Integration test |
| Third-party service integration | Integration test |
| End-to-end user scenarios | E2E test |
| Cross-browser compatibility | E2E test |
| Performance benchmarks | E2E test |
| Accessibility compliance | E2E test |
| Complex state management | Integration test |

### Mocking Strategy

#### When to Mock

- External APIs and third-party services
- Authentication providers
- Payment processors
- Analytics and tracking scripts
- Email and notification services
- Background jobs and queues

#### When NOT to Mock

- Application state management
- Core business logic
- Component interactions within the same feature
- Local storage and cookies (test actual behavior)
- Routing within the application

#### Mocking Implementation

```tsx
// Mock external service
type MockServiceResponse = {
  data: any;
  error: string | null;
};

const mockService = {
  fetchData: jest.fn<() => Promise<MockServiceResponse>>(()
    Promise.resolve({ data: [{ id: 1, name: 'Test' }], error: null })
  ),
  updateData: jest.fn(),
  deleteData: jest.fn()
};

// In test setup
beforeEach(() => {
  jest.resetAllMocks();
  // Set default mock behavior
  mockService.fetchData.mockResolvedValue({
    data: [{ id: 1, name: 'Test Item' }],
    error: null
  });
});
```

```tsx
// Mock API route handler
jest.mock('@/lib/api-client', () => ({
  fetchUsers: jest.fn(() => Promise.resolve([
    { id: 1, name: 'John Doe', email: 'john@example.com' }
  ])),
  createUser: jest.fn()
}));

// Mock database
jest.mock('@/lib/db', () => ({
  query: jest.fn(),
  connect: jest.fn()
}));
```

## Testing Tools and Frameworks

| Test Type | Recommended Tools |
|-----------|-------------------|
| Unit Tests | Jest, Vitest, React Testing Library |
| Integration Tests | Jest, Vitest, Supertest, React Testing Library |
| E2E Tests | Playwright, Cypress |
| Visual Regression | Playwright, Percy, Chromatic |
| Performance Tests | Lighthouse, WebPageTest |
| Accessibility Tests | axe, Pa11y, Lighthouse |

## Test Organization

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # Unit test
│   │   └── Button.stories.tsx
│   └── Form/
│       ├── Form.tsx
│       ├── Form.test.tsx
│       └── __tests__/
│           ├── Form.integration.test.tsx  # Integration test
│           └── Form.validation.test.tsx     # Unit test
├── pages/
│   ├── api/
│   │   ├── users/
│   │   │   ├── route.ts
│   │   │   └── route.test.ts              # Integration test
│   │   └── products/
│   │       ├── route.ts
│   │       └── route.test.ts
│   └── __e2e__/
│       ├── login.test.ts              # E2E test
│       └── checkout.test.ts           # E2E test
└── lib/
    ├── api/
    │   ├── client.ts
    │   └── client.test.ts               # Unit test
    └── auth/
        ├── index.ts
        └── auth.test.ts                 # Unit test
```

## Performance Considerations

- Unit tests should run in < 50ms each
- Integration tests should run in < 500ms each
- E2E tests should run in < 10s each (aim for < 5s)
- Full test suite should complete in < 2 minutes
- Use test parallelization when possible
- Mock slow operations in unit and integration tests

## Continuous Integration

```yaml
# GitHub Actions workflow
name: Test Suite

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: 18
       
    - name: Install dependencies
      run: npm ci
      
    - name: Run unit tests
      run: npm run test:unit
      
    - name: Run integration tests
      run: npm run test:integration
      
    - name: Run E2E tests
      run: npm run test:e2e
      
    - name: Generate coverage report
      run: npm run test:coverage
      
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
```

## Testing Checklist

### Unit Tests

- [ ] Test individual components in isolation
- [ ] Verify component props and state
- [ ] Test user interactions (clicks, input, etc.)
- [ ] Verify appropriate ARIA attributes
- [ ] Test loading and error states
- [ ] Use appropriate testing library utilities
- [ ] Mock external dependencies
- [ ] Keep tests focused and fast
- [ ] Test edge cases and error handling

### Integration Tests

- [ ] Test API endpoints and route handlers
- [ ] Verify database interactions
- [ ] Test component composition
- [ ] Validate state management
- [ ] Test authentication flows
- [ ] Mock external services
- [ ] Test error handling and edge cases
- [ ] Verify data transformations
- [ ] Test caching behavior
- [ ] Validate security measures

### E2E Tests

- [ ] Test critical user journeys
- [ ] Verify navigation and routing
- [ ] Test form submissions
- [ ] Validate data persistence
- [ ] Test error scenarios
- [ ] Verify loading states
- [ ] Test accessibility requirements
- [ ] Validate performance metrics
- [ ] Test cross-browser compatibility
- [ ] Include visual regression checks

## Success Metrics

- Test suite runs in under 2 minutes
- 80%+ code coverage achieved
- No flaky tests (100% pass rate)
- Critical user flows covered by E2E tests
- Each bug fix includes a regression test
- New features include appropriate test coverage
- Test pyramid ratio maintained (50/40/10)