# Test Plan - CP-202

**QA Engineer**: QA Team  
**Date**: 2026-01-29  
**Status**: Ready for Testing

---

## Test Summary

**Feature**: Username/Password Login  
**Test Type**: Functional, Security, Performance, Usability  
**Platforms**: iOS, Android  
**Estimated Effort**: 3 days

---

## Test Scenarios

### 1. Registration Flow

#### TC-001: Successful Registration
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Preconditions**:
- App installed and opened
- User not logged in

**Test Steps**:
1. Navigate to Login screen
2. Tap "Sign Up" link
3. Enter valid email: "newuser@example.com"
4. Enter valid password: "Test123!@#"
5. Enter matching confirm password: "Test123!@#"
6. Check "I agree to Terms & Conditions"
7. Tap "Sign Up" button

**Expected Results**:
- ✓ Loading indicator appears
- ✓ Registration successful
- ✓ User automatically logged in
- ✓ Redirected to home screen
- ✓ Welcome message displayed
- ✓ Verification email sent

**Test Data**:
- Email: newuser@example.com
- Password: Test123!@#

---

#### TC-002: Registration - Invalid Email
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Register screen
2. Enter invalid email: "notanemail"
3. Enter valid password
4. Tap "Sign Up"

**Expected Results**:
- ✓ Inline error message: "Invalid email format"
- ✓ Email field highlighted in red
- ✓ Sign Up button disabled
- ✓ Form not submitted

---

#### TC-003: Registration - Weak Password
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Register screen
2. Enter valid email
3. Enter weak password: "123"
4. Observe password strength indicator

**Expected Results**:
- ✓ Password strength: "Weak" (red)
- ✓ Error message: "Password must be at least 8 characters"
- ✓ Sign Up button disabled

**Test Data**:
| Password | Expected Strength |
|----------|-------------------|
| 123 | Weak (red) |
| test1234 | Medium (yellow) |
| Test1234 | Strong (orange) |
| Test123!@# | Very Strong (green) |

---

#### TC-004: Registration - Password Mismatch
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Register screen
2. Enter password: "Test123!@#"
3. Enter confirm password: "Different123"
4. Tap "Sign Up"

**Expected Results**:
- ✓ Error message: "Passwords do not match"
- ✓ Confirm password field highlighted
- ✓ Form not submitted

---

#### TC-005: Registration - Email Already Exists
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Preconditions**:
- Email "existing@example.com" already registered

**Test Steps**:
1. Navigate to Register screen
2. Enter email: "existing@example.com"
3. Enter valid password
4. Tap "Sign Up"

**Expected Results**:
- ✓ Error message: "Email already registered"
- ✓ Suggestion to login instead
- ✓ Link to login screen

---

### 2. Login Flow

#### TC-006: Successful Login
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Preconditions**:
- User account exists
- User not logged in

**Test Steps**:
1. Open app
2. Navigate to Login screen
3. Enter email: "test@example.com"
4. Enter password: "Test123!@#"
5. Tap "Login" button

**Expected Results**:
- ✓ Loading indicator appears
- ✓ Login successful
- ✓ Redirected to home screen
- ✓ User data loaded
- ✓ Session token saved locally

---

#### TC-007: Login - Invalid Credentials
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Login screen
2. Enter email: "test@example.com"
3. Enter wrong password: "WrongPassword123"
4. Tap "Login"

**Expected Results**:
- ✓ Error message: "Invalid email or password"
- ✓ Password field cleared
- ✓ User remains on login screen
- ✓ Failed attempt counted

---

#### TC-008: Login - Remember Me
**Priority**: Medium  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Login screen
2. Enter valid credentials
3. Check "Remember Me" checkbox
4. Tap "Login"
5. Close app completely
6. Reopen app

**Expected Results**:
- ✓ User automatically logged in
- ✓ No login screen shown
- ✓ Redirected to home screen

---

#### TC-009: Login - Show/Hide Password
**Priority**: Low  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Login screen
2. Enter password: "Test123"
3. Observe password field (should show dots)
4. Tap eye icon
5. Observe password field

**Expected Results**:
- ✓ Initially: Password hidden (••••••)
- ✓ After tap: Password visible (Test123)
- ✓ Icon changes to "eye-off"
- ✓ Tap again: Password hidden

