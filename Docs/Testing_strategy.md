# Testing Strategy - คู่มือกลยุทธ์การทดสอบแบบครบถ้วน

## ภาพรวม

Testing Strategy เป็นแผนการรวมที่กำหนดวิธีการทดสอบ software เพื่อให้มั่นใจในคุณภาพและความน่าเชื่อถือของระบบ คู่มือนี้จะครอบคลุมตั้งแต่หลักการพื้นฐานไปจนถึงการ implement ที่ซับซ้อน พร้อมด้วย best practices และตัวอย่างจริง

---

## 🎯 หลักการพื้นฐานของการทดสอบ

### Testing Philosophy
```yaml
Core Principles:
  Early Testing: "เริ่มทดสอบตั้งแต่เริ่มพัฒนา ไม่ใช่หลังเสร็จ"
  Shift Left: "เลื่อนการทดสอบไปทางซ้าย (เร็วขึ้น) ใน development cycle"
  Risk-Based: "ทดสอบส่วนที่มีความเสี่ยงสูงก่อน"
  Automation First: "Automate ทุกอย่างที่ทำได้ให้ consistent และ repeatable"
  Fast Feedback: "ให้ feedback เร็วที่สุดเท่าที่จะทำได้"

Quality Goals:
  Functional Correctness: "ระบบทำงานตามที่กำหนดไว้"
  Performance: "ระบบตอบสนองในเวลาที่ยอมรับได้"
  Security: "ระบบปลอดภัยจากภัยคุกคาม"
  Usability: "ผู้ใช้สามารถใช้งานได้ง่าย"
  Reliability: "ระบบทำงานได้อย่างเสถียร"
  Maintainability: "Code ง่ายต่อการแก้ไขและขยาย"
```

### Testing ROI Matrix
```markdown
## การลงทุนในการทดสอบ vs ผลตอบแทน

### High ROI Testing Activities:
- **Unit Tests**: ต้นทุนต่ำ, feedback เร็ว, catch bugs เร็ว
- **Smoke Tests**: ทดสอบ critical path หลัก deployment
- **API Contract Tests**: ป้องกัน breaking changes
- **Static Analysis**: หา issues โดยไม่ต้องรัน code

### Medium ROI Testing Activities:
- **Integration Tests**: ค่าใช้จ่ายปานกลาง, จำเป็นสำหรับ complex systems
- **Component Tests**: ทดสอบ UI components แยกจาก backend
- **Performance Tests**: สำคัญแต่ต้องการ infrastructure

### Lower ROI (but still important):
- **Manual Exploratory Testing**: ค่าใช้จ่ายสูง แต่หา unexpected issues ได้
- **Visual Regression Tests**: สำคัญสำหรับ user-facing applications
- **Load Tests**: จำเป็นแต่ expensive และ complex
```

---

## 🏗️ Test Pyramid และ Testing Types

### The Testing Pyramid
```
                    /\
                   /  \
                  / E2E \     
                 /______\     น้อย (5-15%)
                /        \    ช้า, แพง, brittle
               /   API    \   
              /__________\   ปานกลาง (15-25%)
             /            \  เร็วกว่า E2E, stable
            /    Unit      \ 
           /________________\ เยอะ (60-80%)
                              เร็ว, ถูก, stable
```

### 1. Unit Testing

#### หลักการและวัตถุประสงค์
```typescript
// Unit Test คือการทดสอบ function, method, หรือ class แยกจากส่วนอื่น

// ✅ Good Unit Test Example
describe('UserValidator', () => {
  describe('validateEmail', () => {
    test('should return true for valid email', () => {
      // Arrange
      const validator = new UserValidator();
      const validEmail = 'john.doe@example.com';
      
      // Act
      const result = validator.validateEmail(validEmail);
      
      // Assert
      expect(result).toBe(true);
    });
    
    test('should return false for invalid email format', () => {
      const validator = new UserValidator();
      const invalidEmail = 'invalid-email';
      
      const result = validator.validateEmail(invalidEmail);
      
      expect(result).toBe(false);
    });
    
    test('should return false for email without domain', () => {
      const validator = new UserValidator();
      const invalidEmail = 'john@';
      
      const result = validator.validateEmail(invalidEmail);
      
      expect(result).toBe(false);
    });
  });
});

// Unit Test Best Practices:
// 1. Fast - ต้องรันเร็ว (< 1ms per test)
// 2. Isolated - ไม่พึ่งพา external dependencies
// 3. Repeatable - ได้ผลเหมือนเดิมทุกครั้ง
// 4. Self-Validating - มี clear pass/fail result
// 5. Timely - เขียนพร้อมกับ production code
```

#### Mocking และ Stubbing
```typescript
// Mock External Dependencies
import { UserService } from './userService';
import { EmailService } from './emailService';
import { DatabaseService } from './databaseService';

// Jest Mocking Example
jest.mock('./emailService');
jest.mock('./databaseService');

describe('UserService', () => {
  let userService: UserService;
  let mockEmailService: jest.Mocked<EmailService>;
  let mockDatabaseService: jest.Mocked<DatabaseService>;
  
  beforeEach(() => {
    mockEmailService = new EmailService() as jest.Mocked<EmailService>;
    mockDatabaseService = new DatabaseService() as jest.Mocked<DatabaseService>;
    userService = new UserService(mockDatabaseService, mockEmailService);
  });
  
  describe('createUser', () => {
    test('should create user and send welcome email', async () => {
      // Arrange
      const userData = { name: 'John', email: 'john@example.com' };
      const createdUser = { id: 1, ...userData };
      
      mockDatabaseService.saveUser.mockResolvedValue(createdUser);
      mockEmailService.sendWelcomeEmail.mockResolvedValue(true);
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(mockDatabaseService.saveUser).toHaveBeenCalledWith(userData);
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(createdUser);
      expect(result).toEqual(createdUser);
    });
    
    test('should handle database error gracefully', async () => {
      // Arrange
      const userData = { name: 'John', email: 'john@example.com' };
      const dbError = new Error('Database connection failed');
      
      mockDatabaseService.saveUser.mockRejectedValue(dbError);
      
      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects.toThrow('Failed to create user');
      
      expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
    });
  });
});
```

#### Test Coverage Guidelines
```yaml
Coverage Targets:
  Statements: "> 80%"
  Branches: "> 75%"
  Functions: "> 90%"
  Lines: "> 80%"

Focus Areas (100% Coverage Required):
  - Critical business logic
  - Security-related functions
  - Data validation and sanitization
  - Error handling paths
  - Public API interfaces

Coverage Exemptions:
  - Configuration files
  - Generated code
  - External library adapters
  - Simple getters/setters
  - Trivial constructors

Quality over Quantity:
  "80% well-written tests > 95% poor tests"
  - Tests should verify behavior, not implementation
  - Each test should have a clear purpose
  - Tests should be maintainable and readable
```

### 2. Integration Testing

#### API Integration Tests
```typescript
// Testing API endpoints with real HTTP calls
import request from 'supertest';
import { app } from '../app';
import { setupTestDatabase, cleanupTestDatabase } from '../test-utils';

describe('User API Integration Tests', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });
  
  afterAll(async () => {
    await cleanupTestDatabase();
  });
  
  beforeEach(async () => {
    // Reset database state for each test
    await clearAllTables();
  });
  
  describe('POST /api/users', () => {
    test('should create new user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'SecurePassword123!'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(Number),
        name: userData.name,
        email: userData.email,
        createdAt: expect.any(String)
      });
      
      // Verify password is not returned
      expect(response.body).not.toHaveProperty('password');
      
      // Verify user was actually saved to database
      const savedUser = await getUserById(response.body.id);
      expect(savedUser).toBeTruthy();
      expect(savedUser.email).toBe(userData.email);
    });
    
    test('should return 400 for invalid email format', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email',
        password: 'SecurePassword123!'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);
      
      expect(response.body).toMatchObject({
        error: 'Validation failed',
        details: expect.arrayContaining([
          expect.objectContaining({
            field: 'email',
            message: expect.stringContaining('valid email')
          })
        ])
      });
    });
    
    test('should return 409 for duplicate email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'SecurePassword123!'
      };
      
      // Create first user
      await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      // Try to create second user with same email
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(409);
      
      expect(response.body.error).toContain('Email already exists');
    });
  });
  
  describe('Authentication Flow', () => {
    test('should complete full login flow', async () => {
      // 1. Create user
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'SecurePassword123!'
      };
      
      const createResponse = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      // 2. Login with created user
      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: userData.email,
          password: userData.password
        })
        .expect(200);
      
      expect(loginResponse.body).toMatchObject({
        token: expect.any(String),
        user: {
          id: createResponse.body.id,
          email: userData.email
        }
      });
      
      // 3. Use token to access protected endpoint
      const protectedResponse = await request(app)
        .get('/api/profile')
        .set('Authorization', `Bearer ${loginResponse.body.token}`)
        .expect(200);
      
      expect(protectedResponse.body.id).toBe(createResponse.body.id);
    });
  });
});
```

#### Database Integration Tests
```typescript
// Testing database operations with real database
import { DatabaseService } from '../services/databaseService';
import { User, CreateUserData } from '../types';

describe('Database Integration Tests', () => {
  let dbService: DatabaseService;
  
  beforeAll(async () => {
    dbService = new DatabaseService({
      host: process.env.TEST_DB_HOST,
      database: process.env.TEST_DB_NAME,
      // Use test database configuration
    });
    await dbService.connect();
  });
  
  afterAll(async () => {
    await dbService.disconnect();
  });
  
  beforeEach(async () => {
    await dbService.clearAllTables();
  });
  
  describe('User Operations', () => {
    test('should save and retrieve user with all fields', async () => {
      const userData: CreateUserData = {
        name: 'John Doe',
        email: 'john@example.com',
        hashedPassword: 'hashed_password_here',
        profileData: {
          age: 30,
          preferences: { theme: 'dark' }
        }
      };
      
      // Save user
      const savedUser = await dbService.users.create(userData);
      
      expect(savedUser).toMatchObject({
        id: expect.any(Number),
        name: userData.name,
        email: userData.email,
        createdAt: expect.any(Date),
        updatedAt: expect.any(Date)
      });
      
      // Retrieve user
      const retrievedUser = await dbService.users.findById(savedUser.id);
      
      expect(retrievedUser).toEqual(savedUser);
      expect(retrievedUser.profileData).toEqual(userData.profileData);
    });
    
    test('should handle database constraints', async () => {
      const userData: CreateUserData = {
        name: 'John Doe',
        email: 'john@example.com',
        hashedPassword: 'hashed_password_here'
      };
      
      // Create first user
      await dbService.users.create(userData);
      
      // Try to create second user with same email (should fail)
      await expect(dbService.users.create(userData))
        .rejects.toThrow(/unique constraint|duplicate key/i);
    });
    
    test('should support transactions', async () => {
      const userData: CreateUserData = {
        name: 'John Doe',
        email: 'john@example.com',
        hashedPassword: 'hashed_password_here'
      };
      
      // Test successful transaction
      const result = await dbService.transaction(async (trx) => {
        const user = await dbService.users.create(userData, trx);
        await dbService.userProfiles.create({
          userId: user.id,
          bio: 'Test bio'
        }, trx);
        return user;
      });
      
      expect(result.id).toBeDefined();
      
      const profile = await dbService.userProfiles.findByUserId(result.id);
      expect(profile.bio).toBe('Test bio');
      
      // Test failed transaction (should rollback)
      await expect(dbService.transaction(async (trx) => {
        await dbService.users.create({
          ...userData,
          email: 'another@example.com'
        }, trx);
        
        // This should cause transaction to fail
        throw new Error('Simulated error');
      })).rejects.toThrow('Simulated error');
      
      // Verify rollback - user should not exist
      const users = await dbService.users.findByEmail('another@example.com');
      expect(users).toBeNull();
    });
  });
});
```

