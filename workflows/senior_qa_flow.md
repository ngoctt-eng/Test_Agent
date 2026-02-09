---
description: Quy trình QA testing và quality assurance
---

# Senior QA Flow

## Mục đích
Quy trình testing toàn diện để đảm bảo chất lượng sản phẩm trước khi release.

## Vai trò tham gia
- **QA Engineer**: Lead testing activities
- **Senior Developer**: Support với technical issues
- **Performance Expert**: Validate performance metrics

---

## Quy trình QA Testing

### Phase 1: Test Preparation
**Người thực hiện**: QA Engineer

**Bước 1.1: Review Documentation**
- [ ] Đọc `tasks/{TASK_ID}/requirements.md`
- [ ] Đọc `tasks/{TASK_ID}/test_plan.md`
- [ ] Review acceptance criteria
- [ ] Understand feature scope

**Bước 1.2: Setup Test Environment**
```bash
# Checkout feature branch
git checkout feature/{TASK_ID}-{feature-name}

# Install dependencies
flutter pub get

# Build app
flutter build apk --debug
flutter build ios --debug
```

**Bước 1.3: Prepare Test Data**
- [ ] Create test accounts
- [ ] Prepare test datasets
- [ ] Setup mock API responses (if needed)
- [ ] Configure test environment variables

---

### Phase 2: Automated Testing

**Bước 2.1: Run Unit Tests**
```bash
# Run all unit tests
flutter test

# Run with coverage
flutter test --coverage

# Generate coverage report
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

**Verification:**
- [ ] All unit tests pass
- [ ] Test coverage > 80%
- [ ] No flaky tests
- [ ] Test execution time reasonable

**Bước 2.2: Run Widget Tests**
```bash
# Run widget tests
flutter test test/presentation/widgets/

# Run specific widget test
flutter test test/presentation/widgets/campaign_card_test.dart
```

**Verification:**
- [ ] All widget tests pass
- [ ] UI components render correctly
- [ ] User interactions work
- [ ] Edge cases handled

**Bước 2.3: Run BLoC Tests**
```bash
# Run BLoC tests
flutter test test/presentation/bloc/
```

**Verification:**
- [ ] All state transitions correct
- [ ] Events handled properly
- [ ] Error states tested
- [ ] Side effects verified

**Bước 2.4: Run Integration Tests**
```bash
# Run integration tests on device
flutter test integration_test/campaign_flow_test.dart
```

**Verification:**
- [ ] End-to-end flows work
- [ ] Navigation correct
- [ ] Data persistence works
- [ ] API integration successful

---

### Phase 3: Manual Testing - Functional

**Bước 3.1: Happy Path Testing**

**Test Case: Create Campaign**
```
Preconditions: User logged in

Steps:
1. Navigate to Campaigns screen
2. Tap "Create Campaign" button
3. Fill in campaign name: "Test Campaign"
4. Select start date: Today
5. Select end date: +30 days
6. Tap "Submit"

Expected Results:
✓ Loading indicator appears
✓ Success message shown
✓ Campaign appears in list
✓ Campaign details correct

Actual Results: [PASS/FAIL]
Notes: ___________
```

**Bước 3.2: Error Scenario Testing**

**Test Case: Create Campaign - Validation Error**
```
Steps:
1. Navigate to Campaigns screen
2. Tap "Create Campaign" button
3. Leave name field empty
4. Tap "Submit"

Expected Results:
✓ Validation error shown
✓ "Name is required" message
✓ Form not submitted
✓ User stays on form

Actual Results: [PASS/FAIL]
Notes: ___________
```

**Bước 3.3: Edge Case Testing**

Test scenarios:
- [ ] Very long input (>100 characters)
- [ ] Special characters in input
- [ ] Boundary values (min/max dates)
- [ ] Rapid button tapping
- [ ] Back button during loading
- [ ] App backgrounding during operation

---

### Phase 4: Manual Testing - Non-Functional

**Bước 4.1: Performance Testing**

**Metrics to measure:**
```
Screen Load Time:
- Campaign List: _____ ms (target: <500ms)
- Campaign Detail: _____ ms (target: <300ms)

Scroll Performance:
- FPS during scroll: _____ (target: 60 FPS)
- Jank detected: Yes/No

Memory Usage:
- Initial: _____ MB
- After 5 min usage: _____ MB
- Memory leak: Yes/No

Network Performance:
- API response time: _____ ms
- Offline mode works: Yes/No
```

**Bước 4.2: Compatibility Testing**

**iOS Testing:**
- [ ] iOS 15.0 - iPhone 11
- [ ] iOS 16.0 - iPhone 13
- [ ] iOS 17.0 - iPhone 15
- [ ] iPad (latest)

**Android Testing:**
- [ ] Android 11 - Samsung Galaxy S10
- [ ] Android 12 - Pixel 5
- [ ] Android 13 - Pixel 7
- [ ] Android 14 - Pixel 8

**Screen Sizes:**
- [ ] Small (iPhone SE)
- [ ] Medium (iPhone 14)
- [ ] Large (iPhone 14 Pro Max)
- [ ] Tablet (iPad)

**Bước 4.3: Usability Testing**

Checklist:
- [ ] UI elements clearly visible
- [ ] Text readable (font size, contrast)
- [ ] Touch targets adequate size (44x44 pt)
- [ ] Feedback for user actions
- [ ] Error messages helpful
- [ ] Navigation intuitive
- [ ] Loading states clear

---

### Phase 5: Regression Testing

**Bước 5.1: Identify Affected Features**

Features potentially impacted:
- [ ] Authentication flow
- [ ] Home screen
- [ ] Related features (list them)

**Bước 5.2: Run Regression Test Suite**

```bash
# Run full regression test suite
flutter test test/regression/

