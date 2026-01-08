# 🎭 Playwright Testing - Developer Workflow Guide

> Complete guide to get started with Playwright E2E testing for modern web applications

## Table of Contents

- [Getting Started](#-getting-started)
- [Installation & Setup](#-installation--setup)
- [Project Structure](#-project-structure)
- [Writing Tests](#-writing-tests)
- [Running Tests](#-running-tests)
- [Developer Tools](#-developer-tools)
- [Best Practices](#-best-practices)
- [CI/CD Integration](#-cicd-integration)
- [Troubleshooting](#-troubleshooting)

---

## Getting Started

### What is Playwright?

Playwright is a modern end-to-end testing framework that allows you to test web applications across multiple browsers (Chromium, Firefox, WebKit) with a single API.

### Why Playwright?

- ✅ **Cross-Browser Testing** - Test on Chromium, Firefox, and WebKit
- ✅ **Auto-Wait** - No more flaky tests with artificial timeouts
- ✅ **Fast Execution** - Parallel test execution and browser contexts
- ✅ **Great Developer Tools** - Codegen, Inspector, Trace Viewer
- ✅ **Web-First Assertions** - Assertions designed for the dynamic web
- ✅ **Network Control** - Mock APIs and intercept requests
- ✅ **Mobile Testing** - Device emulation for responsive testing

---

##  Installation & Setup

### Prerequisites

- **Node.js** version 18 or higher
- **npm** or **yarn** package manager

### Step 1: Initialize Playwright

```bash
# Create a new project or navigate to existing project
cd your-project

# Initialize Playwright (recommended)
npm init playwright@latest
```

The initialization wizard will prompt you for:
- Where to put your tests (default: `tests/` or `e2e/`)
- Add GitHub Actions workflow? (Yes/No)
- Install browsers? (Yes)

### Step 2: Manual Installation (Alternative)

```bash
# Install Playwright
npm install -D @playwright/test

# Install browsers
npx playwright install

# Install system dependencies (Linux only)
npx playwright install-deps
```

### Step 3: Verify Installation

```bash
# Run example tests
npx playwright test

# Open HTML test report
npx playwright show-report
```

---

## Project Structure

After installation, your project will look like this:

```
your-project/
├── tests/                          # Test directory
│   ├── example.spec.ts            # Example test file
│   └── auth.setup.ts              # Authentication setup (optional)
├── playwright.config.ts           # Playwright configuration
├── package.json
└── playwright-report/             # HTML reports (generated)
    └── index.html
```

### Configuration File: `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Test directory
  testDir: './tests',
  
  // Maximum time one test can run
  timeout: 30 * 1000,
  
  // Run tests in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only
  forbidOnly: !!process.env.CI,
  
  // Retry on CI only
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter to use
  reporter: 'html',
  
  // Shared settings for all projects
  use: {
    // Base URL for navigation
    baseURL: 'http://localhost:3000',
    
    // Collect trace when retrying failed test
    trace: 'on-first-retry',
    
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Video on failure
    video: 'retain-on-failure',
  },

  // Configure projects for major browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    // Mobile browsers
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 13'] },
    },
  ],

  // Run your local dev server before starting tests
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

##  Writing Tests

### Test File Structure

Create test files with `.spec.ts` or `.spec.js` extension:

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';

test('basic test example', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});
```

### Example 1: Basic Navigation & Assertions

```typescript
// tests/homepage.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Homepage Tests', () => {
  test('should load homepage successfully', async ({ page }) => {
    // Navigate to homepage
    await page.goto('/');
    
    // Check page title
    await expect(page).toHaveTitle('My Awesome App');
    
    // Check main heading is visible
    await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
    
    // Check URL
    await expect(page).toHaveURL('/');
  });
  
  test('should navigate to about page', async ({ page }) => {
    await page.goto('/');
    
    // Click navigation link
    await page.getByRole('link', { name: 'About' }).click();
    
    // Verify navigation
    await expect(page).toHaveURL('/about');
    await expect(page.getByRole('heading', { name: 'About Us' })).toBeVisible();
  });
});
```

### Example 2: Form Interactions

```typescript
// tests/contact-form.spec.ts
import { test, expect } from '@playwright/test';

test('submit contact form', async ({ page }) => {
  await page.goto('/contact');
  
  // Fill form fields
  await page.getByLabel('Name').fill('John Doe');
  await page.getByLabel('Email').fill('john@example.com');
  await page.getByLabel('Message').fill('Hello, this is a test message!');
  
  // Select from dropdown
  await page.getByLabel('Subject').selectOption('general');
  
  // Check checkbox
  await page.getByLabel('Subscribe to newsletter').check();
  
  // Submit form
  await page.getByRole('button', { name: 'Send Message' }).click();
  
  // Verify success message
  await expect(page.getByText('Message sent successfully')).toBeVisible();
});
```

### Example 3: Login Flow

```typescript
// tests/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('successful login', async ({ page }) => {
    await page.goto('/login');
    
    // Fill login form
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('SecurePassword123!');
    
    // Click login button
    await page.getByRole('button', { name: 'Login' }).click();
    
    // Wait for redirect
    await expect(page).toHaveURL('/dashboard');
    
    // Verify user is logged in
    await expect(page.getByText('Welcome back, User!')).toBeVisible();
  });
  
  test('login with invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('invalid@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Login' }).click();
    
    // Verify error message
    await expect(page.getByText('Invalid email or password')).toBeVisible();
    
    // Should stay on login page
    await expect(page).toHaveURL('/login');
  });
});
```

### Example 4: E-Commerce Flow

```typescript
// tests/checkout.spec.ts
import { test, expect } from '@playwright/test';