### 3. End-to-End (E2E) Testing

#### Playwright E2E Tests
```typescript
// Full user journey testing with real browser
import { test, expect, Page } from '@playwright/test';

// Page Object Model
class LoginPage {
  constructor(private page: Page) {}
  
  async goto() {
    await this.page.goto('/login');
  }
  
  async fillEmail(email: string) {
    await this.page.fill('[data-testid="email-input"]', email);
  }
  
  async fillPassword(password: string) {
    await this.page.fill('[data-testid="password-input"]', password);
  }
  
  async clickLogin() {
    await this.page.click('[data-testid="login-button"]');
  }
  
  async getErrorMessage() {
    return await this.page.textContent('[data-testid="error-message"]');
  }
}

class DocumentRequestPage {
  constructor(private page: Page) {}
  
  async goto() {
    await this.page.goto('/request-document');
  }
  
  async selectDocumentType(type: string) {
    await this.page.selectOption('[data-testid="document-type"]', type);
  }
  
  async fillPurpose(purpose: string) {
    await this.page.fill('[data-testid="purpose"]', purpose);
  }
  
  async uploadFile(filePath: string) {
    await this.page.setInputFiles('[data-testid="file-upload"]', filePath);
  }
  
  async submitRequest() {
    await this.page.click('[data-testid="submit-button"]');
  }
  
  async getSuccessMessage() {
    await this.page.waitForSelector('[data-testid="success-message"]');
    return await this.page.textContent('[data-testid="success-message"]');
  }
  
  async getTrackingNumber() {
    return await this.page.textContent('[data-testid="tracking-number"]');
  }
}

// E2E Test Suite
test.describe('Document Request System E2E', () => {
  test.beforeEach(async ({ page }) => {
    // Setup test data
    await setupTestUser();
    await setupTestDocumentTypes();
  });
  
  test.afterEach(async ({ page }) => {
    // Cleanup test data
    await cleanupTestData();
  });
  
  test('complete document request workflow', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const requestPage = new DocumentRequestPage(page);
    
    // Step 1: Login
    await loginPage.goto();
    await loginPage.fillEmail('student@university.edu');
    await loginPage.fillPassword('password123');
    await loginPage.clickLogin();
    
    // Verify successful login
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-name"]')).toContainText('Test Student');
    
    // Step 2: Navigate to document request
    await page.click('[data-testid="request-document-nav"]');
    await expect(page).toHaveURL('/request-document');
    
    // Step 3: Fill document request form
    await requestPage.selectDocumentType('transcript');
    await requestPage.fillPurpose('Job application');
    
    // Step 4: Upload supporting documents
    await requestPage.uploadFile('./test-files/id-card.pdf');
    
    // Verify file upload success
    await expect(page.locator('[data-testid="uploaded-file"]')).toContainText('id-card.pdf');
    
    // Step 5: Submit request
    await requestPage.submitRequest();
    
    // Step 6: Verify success
    const successMessage = await requestPage.getSuccessMessage();
    expect(successMessage).toContain('Request submitted successfully');
    
    const trackingNumber = await requestPage.getTrackingNumber();
    expect(trackingNumber).toMatch(/REQ-\d{8}/);
    
    // Step 7: Verify request appears in user's dashboard
    await page.click('[data-testid="dashboard-nav"]');
    await expect(page.locator('[data-testid="request-list"]')).toContainText(trackingNumber);
    await expect(page.locator('[data-testid="request-status"]')).toContainText('Pending Advisor Approval');
  });
  
  test('advisor approval workflow', async ({ page }) => {
    // Pre-condition: Create a pending request
    const requestId = await createTestRequest();
    
    const loginPage = new LoginPage(page);
    
    // Login as advisor
    await loginPage.goto();
    await loginPage.fillEmail('advisor@university.edu');
    await loginPage.fillPassword('password123');
    await loginPage.clickLogin();
    
    // Navigate to pending approvals
    await page.click('[data-testid="pending-approvals-nav"]');
    
    // Find and approve the request
    await page.click(`[data-testid="request-${requestId}"]`);
    
    // Review request details
    await expect(page.locator('[data-testid="request-details"]')).toBeVisible();
    await expect(page.locator('[data-testid="student-name"]')).toContainText('Test Student');
    
    // Add approval comment and approve
    await page.fill('[data-testid="approval-comment"]', 'Student is in good standing. Approved.');
    await page.click('[data-testid="approve-button"]');
    
    // Verify approval success
    await expect(page.locator('[data-testid="approval-success"]')).toContainText('Request approved successfully');
    
    // Verify request no longer in pending list
    await page.click('[data-testid="pending-approvals-nav"]');
    await expect(page.locator(`[data-testid="request-${requestId}"]`)).not.toBeVisible();
    
    // Verify notification sent (check database or email service)
    const notifications = await getNotificationsSent();
    expect(notifications).toContainEqual(
      expect.objectContaining({
        type: 'REQUEST_APPROVED',
        recipientEmail: 'student@university.edu'
      })
    );
  });
  
  test('error handling and validation', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const requestPage = new DocumentRequestPage(page);
    
    // Test login with invalid credentials
    await loginPage.goto();
    await loginPage.fillEmail('invalid@example.com');
    await loginPage.fillPassword('wrongpassword');
    await loginPage.clickLogin();
    
    const errorMessage = await loginPage.getErrorMessage();
    expect(errorMessage).toContain('Invalid credentials');
    
    // Test form validation
    await loginPage.fillEmail('student@university.edu');
    await loginPage.fillPassword('password123');
    await loginPage.clickLogin();
    
    await requestPage.goto();
    
    // Try to submit empty form
    await requestPage.submitRequest();
    
    // Verify validation errors
    await expect(page.locator('[data-testid="validation-error"]')).toContainText('Document type is required');
    await expect(page.locator('[data-testid="validation-error"]')).toContainText('Purpose is required');
    
    // Test file upload validation
    await requestPage.selectDocumentType('transcript');
    await requestPage.fillPurpose('Test purpose');
    
    // Try to upload invalid file type
    await requestPage.uploadFile('./test-files/invalid.txt');
    
    await expect(page.locator('[data-testid="upload-error"]')).toContainText('Only PDF, JPG, and PNG files are allowed');
  });
});

// Visual Regression Testing
test('visual regression tests', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Take screenshot and compare with baseline
  await expect(page).toHaveScreenshot('dashboard.png');
  
  // Test responsive design
  await page.setViewportSize({ width: 375, height: 667 }); // Mobile
  await expect(page).toHaveScreenshot('dashboard-mobile.png');
  
  await page.setViewportSize({ width: 768, height: 1024 }); // Tablet  
  await expect(page).toHaveScreenshot('dashboard-tablet.png');
});
```

---

## 🔄 Test-Driven Development (TDD)

### TDD Cycle Implementation
```typescript
// Red-Green-Refactor Cycle Example

// 1. RED: Write a failing test first
describe('PasswordValidator', () => {
  test('should validate password strength', () => {
    const validator = new PasswordValidator();
    
    // This will fail because PasswordValidator doesn't exist yet
    expect(validator.isStrong('weak')).toBe(false);
    expect(validator.isStrong('StrongP@ssw0rd')).toBe(true);
  });
});

// 2. GREEN: Write minimal code to make test pass
class PasswordValidator {
  isStrong(password: string): boolean {
    // Minimal implementation to pass the test
    return password === 'StrongP@ssw0rd';
  }
}

// 3. REFACTOR: Improve the implementation
class PasswordValidator {
  isStrong(password: string): boolean {
    if (password.length < 8) return false;
    if (!/[A-Z]/.test(password)) return false;
    if (!/[a-z]/.test(password)) return false;
    if (!/\d/.test(password)) return false;
    if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) return false;
    return true;
  }
}

// 4. Add more test cases (RED again)
describe('PasswordValidator', () => {
  let validator: PasswordValidator;
  
  beforeEach(() => {
    validator = new PasswordValidator();
  });
  
  describe('isStrong', () => {
    test('should return false for passwords shorter than 8 characters', () => {
      expect(validator.isStrong('Short1!')).toBe(false);
    });
    
    test('should return false for passwords without uppercase letter', () => {
      expect(validator.isStrong('lowercase123!')).toBe(false);
    });
    
    test('should return false for passwords without lowercase letter', () => {
      expect(validator.isStrong('UPPERCASE123!')).toBe(false);
    });
    
    test('should return false for passwords without numbers', () => {
      expect(validator.isStrong('NoNumbers!')).toBe(false);
    });
    
    test('should return false for passwords without special characters', () => {
      expect(validator.isStrong('NoSpecial123')).toBe(false);
    });
    
    test('should return true for strong passwords', () => {
      expect(validator.isStrong('StrongP@ssw0rd')).toBe(true);
      expect(validator.isStrong('Another$trong1')).toBe(true);
    });
  });
});
```

### BDD (Behavior-Driven Development)
```typescript
// Gherkin-style tests with Jest
import { defineFeature, loadFeature } from 'jest-cucumber';

const feature = loadFeature('./features/user-authentication.feature');

defineFeature(feature, test => {
  test('User logs in with valid credentials', ({ given, when, then }) => {
    let authService: AuthService;
    let loginResult: LoginResult;
    
    given('a user with valid credentials exists', () => {
      authService = new AuthService();
      // Setup user in test database
      createTestUser({
        email: 'test@example.com',
        password: 'hashedPassword'
      });
    });
    
    when('the user attempts to log in with correct credentials', async () => {
      loginResult = await authService.login('test@example.com', 'password123');
    });
    
    then('the login should be successful', () => {
      expect(loginResult.success).toBe(true);
      expect(loginResult.token).toBeDefined();
    });
    
    then('a valid JWT token should be returned', () => {
      expect(loginResult.token).toMatch(/^[A-Za-z0-9-_=]+\.[A-Za-z0-9-_=]+\.?[A-Za-z0-9-_.+/=]*$/);
    });
  });
  
  test('User login fails with invalid credentials', ({ given, when, then }) => {
    let authService: AuthService;
    let loginResult: LoginResult;
    let errorThrown: Error;
    
    given('a user attempts to log in', () => {
      authService = new AuthService();
    });
    
    when('the user provides invalid credentials', async () => {
      try {
        loginResult = await authService.login('test@example.com', 'wrongPassword');
      } catch (error) {
        errorThrown = error;
      }
    });
    
    then('the login should fail', () => {
      expect(errorThrown).toBeDefined();
      expect(errorThrown.message).toContain('Invalid credentials');
    });
    
    then('no token should be returned', () => {
      expect(loginResult).toBeUndefined();
    });
  });
});
```

```gherkin
# features/user-authentication.feature
Feature: User Authentication
  As a student
  I want to log in to the system
  So that I can access my account and request documents

  Scenario: User logs in with valid credentials
    Given a user with valid credentials exists
    When the user attempts to log in with correct credentials
    Then the login should be successful
    And a valid JWT token should be returned

  Scenario: User login fails with invalid credentials
    Given a user attempts to log in
    When the user provides invalid credentials
    Then the login should fail
    And no token should be returned

  Scenario: User account is locked after multiple failed attempts
    Given a user account exists
    When the user fails to log in 5 times consecutively
    Then the account should be locked
    And subsequent login attempts should be rejected
    And an email notification should be sent to the user
```

---

## 🛠️ Testing Tools และ Frameworks

