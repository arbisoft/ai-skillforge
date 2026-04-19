---
name: exploratory-testing
trigger: "use PROACTIVELY when conducting QA, debugging, or user testing"
description: Comprehensive guide to exploratory testing methodologies, charter development, session reporting, and best practices.
---

# Exploratory Testing Mastery

Practical framework for structured exploratory testing that balances freedom with accountability. Based on ISTQB best practices and real-world application.

## 1. Exploratory Testing Charters

A charter defines the mission and scope for a testing session. It should be specific enough to provide direction, but flexible enough to allow discovery.

### Charter Template

```
**Mission**: [What are you trying to achieve?]
**Scope**: [Specific features/components to explore]
**Risks**: [Known risk areas to investigate]
**Timebox**: [Duration of session, typically 60-90 minutes]
**Reference Materials**: [PRs, requirements, user stories to review]
**Success Metrics**: [What does a successful session look like?]
```

### Example Charters

**Authentication Flow Exploration**
```
Mission: Uncover edge cases in the multi-factor authentication flow
Scope: Login page, 2FA entry, recovery codes, biometric authentication
Risks: Session hijacking, brute force attacks, accessibility issues
Timebox: 75 minutes
Reference Materials: PR #1423, Security RFC-2024-001, User Story AUTH-45
Success Metrics: Discover at least 3 edge cases, document 2 potential security improvements
```

**Checkout Process Resilience**
```
Mission: Test system behavior under failure conditions during checkout
Scope: Payment processing, order confirmation, email notifications
Risks: Payment gateway timeouts, inventory race conditions, network interruptions
Timebox: 90 minutes
Reference Materials: Payment Integration Doc v2.3, Order Service Architecture
Success Metrics: Identify failure recovery gaps, suggest 3 resilience improvements
```

## 2. Test Session Structure

Follow a disciplined approach to ensure productive sessions:

### Pre-Session Preparation
- Review charter and reference materials
- Set up test environment and data
- Configure necessary tools (browser dev tools, network throttling, etc.)
- Document starting state

### Session Execution (90-Minute Format)
```
0-15 min:  Familiarization & hypothesis generation
15-60 min: Deep exploration with varied inputs and sequences
60-75 min: Focus on highest-risk areas identified
75-90 min: Wrap-up and initial findings documentation
```

### Heuristic Test Strategy Model (HTSM)

Use these dimensions to guide exploration:

**FUNCTIONS**: What does it do? What doesn't it do?
**DATA**: Valid/invalid inputs, boundary conditions, data dependencies
**PLATFORMS**: Different browsers, devices, operating systems
**OPERATIONS**: Typical user workflows, edge case scenarios
**TIME**: Performance under load, timing dependencies, timeouts

## 3. Session Reporting

Document findings in a structured format that enables action:

### Minimal Session Report
```
# Exploratory Testing Session Report

**Date**: YYYY-MM-DD
**Tester**: [Name]
**Charter**: [Mission]
**Duration**: [Start Time] - [End Time] ([Total Minutes] min)

## Summary
[2-3 sentence overview of key findings]

## Areas Explored
- [Feature/Component 1]
- [Feature/Component 2]
- [Feature/Component 3]

## Findings

### Critical Issues
- [Issue 1: Brief description with reproduction steps]
- [Issue 2: Brief description with reproduction steps]

### Improvement Opportunities
- [Suggestion 1: Clear, actionable recommendation]
- [Suggestion 2: Clear, actionable recommendation]

### Questions for Team
- [Question 1: Specific question requiring clarification]
- [Question 2: Specific question about expected behavior]

## Time Allocation
- Familiarization: X minutes
- Deep exploration: X minutes
- Focus on risks: X minutes
- Documentation: X minutes

## Confidence Rating
[High/Medium/Low] - [Brief justification]
```

