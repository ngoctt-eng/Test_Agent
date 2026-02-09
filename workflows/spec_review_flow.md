---
description: Quy trình review specification và requirements
---

# Spec Review Flow

## Mục đích
Đảm bảo specification được review kỹ lưỡng trước khi bắt đầu development, tránh hiểu sai requirements và thiết kế sai hướng.

## Vai trò tham gia
- **Product Owner/BA**: Cung cấp spec
- **Architecture Expert**: Review technical feasibility
- **Senior Developer**: Review implementation details
- **QA Engineer**: Review testability và acceptance criteria

---

## Quy trình

### Bước 1: Nhận Specification
**Người thực hiện**: Product Owner/BA

**Deliverables**:
- `tasks/{TASK_ID}/requirements.md` - Business requirements
- `tasks/{TASK_ID}/specs.md` - Technical specifications
- Design mockups (nếu có)
- API documentation (nếu có)

**Checklist**:
- [ ] Requirements rõ ràng và đầy đủ
- [ ] Có acceptance criteria cụ thể
- [ ] Có user stories hoặc use cases
- [ ] Có mockups/wireframes cho UI features

---

### Bước 2: Architecture Review
**Người thực hiện**: Architecture Expert

**Nhiệm vụ**:
1. Đọc và phân tích requirements
2. Xác định feature module cần tạo/modify
3. Thiết kế high-level architecture
4. Identify dependencies với features khác
5. Đánh giá technical risks

**Output**:
```markdown
## Architecture Review - {TASK_ID}

### Affected Modules
- `features/campaign` - CREATE NEW
- `features/payment` - MODIFY
- `infrastructure/routing` - UPDATE

### Architecture Design
[Mô tả kiến trúc, layer structure, data flow]

### Dependencies
- Depends on: Payment service API
- Impacts: Reporting module

### Technical Risks
- [ ] High: Complex state synchronization
- [ ] Medium: Performance with large datasets
- [ ] Low: UI complexity

### Recommendations
- Use BLoC for state management
- Implement caching strategy
- Add pagination for lists
```

**Checklist**:
- [ ] Feature fits trong existing architecture
- [ ] Không tạo circular dependencies
- [ ] Có clear separation of concerns
- [ ] Scalable và maintainable

---

### Bước 3: Development Review
**Người thực hiện**: Senior Developer

**Nhiệm vụ**:
1. Review architecture design
2. Break down implementation tasks
3. Estimate effort
4. Identify reusable components
5. Plan code structure

**Output**:
```markdown
## Development Plan - {TASK_ID}

### Implementation Tasks
1. **Domain Layer** (2 days)
   - [ ] Create Campaign entity
   - [ ] Define ICampaignRepository interface
   - [ ] Write GetCampaigns use case
   - [ ] Write CreateCampaign use case

2. **Data Layer** (3 days)
   - [ ] Create CampaignModel with JSON serialization
   - [ ] Implement CampaignRepository
   - [ ] Create CampaignApiClient
   - [ ] Setup Hive local storage

3. **Presentation Layer** (4 days)
   - [ ] Define CampaignBloc events/states
   - [ ] Implement CampaignBloc
   - [ ] Create CampaignListPage
   - [ ] Create CampaignDetailPage
   - [ ] Create CampaignForm widgets

4. **Testing** (2 days)
   - [ ] Unit tests for use cases
   - [ ] BLoC tests
   - [ ] Widget tests
   - [ ] Integration tests

### Effort Estimate
- Total: 11 days
- Complexity: Medium-High

### Reusable Components
- FormField widgets from common/widgets
- LoadingIndicator from common/widgets
- ErrorView from common/widgets

### Technical Considerations
- Use cached_network_image for campaign images
- Implement pull-to-refresh
- Add shimmer loading effect
```

**Checklist**:
- [ ] Tasks được break down rõ ràng
- [ ] Estimate hợp lý
- [ ] Identify được reusable code
- [ ] Plan cho testing

---

### Bước 4: QA Review
**Người thực hiện**: QA Engineer