### JavaScript/TypeScript Testing Stack
```yaml
Unit Testing:
  Jest:
    pros: "Zero config, built-in mocking, snapshot testing"
    cons: "Can be slow for large projects"
    best_for: "React/Node.js projects"
    
  Vitest:
    pros: "Vite-native, very fast, Jest-compatible API"
    cons: "Newer ecosystem, fewer plugins"
    best_for: "Vite-based projects"

  Mocha + Chai:
    pros: "Flexible, modular, lots of plugins"
    cons: "More configuration required"
    best_for: "Custom testing setups"

Integration Testing:
  Supertest:
    purpose: "HTTP assertion library for Node.js"
    usage: "API endpoint testing"
    
  TestContainers:
    purpose: "Real database/service testing"
    usage: "Integration tests with Docker"

E2E Testing:
  Playwright:
    pros: "Cross-browser, auto-wait, debugging tools"
    cons: "Learning curve"
    best_for: "Modern web applications"
    
  Cypress:
    pros: "Developer-friendly, great debugging"
    cons: "Limited to Chromium family"
    best_for: "Single-page applications"
    
  Selenium WebDriver:
    pros: "Mature, supports all browsers"
    cons: "Complex setup, flaky tests"
    best_for: "Legacy applications"
```

### Testing Configuration Examples

#### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  // Test environment
  testEnvironment: 'node', // or 'jsdom' for frontend
  
  // File patterns
  testMatch: [
    '**/__tests__/**/*.(test|spec).(js|ts)',
    '**/*.(test|spec).(js|ts)'
  ],
  
  // Coverage settings
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.d.ts',
    '!src/**/*.test.{js,ts}',
    '!src/test-utils/**',
    '!src/migrations/**'
  ],
  coverageThreshold: {
    global: {
      branches: 75,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/services/': {
      branches: 90,
      functions: 95,
      lines: 90,
      statements: 90
    }
  },
  
  // Setup files
  setupFilesAfterEnv: ['<rootDir>/src/test-setup.ts'],
  
  // Module mapping
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  
  // Transform files
  transform: {
    '^.+\\.ts$': 'ts-jest'
  },
  
  // Ignore patterns
  testPathIgnorePatterns: [
    '/node_modules/',
    '/dist/',
    '/coverage/'
  ],
  
  // Reporters
  reporters: [
    'default',
    ['jest-junit', {
      outputDirectory: './test-results',
      outputName: 'junit.xml'
    }]
  ]
};
```

#### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Test directory
  testDir: './e2e',
  
  // Timeout settings
  timeout: 30 * 1000,
  expect: {
    timeout: 5 * 1000
  },
  
  // Test execution settings
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  
  // Reporting
  reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/junit.xml' }],
    ['json', { outputFile: 'test-results/results.json' }]
  ],
  
  // Global test settings
  use: {
    baseURL: process.env.E2E_BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  
  // Project configurations for different browsers
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
  
  // Development server
  webServer: {
    command: 'npm run dev',
    port: 3000,
    reuseExistingServer: !process.env.CI
  }
});
```

### Test Utilities และ Helpers
```typescript
// test-utils/database.ts
import { Pool } from 'pg';

let testDbPool: Pool;

export async function setupTestDatabase() {
  testDbPool = new Pool({
    host: process.env.TEST_DB_HOST || 'localhost',
    port: parseInt(process.env.TEST_DB_PORT || '5432'),
    database: process.env.TEST_DB_NAME || 'test_db',
    user: process.env.TEST_DB_USER || 'test_user',
    password: process.env.TEST_DB_PASSWORD || 'test_password'
  });
  
  // Run migrations
  await runMigrations(testDbPool);
}

export async function cleanupTestDatabase() {
  if (testDbPool) {
    await testDbPool.end();
  }
}

export async function clearAllTables() {
  const client = await testDbPool.connect();
  try {
    await client.query('BEGIN');
    
    // Get all table names
    const result = await client.query(`
      SELECT tablename FROM pg_tables 
      WHERE schemaname = 'public' 
      AND tablename NOT LIKE 'migrations%'
    `);
    
    // Truncate all tables
    for (const row of result.rows) {
      await client.query(`TRUNCATE TABLE ${row.tablename} CASCADE`);
    }
    
    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// test-utils/fixtures.ts
export interface TestUser {
  id?: number;
  name: string;
  email: string;
  password: string;
  role: 'student' | 'advisor' | 'registrar';
}

export interface TestRequest {
  id?: number;
  studentId: number;
  documentType: string;
  purpose: string;
  status: string;
}

export class TestDataBuilder {
  static user(overrides: Partial<TestUser> = {}): TestUser {
    return {
      name: 'Test User',
      email: `test${Date.now()}@example.com`,
      password: 'password123',
      role: 'student',
      ...overrides
    };
  }
  
  static request(overrides: Partial<TestRequest> = {}): TestRequest {
    return {
      studentId: 1,
      documentType: 'transcript',
      purpose: 'Job application',
      status: 'PENDING_ADVISOR',
      ...overrides
    };
  }
  
  static async createUser(userData: Partial<TestUser> = {}): Promise<TestUser> {
    const user = this.user(userData);
    const savedUser = await userRepository.create(user);
    return savedUser;
  }
  
  static async createRequest(requestData: Partial<TestRequest> = {}): Promise<TestRequest> {
    const request = this.request(requestData);
    const savedRequest = await requestRepository.create(request);
    return savedRequest;
  }
}

// test-utils/matchers.ts
import { expect } from '@jest/globals';

// Custom Jest matchers
expect.extend({
  toBeValidEmail(received: string) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const pass = emailRegex.test(received);
    
    return {
      message: () => `expected ${received} ${pass ? 'not ' : ''}to be a valid email`,
      pass
    };
  },
  
  toBeWithinTimeRange(received: Date, expected: Date, rangeInMs: number = 1000) {
    const diff = Math.abs(received.getTime() - expected.getTime());
    const pass = diff <= rangeInMs;
    
    return {
      message: () => `expected ${received.toISOString()} to be within ${rangeInMs}ms of ${expected.toISOString()}`,
      pass
    };
  },
  
  toHaveValidJWT(received: string) {
    const jwtRegex = /^[A-Za-z0-9-_=]+\.[A-Za-z0-9-_=]+\.?[A-Za-z0-9-_.+/=]*$/;
    const pass = jwtRegex.test(received);
    
    return {
      message: () => `expected ${received} ${pass ? 'not ' : ''}to be a valid JWT token`,
      pass
    };
  }
});

// Usage in tests
test('should create user with valid email', async () => {
  const user = await createUser({ email: 'test@example.com' });
  
  expect(user.email).toBeValidEmail();
  expect(user.createdAt).toBeWithinTimeRange(new Date());
});
```

---

## ⚡ Performance Testing

### Load Testing Strategy
```yaml
Performance Testing Types:

Load Testing:
  purpose: "ทดสอบ normal expected load"
  metrics: "Response time, throughput, resource utilization"
  tools: "Artillery, k6, JMeter"

Stress Testing:
  purpose: "ทดสอบ breaking point ของระบบ"
  metrics: "Maximum load, failure modes, recovery time"
  approach: "Gradually increase load until failure"

Spike Testing:
  purpose: "ทดสอบ sudden traffic spikes"
  scenario: "Black Friday sales, viral content"
  focus: "Auto-scaling, circuit breakers"

Volume Testing:
  purpose: "ทดสอบ large amounts of data"
  focus: "Database performance, memory usage"
  scenarios: "Large file uploads, batch processing"

Endurance Testing:
  purpose: "ทดสอบ prolonged usage"
  duration: "Hours to days"
  focus: "Memory leaks, resource cleanup"
```

### k6 Performance Tests
```javascript
// performance-tests/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');

// Test configuration
export const options = {
  stages: [
    { duration: '2m', target: 10 },   // Ramp up to 10 users over 2 minutes
    { duration: '5m', target: 10 },   // Stay at 10 users for 5 minutes
    { duration: '2m', target: 50 },   // Ramp up to 50 users over 2 minutes
    { duration: '10m', target: 50 },  // Stay at 50 users for 10 minutes
    { duration: '2m', target: 0 },    // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests must complete below 500ms
    http_req_failed: ['rate<0.1'],    // Error rate must be below 10%
    errors: ['rate<0.1'],             // Custom error rate below 10%
  },
};

// Test data
const users = [
  { email: 'student1@university.edu', password: 'password123' },
  { email: 'student2@university.edu', password: 'password123' },
  { email: 'student3@university.edu', password: 'password123' },
];

let authToken = '';

export function setup() {
  // Setup test data before load test
  const setupResponse = http.post(`${__ENV.BASE_URL}/api/test/setup`, {
    users: users
  });
  
  check(setupResponse, {
    'setup successful': (r) => r.status === 200,
  });
  
  return { setupData: setupResponse.json() };
}

export default function (data) {
  // Step 1: Login
  const loginPayload = JSON.stringify({
    email: users[Math.floor(Math.random() * users.length)].email,
    password: 'password123'
  });
  
  const loginResponse = http.post(`${__ENV.BASE_URL}/api/auth/login`, loginPayload, {
    headers: { 'Content-Type': 'application/json' },
  });
  
  const loginSuccess = check(loginResponse, {
    'login status is 200': (r) => r.status === 200,
    'login response has token': (r) => r.json('token') !== undefined,
  });
  
  if (!loginSuccess) {
    errorRate.add(1);
    return;
  }
  
  authToken = loginResponse.json('token');
  
  sleep(1);
  
  // Step 2: Get dashboard data
  const dashboardResponse = http.get(`${__ENV.BASE_URL}/api/dashboard`, {
    headers: { 
      'Authorization': `Bearer ${authToken}`,
      'Content-Type': 'application/json'
    },
  });
  
  check(dashboardResponse, {
    'dashboard status is 200': (r) => r.status === 200,
    'dashboard loads in reasonable time': (r) => r.timings.duration < 1000,
  });
  
  sleep(2);
  
  // Step 3: Create document request
  const requestPayload = JSON.stringify({
    documentType: 'transcript',
    purpose: 'Job application',
    urgency: 'normal'
  });
  
  const createResponse = http.post(`${__ENV.BASE_URL}/api/requests`, requestPayload, {
    headers: { 
      'Authorization': `Bearer ${authToken}`,
      'Content-Type': 'application/json'
    },
  });
  
  const createSuccess = check(createResponse, {
    'create request status is 201': (r) => r.status === 201,
    'create request has tracking number': (r) => r.json('trackingNumber') !== undefined,
  });
  
  if (!createSuccess) {
    errorRate.add(1);
  }
  
  sleep(1);
  
  // Step 4: Get request list
  const listResponse = http.get(`${__ENV.BASE_URL}/api/requests`, {
    headers: { 
      'Authorization': `Bearer ${authToken}`,
      'Content-Type': 'application/json'
    },
  });
  
  check(listResponse, {
    'request list status is 200': (r) => r.status === 200,
    'request list contains data': (r) => r.json('data').length > 0,
  });
}

export function teardown(data) {
  // Cleanup test data after load test
  http.post(`${__ENV.BASE_URL}/api/test/cleanup`);
}
```