---

#### TC-010: Login - Rate Limiting
**Priority**: High  
**Type**: Security  
**Automation**: Yes

**Test Steps**:
1. Navigate to Login screen
2. Enter valid email
3. Enter wrong password
4. Tap "Login" (repeat 5 times)
5. Attempt 6th login

**Expected Results**:
- ✓ After 5 failed attempts: Account locked
- ✓ Error message: "Too many failed attempts. Try again in 15 minutes"
- ✓ Login button disabled
- ✓ Timer shown

---

### 3. Forgot Password Flow

#### TC-011: Forgot Password - Success
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Navigate to Login screen
2. Tap "Forgot Password?" link
3. Enter email: "test@example.com"
4. Tap "Send Reset Link"

**Expected Results**:
- ✓ Loading indicator appears
- ✓ Success message: "Reset link sent to test@example.com"
- ✓ Email received with reset link
- ✓ Link expires in 24 hours

---

#### TC-012: Reset Password - Success
**Priority**: High  
**Type**: Functional  
**Automation**: Partial (manual email check)

**Preconditions**:
- Reset link received via email

**Test Steps**:
1. Open reset link from email
2. Enter new password: "NewTest123!@#"
3. Enter confirm password: "NewTest123!@#"
4. Tap "Reset Password"

**Expected Results**:
- ✓ Password reset successful
- ✓ Success message shown
- ✓ Automatically logged in
- ✓ Redirected to home screen
- ✓ Old password no longer works

---

#### TC-013: Reset Password - Expired Link
**Priority**: Medium  
**Type**: Functional  
**Automation**: No (requires time manipulation)

**Preconditions**:
- Reset link older than 24 hours

**Test Steps**:
1. Open expired reset link
2. Attempt to reset password

**Expected Results**:
- ✓ Error message: "Reset link expired"
- ✓ Option to request new link

---

### 4. Logout Flow

#### TC-014: Logout
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Preconditions**:
- User logged in

**Test Steps**:
1. Navigate to Menu/Settings
2. Tap "Logout" button
3. Confirm logout

**Expected Results**:
- ✓ Confirmation dialog shown
- ✓ After confirm: User logged out
- ✓ Session token cleared
- ✓ Redirected to login screen
- ✓ Cannot access protected screens

---

### 5. Token Management

#### TC-015: Token Auto-Refresh
**Priority**: High  
**Type**: Functional  
**Automation**: Partial

**Preconditions**:
- User logged in
- Token about to expire (< 5 mins)

**Test Steps**:
1. Wait for token to be near expiration
2. Make an API request
3. Observe token refresh

**Expected Results**:
- ✓ Token automatically refreshed
- ✓ Request succeeds
- ✓ New token saved
- ✓ User not logged out

---

#### TC-016: Invalid Token - Auto Logout
**Priority**: High  
**Type**: Functional  
**Automation**: Yes

**Test Steps**:
1. Login successfully
2. Manually invalidate token (backend)
3. Make an API request

**Expected Results**:
- ✓ Request fails with 401
- ✓ User automatically logged out
- ✓ Redirected to login screen
- ✓ Error message: "Session expired. Please login again"

---

## Edge Cases

### EC-001: Very Long Email
**Test**: Enter email with 100+ characters  
**Expected**: Validation error or truncation

### EC-002: Special Characters in Password
**Test**: Password with emojis, unicode  
**Expected**: Accepted or clear error message

### EC-003: Rapid Button Tapping
**Test**: Tap "Login" button multiple times rapidly  
**Expected**: Only one request sent, button disabled during loading

### EC-004: Network Interruption During Login
**Test**: Disconnect network mid-login  
**Expected**: Error message, retry option

### EC-005: App Backgrounding During Login
**Test**: Background app during login process  
**Expected**: Resume correctly or show error

### EC-006: Simultaneous Login on Multiple Devices
**Test**: Login with same account on 2 devices  
**Expected**: Both sessions valid OR last login invalidates previous

---

## Performance Testing

### PT-001: Login Response Time
**Metric**: Time from tap "Login" to home screen  
**Target**: < 2 seconds  
**Test Method**: Measure with DevTools

**Test Cases**:
- Fast network (WiFi): _____ ms
- Slow network (3G): _____ ms
- Average: _____ ms

