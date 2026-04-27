---
type: skill
name: test-automation
description: Test automation framework development, scripting, integration, maintenance, and best practices across multiple technologies.
origin: ECC
---

# Test Automation Framework

Comprehensive guide to developing and maintaining robust test automation frameworks for various technology stacks.

## Framework Architecture

### Layered Design

```
test-automation/
├── framework/
│   ├── drivers/
│   │   ├── web-driver.js
│   │   ├── api-client.js
│   │   └── mobile-driver.js
│   ├── utils/
│   │   ├── reporter.js
│   │   ├── retry.js
│   │   └── screenshot.js
│   ├── assertions/
│   │   ├── base-assertions.js
│   │   └── custom-assertions.js
│   └── config/
│       ├── default.js
│       ├── ci.js
│       └── local.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── page-objects/
├── fixtures/
└── scripts/
    ├── setup.js
    └── cleanup.js
```

### Core Components

**Test Runner Integration**:
```javascript
// Example: Jest custom environment
const PlaywrightEnvironment = require('@playwright/test/lib/integration').PlaywrightTestEnvironment;

class CustomTestEnvironment extends PlaywrightEnvironment {
  async setup() {
    await super.setup();
    this.global.page = this.playwright._getOrCreateBrowser().newPage();
    this.global.api = new APIClient(process.env.API_BASE_URL);
  }

  async teardown() {
    await this.global.page.close();
    await super.teardown();
  }
}

module.exports = CustomTestEnvironment;
```

**Configuration Management**:
```javascript
// config/default.js
module.exports = {
  baseUrl: 'https://staging.example.com',
  timeout: 30000,
  retryAttempts: 3,
  headless: true,
  screenshotOnFailure: true,
  reporters: ['spec', 'junit', 'allure'],
  environments: {
    development: { baseUrl: 'http://localhost:3000' },
    staging: { baseUrl: 'https://staging.example.com' },
    production: { baseUrl: 'https://example.com' }
  }
};
```

## Automation Patterns

### Page Object Model (POM)

```javascript
// page-objects/login.page.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('[data-testid="login-btn"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async navigate() {
    await this.page.goto('/login');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

module.exports = LoginPage;
```

### Component Object Model

```javascript
// components/header.component.js
class HeaderComponent {
  constructor(page) {
    this.page = page;
    this.logo = page.locator('[data-testid="logo"]');
    this.userMenu = page.locator('[data-testid="user-menu"]');
    this.notifications = page.locator('[data-testid="notifications"]');
  }

  async verifyHeaderVisible() {
    await expect(this.logo).toBeVisible();
    await expect(this.userMenu).toBeVisible();
  }

  async navigateToProfile() {
    await this.userMenu.click();
    await this.page.locator('[data-testid="profile-link"]').click();
  }
}

module.exports = HeaderComponent;
```

### Screenplay Pattern

```javascript
// actors/user.actor.js
class User {
  constructor(name) {
    this.name = name;
    this.tasks = [];
    this.questions = [];
  }

  async attemptsTo(...tasks) {
    for (const task of tasks) {
      await task.performAs(this);
    }
  }

  async asks(...questions) {
    return await Promise.all(questions.map(q => q.answeredBy(this)));
  }
}

// tasks/login.task.js
class Login {
  constructor(username, password) {
    this.username = username;
    this.password = password;
  }

  async performAs(actor) {
    const loginPage = new LoginPage(actor.page);
    await loginPage.navigate();
    await loginPage.login(this.username, this.password);
  }
}
```

## CI/CD Integration

### Pipeline Configuration

```yaml
# .github/workflows/test-automation.yml
name: Test Automation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x]

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Install Playwright browsers
      run: npx playwright install --with-deps

    - name: Run tests
      run: npm run test:automation
      env:
        BASE_URL: ${{ secrets.STAGING_URL }}
        HEADLESS: true

    - name: Upload test results
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: test-results
        path: |
          test-results/
          screenshots/
          videos/
        retention-days: 30
```