### Database Performance Testing
```sql
-- Database performance test queries
-- Create test data for performance testing

-- Generate test users
INSERT INTO users (name, email, password_hash, role, created_at)
SELECT 
  'Test User ' || generate_series,
  'testuser' || generate_series || '@university.edu',
  '$2b$10$example_hash_here',
  CASE 
    WHEN generate_series % 10 = 0 THEN 'advisor'
    WHEN generate_series % 20 = 0 THEN 'registrar'
    ELSE 'student'
  END,
  NOW() - (random() * interval '365 days')
FROM generate_series(1, 10000);

-- Generate test requests
INSERT INTO requests (student_id, document_type_id, status, purpose, created_at)
SELECT 
  (random() * 9000 + 1)::int, -- Random student ID
  (random() * 5 + 1)::int,     -- Random document type
  CASE 
    WHEN random() < 0.3 THEN 'PENDING_ADVISOR'
    WHEN random() < 0.6 THEN 'PENDING_REGISTRAR'
    WHEN random() < 0.9 THEN 'COMPLETED'
    ELSE 'REJECTED'
  END,
  'Performance test request ' || generate_series,
  NOW() - (random() * interval '90 days')
FROM generate_series(1, 50000);

-- Performance test queries with EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT r.id, r.status, r.created_at, u.name, u.email, dt.name as document_type
FROM requests r
JOIN users u ON r.student_id = u.id
JOIN document_types dt ON r.document_type_id = dt.id
WHERE r.status = 'PENDING_ADVISOR'
ORDER BY r.created_at DESC
LIMIT 20;

-- Test complex aggregation query
EXPLAIN ANALYZE
SELECT 
  dt.name as document_type,
  COUNT(*) as total_requests,
  COUNT(CASE WHEN r.status = 'COMPLETED' THEN 1 END) as completed,
  COUNT(CASE WHEN r.status = 'REJECTED' THEN 1 END) as rejected,
  AVG(EXTRACT(EPOCH FROM (r.updated_at - r.created_at))/3600) as avg_hours_to_process
FROM requests r
JOIN document_types dt ON r.document_type_id = dt.id
WHERE r.created_at >= NOW() - interval '30 days'
GROUP BY dt.id, dt.name
ORDER BY total_requests DESC;

-- Index performance testing
CREATE INDEX CONCURRENTLY idx_requests_status_created 
ON requests(status, created_at DESC);

CREATE INDEX CONCURRENTLY idx_requests_student_status 
ON requests(student_id, status);

-- Test query performance with new indexes
EXPLAIN ANALYZE
SELECT * FROM requests 
WHERE student_id = 1234 
AND status IN ('PENDING_ADVISOR', 'PENDING_REGISTRAR')
ORDER BY created_at DESC;
```

### API Performance Monitoring
```typescript
// middleware/performance-monitor.ts
import { Request, Response, NextFunction } from 'express';
import { Histogram, Counter } from 'prom-client';

// Metrics
const httpDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequests = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const dbQueryDuration = new Histogram({
  name: 'db_query_duration_seconds',
  help: 'Duration of database queries in seconds',
  labelNames: ['query_type', 'table']
});

export function performanceMonitorMiddleware(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();
  
  // Override res.end to capture metrics
  const originalEnd = res.end;
  res.end = function(chunk: any, encoding: any) {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    // Record metrics
    httpDuration
      .labels(req.method, route, res.statusCode.toString())
      .observe(duration);
    
    httpRequests
      .labels(req.method, route, res.statusCode.toString())
      .inc();
    
    // Log slow requests
    if (duration > 2) {
      console.warn(`Slow request: ${req.method} ${route} took ${duration}s`);
    }
    
    originalEnd.call(this, chunk, encoding);
  };
  
  next();
}

// Database query monitoring
export function monitorDatabaseQuery<T>(
  queryType: string,
  table: string,
  queryFn: () => Promise<T>
): Promise<T> {
  const start = Date.now();
  
  return queryFn()
    .then(result => {
      const duration = (Date.now() - start) / 1000;
      dbQueryDuration.labels(queryType, table).observe(duration);
      
      if (duration > 1) {
        console.warn(`Slow query: ${queryType} on ${table} took ${duration}s`);
      }
      
      return result;
    })
    .catch(error => {
      const duration = (Date.now() - start) / 1000;
      dbQueryDuration.labels(queryType, table).observe(duration);
      throw error;
    });
}

// Usage in service
export class UserService {
  async findById(id: number): Promise<User | null> {
    return monitorDatabaseQuery('SELECT', 'users', async () => {
      return await this.userRepository.findById(id);
    });
  }
  
  async create(userData: CreateUserData): Promise<User> {
    return monitorDatabaseQuery('INSERT', 'users', async () => {
      return await this.userRepository.create(userData);
    });
  }
}
```

---

## 🔒 Security Testing

### Security Testing Framework
```yaml
Security Testing Categories:

Authentication Testing:
  - Weak password policies
  - Brute force attacks
  - Session management
  - Multi-factor authentication bypass

Authorization Testing:
  - Privilege escalation
  - Access control bypass
  - Role-based security
  - Resource-level permissions

Input Validation Testing:
  - SQL injection
  - XSS (Cross-site scripting)
  - Command injection
  - File upload vulnerabilities

Session Management:
  - Session fixation
  - Session hijacking
  - Concurrent session limits
  - Secure logout

Data Protection:
  - Encryption at rest
  - Encryption in transit
  - Sensitive data exposure
  - Data leakage
```

### Automated Security Tests
```typescript
// security-tests/auth-security.test.ts
import request from 'supertest';
import { app } from '../app';

describe('Authentication Security Tests', () => {
  describe('Brute Force Protection', () => {
    test('should rate limit failed login attempts', async () => {
      const credentials = {
        email: 'test@example.com',
        password: 'wrongpassword'
      };
      
      // Make multiple failed attempts
      const promises = Array(10).fill(null).map(() =>
        request(app)
          .post('/api/auth/login')
          .send(credentials)
      );
      
      const responses = await Promise.all(promises);
      
      // Should start rate limiting after 5 attempts
      const rateLimitedResponses = responses.filter(r => r.status === 429);
      expect(rateLimitedResponses.length).toBeGreaterThan(0);
      
      // Should include proper rate limit headers
      const rateLimitResponse = rateLimitedResponses[0];
      expect(rateLimitResponse.headers['retry-after']).toBeDefined();
      expect(rateLimitResponse.body.error).toContain('Too many attempts');
    });
    
    test('should temporarily lock account after multiple failures', async () => {
      const credentials = {
        email: 'test@example.com',
        password: 'wrongpassword'
      };
      
      // Make multiple failed attempts
      for (let i = 0; i < 5; i++) {
        await request(app)
          .post('/api/auth/login')
          .send(credentials);
      }
      
      // Try with correct password - should still be locked
      const correctAttempt = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'correctpassword'
        });
      
      expect(correctAttempt.status).toBe(423); // Locked
      expect(correctAttempt.body.error).toContain('Account temporarily locked');
    });
  });
  
  describe('Password Security', () => {
    test('should reject weak passwords', async () => {
      const weakPasswords = [
        'password',
        '123456',
        'qwerty',
        'password123',
        'Password', // No numbers or special chars
        'Pass1',    // Too short
        'PASSWORD123!' // No lowercase
      ];
      
      for (const password of weakPasswords) {
        const response = await request(app)
          .post('/api/users')
          .send({
            name: 'Test User',
            email: 'test@example.com',
            password: password
          });
        
        expect(response.status).toBe(400);
        expect(response.body.error).toContain('Password does not meet requirements');
      }
    });
    
    test('should accept strong passwords', async () => {
      const strongPasswords = [
        'MySecureP@ssw0rd!',
        'Anoth3r$trongP@ss',
        'C0mpl3x&Secure!23'
      ];
      
      for (const password of strongPasswords) {
        const response = await request(app)
          .post('/api/users')
          .send({
            name: 'Test User',
            email: `test${Date.now()}@example.com`,
            password: password
          });
        
        expect(response.status).toBe(201);
      }
    });
  });
  
  describe('JWT Security', () => {
    test('should reject tampered JWT tokens', async () => {
      // Get valid token
      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
      
      const validToken = loginResponse.body.token;
      
      // Tamper with token
      const tamperedToken = validToken.slice(0, -5) + 'XXXXX';
      
      const response = await request(app)
        .get('/api/profile')
        .set('Authorization', `Bearer ${tamperedToken}`);
      
      expect(response.status).toBe(401);
      expect(response.body.error).toContain('Invalid token');
    });
    
    test('should reject expired JWT tokens', async () => {
      // Create token with short expiration for testing
      const shortLivedToken = jwt.sign(
        { userId: 1, email: 'test@example.com' },
        process.env.JWT_SECRET,
        { expiresIn: '1ms' }
      );
      
      // Wait for token to expire
      await new Promise(resolve => setTimeout(resolve, 10));
      
      const response = await request(app)
        .get('/api/profile')
        .set('Authorization', `Bearer ${shortLivedToken}`);
      
      expect(response.status).toBe(401);
      expect(response.body.error).toContain('Token expired');
    });
  });
});

// security-tests/injection-attacks.test.ts
describe('Injection Attack Prevention', () => {
  describe('SQL Injection', () => {
    test('should prevent SQL injection in user queries', async () => {
      const maliciousInputs = [
        "'; DROP TABLE users; --",
        "1' OR '1'='1",
        "'; INSERT INTO users (email) VALUES ('hacker@evil.com'); --",
        "1'; UPDATE users SET role='admin' WHERE id=1; --"
      ];
      
      for (const maliciousInput of maliciousInputs) {
        const response = await request(app)
          .get(`/api/users/${maliciousInput}`);
        
        // Should either return 400 (validation error) or 404 (not found)
        // Should NOT return 500 (which might indicate SQL error)
        expect([400, 404]).toContain(response.status);
        
        // Response should not contain SQL error messages
        expect(response.body.error || '').not.toMatch(/sql|syntax|database/i);
      }
    });
  });
  
  describe('XSS Prevention', () => {
    test('should sanitize user input to prevent XSS', async () => {
      const xssPayloads = [
        '<script>alert("XSS")</script>',
        '<img src="x" onerror="alert(\'XSS\')">',
        '"><script>document.location="http://evil.com"</script>',
        'javascript:alert("XSS")'
      ];
      
      for (const payload of xssPayloads) {
        const response = await request(app)
          .post('/api/users')
          .send({
            name: payload,
            email: 'test@example.com',
            password: 'SecurePassword123!'
          });
        
        if (response.status === 201) {
          // If user was created, name should be sanitized
          expect(response.body.name).not.toContain('<script>');
          expect(response.body.name).not.toContain('javascript:');
          expect(response.body.name).not.toContain('onerror');
        } else {
          // Should be rejected due to validation
          expect(response.status).toBe(400);
        }
      }
    });
  });
  
  describe('File Upload Security', () => {
    test('should reject malicious file uploads', async () => {
      const maliciousFiles = [
        { filename: 'virus.exe', content: 'MZ\x90\x00\x03', type: 'application/octet-stream' },
        { filename: 'script.php', content: '<?php system($_GET["cmd"]); ?>', type: 'application/x-php' },
        { filename: 'backdoor.jsp', content: '<%@page import="java.io.*"%>', type: 'application/x-jsp' },
        { filename: 'malware.js', content: 'eval(atob("alert(\'XSS\')"))', type: 'application/javascript' }
      ];
      
      for (const file of maliciousFiles) {
        const response = await request(app)
          .post('/api/upload')
          .attach('file', Buffer.from(file.content), {
            filename: file.filename,
            contentType: file.type
          });
        
        expect(response.status).toBe(400);
        expect(response.body.error).toMatch(/file type not allowed|invalid file/i);
      }
    });
    
    test('should accept only allowed file types', async () => {
      const allowedFiles = [
        { filename: 'document.pdf', content: '%PDF-1.4', type: 'application/pdf' },
        { filename: 'image.jpg', content: '\xFF\xD8\xFF\xE0', type: 'image/jpeg' },
        { filename: 'photo.png', content: '\x89PNG\r\n\x1a\n', type: 'image/png' }
      ];
      
      for (const file of allowedFiles) {
        const response = await request(app)
          .post('/api/upload')
          .attach('file', Buffer.from(file.content), {
            filename: file.filename,
            contentType: file.type
          });
        
        expect(response.status).toBe(200);
        expect(response.body.filename).toBe(file.filename);
      }
    });
  });
});
```

### OWASP ZAP Integration
```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Start application
      run: |
        npm run build
        npm start &
        sleep 30 # Wait for app to start
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Run OWASP ZAP Scan
      uses: zaproxy/action-full-scan@v0.4.0
      with:
        target: 'http://localhost:3000'
        rules_file_name: '.zap/rules.tsv'
        cmd_options: '-a'
        issue_title: 'ZAP Security Scan'
        token: ${{ secrets.GITHUB_TOKEN }}
```

---

## 📊 Test Data Management