### Example Session Report
```
# Exploratory Testing Session Report

**Date**: 2025-03-15
**Tester**: Jane Doe
**Charter**: Authentication Flow Exploration
**Duration**: 10:00 - 11:15 (75 min)

## Summary
Uncovered three edge cases in 2FA recovery flow and identified accessibility gaps in biometric authentication. Discovered potential session fixation vulnerability when switching authentication methods.

## Areas Explored
- Login with email/password
- SMS-based 2FA
- Authenticator app 2FA
- Biometric authentication (Touch ID)
- Recovery code generation and use
- Authentication method switching

## Findings

### Critical Issues
- Session not invalidated when switching from password to biometric authentication, creating session fixation risk
- Recovery codes not properly rate-limited, enabling brute force attacks
- Biometric fallback not available when Face ID fails multiple times

### Improvement Opportunities
- Implement step-up authentication when switching authentication methods
- Add progressive rate limiting for recovery code attempts
- Provide alternative 2FA method when biometrics fail
- Enhance error messages for failed biometric attempts

### Questions for Team
- What is the expected behavior when a user's biometric enrollment changes (e.g., new phone)?
- Are there plans to support security keys as a 2FA option?

## Time Allocation
- Familiarization: 12 minutes
- Deep exploration: 48 minutes
- Focus on risks: 10 minutes
- Documentation: 5 minutes

## Confidence Rating
Medium - Covered primary attack vectors but limited time to test edge network conditions
```

## 4. Exploratory Testing Techniques

### Touring Heuristics
Apply these mental models during exploration:

**Bug Taxonomy Tour**: Systematically test for common bug patterns:
- Input validation failures
- Error handling gaps
- State management issues
- Race conditions
- Security vulnerabilities
- Accessibility barriers

**Quality Attributes Tour**: Focus on non-functional aspects:
- Performance under various loads
- Usability across different user personas
- Reliability during network interruptions
- Security against common attack vectors
- Compatibility across platforms

**Data Life Cycle Tour**: Follow data from creation to deletion:
- Data entry and validation
- Data storage and encryption
- Data processing and transformation
- Data display and formatting
- Data sharing and export
- Data deletion and retention

### Session-Based Test Management (SBTM)

Track exploratory testing as a structured practice:

```
| Session | Charter | Tester | Date | Duration | Bugs Found | Status |
|---------|---------|--------|------|----------|------------|--------|
| 001 | Auth Flow Exploration | Jane Doe | 2025-03-15 | 75 min | 3 | Complete |
| 002 | Checkout Resilience | John Smith | 2025-03-16 | 90 min | 5 | Complete |
| 003 | Search Relevance | Jane Doe | 2025-03-17 | 60 min | 2 | In Progress |
```

## 5. Integrating with Agile Workflow

### When to Use Exploratory Testing
- After major feature implementation
- Before production releases
- In response to user-reported issues
- Following security incidents
- When automated tests fail to catch critical bugs

### Complementary Practices
- Pair testing with developers or product owners
- Bug bash sessions before releases
- Context-driven test tours for new team members
- Exploratory testing workshops to share findings

## 6. Advanced Techniques

### Exploratory Automation
Combine exploratory principles with automation:

```javascript
// Randomized input generator for form testing
function generateRandomUser() {
  return {
    email: `${randomString(8)}@${randomDomain()}`,
    password: generateStrongPassword(),
    age: randomInt(13, 100),
    phone: generatePhoneNumber()
  };
}

// Property-based testing for API endpoints
function testAPIEndpointProperty(endpoint, property, generator) {
  const testCases = Array.from({length: 100}, () => generator());
  
  testCases.forEach(testCase => {
    const response = callAPI(endpoint, testCase);
    assert(property(response, testCase), 
      `Property violation for input: ${JSON.stringify(testCase)}`);
  });
}
```

### Cognitive Bias Mitigation

Common biases and countermeasures:

**Confirmation Bias**: Seek to disprove hypotheses, not confirm them
**Automation Bias**: Don't trust automated results blindly; verify manually
**Anchoring Bias**: Don't fixate on first observed behavior; explore alternatives
**Availability Bias**: Don't focus only on obvious test areas; use checklists

## 7. References

- ISTQB Advanced Level Test Analyst Syllabus
- Cem Kaner's "Exploratory Testing" concepts
- James Bach's "Session-Based Test Management"
- "Lessons Learned in Software Testing" by Cem Kaner, James Bach, and Bret Pettichord