**Pass Criteria**: Average < 2000ms

---

### PT-002: Registration Response Time
**Metric**: Time from tap "Sign Up" to home screen  
**Target**: < 3 seconds  
**Test Method**: Measure with DevTools

---

### PT-003: Password Strength Calculation
**Metric**: Time to calculate and display strength  
**Target**: < 100ms (should not block UI)  
**Test Method**: Measure with DevTools

---

### PT-004: Memory Usage
**Metric**: Memory consumption during auth flow  
**Target**: < 50MB increase  
**Test Method**: Memory profiler

**Measurements**:
- Before login: _____ MB
- After login: _____ MB
- Increase: _____ MB

**Pass Criteria**: Increase < 50MB

---

### PT-005: Frame Rate
**Metric**: FPS during auth screens  
**Target**: 60 FPS  
**Test Method**: Performance overlay

**Measurements**:
- Login screen: _____ FPS
- Register screen: _____ FPS
- Password typing: _____ FPS

**Pass Criteria**: All > 58 FPS (allowing 2 FPS margin)

---

## Security Testing

### ST-001: Password Hashing
**Test**: Inspect network traffic during login  
**Expected**: Password is hashed, not sent in plain text

**Method**:
1. Use Charles Proxy / Wireshark
2. Capture login request
3. Verify password field is hashed

**Pass Criteria**: Password is SHA-256 hashed

---

### ST-002: Token Storage
**Test**: Inspect local storage  
**Expected**: Tokens stored securely (encrypted)

**Method**:
1. Login successfully
2. Inspect device storage (Keychain/KeyStore)
3. Verify tokens are encrypted

**Pass Criteria**: Tokens not readable in plain text

---

### ST-003: SQL Injection
**Test**: Enter SQL injection strings in email field  
**Expected**: Properly escaped, no injection

**Test Data**:
```
' OR '1'='1
'; DROP TABLE users; --
admin'--
```

**Pass Criteria**: All treated as literal strings, no injection

---

### ST-004: XSS Attack
**Test**: Enter XSS payloads in input fields  
**Expected**: Properly sanitized

**Test Data**:
```
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
```

**Pass Criteria**: Displayed as text, no script execution

---

## Compatibility Testing

### iOS Testing

| Device | OS Version | Status | Notes |
|--------|------------|--------|-------|
| iPhone SE (2nd gen) | iOS 15.0 | ⏳ | Small screen |
| iPhone 13 | iOS 16.0 | ⏳ | Standard |
| iPhone 14 Pro | iOS 17.0 | ⏳ | Latest |
| iPad Air | iOS 17.0 | ⏳ | Tablet |

### Android Testing

| Device | OS Version | Status | Notes |
|--------|------------|--------|-------|
| Samsung Galaxy S10 | Android 11 | ⏳ | Popular device |
| Google Pixel 5 | Android 12 | ⏳ | Stock Android |
| Google Pixel 7 | Android 13 | ⏳ | Recent |
| Google Pixel 8 | Android 14 | ⏳ | Latest |

---

## Usability Testing

### UT-001: UI Clarity
- [ ] All labels clearly visible
- [ ] Input fields have clear placeholders
- [ ] Error messages are helpful
- [ ] Button states are obvious (enabled/disabled)
- [ ] Loading states are clear

### UT-002: Touch Targets
- [ ] All buttons meet 44x44 pt minimum
- [ ] Adequate spacing between elements
- [ ] No accidental taps

### UT-003: Accessibility
- [ ] Screen reader compatible
- [ ] Sufficient color contrast (WCAG AA)
- [ ] Font sizes readable
- [ ] Works with large text settings

### UT-004: Error Recovery
- [ ] Clear error messages
- [ ] Suggestions for fixing errors
- [ ] Easy to retry after errors

---

## Automation Plan

### Unit Tests (70%)
**Coverage Target**: 90%

**Files to Test**:
- All use cases
- Repository
- API client
- Local storage
- BLoC

**Tools**: flutter_test, mockito

---

### Widget Tests (20%)
**Coverage Target**: 80%

**Widgets to Test**:
- EmailInputField
- PasswordInputField
- PasswordStrengthIndicator
- AuthButton
- LoginPage
- RegisterPage
- ForgotPasswordPage