### Test Data Strategy
```yaml
Test Data Categories:

Static Test Data:
  purpose: "Consistent, predictable data for repeatable tests"
  examples: "Reference data, lookup tables, configuration"
  storage: "SQL scripts, JSON files, fixtures"
  
Dynamic Test Data:
  purpose: "Generated data for each test run"
  examples: "User accounts, transactions, temporary records"
  generation: "Factories, builders, fake data libraries"
  
Sensitive Test Data:
  purpose: "Realistic data without real user information"
  approach: "Data masking, synthetic data generation"
  tools: "Faker.js, data anonymization tools"

Production-like Data:
  purpose: "Performance and load testing"
  approach: "Sanitized production data copies"
  considerations: "Privacy, data protection laws"
```

### Test Data Factories
```typescript
// test-data/factories.ts
import { faker } from '@faker-js/faker';

export interface UserFactory {
  id?: number;
  name?: string;
  email?: string;
  password?: string;
  role?: 'student' | 'advisor' | 'registrar' | 'admin';
  createdAt?: Date;
  isActive?: boolean;
}

export interface RequestFactory {
  id?: number;
  studentId?: number;
  documentTypeId?: number;
  purpose?: string;
  status?: 'DRAFT' | 'PENDING_ADVISOR' | 'PENDING_REGISTRAR' | 'COMPLETED' | 'REJECTED';
  createdAt?: Date;
  updatedAt?: Date;
}

export class TestDataFactory {
  static user(overrides: UserFactory = {}): UserFactory {
    return {
      name: faker.person.fullName(),
      email: faker.internet.email(),
      password: 'TestPassword123!',
      role: 'student',
      createdAt: faker.date.past(),
      isActive: true,
      ...overrides
    };
  }
  
  static advisor(overrides: UserFactory = {}): UserFactory {
    return this.user({
      role: 'advisor',
      email: faker.internet.email({ firstName: 'advisor' }),
      ...overrides
    });
  }
  
  static registrar(overrides: UserFactory = {}): UserFactory {
    return this.user({
      role: 'registrar',
      email: faker.internet.email({ firstName: 'registrar' }),
      ...overrides
    });
  }
  
  static request(overrides: RequestFactory = {}): RequestFactory {
    return {
      studentId: faker.number.int({ min: 1, max: 1000 }),
      documentTypeId: faker.number.int({ min: 1, max: 5 }),
      purpose: faker.lorem.sentence(),
      status: faker.helpers.arrayElement(['PENDING_ADVISOR', 'PENDING_REGISTRAR', 'COMPLETED']),
      createdAt: faker.date.past(),
      updatedAt: faker.date.recent(),
      ...overrides
    };
  }
  
  static pendingRequest(overrides: RequestFactory = {}): RequestFactory {
    return this.request({
      status: 'PENDING_ADVISOR',
      createdAt: faker.date.recent({ days: 7 }),
      ...overrides
    });
  }
  
  static completedRequest(overrides: RequestFactory = {}): RequestFactory {
    return this.request({
      status: 'COMPLETED',
      createdAt: faker.date.past({ days: 30 }),
      updatedAt: faker.date.recent({ days: 7 }),
      ...overrides
    });
  }
}

// Builder Pattern for Complex Data
export class RequestBuilder {
  private request: RequestFactory = {};
  
  withStudent(student: UserFactory): this {
    this.request.studentId = student.id;
    return this;
  }
  
  withDocumentType(type: 'transcript' | 'certificate' | 'recommendation'): this {
    const typeMap = {
      transcript: 1,
      certificate: 2,
      recommendation: 3
    };
    this.request.documentTypeId = typeMap[type];
    return this;
  }
  
  withStatus(status: RequestFactory['status']): this {
    this.request.status = status;
    return this;
  }
  
  withPurpose(purpose: string): this {
    this.request.purpose = purpose;
    return this;
  }
  
  createdDaysAgo(days: number): this {
    this.request.createdAt = new Date(Date.now() - days * 24 * 60 * 60 * 1000);
    return this;
  }
  
  build(): RequestFactory {
    return { ...this.request };
  }
}

// Usage examples
const testUser = TestDataFactory.user({ email: 'specific@example.com' });
const testAdvisor = TestDataFactory.advisor();
const testRequest = new RequestBuilder()
  .withStudent(testUser)
  .withDocumentType('transcript')
  .withStatus('PENDING_ADVISOR')
  .createdDaysAgo(3)
  .build();
```

### Database Seeding
```typescript
// test-data/seeders.ts
import { DatabaseService } from '../services/databaseService';
import { TestDataFactory, RequestBuilder } from './factories';

export class TestDataSeeder {
  constructor(private db: DatabaseService) {}
  
  async seedBasicData(): Promise<void> {
    // Seed document types
    await this.db.documentTypes.createMany([
      { id: 1, name: 'Official Transcript', fee: 50, processingDays: 5 },
      { id: 2, name: 'Graduation Certificate', fee: 100, processingDays: 7 },
      { id: 3, name: 'Recommendation Letter', fee: 25, processingDays: 3 },
      { id: 4, name: 'Enrollment Verification', fee: 20, processingDays: 2 },
      { id: 5, name: 'Grade Report', fee: 30, processingDays: 3 }
    ]);
    
    // Seed departments
    await this.db.departments.createMany([
      { id: 1, name: 'Computer Science', code: 'CS' },
      { id: 2, name: 'Engineering', code: 'ENG' },
      { id: 3, name: 'Business Administration', code: 'BA' },
      { id: 4, name: 'Liberal Arts', code: 'LA' }
    ]);
  }
  
  async seedUsers(count: number = 100): Promise<any[]> {
    const users = [];
    
    // Create advisors (10% of users)
    const advisorCount = Math.floor(count * 0.1);
    for (let i = 0; i < advisorCount; i++) {
      const advisor = TestDataFactory.advisor({
        email: `advisor${i + 1}@university.edu`
      });
      const savedAdvisor = await this.db.users.create(advisor);
      users.push(savedAdvisor);
    }
    
    // Create registrar staff (5% of users)
    const registrarCount = Math.floor(count * 0.05);
    for (let i = 0; i < registrarCount; i++) {
      const registrar = TestDataFactory.registrar({
        email: `registrar${i + 1}@university.edu`
      });
      const savedRegistrar = await this.db.users.create(registrar);
      users.push(savedRegistrar);
    }
    
    // Create students (85% of users)
    const studentCount = count - advisorCount - registrarCount;
    for (let i = 0; i < studentCount; i++) {
      const advisorId = users.find(u => u.role === 'advisor')?.id;
      const student = TestDataFactory.user({
        email: `student${i + 1}@university.edu`,
        advisorId: advisorId
      });
      const savedStudent = await this.db.users.create(student);
      users.push(savedStudent);
    }
    
    return users;
  }
  
  async seedRequests(users: any[], count: number = 500): Promise<any[]> {
    const students = users.filter(u => u.role === 'student');
    const requests = [];
    
    for (let i = 0; i < count; i++) {
      const student = faker.helpers.arrayElement(students);
      const daysAgo = faker.number.int({ min: 1, max: 90 });
      
      // Create more realistic status distribution
      let status: any;
      if (daysAgo < 3) {
        status = 'PENDING_ADVISOR';
      } else if (daysAgo < 7) {
        status = faker.helpers.arrayElement(['PENDING_ADVISOR', 'PENDING_REGISTRAR']);
      } else if (daysAgo < 30) {
        status = faker.helpers.arrayElement(['PENDING_REGISTRAR', 'COMPLETED', 'REJECTED']);
      } else {
        status = faker.helpers.arrayElement(['COMPLETED', 'REJECTED']);
      }
      
      const request = new RequestBuilder()
        .withStudent(student)
        .withDocumentType(faker.helpers.arrayElement(['transcript', 'certificate', 'recommendation']))
        .withStatus(status)
        .createdDaysAgo(daysAgo)
        .build();
      
      const savedRequest = await this.db.requests.create({
        ...request,
        studentId: student.id
      });
      
      requests.push(savedRequest);
    }
    
    return requests;
  }
  
  async seedFullDatabase(): Promise<{
    users: any[];
    requests: any[];
  }> {
    await this.seedBasicData();
    const users = await this.seedUsers(1000);
    const requests = await this.seedRequests(users, 5000);
    
    return { users, requests };
  }
  
  async clearAllData(): Promise<void> {
    // Clear in correct order to respect foreign key constraints
    await this.db.requests.deleteAll();
    await this.db.users.deleteAll();
    await this.db.documentTypes.deleteAll();
    await this.db.departments.deleteAll();
  }
}

// Usage in tests
beforeAll(async () => {
  const seeder = new TestDataSeeder(testDatabase);
  await seeder.seedFullDatabase();
});

afterAll(async () => {
  const seeder = new TestDataSeeder(testDatabase);
  await seeder.clearAllData();
});
```

---

## 📈 Quality Metrics และ Reporting

### Coverage Reporting
```typescript
// test-utils/coverage-reporter.ts
export interface CoverageReport {
  statements: CoverageMetric;
  branches: CoverageMetric;
  functions: CoverageMetric;
  lines: CoverageMetric;
}

export interface CoverageMetric {
  total: number;
  covered: number;
  percentage: number;
}

export class CoverageAnalyzer {
  static analyzeCoverage(coverageData: any): CoverageReport {
    return {
      statements: this.calculateMetric(coverageData.statements),
      branches: this.calculateMetric(coverageData.branches),
      functions: this.calculateMetric(coverageData.functions),
      lines: this.calculateMetric(coverageData.lines)
    };
  }
  
  private static calculateMetric(data: any): CoverageMetric {
    const total = Object.keys(data).length;
    const covered = Object.values(data).filter(count => count > 0).length;
    const percentage = total > 0 ? (covered / total) * 100 : 0;
    
    return { total, covered, percentage };
  }
  
  static generateReport(coverage: CoverageReport): string {
    return `
Coverage Report:
================
Statements: ${coverage.statements.covered}/${coverage.statements.total} (${coverage.statements.percentage.toFixed(2)}%)
Branches:   ${coverage.branches.covered}/${coverage.branches.total} (${coverage.branches.percentage.toFixed(2)}%)
Functions:  ${coverage.functions.covered}/${coverage.functions.total} (${coverage.functions.percentage.toFixed(2)}%)
Lines:      ${coverage.lines.covered}/${coverage.lines.total} (${coverage.lines.percentage.toFixed(2)}%)

Quality Gates:
- Statements: ${coverage.statements.percentage >= 80 ? '✅ PASS' : '❌ FAIL'} (target: 80%)
- Branches:   ${coverage.branches.percentage >= 75 ? '✅ PASS' : '❌ FAIL'} (target: 75%)
- Functions:  ${coverage.functions.percentage >= 90 ? '✅ PASS' : '❌ FAIL'} (target: 90%)
- Lines:      ${coverage.lines.percentage >= 80 ? '✅ PASS' : '❌ FAIL'} (target: 80%)
`;
  }
}
```

### Test Metrics Dashboard
```yaml
# Test Metrics Configuration
test_metrics:
  collection:
    test_execution_time: true
    test_success_rate: true
    code_coverage: true
    flaky_test_detection: true
    test_trend_analysis: true
  
  thresholds:
    min_coverage_percentage: 80
    max_test_execution_time: 300 # 5 minutes
    max_flaky_test_percentage: 5
    min_success_rate: 95
  
  reporting:
    frequency: "daily"
    recipients: ["team-leads@company.com", "qa-team@company.com"]
    formats: ["html", "json", "slack"]
  
  quality_gates:
    - name: "Coverage Gate"
      condition: "coverage >= 80%"
      action: "block_deployment"
    
    - name: "Test Success Rate"
      condition: "success_rate >= 95%"
      action: "block_deployment"
    
    - name: "Performance Gate"
      condition: "execution_time <= 300s"
      action: "warning"
```

---

## 🔧 Testing Process และ Workflows