test('complete purchase flow', async ({ page }) => {
  // Login first
  await page.goto('/login');
  await page.getByLabel('Email').fill('customer@test.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Login' }).click();
  
  // Browse products
  await page.goto('/products');
  
  // Search for product
  await page.getByPlaceholder('Search products').fill('laptop');
  await page.getByRole('button', { name: 'Search' }).click();
  
  // Select product
  await page.getByRole('link', { name: 'MacBook Pro' }).click();
  
  // Add to cart
  await page.getByRole('button', { name: 'Add to Cart' }).click();
  await expect(page.getByText('Added to cart')).toBeVisible();
  
  // Go to cart
  await page.getByRole('link', { name: 'Cart' }).click();
  
  // Verify cart item
  await expect(page.getByText('MacBook Pro')).toBeVisible();
  
  // Proceed to checkout
  await page.getByRole('button', { name: 'Checkout' }).click();
  
  // Fill shipping information
  await page.getByLabel('Full Name').fill('John Doe');
  await page.getByLabel('Address').fill('123 Main Street');
  await page.getByLabel('City').fill('Casablanca');
  await page.getByLabel('Postal Code').fill('20000');
  
  // Complete order
  await page.getByRole('button', { name: 'Place Order' }).click();
  
  // Verify success
  await expect(page).toHaveURL(/.*order-confirmation/);
  await expect(page.getByText('Order placed successfully!')).toBeVisible();
});
```

### Example 5: API Testing

```typescript
// tests/api.spec.ts
import { test, expect } from '@playwright/test';

test('API - get users list', async ({ request }) => {
  const response = await request.get('/api/users');
  
  // Check status code
  expect(response.status()).toBe(200);
  
  // Parse JSON response
  const users = await response.json();
  
  // Verify data structure
  expect(users).toHaveLength(10);
  expect(users[0]).toHaveProperty('id');
  expect(users[0]).toHaveProperty('email');
});

test('API - create new user', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: {
      name: 'Test User',
      email: 'test@example.com',
    }
  });
  
  expect(response.status()).toBe(201);
  
  const user = await response.json();
  expect(user.name).toBe('Test User');
  expect(user.email).toBe('test@example.com');
});
```

### Example 6: Network Mocking

```typescript
// tests/mocking.spec.ts
import { test, expect } from '@playwright/test';