# Run smoke tests
flutter test test/smoke/
```

**Bước 5.3: Manual Regression Checks**

Critical flows to verify:
- [ ] User login/logout
- [ ] Main navigation
- [ ] Data synchronization
- [ ] Push notifications
- [ ] Payment flows (if applicable)

---

### Phase 6: Bug Reporting

**Bước 6.1: Document Bugs**

Use bug template:
```markdown
## BUG-{ID}: {Title}

**Severity**: Critical/High/Medium/Low
**Priority**: P0/P1/P2/P3
**Status**: Open

**Environment**:
- Device: iPhone 14 Pro
- OS: iOS 17.2
- App Version: 3.2.3+654
- Build: feature/{TASK_ID}

**Steps to Reproduce**:
1. ...
2. ...

**Expected**: ...
**Actual**: ...

**Screenshots**: [attach]
**Logs**: [attach]
```

**Bước 6.2: Categorize Bugs**

**By Severity:**
- **Critical**: App crash, data loss, security
- **High**: Major feature broken
- **Medium**: Minor feature issue
- **Low**: UI cosmetic issue

**By Priority:**
- **P0**: Blocks release, fix immediately
- **P1**: Fix before release
- **P2**: Fix in next release
- **P3**: Nice to have

**Bước 6.3: Track Bugs**

Create issues in issue tracker:
- Assign to developer
- Set priority and severity
- Link to feature branch
- Add to sprint board

---

### Phase 7: Verification & Sign-off

**Bước 7.1: Verify Bug Fixes**

For each fixed bug:
- [ ] Re-test original scenario
- [ ] Test related scenarios
- [ ] Verify no regression
- [ ] Close bug if fixed

**Bước 7.2: Final Verification**

Checklist:
- [ ] All acceptance criteria met
- [ ] All automated tests pass
- [ ] All manual tests pass
- [ ] No P0/P1 bugs open
- [ ] Performance benchmarks met
- [ ] Regression tests pass
- [ ] Documentation updated

**Bước 7.3: QA Sign-off**

```markdown
## QA Sign-off - {TASK_ID}

**Feature**: Campaign Management
**Tested by**: [QA Name]
**Date**: [Date]

**Test Summary**:
- Total test cases: 45
- Passed: 43
- Failed: 2 (P3 - non-blocking)
- Blocked: 0

**Coverage**:
- Unit tests: 85%
- Integration tests: Complete
- Manual tests: Complete
- Regression tests: Complete

**Devices Tested**:
- iOS: 3 devices, 3 OS versions
- Android: 4 devices, 4 OS versions

**Performance**:
- Load time: ✓ Meets target
- FPS: ✓ 60 FPS
- Memory: ✓ No leaks

**Bugs**:
- Critical: 0
- High: 0
- Medium: 0
- Low: 2 (documented, non-blocking)

**Recommendation**: ✅ APPROVED FOR RELEASE

**Notes**: 
Two low-priority UI alignment issues documented as 
BUG-123 and BUG-124. Can be addressed in next release.

**Signature**: [QA Engineer]
```

---

### Phase 8: Release Testing (Staging)

**Bước 8.1: Deploy to Staging**
```bash
# Deploy to staging environment
# (Usually automated via CI/CD)
```

**Bước 8.2: Smoke Testing on Staging**

Quick verification:
- [ ] App launches successfully
- [ ] User can login
- [ ] New feature accessible
- [ ] Critical flows work
- [ ] No console errors

**Bước 8.3: Stakeholder Demo**

Prepare demo:
- [ ] Demo script ready
- [ ] Test data prepared
- [ ] Happy path demonstrated
- [ ] Edge cases shown
- [ ] Q&A session

**Bước 8.4: Final Approval**

Sign-offs required:
- [ ] QA Engineer approved
- [ ] Product Owner approved
- [ ] Tech Lead approved
- [ ] Stakeholders approved

---

## Quality Metrics Dashboard

### Test Execution Metrics
```
Total Test Cases: _____
Passed: _____
Failed: _____
Blocked: _____
Pass Rate: _____%

Test Coverage:
- Unit: _____%
- Widget: _____%
- Integration: _____%
- Overall: _____%
```

### Bug Metrics
```
Total Bugs Found: _____
By Severity:
- Critical: _____
- High: _____
- Medium: _____
- Low: _____

By Status:
- Open: _____
- In Progress: _____
- Fixed: _____
- Closed: _____
```

### Performance Metrics
```
Load Time (avg): _____ ms
FPS (avg): _____
Memory Usage: _____ MB
API Response Time: _____ ms
```

---

## Checklist Tổng hợp

### Test Preparation
- [x] Test plan reviewed
- [x] Test environment setup
- [x] Test data prepared

### Automated Testing
- [x] Unit tests pass (>80% coverage)
- [x] Widget tests pass
- [x] BLoC tests pass
- [x] Integration tests pass

### Manual Testing
- [x] Functional testing complete
- [x] Performance testing complete
- [x] Compatibility testing complete
- [x] Usability testing complete

### Regression Testing
- [x] Regression suite executed
- [x] No breaking changes
- [x] Critical flows verified

### Bug Management
- [x] All bugs documented
- [x] P0/P1 bugs fixed
- [x] Bug fixes verified

### Sign-off
- [x] QA approved
- [x] Product Owner approved
- [x] Ready for release

---

**Remember**: Quality is everyone's responsibility! ✅