### CI/CD Testing Pipeline
```yaml
# .github/workflows/test-pipeline.yml
name: Test Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

jobs:
  lint-and-format:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run ESLint
      run: npm run lint
    
    - name: Check code formatting
      run: npm run format:check
    
    - name: TypeScript compilation
      run: npm run type-check

  unit-tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run unit tests
      run: npm run test:unit -- --coverage --maxWorkers=2
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        file: ./coverage/lcov.info
        fail_ci_if_error: true

  integration-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run database migrations
      run: npm run migrate:test
      env:
        DATABASE_URL: ${{ env.DATABASE_URL }}
    
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: ${{ env.DATABASE_URL }}
        REDIS_URL: redis://localhost:6379

  e2e-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright browsers
      run: npx playwright install
    
    - name: Build application
      run: npm run build
    
    - name: Start application
      run: |
        npm run migrate:test
        npm start &
        npx wait-on http://localhost:3000
      env:
        DATABASE_URL: ${{ env.DATABASE_URL }}
        NODE_ENV: test
    
    - name: Run E2E tests
      run: npm run test:e2e
    
    - name: Upload E2E artifacts
      uses: actions/upload-artifact@v3
      if: failure()
      with:
        name: playwright-report
        path: |
          test-results/
          playwright-report/

  security-tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run security audit
      run: npm audit --audit-level=high
    
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high

  performance-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install k6
      run: |
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6
    
    - name: Run performance tests
      run: k6 run performance-tests/load-test.js
      env:
        BASE_URL: https://staging.yourapp.com

  quality-gate:
    runs-on: ubuntu-latest
    needs: [lint-and-format, unit-tests, integration-tests, e2e-tests, security-tests]
    
    steps:
    - name: Download coverage reports
      uses: actions/download-artifact@v3
      with:
        name: coverage-report
    
    - name: Quality Gate Check
      run: |
        # Check coverage thresholds
        # Check test success rates
        # Generate quality report
        echo "Quality gate passed ✅"
```

### Test Environment Management
```typescript
// test-utils/environment-manager.ts
export class TestEnvironmentManager {
  private static environments = new Map<string, TestEnvironment>();
  
  static async createEnvironment(name: string, config: EnvironmentConfig): Promise<TestEnvironment> {
    const env = new TestEnvironment(name, config);
    await env.setup();
    this.environments.set(name, env);
    return env;
  }
  
  static async destroyEnvironment(name: string): Promise<void> {
    const env = this.environments.get(name);
    if (env) {
      await env.teardown();
      this.environments.delete(name);
    }
  }
  
  static async destroyAllEnvironments(): Promise<void> {
    for (const [name, env] of this.environments) {
      await env.teardown();
    }
    this.environments.clear();
  }
}

export class TestEnvironment {
  private database?: DatabaseService;
  private server?: Server;
  
  constructor(
    private name: string,
    private config: EnvironmentConfig
  ) {}
  
  async setup(): Promise<void> {
    console.log(`Setting up test environment: ${this.name}`);
    
    // Setup database
    this.database = new DatabaseService({
      host: this.config.database.host,
      port: this.config.database.port,
      database: `${this.config.database.name}_${this.name}`,
      user: this.config.database.user,
      password: this.config.database.password
    });
    
    await this.database.connect();
    await this.database.migrate();
    
    // Setup application server
    if (this.config.server.enabled) {
      this.server = await this.startServer();
    }
    
    // Seed initial data
    if (this.config.seedData) {
      await this.seedData();
    }
    
    console.log(`Test environment ${this.name} ready`);
  }
  
  async teardown(): Promise<void> {
    console.log(`Tearing down test environment: ${this.name}`);
    
    if (this.server) {
      await this.stopServer();
    }
    
    if (this.database) {
      await this.database.dropDatabase();
      await this.database.disconnect();
    }
    
    console.log(`Test environment ${this.name} destroyed`);
  }
  
  private async startServer(): Promise<Server> {
    const app = createApp({
      database: this.database,
      port: this.config.server.port
    });
    
    return new Promise((resolve) => {
      const server = app.listen(this.config.server.port, () => {
        resolve(server);
      });
    });
  }
  
  private async stopServer(): Promise<void> {
    if (this.server) {
      return new Promise((resolve) => {
        this.server!.close(() => resolve());
      });
    }
  }
  
  private async seedData(): Promise<void> {
    const seeder = new TestDataSeeder(this.database!);
    await seeder.seedFullDatabase();
  }
}

// Usage in test setup
beforeAll(async () => {
  await TestEnvironmentManager.createEnvironment('integration', {
    database: {
      host: 'localhost',
      port: 5432,
      name: 'test',
      user: 'test',
      password: 'test'
    },
    server: {
      enabled: true,
      port: 3001
    },
    seedData: true
  });
});

afterAll(async () => {
  await TestEnvironmentManager.destroyAllEnvironments();
});
```

---

## 🎯 Testing Best Practices และ Anti-patterns

### Testing Best Practices
```markdown
## DO's - สิ่งที่ควรทำ

### 1. Test Structure และ Organization
✅ **ใช้ AAA Pattern (Arrange-Act-Assert)**
```typescript
test('should calculate total price with discount', () => {
  // Arrange
  const cart = new ShoppingCart();
  cart.addItem(new Item('Book', 100));
  const discount = new Discount(0.1); // 10%
  
  // Act
  const total = cart.calculateTotal(discount);
  
  // Assert
  expect(total).toBe(90);
});
```

✅ **ใช้ describe blocks เพื่อจัดกลุ่ม related tests**
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    test('should create user with valid data', () => {});
    test('should throw error for invalid email', () => {});
  });
  
  describe('updateUser', () => {
    test('should update user profile', () => {});
    test('should not update non-existent user', () => {});
  });
});
```

### 2. Test Independence
✅ **แต่ละ test ต้องสามารถรันแยกได้**
```typescript
describe('UserRepository', () => {
  beforeEach(async () => {
    await clearDatabase(); // Clean state for each test
    await seedBasicData();
  });
  
  test('should find user by email', async () => {
    const user = await createTestUser({ email: 'test@example.com' });
    const found = await userRepo.findByEmail('test@example.com');
    expect(found).toEqual(user);
  });
});
```

✅ **ใช้ fixtures และ factories สำหรับ test data**
```typescript
const testUser = TestDataFactory.user({
  email: 'specific@example.com',
  role: 'admin'
});
```

### 3. Test Naming และ Documentation
✅ **ใช้ชื่อ test ที่อธิบายพฤติกรรม**
```typescript
// ✅ Good
test('should return 401 when user provides invalid credentials', () => {});
test('should send welcome email after successful user registration', () => {});

// ❌ Bad
test('login test', () => {});
test('test user creation', () => {});
```

### 4. Mocking และ Stubbing
✅ **Mock external dependencies เท่านั้น**
```typescript
// ✅ Good - Mock external service
jest.mock('./emailService');

// ❌ Bad - Don't mock internal business logic
jest.mock('./userValidator'); // This is part of what we're testing
```

### 5. Assertions
✅ **ใช้ specific assertions**
```typescript
// ✅ Good
expect(response.status).toBe(201);
expect(response.body.email).toBe('test@example.com');
expect(response.body.createdAt).toBeInstanceOf(Date);

// ❌ Bad
expect(response).toBeTruthy();
expect(response.body).toBeDefined();
```
```

### Testing Anti-patterns
```markdown
## DON'Ts - สิ่งที่ไม่ควรทำ

### 1. Test Interdependence
❌ **Tests ที่พึ่งพากัน**
```typescript
// ❌ Bad - Tests depend on each other
describe('User Management', () => {
  let createdUserId;
  
  test('should create user', () => {
    const user = createUser({ name: 'John' });
    createdUserId = user.id; // Shared state!
    expect(user.name).toBe('John');
  });
  
  test('should update user', () => {
    updateUser(createdUserId, { name: 'Jane' }); // Depends on previous test!
    expect(getUser(createdUserId).name).toBe('Jane');
  });
});

// ✅ Good - Independent tests
describe('User Management', () => {
  test('should create user', () => {
    const user = createUser({ name: 'John' });
    expect(user.name).toBe('John');
  });
  
  test('should update user', () => {
    const user = createUser({ name: 'John' }); // Create own data
    const updated = updateUser(user.id, { name: 'Jane' });
    expect(updated.name).toBe('Jane');
  });
});
```

### 2. Testing Implementation Details
❌ **ทดสอบ implementation แทน behavior**
```typescript
// ❌ Bad - Testing internal method calls
test('should call validation method', () => {
  const validator = jest.spyOn(userService, 'validateEmail');
  userService.createUser({ email: 'test@example.com' });
  expect(validator).toHaveBeenCalled(); // Testing implementation
});

// ✅ Good - Testing behavior
test('should reject invalid email format', () => {
  expect(() => {
    userService.createUser({ email: 'invalid-email' });
  }).toThrow('Invalid email format'); // Testing behavior
});
```

### 3. Overly Complex Tests
❌ **Tests ที่ซับซ้อนเกินไป**
```typescript
// ❌ Bad - Too complex, testing multiple things
test('complete user workflow', async () => {
  const user = await createUser({ email: 'test@example.com' });
  await activateUser(user.id);
  const loginResult = await loginUser(user.email, 'password');
  const profile = await getProfile(loginResult.token);
  await updateProfile(loginResult.token, { name: 'Updated' });
  const updatedProfile = await getProfile(loginResult.token);
  
  expect(user.email).toBe('test@example.com');
  expect(loginResult.success).toBe(true);
  expect(profile.name).toBe('Test User');
  expect(updatedProfile.name).toBe('Updated');
  // If this fails, which part is broken?
});

// ✅ Good - Focused, single responsibility tests
test('should create user with valid email', async () => {
  const user = await createUser({ email: 'test@example.com' });
  expect(user.email).toBe('test@example.com');
});

test('should activate user successfully', async () => {
  const user = await createUser({ email: 'test@example.com' });
  await activateUser(user.id);
  const activated = await getUser(user.id);
  expect(activated.isActive).toBe(true);
});
```

### 4. Magic Numbers และ Unclear Data
❌ **ใช้ magic numbers และ unclear test data**
```typescript
// ❌ Bad - Magic numbers and unclear data
test('should calculate discount', () => {
  const result = calculateDiscount(1000, 0.15, 2);
  expect(result).toBe(850); // What do these numbers mean?
});

// ✅ Good - Clear, named values
test('should apply 15% bulk discount for orders over 2 items', () => {
  const orderAmount = 1000;
  const bulkDiscountRate = 0.15;
  const qualifyingItemCount = 2;
  
  const discountedAmount = calculateDiscount(
    orderAmount, 
    bulkDiscountRate, 
    qualifyingItemCount
  );
  
  const expectedAmount = orderAmount * (1 - bulkDiscountRate);
  expect(discountedAmount).toBe(expectedAmount);
});
```

### 5. Inadequate Error Testing
❌ **ไม่ทดสอบ error cases เพียงพอ**
```typescript
// ❌ Bad - Only testing happy path
test('should create user', () => {
  const user = createUser({ name: 'John', email: 'john@example.com' });
  expect(user.name).toBe('John');
});

// ✅ Good - Testing both success and failure cases
describe('createUser', () => {
  test('should create user with valid data', () => {
    const user = createUser({ name: 'John', email: 'john@example.com' });
    expect(user.name).toBe('John');
  });
  
  test('should throw error for invalid email', () => {
    expect(() => {
      createUser({ name: 'John', email: 'invalid-email' });
    }).toThrow('Invalid email format');
  });
  
  test('should throw error for missing name', () => {
    expect(() => {
      createUser({ email: 'john@example.com' });
    }).toThrow('Name is required');
  });
});
```
```

---

## 🏆 Advanced Testing Patterns

### Test Doubles Strategy
```typescript
// Different types of test doubles and when to use them

// 1. Dummy - Objects passed around but not used
class DummyEmailService implements EmailService {
  async sendEmail(): Promise<boolean> {
    return true; // Not actually implemented
  }
}