test('mock API response', async ({ page }) => {
  // Mock API endpoint
  await page.route('**/api/products', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'Mocked Product', price: 99.99 }
      ])
    });
  });
  
  await page.goto('/products');
  
  // Verify mocked data is displayed
  await expect(page.getByText('Mocked Product')).toBeVisible();
  await expect(page.getByText('$99.99')).toBeVisible();
});
```

### Common Locators (Selectors)

```typescript
// Recommended: Role-based selectors (accessible)
page.getByRole('button', { name: 'Submit' })
page.getByRole('link', { name: 'Home' })
page.getByRole('textbox', { name: 'Email' })

// By label (forms)
page.getByLabel('Password')
page.getByLabel('Remember me')

// By placeholder
page.getByPlaceholder('Enter your email')

// By text content
page.getByText('Welcome back')
page.getByText(/Welcome .*/i) // Regex

// By test ID (recommended for complex UIs)
page.getByTestId('submit-button')

// CSS selectors (use sparingly)
page.locator('.btn-primary')
page.locator('#login-form')

// XPath (use sparingly)
page.locator('xpath=//button[@type="submit"]')
```

### Test Hooks

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Dashboard', () => {
  // Runs before each test in the describe block
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@test.com');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Login' }).click();
    await expect(page).toHaveURL('/dashboard');
  });
  
  // Runs after each test
  test.afterEach(async ({ page }) => {
    // Cleanup or logout
    await page.getByRole('button', { name: 'Logout' }).click();
  });
  
  test('view profile', async ({ page }) => {
    await page.getByRole('link', { name: 'Profile' }).click();
    await expect(page.getByText('User Profile')).toBeVisible();
  });
  
  test('view settings', async ({ page }) => {
    await page.getByRole('link', { name: 'Settings' }).click();
    await expect(page.getByText('Account Settings')).toBeVisible();
  });
});
```

---

##  Running Tests

### Basic Commands

```bash
# Run all tests
npx playwright test

# Run specific test file
npx playwright test tests/login.spec.ts

# Run tests in specific browser
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# Run tests in headed mode (see the browser)
npx playwright test --headed

# Run tests in debug mode
npx playwright test --debug

# Run specific test by name
npx playwright test -g "should login successfully"

# Run tests in a specific directory
npx playwright test tests/auth/
```

### Interactive UI Mode

```bash
# Open UI mode (recommended for development)
npx playwright test --ui
```

UI mode features:
- Watch mode - tests re-run on file changes
- Time travel debugging
- DOM snapshots
- Network activity
- Console logs

### Generate Test Report

```bash
# Generate and open HTML report
npx playwright show-report

# Generate report without opening
npx playwright test --reporter=html
```

### Parallel Execution

```bash
# Run tests in parallel (default)
npx playwright test

# Specify number of workers
npx playwright test --workers=4

# Run tests serially (one at a time)
npx playwright test --workers=1
```

---

## 🛠️ Developer Tools

### 1. Codegen - Test Generator

Generate tests by recording your browser actions:

```bash
# Basic usage
npx playwright codegen

# With specific URL
npx playwright codegen https://example.com

# With specific browser
npx playwright codegen --browser=firefox https://example.com

# Save to file
npx playwright codegen --target=typescript -o tests/generated.spec.ts https://example.com

# With specific device emulation
npx playwright codegen --device="iPhone 13" https://example.com
```

**How to use Codegen:**
1. Run the command
2. Browser opens with Playwright Inspector
3. Perform actions in the browser (click, type, navigate)
4. Playwright generates test code in real-time
5. Copy the generated code to your test file

### 2. Playwright Inspector

Debug tests step-by-step:

```bash
# Run test in debug mode
npx playwright test --debug

# Debug specific test
npx playwright test tests/login.spec.ts --debug

# Set breakpoint in code
await page.pause(); // Add this line in your test
```

**Inspector Features:**
- Step through test execution
- Inspect DOM elements
- Try out selectors
- View action logs
- See screenshots