**Tools**: flutter_test

---

### Integration Tests (10%)
**Flows to Test**:
- Complete registration flow
- Complete login flow
- Forgot password flow
- Logout flow

**Tools**: integration_test

---

## Manual Testing Checklist

### Pre-Testing
- [ ] Build debug APK
- [ ] Build debug IPA
- [ ] Setup test accounts
- [ ] Prepare test data
- [ ] Setup test environment

### Functional Testing
- [ ] All test cases executed
- [ ] Edge cases tested
- [ ] Error scenarios tested

### Performance Testing
- [ ] Response times measured
- [ ] Memory usage checked
- [ ] FPS verified
- [ ] No memory leaks

### Security Testing
- [ ] Password hashing verified
- [ ] Token storage verified
- [ ] Injection attacks tested
- [ ] Rate limiting verified

### Compatibility Testing
- [ ] All iOS devices tested
- [ ] All Android devices tested
- [ ] Different screen sizes tested

### Usability Testing
- [ ] UI clarity verified
- [ ] Touch targets adequate
- [ ] Accessibility checked
- [ ] Error recovery tested

---

## Bug Tracking

### Bug Template
```markdown
## BUG-XXX: [Title]

**Severity**: Critical/High/Medium/Low
**Priority**: P0/P1/P2/P3
**Found in**: TC-XXX
**Environment**: iOS 17.0 / iPhone 14 Pro

**Steps to Reproduce**:
1. ...

**Expected**: ...
**Actual**: ...

**Screenshots**: [attach]
**Logs**: [attach]
```

---

## Test Metrics

### Coverage Metrics
```
Total Test Cases: 50
- Functional: 30
- Performance: 5
- Security: 4
- Compatibility: 8
- Usability: 3

Automation:
- Automated: 35 (70%)
- Manual: 15 (30%)
```

### Execution Metrics
```
Executed: _____ / 50
Passed: _____
Failed: _____
Blocked: _____
Pass Rate: _____%
```

### Bug Metrics
```
Total Bugs: _____
- Critical: _____
- High: _____
- Medium: _____
- Low: _____

Status:
- Open: _____
- Fixed: _____
- Verified: _____
```

---

## Test Schedule

### Day 1: Automated Testing
- Morning: Run all unit tests
- Afternoon: Run widget tests
- Evening: Run integration tests
- Fix any test failures

### Day 2: Manual Testing
- Morning: Functional testing (TC-001 to TC-014)
- Afternoon: Performance testing
- Evening: Security testing

### Day 3: Compatibility & Final
- Morning: iOS devices testing
- Afternoon: Android devices testing
- Evening: Bug verification, final report

---

## Exit Criteria

- [ ] All P0/P1 test cases executed
- [ ] Test pass rate > 95%
- [ ] No critical/high bugs open
- [ ] All P0/P1 bugs fixed and verified
- [ ] Performance benchmarks met
- [ ] Security tests passed
- [ ] Compatibility verified on all target devices
- [ ] Test coverage > 80%

---

## QA Sign-off Template

```markdown
## QA Sign-off - CP-202

**Feature**: Username/Password Login
**Tested by**: [QA Name]
**Date**: [Date]

**Test Summary**:
- Total test cases: 50
- Executed: _____
- Passed: _____
- Failed: _____
- Pass Rate: _____%

**Coverage**:
- Unit tests: _____%
- Widget tests: _____%
- Integration tests: Complete/Incomplete
- Manual tests: Complete/Incomplete

**Devices Tested**:
- iOS: X devices, Y OS versions
- Android: X devices, Y OS versions

**Performance**:
- Login time: _____ ms (Target: <2000ms)
- FPS: _____ (Target: 60)
- Memory: _____ MB increase (Target: <50MB)

**Security**:
- Password hashing: ✓/✗
- Token storage: ✓/✗
- Rate limiting: ✓/✗
- Injection prevention: ✓/✗

**Bugs**:
- Critical: _____
- High: _____
- Medium: _____
- Low: _____

**Recommendation**: ✅ APPROVED / ⚠️ APPROVED WITH CONDITIONS / ❌ NOT APPROVED

**Notes**: _____

**Signature**: [QA Engineer]
```

---

**QA Team**: Ready to begin testing upon feature completion