// 2. Fake - Working implementation but simplified
class FakeUserRepository implements UserRepository {
  private users: User[] = [];
  
  async save(user: User): Promise<User> {
    const saved = { ...user, id: this.users.length + 1 };
    this.users.push(saved);
    return saved;
  }
  
  async findById(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
}

// 3. Stub - Returns predetermined responses
class StubPaymentGateway implements PaymentGateway {
  constructor(private shouldSucceed: boolean = true) {}
  
  async processPayment(amount: number): Promise<PaymentResult> {
    if (this.shouldSucceed) {
      return { success: true, transactionId: 'tx_123' };
    } else {
      return { success: false, error: 'Payment failed' };
    }
  }
}

// 4. Spy - Records how it was called
class SpyNotificationService implements NotificationService {
  public sentNotifications: Notification[] = [];
  
  async send(notification: Notification): Promise<void> {
    this.sentNotifications.push(notification);
  }
  
  getNotificationsSentTo(email: string): Notification[] {
    return this.sentNotifications.filter(n => n.recipient === email);
  }
}

// 5. Mock - Verifies behavior and interactions
const mockAuditLogger = {
  log: jest.fn(),
  getLogEntries: jest.fn().mockReturnValue([])
};

// Usage examples
describe('UserService with Test Doubles', () => {
  test('should save user using fake repository', async () => {
    const fakeRepo = new FakeUserRepository();
    const service = new UserService(fakeRepo);
    
    const user = await service.createUser({ name: 'John', email: 'john@example.com' });
    
    expect(user.id).toBeDefined();
    expect(user.name).toBe('John');
  });
  
  test('should handle payment failure with stub', async () => {
    const failingPaymentGateway = new StubPaymentGateway(false);
    const service = new PaymentService(failingPaymentGateway);
    
    const result = await service.processOrder({ amount: 100 });
    
    expect(result.success).toBe(false);
    expect(result.error).toContain('Payment failed');
  });
  
  test('should send notification using spy', async () => {
    const notificationSpy = new SpyNotificationService();
    const service = new UserService(new FakeUserRepository(), notificationSpy);
    
    await service.createUser({ name: 'John', email: 'john@example.com' });
    
    expect(notificationSpy.sentNotifications).toHaveLength(1);
    expect(notificationSpy.sentNotifications[0].type).toBe('WELCOME');
  });
  
  test('should log audit events using mock', async () => {
    const service = new UserService(new FakeUserRepository(), undefined, mockAuditLogger);
    
    await service.createUser({ name: 'John', email: 'john@example.com' });
    
    expect(mockAuditLogger.log).toHaveBeenCalledWith({
      action: 'USER_CREATED',
      userId: expect.any(Number),
      timestamp: expect.any(Date)
    });
  });
});
```

### Property-Based Testing
```typescript
// Using fast-check for property-based testing
import fc from 'fast-check';

describe('Property-Based Tests', () => {
  test('email validation should be consistent', () => {
    fc.assert(fc.property(
      fc.emailAddress(),
      (email) => {
        const isValid = validateEmail(email);
        // Property: valid emails should always pass validation
        expect(isValid).toBe(true);
      }
    ));
  });
  
  test('password hashing should be deterministic', () => {
    fc.assert(fc.property(
      fc.string({ minLength: 8, maxLength: 100 }),
      async (password) => {
        const hash1 = await hashPassword(password);
        const hash2 = await hashPassword(password);
        
        // Property: same password should produce same hash
        expect(hash1).toBe(hash2);
        
        // Property: hash should be different from original password
        expect(hash1).not.toBe(password);
      }
    ));
  });
  
  test('array sorting should be idempotent', () => {
    fc.assert(fc.property(
      fc.array(fc.integer()),
      (numbers) => {
        const sorted1 = [...numbers].sort((a, b) => a - b);
        const sorted2 = [...sorted1].sort((a, b) => a - b);
        
        // Property: sorting already sorted array should not change it
        expect(sorted1).toEqual(sorted2);
      }
    ));
  });
  
  test('price calculation should be commutative for addition', () => {
    fc.assert(fc.property(
      fc.float({ min: 0, max: 1000 }),
      fc.float({ min: 0, max: 1000 }),
      (price1, price2) => {
        const total1 = calculateTotal([price1, price2]);
        const total2 = calculateTotal([price2, price1]);
        
        // Property: order shouldn't matter for addition
        expect(total1).toBeCloseTo(total2, 2);
      }
    ));
  });
});
```

### Mutation Testing
```typescript
// Mutation testing helps find weaknesses in test suites
// Example of what mutation testing might reveal

// Original code
function isAdult(age: number): boolean {
  return age >= 18; // Mutation: change >= to >
}

// Current test
test('should identify adults correctly', () => {
  expect(isAdult(18)).toBe(true);
  expect(isAdult(17)).toBe(false);
});

// Mutation testing would create this mutant:
function isAdultMutant(age: number): boolean {
  return age > 18; // Changed >= to >
}

// The test would still pass with age 17, but fail with age 18
// This reveals we need an additional test case:
test('should handle edge case at exactly 18', () => {
  expect(isAdult(18)).toBe(true); // This test catches the mutation
});

// Stryker configuration for JavaScript mutation testing
module.exports = {
  mutate: [
    'src/**/*.ts',
    '!src/**/*.test.ts',
    '!src/test-utils/**/*.ts'
  ],
  testRunner: 'jest',
  jest: {
    projectType: 'custom',
    configFile: 'jest.config.js'
  },
  coverageAnalysis: 'perTest',
  thresholds: {
    high: 80,
    low: 60,
    break: 50
  },
  mutator: {
    excludedMutations: [
      'StringLiteral', // Don't mutate string literals
      'BlockStatement' // Don't remove block statements
    ]
  }
};
```

---

## 🔍 Test Analysis และ Debugging

### Flaky Test Detection
```typescript
// test-utils/flaky-test-detector.ts
export interface TestRun {
  testName: string;
  passed: boolean;
  duration: number;
  timestamp: Date;
  error?: string;
}

export class FlakyTestDetector {
  private testRuns: Map<string, TestRun[]> = new Map();
  
  recordTestRun(run: TestRun): void {
    const runs = this.testRuns.get(run.testName) || [];
    runs.push(run);
    
    // Keep only last 100 runs
    if (runs.length > 100) {
      runs.shift();
    }
    
    this.testRuns.set(run.testName, runs);
  }
  
  detectFlakyTests(minRuns: number = 10, flakyThreshold: number = 0.9): string[] {
    const flakyTests: string[] = [];
    
    for (const [testName, runs] of this.testRuns) {
      if (runs.length < minRuns) continue;
      
      const passRate = runs.filter(r => r.passed).length / runs.length;
      
      // Test is flaky if it sometimes passes and sometimes fails
      if (passRate > 0.1 && passRate < flakyThreshold) {
        flakyTests.push(testName);
      }
    }
    
    return flakyTests;
  }
  
  getTestStability(testName: string): {
    passRate: number;
    averageDuration: number;
    lastFailures: string[];
    trends: 'improving' | 'declining' | 'stable';
  } {
    const runs = this.testRuns.get(testName) || [];
    
    if (runs.length === 0) {
      return {
        passRate: 0,
        averageDuration: 0,
        lastFailures: [],
        trends: 'stable'
      };
    }
    
    const passRate = runs.filter(r => r.passed).length / runs.length;
    const averageDuration = runs.reduce((sum, r) => sum + r.duration, 0) / runs.length;
    const lastFailures = runs
      .filter(r => !r.passed)
      .slice(-5)
      .map(r => r.error || 'Unknown error');
    
    // Analyze trends (last 20 runs vs previous 20 runs)
    let trends: 'improving' | 'declining' | 'stable' = 'stable';
    if (runs.length >= 40) {
      const recent = runs.slice(-20);
      const previous = runs.slice(-40, -20);
      
      const recentPassRate = recent.filter(r => r.passed).length / recent.length;
      const previousPassRate = previous.filter(r => r.passed).length / previous.length;
      
      if (recentPassRate > previousPassRate + 0.1) {
        trends = 'improving';
      } else if (recentPassRate < previousPassRate - 0.1) {
        trends = 'declining';
      }
    }
    
    return { passRate, averageDuration, lastFailures, trends };
  }
  
  generateReport(): string {
    const flakyTests = this.detectFlakyTests();
    let report = 'Flaky Test Report\n================\n\n';
    
    if (flakyTests.length === 0) {
      report += '✅ No flaky tests detected!\n';
    } else {
      report += `⚠️  Found ${flakyTests.length} potentially flaky tests:\n\n`;
      
      for (const testName of flakyTests) {
        const stability = this.getTestStability(testName);
        report += `📊 ${testName}\n`;
        report += `   Pass Rate: ${(stability.passRate * 100).toFixed(1)}%\n`;
        report += `   Avg Duration: ${stability.averageDuration.toFixed(0)}ms\n`;
        report += `   Trend: ${stability.trends}\n`;
        
        if (stability.lastFailures.length > 0) {
          report += `   Recent Failures:\n`;
          stability.lastFailures.forEach(error => {
            report += `     - ${error}\n`;
          });
        }
        report += '\n';
      }
    }
    
    return report;
  }
}

// Jest reporter to track test stability
class TestStabilityReporter {
  private detector = new FlakyTestDetector();
  
  onTestResult(test: any, testResult: any): void {
    testResult.testResults.forEach((result: any) => {
      this.detector.recordTestRun({
        testName: result.fullName,
        passed: result.status === 'passed',
        duration: result.duration || 0,
        timestamp: new Date(),
        error: result.failureMessages.join('; ')
      });
    });
  }
  
  onRunComplete(): void {
    const report = this.detector.generateReport();
    console.log(report);
    
    // Save to file for CI/CD pipeline
    require('fs').writeFileSync('flaky-test-report.txt', report);
  }
}
```

### Test Debugging Tools
```typescript
// test-utils/debugging-helpers.ts
export class TestDebugger {
  static enableDetailedLogging(): void {
    // Enable detailed console logging for tests
    const originalLog = console.log;
    console.log = (...args) => {
      const timestamp = new Date().toISOString();
      originalLog(`[${timestamp}] TEST:`, ...args);
    };
  }
  
  static captureAsyncErrors(): void {
    // Capture unhandled promise rejections in tests
    process.on('unhandledRejection', (reason, promise) => {
      console.error('Unhandled Rejection in test:', reason);
      console.error('Promise:', promise);
    });
  }
  
  static timeoutWarning(testName: string, maxDuration: number = 5000): void {
    const startTime = Date.now();
    
    const checkTimeout = () => {
      const elapsed = Date.now() - startTime;
      if (elapsed > maxDuration) {
        console.warn(`⚠️  Test "${testName}" is taking longer than expected (${elapsed}ms)`);
      } else {
        setTimeout(checkTimeout, 1000);
      }
    };
    
    setTimeout(checkTimeout, 1000);
  }
  
  static async waitForCondition(
    condition: () => boolean | Promise<boolean>,
    timeout: number = 5000,
    interval: number = 100
  ): Promise<void> {
    const startTime = Date.now();
    
    while (Date.now() - startTime < timeout) {
      const result = await condition();
      if (result) return;
      
      await new Promise(resolve => setTimeout(resolve, interval));
    }
    
    throw new Error(`Condition not met within ${timeout}ms`);
  }
  