### 3. Trace Viewer

View detailed execution traces:

```bash
# Record trace (already configured in playwright.config.ts)
npx playwright test --trace on

# View trace file
npx playwright show-trace trace.zip

# View trace from test results
npx playwright show-trace test-results/path-to-trace.zip
```

**Trace Viewer Shows:**
- Complete timeline of test execution
- Screenshots at each step
- DOM snapshots
- Network requests and responses
- Console logs
- Action details

**Enable traces in config:**

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry', // or 'on', 'retain-on-failure'
  },
});
```

### 4. VS Code Extension

Install the **Playwright Test for VS Code** extension:

**Features:**
- Run tests directly from editor
- Set breakpoints
- View test results inline
- Generate tests with Codegen
- Pick locators from webpage

**Shortcuts:**
- Run test: Click green play button next to test
- Debug test: Right-click → Debug Test
- Pick locator: Command Palette → "Playwright: Pick Locator"

### 5. Pick Locators

Find the best selector for an element:

```bash
# Open picker
npx playwright codegen --target=typescript
```

Then click on any element to see suggested selectors.

### 6. Screenshots & Videos

```typescript
// Take screenshot
await page.screenshot({ path: 'screenshot.png' });

// Screenshot specific element
await page.getByRole('button').screenshot({ path: 'button.png' });

// Full page screenshot
await page.screenshot({ path: 'full-page.png', fullPage: true });
```

**Configure in playwright.config.ts:**

```typescript
use: {
  screenshot: 'only-on-failure', // or 'on', 'off'
  video: 'retain-on-failure',    // or 'on', 'off'
}
```

---

##  Best Practices

### 1. Use User-Centric Selectors

```typescript
// ✅ Good - Accessible and stable
await page.getByRole('button', { name: 'Submit' });
await page.getByLabel('Email');
await page.getByText('Welcome');

// ❌ Avoid - Brittle and implementation-dependent
await page.locator('.btn-submit-primary');
await page.locator('#form > div:nth-child(2) > input');
```

### 2. Leverage Auto-Wait

```typescript
// ✅ Good - Playwright waits automatically
await page.getByRole('button').click();

// ❌ Avoid - Manual waits cause flaky tests
await page.waitForTimeout(3000);
await page.locator('.button').click();
```

### 3. Use Web-First Assertions

```typescript
// ✅ Good - Auto-retrying assertions
await expect(page.getByText('Success')).toBeVisible();
await expect(page).toHaveURL('/dashboard');
await expect(page.getByRole('button')).toBeEnabled();

// ❌ Avoid - Non-retrying assertions
const text = await page.textContent('.message');
expect(text).toBe('Success'); // No retry
```

### 4. Isolate Tests

```typescript
// ✅ Good - Each test is independent
test('test 1', async ({ page }) => {
  // Create own test data
  await createTestUser();
  // Run test
  // Cleanup
});

// ❌ Avoid - Tests depend on each other
let userId;
test('create user', async () => {
  userId = await createUser(); // Other tests depend on this
});
test('update user', async () => {
  await updateUser(userId); // Breaks if first test fails
});
```

### 5. Use Page Object Model

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Login' }).click();
  }

  async getErrorMessage() {
    return this.page.getByRole('alert');
  }
}

// tests/login.spec.ts
test('login flow', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@test.com', 'password');
  await expect(page).toHaveURL('/dashboard');
});
```

### 6. Use Test Data Effectively

```typescript
// test-data/users.ts
export const testUsers = {
  validUser: {
    email: 'valid@example.com',
    password: 'SecurePass123!',
  },
  adminUser: {
    email: 'admin@example.com',
    password: 'AdminPass123!',
  },
};

// tests/auth.spec.ts
import { testUsers } from '../test-data/users';

test('login as valid user', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(testUsers.validUser.email);
  await page.getByLabel('Password').fill(testUsers.validUser.password);
  await page.getByRole('button', { name: 'Login' }).click();
});
```