**Nhiệm vụ**:
1. Review acceptance criteria
2. Thiết kế test scenarios
3. Identify edge cases
4. Plan test data
5. Define quality metrics

**Output**:
```markdown
## Test Plan - {TASK_ID}

### Test Scenarios

#### Happy Path
1. **Create Campaign Successfully**
   - User fills all required fields
   - Submits form
   - Campaign appears in list
   - Success message shown

2. **View Campaign Details**
   - User taps campaign card
   - Details page loads
   - All information displayed correctly

#### Error Cases
1. **Create Campaign - Validation Errors**
   - Empty name field → Show error
   - Invalid date range → Show error
   - Network error → Show retry option

2. **View Campaign - Not Found**
   - Campaign deleted → Show error message
   - Network error → Show cached data

#### Edge Cases
- Very long campaign name (>100 chars)
- Campaign with past end date
- Offline mode
- Slow network
- Large number of campaigns (1000+)

### Test Data
- Valid campaign: name="Test Campaign", dates=valid range
- Invalid campaign: name="", dates=null
- Edge case: name=100 chars, dates=same day

### Automation Plan
- Unit tests: 70% coverage target
- Widget tests: All major UI components
- Integration tests: Create → View → Edit flow

### Manual Testing
- Test on iOS 15, 16, 17
- Test on Android 11, 12, 13, 14
- Test on different screen sizes
- Performance testing with 1000+ items
```

**Checklist**:
- [ ] Acceptance criteria đầy đủ và testable
- [ ] Test scenarios cover happy path và error cases
- [ ] Edge cases được identify
- [ ] Test data được define rõ ràng
- [ ] Automation plan khả thi

---

### Bước 5: Review Meeting
**Người tham gia**: Tất cả stakeholders

**Agenda**:
1. Product Owner present requirements (10 mins)
2. Architecture Expert present design (15 mins)
3. Senior Developer present implementation plan (15 mins)
4. QA Engineer present test plan (10 mins)
5. Q&A và discussion (20 mins)
6. Decision: Approve / Request Changes (10 mins)

**Outcome**:
- ✅ **APPROVED** → Proceed to development
- ⚠️ **APPROVED WITH CHANGES** → Minor updates needed
- ❌ **REJECTED** → Major rework required

---

### Bước 6: Documentation
**Người thực hiện**: Tất cả

**Deliverables**:
- Updated `tasks/{TASK_ID}/requirements.md`
- `tasks/{TASK_ID}/architecture_review.md`
- `tasks/{TASK_ID}/development_plan.md`
- `tasks/{TASK_ID}/test_plan.md`
- Meeting notes

---

## Checklist tổng hợp

### Requirements Quality
- [ ] Clear và unambiguous
- [ ] Measurable acceptance criteria
- [ ] Complete user stories
- [ ] UI/UX mockups available

### Technical Feasibility
- [ ] Fits existing architecture
- [ ] No major technical blockers
- [ ] Dependencies identified
- [ ] Risks assessed

### Implementation Readiness
- [ ] Tasks broken down
- [ ] Effort estimated
- [ ] Resources available
- [ ] Timeline realistic

### Quality Assurance
- [ ] Testable requirements
- [ ] Test scenarios defined
- [ ] Test data prepared
- [ ] Automation planned

### Sign-off
- [ ] Architecture Expert approved
- [ ] Senior Developer approved
- [ ] QA Engineer approved
- [ ] Product Owner approved

---

## Templates

### Requirements Template
```markdown
# {TASK_ID}: {Feature Name}

## Business Context
[Why we need this feature]

## User Stories
As a [user type], I want to [action] so that [benefit]

## Acceptance Criteria
- [ ] Given [context], when [action], then [result]
- [ ] ...

## UI/UX Requirements
[Mockups, wireframes, design specs]

## Technical Requirements
- API endpoints needed
- Data models
- Third-party integrations

## Out of Scope
[What is NOT included in this feature]
```

---

**Remember**: Measure twice, cut once! 📐