  static createNetworkSnapshot(): NetworkSnapshot {
    // Capture network requests during test execution
    const requests: NetworkRequest[] = [];
    
    // Mock fetch to capture requests
    const originalFetch = global.fetch;
    global.fetch = jest.fn(async (url, options) => {
      const startTime = Date.now();
      
      try {
        const response = await originalFetch(url, options);
        
        requests.push({
          url: url.toString(),
          method: options?.method || 'GET',
          status: response.status,
          duration: Date.now() - startTime,
          success: response.ok
        });
        
        return response;
      } catch (error) {
        requests.push({
          url: url.toString(),
          method: options?.method || 'GET',
          status: 0,
          duration: Date.now() - startTime,
          success: false,
          error: error.message
        });
        
        throw error;
      }
    });
    
    return {
      getRequests: () => requests,
      reset: () => {
        requests.length = 0;
        global.fetch = originalFetch;
      }
    };
  }
}

// Usage in tests
describe('API Integration with Debugging', () => {
  beforeEach(() => {
    TestDebugger.enableDetailedLogging();
    TestDebugger.captureAsyncErrors();
  });
  
  test('should handle slow API responses', async () => {
    TestDebugger.timeoutWarning('slow API test', 3000);
    
    const networkSnapshot = TestDebugger.createNetworkSnapshot();
    
    // Test slow API call
    const response = await fetch('/api/slow-endpoint');
    
    // Wait for async processing to complete
    await TestDebugger.waitForCondition(
      () => response.status === 200,
      5000
    );
    
    // Analyze network activity
    const requests = networkSnapshot.getRequests();
    expect(requests).toHaveLength(1);
    expect(requests[0].duration).toBeLessThan(3000);
    
    networkSnapshot.reset();
  });
});
```

---

## 📚 Testing Documentation และ Knowledge Sharing

### Test Documentation Standards
```markdown
# Testing Documentation Template

## Test Plan: User Authentication Module

### Overview
This document outlines the testing strategy for the user authentication module, including login, logout, password reset, and session management functionality.

### Scope
**In Scope:**
- User login with email/password
- JWT token generation and validation
- Password reset workflow
- Session timeout handling
- Account lockout after failed attempts

**Out of Scope:**
- OAuth integration (separate test plan)
- Multi-factor authentication (future release)
- Password history validation

### Test Objectives
1. **Functional Correctness**: Verify all authentication flows work as specified
2. **Security**: Ensure secure handling of credentials and tokens
3. **Performance**: Login should complete within 500ms
4. **Usability**: Clear error messages for failed attempts

### Test Strategy

#### Unit Tests (80% coverage target)
- Password validation rules
- JWT token generation/verification
- Input sanitization
- Error handling

#### Integration Tests
- API endpoint testing
- Database integration
- Email service integration
- Rate limiting functionality

#### E2E Tests
- Complete login workflow
- Password reset workflow
- Session timeout scenarios
- Error scenarios (invalid credentials, locked account)

### Test Data Requirements
- Valid user accounts (student, advisor, registrar roles)
- Invalid email formats
- Weak passwords for validation testing
- Expired tokens for security testing

### Environment Setup
```bash
# Database setup
docker run -d --name test-db -p 5432:5432 postgres:13

# Start test environment
npm run test:setup

# Run specific test suite
npm run test:auth
```

### Success Criteria
- All tests pass with 0 flaky tests
- Code coverage >= 80%
- Security scan passes with no high/critical issues
- Performance tests meet response time requirements

### Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Password data exposure | High | Encrypt all password data, never log passwords |
| JWT token vulnerabilities | Medium | Use secure signing, implement token rotation |
| Brute force attacks | Medium | Implement rate limiting and account lockout |

### Test Schedule
- Unit Tests: Continuous (with each commit)
- Integration Tests: Daily
- E2E Tests: Before each release
- Security Tests: Weekly
- Performance Tests: Before major releases
```

### Knowledge Sharing Sessions
```yaml
Testing Knowledge Sharing Program:

Monthly Sessions:
  - "Testing Best Practices" - sharing experiences and lessons learned
  - "Tool Deep Dive" - exploring new testing tools and techniques
  - "Bug Post-Mortem" - analyzing production issues and testing gaps
  - "Performance Testing Workshop" - hands-on performance testing

Quarterly Sessions:
  - "Testing Strategy Review" - evaluating and improving test strategy
  - "Cross-team Testing Collaboration" - sharing practices between teams
  - "Industry Trends" - new testing methodologies and tools

Documentation:
  - Test Case Library - reusable test scenarios
  - Testing Playbooks - step-by-step testing guides
  - Troubleshooting Guides - common issues and solutions
  - Tool Tutorials - how-to guides for testing tools

Mentorship Program:
  - Senior testers mentor junior developers
  - Code review sessions focused on testability
  - Pair testing sessions
  - Testing skills assessment and development plans
```

---

## 🎯 Implementation Roadmap

### Phase 1: Foundation (Month 1-2)
```markdown
## Week 1-2: Setup and Planning
- [ ] Conduct testing strategy workshop with team
- [ ] Define testing standards and guidelines
- [ ] Set up basic testing infrastructure (Jest, test database)
- [ ] Create initial test data factories and utilities
- [ ] Establish CI/CD pipeline with basic test automation

## Week 3-4: Unit Testing Framework
- [ ] Implement comprehensive unit tests for core business logic
- [ ] Set up code coverage reporting and thresholds
- [ ] Create mocking strategies for external dependencies
- [ ] Establish unit testing best practices and code review guidelines
- [ ] Train team on unit testing patterns and TDD

## Week 5-8: Integration Testing
- [ ] Set up integration test environment
- [ ] Implement API integration tests
- [ ] Create database integration tests
- [ ] Set up test data management and cleanup procedures
- [ ] Establish integration testing in CI/CD pipeline
```

### Phase 2: Advanced Testing (Month 3-4)
```markdown
## Week 9-12: E2E Testing Implementation
- [ ] Set up Playwright/Cypress for E2E testing
- [ ] Implement critical user journey tests
- [ ] Create page object models and test utilities
- [ ] Set up visual regression testing
- [ ] Integrate E2E tests into deployment pipeline

## Week 13-16: Performance and Security Testing
- [ ] Implement load testing with k6 or Artillery
- [ ] Set up database performance testing
- [ ] Create security test suite
- [ ] Implement automated security scanning
- [ ] Establish performance benchmarks and monitoring
```

### Phase 3: Optimization (Month 5-6)
```markdown
## Week 17-20: Test Quality and Efficiency
- [ ] Implement flaky test detection and monitoring
- [ ] Optimize test execution speed and parallelization
- [ ] Create comprehensive test documentation
- [ ] Implement mutation testing for critical components
- [ ] Set up test metrics and reporting dashboard

## Week 21-24: Team Scaling and Process
- [ ] Create testing onboarding program
- [ ] Establish test review and quality processes
- [ ] Implement cross-team testing collaboration
- [ ] Create testing knowledge base and guidelines
- [ ] Plan for continuous improvement and tool evaluation
```

---

## 🔄 Continuous Improvement Process

### Testing Retrospectives
```markdown
## Monthly Testing Retrospective Template

### Metrics Review
- Test execution time trends
- Test success rate and flaky test count
- Code coverage trends
- Bug escape rate from testing
- Time spent on testing vs development

### Process Evaluation
- What testing practices worked well this month?
- What testing challenges did we face?
- Which tools helped or hindered our testing efforts?
- How effective were our test reviews?

### Quality Assessment
- Did our tests catch issues before production?
- Were there any production bugs that should have been caught by tests?
- How was the quality of our test code?
- Did our tests help or hinder refactoring efforts?

### Improvement Actions
- What should we start doing in our testing process?
- What should we stop doing that isn't adding value?
- What should we continue doing that's working well?
- What tools or training do we need?

### Next Month Goals
- Specific testing improvements to implement
- Test coverage targets
- New testing techniques to try
- Knowledge sharing sessions to organize
```

### Testing Maturity Assessment
```yaml
Testing Maturity Levels:

Level 1 - Basic:
  - Manual testing only
  - No automated tests
  - Testing happens after development
  - No test documentation

Level 2 - Developing:
  - Some unit tests
  - Basic CI/CD with test automation
  - Testing considered during development
  - Basic test documentation

Level 3 - Mature:
  - Comprehensive unit and integration tests
  - TDD/BDD practices adopted
  - Automated testing in pipeline
  - Good test documentation and standards

Level 4 - Advanced:
  - Full test pyramid implemented
  - Performance and security testing automated
  - Test-first development culture
  - Continuous testing optimization

Level 5 - Optimized:
  - AI-assisted testing
  - Self-healing test suites
  - Predictive quality analytics
  - Industry-leading practices

Current Assessment Tools:
- Team self-assessment surveys
- Automated metrics collection
- External testing audits
- Benchmark comparisons
```

---

## 📋 Quick Reference Checklists

### Pre-Development Testing Checklist
```markdown
## Before Writing Code

### Requirements Analysis
- [ ] Acceptance criteria are clear and testable
- [ ] Edge cases and error conditions identified
- [ ] Performance requirements specified
- [ ] Security considerations documented

### Test Planning
- [ ] Test approach decided (TDD, test-after, etc.)
- [ ] Test data requirements identified
- [ ] Dependencies and external services identified
- [ ] Test environment requirements clarified

### Infrastructure Ready
- [ ] Test databases accessible
- [ ] CI/CD pipeline configured
- [ ] Test data factories available
- [ ] Mocking strategies defined
```

### Code Review Testing Checklist
```markdown
## During Code Review

### Test Coverage
- [ ] New code has appropriate tests
- [ ] Tests cover both positive and negative scenarios
- [ ] Edge cases are tested
- [ ] Error handling is tested

### Test Quality
- [ ] Tests are readable and maintainable
- [ ] Tests follow naming conventions
- [ ] Tests are focused and not overly complex
- [ ] Proper assertions used (specific, not just truthy)

### Test Independence
- [ ] Tests can run in any order
- [ ] Tests clean up after themselves
- [ ] No shared mutable state between tests
- [ ] External dependencies properly mocked
```

### Production Deployment Testing Checklist
```markdown
## Before Production Release

### Test Execution
- [ ] All unit tests passing
- [ ] Integration tests passing
- [ ] E2E tests passing
- [ ] Performance tests meeting benchmarks
- [ ] Security scans completed

### Quality Gates
- [ ] Code coverage meets thresholds
- [ ] No critical or high severity bugs
- [ ] Flaky test count within acceptable limits
- [ ] Performance regression checks passed

### Documentation
- [ ] Test results documented and reviewed
- [ ] Known issues documented
- [ ] Rollback procedures tested
- [ ] Monitoring and alerting configured
```

---

## 🎯 Conclusion

การทดสอบที่มีประสิทธิภาพเป็นรากฐานสำคัญของการพัฒนา software ที่มีคุณภาพ การมีกลยุทธ์การทดสอบที่ครอบคลุมจะช่วยให้ทีมสามารถพัฒนาและส่งมอบ software ได้อย่างมั่นใจ

### จุดสำคัญที่ต้องจำ:

1. **Test Pyramid เป็นแนวทาง**: มี unit tests มาก, integration tests ปานกลาง, E2E tests น้อย
2. **เริ่มจากพื้นฐาน**: สร้างนิสัยการเขียน test ควบคู่กับ production code
3. **Quality over Quantity**: tests ที่ดี 80% ดีกว่า tests ที่ไม่ดี 100%
4. **Automation คือกุญแจ**: Automate ทุกอย่างที่ทำได้เพื่อ fast feedback
5. **ปรับปรุงอย่างต่อเนื่อง**: วิเคราะห์และปรับปรุงกระบวนการทดสอบอย่างสม่ำเสมอ

### การเริ่มต้น:
1. เริ่มจาก unit testing framework
2. เพิ่ม integration tests ทีละน้อย
3. Setup CI/CD pipeline พร้อม automated testing
4. ค่อยๆ เพิ่ม E2E และ performance testing
5. สร้างวัฒนธรรมการทดสอบในทีม

**Testing ไม่ใช่แค่การหา bugs แต่เป็นการสร้างความมั่นใจในการส่งมอบคุณภาพให้ผู้ใช้** 🚀

---

*คู่มือนี้เป็น living document ที่ควรปรับปรุงตามประสบการณ์และเทคโนโลยีใหม่ๆ*