### 7. Add data-testid for Complex UIs

```html
<!-- Component -->
<button class="btn btn-primary btn-lg custom-class" data-testid="submit-form">
  Submit
</button>
```

```typescript
// Test
await page.getByTestId('submit-form').click();
```

---

##  CI/CD Integration

### GitHub Actions

Create `.github/workflows/playwright.yml`:

```yaml
name: Playwright Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: 18
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    
    - name: Run Playwright tests
      run: npx playwright test
    
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

### GitLab CI

Create `.gitlab-ci.yml`:

```yaml
image: mcr.microsoft.com/playwright:v1.40.0-jammy

stages:
  - test

playwright-tests:
  stage: test
  script:
    - npm ci
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 1 week
```

### Docker

```dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

Run with:
```bash
docker build -t playwright-tests .
docker run -it playwright-tests
```

---

##  Troubleshooting

### Common Issues

#### 1. Browser Not Installed

```bash
# Error: Executable doesn't exist
# Solution: Install browsers
npx playwright install
```

#### 2. Timeout Errors

```typescript
// Increase timeout for specific test
test('slow operation', async ({ page }) => {
  test.setTimeout(60000); // 60 seconds
  // ... test code
});

// Or in config
export default defineConfig({
  timeout: 60000,
});
```

#### 3. Element Not Found

```typescript
// Wait for element to be available
await page.getByRole('button', { name: 'Submit' }).waitFor();

// Or use better locator
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
```

#### 4. Flaky Tests

```typescript
// Enable retries
export default defineConfig({
  retries: 2, // Retry failed tests
});

// Use stable locators
await page.getByRole('button', { name: 'Submit' }); // Good
await page.locator('.btn'); // Bad - can change
```

#### 5. Tests Fail in CI but Pass Locally

```typescript
// Add waiting for network idle
await page.goto('/', { waitUntil: 'networkidle' });

// Take screenshots on failure (already configured)
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
}
```

---

##  Additional Resources

### Official Documentation
- **Playwright Docs**: https://playwright.dev
- **API Reference**: https://playwright.dev/docs/api/class-playwright
- **GitHub**: https://github.com/microsoft/playwright

### Community
- **Discord**: https://discord.gg/playwright
- **Stack Overflow**: Tag `playwright`

### Useful Commands Cheat Sheet

```bash
# Installation
npm init playwright@latest

# Run tests
npx playwright test                    # All tests
npx playwright test --headed           # With browser UI
npx playwright test --debug            # Debug mode
npx playwright test --ui               # Interactive UI mode

# Generate tests
npx playwright codegen [url]           # Record test

# View reports
npx playwright show-report             # HTML report
npx playwright show-trace trace.zip    # Trace viewer

# Update snapshots
npx playwright test --update-snapshots

# List tests
npx playwright test --list

# Specific browser
npx playwright test --project=chromium
```

---

##  Quick Start Checklist

- [ ] Install Playwright: `npm init playwright@latest`
- [ ] Review generated `playwright.config.ts`
- [ ] Write your first test in `tests/` directory
- [ ] Run test: `npx playwright test`
- [ ] Try Codegen: `npx playwright codegen https://your-app.com`
- [ ] Debug with Inspector: `npx playwright test --debug`
- [ ] View trace: `npx playwright show-trace`
- [ ] Add to CI/CD pipeline
- [ ] Set up Page Object Model for better maintainability

---

##  Next Steps

1. **Start Small**: Write 2-3 tests for critical user journeys
2. **Use Codegen**: Generate initial tests to learn the syntax
3. **Review Traces**: When tests fail, analyze traces to understand why
4. **Adopt Page Objects**: Organize tests as your suite grows
5. **Integrate CI/CD**: Automate test execution on every commit
6. **Monitor Flakiness**: Track and fix unstable tests
7. **Expand Coverage**: Gradually add more test scenarios

---

**Happy Testing with Playwright! 🎭**

--- 
By : NOUHAYLA MOUAKKAL
