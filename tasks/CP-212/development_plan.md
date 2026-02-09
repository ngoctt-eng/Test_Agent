# Development Plan - CP-212

**Developer**: Senior Developer  
**Date**: 2026-01-29  
**Status**: Ready for Review

---

## Implementation Tasks

### 1. Domain Layer (2 days)

#### 1.1 Create Entities
**Files**:
- `lib/src/features/assistant/domain/entities/chat_message.dart`
- `lib/src/features/assistant/domain/entities/chat_session.dart`
- `lib/src/features/assistant/domain/entities/thinking_process.dart`
- `lib/src/features/assistant/domain/entities/prompt_item.dart`

**Tasks**:
- [ ] Define ChatMessage entity with all properties
- [ ] Define MessageSender, MessageStatus, FeedbackType enums
- [ ] Define ChatSession entity
- [ ] Define ThinkingProcess and ThinkingStep entities
- [ ] Define PromptItem entity
- [ ] Add equality and copyWith methods

**Estimate**: 3 hours

#### 1.2 Define Repository Interface
**File**: `lib/src/features/assistant/domain/repositories/i_assistant_repository.dart`

**Tasks**:
- [ ] Define IAssistantRepository interface
- [ ] Add sendMessage method
- [ ] Add getSessions method
- [ ] Add getSession method
- [ ] Add createSession method
- [ ] Add deleteSession method
- [ ] Add renameSession method
- [ ] Add submitFeedback method
- [ ] Add getPrompts method
- [ ] Add watchSessions stream
- [ ] Add cleanupOldSessions method

**Estimate**: 2 hours

#### 1.3 Write Use Cases
**Files**:
- `lib/src/features/assistant/domain/usecases/send_message.dart`
- `lib/src/features/assistant/domain/usecases/get_sessions.dart`
- `lib/src/features/assistant/domain/usecases/create_session.dart`
- `lib/src/features/assistant/domain/usecases/delete_session.dart`
- `lib/src/features/assistant/domain/usecases/rename_session.dart`
- `lib/src/features/assistant/domain/usecases/submit_feedback.dart`
- `lib/src/features/assistant/domain/usecases/get_prompts.dart`

**Tasks**:
- [ ] Implement SendMessage use case
  - Validate message length (max 355 chars)
  - Validate not empty
  - Call repository
- [ ] Implement GetSessions use case
- [ ] Implement CreateSession use case
- [ ] Implement DeleteSession use case
- [ ] Implement RenameSession use case
  - Validate name length (1-50 chars)
- [ ] Implement SubmitFeedback use case
- [ ] Implement GetPrompts use case

**Estimate**: 5 hours

#### 1.4 Write Unit Tests
**Files**: `test/features/assistant/domain/usecases/*_test.dart`

**Tasks**:
- [ ] Test SendMessage
  - Happy path
  - Empty message
  - Message too long
  - Repository errors
- [ ] Test RenameSession
  - Happy path
  - Empty name
  - Name too long
- [ ] Test other use cases

**Estimate**: 6 hours

**Total Domain Layer**: 16 hours (2 days)

---

### 2. Data Layer (3 days)

#### 2.1 Create Models
**Files**:
- `lib/src/features/assistant/data/models/chat_message_model.dart`
- `lib/src/features/assistant/data/models/chat_session_model.dart`
- `lib/src/features/assistant/data/models/thinking_process_model.dart`
- `lib/src/features/assistant/data/models/prompt_item_model.dart`

**Tasks**:
- [ ] Create ChatMessageModel with JSON serialization
- [ ] Create ChatSessionModel with JSON serialization
- [ ] Create ThinkingProcessModel with JSON serialization
- [ ] Create PromptItemModel with JSON serialization
- [ ] Add toEntity() methods
- [ ] Add fromEntity() methods
- [ ] Run build_runner for code generation

**Estimate**: 4 hours

#### 2.2 Implement API Client
**File**: `lib/src/features/assistant/data/datasources/ai_api_client.dart`

**Tasks**:
- [ ] Implement sendMessageStream() method with streaming support
  - Configure Dio for Server-Sent Events (SSE)
  - Parse streaming response chunks
  - Handle reasoning chunks
  - Handle content delta chunks
  - Handle done event
- [ ] Implement getConversations() method with pagination
- [ ] Implement getConversationMessages() method with pagination
- [ ] Implement deleteConversation() method (if API supports)
- [ ] Implement renameConversation() method (if API supports)
- [ ] Implement submitFeedback() method (if API supports)
- [ ] Implement getPrompts() method (or use Firebase Remote Config)
- [ ] Add error mapping for HTTP status codes (400, 401, 403, 500)
- [ ] Add timeout handling (30 seconds)
- [ ] Create response models (ConversationsResponse, MessagesResponse, etc.)

**Estimate**: 8 hours

#### 2.3 Implement Local Storage
**File**: `lib/src/features/assistant/data/datasources/chat_local_storage.dart`

**Tasks**:
- [ ] Setup Hive boxes for sessions, messages, prompts
- [ ] Implement saveSession()
- [ ] Implement getSessions() with sorting and limit
- [ ] Implement getSession()
- [ ] Implement deleteSession()
- [ ] Implement saveMessage()
- [ ] Implement cleanupOldSessions() (> 60 days)
- [ ] Add caching for prompts

**Estimate**: 5 hours

#### 2.4 Implement Repository
**File**: `lib/src/features/assistant/data/repositories/assistant_repository.dart`

**Tasks**:
- [ ] Implement IAssistantRepository interface
- [ ] Implement sendMessage()
  - Listen to streaming API
  - Parse reasoning chunks and emit as state updates
  - Parse content deltas and build complete message
  - Save complete message locally
  - Handle errors and timeouts
  - Offline fallback (show error, no cached response for new messages)
- [ ] Implement getSessions()
  - Try API first with pagination
  - Fallback to local cache
  - Merge and deduplicate
- [ ] Implement getSession()
  - Load from API with message pagination
  - Cache locally
- [ ] Implement deleteSession()
  - Delete from API (if supported)
  - Delete from local
- [ ] Implement renameSession()
  - Update via API (if supported)
  - Update local cache
- [ ] Implement submitFeedback()
  - Send to API (if supported)
  - Update local message
- [ ] Implement getPrompts()
  - Fetch from API or Firebase Remote Config
  - Cache locally
- [ ] Implement watchSessions()
  - Stream updates from local storage
- [ ] Implement cleanupOldSessions()
- [ ] Add caching strategy
- [ ] Add error handling

**Estimate**: 10 hours

#### 2.5 Write Unit Tests
**Files**: `test/features/assistant/data/*_test.dart`

**Tasks**:
- [ ] Test AIApiClient
  - Mock Dio responses
  - Test error scenarios
  - Test timeout
- [ ] Test ChatLocalStorage
  - Mock Hive
  - Test cleanup logic
- [ ] Test AssistantRepository
  - Mock API client
  - Mock local storage
  - Test caching logic
  - Test error handling

**Estimate**: 6 hours

**Total Data Layer**: 33 hours (4 days)

---

### 3. Presentation Layer (5 days)

#### 3.1 Define BLoC Events & States
**Files**:
- `lib/src/features/assistant/presentation/bloc/assistant_event.dart`
- `lib/src/features/assistant/presentation/bloc/assistant_state.dart`

**Tasks**:
- [ ] Define all assistant events
- [ ] Define all assistant states
- [ ] Add equality for events/states

**Estimate**: 2 hours

#### 3.2 Implement BLoC
**File**: `lib/src/features/assistant/presentation/bloc/assistant_bloc.dart`

**Tasks**:
- [ ] Implement AssistantBloc
- [ ] Handle LoadSessions event
- [ ] Handle LoadSession event with message pagination
- [ ] Handle CreateNewSession event
- [ ] Handle SendMessageRequested event
  - Optimistic UI update (add user message immediately)
  - Listen to streaming response
  - Emit reasoning state updates
  - Emit content delta state updates
  - Build complete AI message from deltas
  - Handle errors and timeouts
- [ ] Handle DeleteSessionRequested event
- [ ] Handle RenameSessionRequested event
- [ ] Handle SubmitFeedbackRequested event
- [ ] Handle LoadPrompts event
- [ ] Handle HideMiniConversation event
- [ ] Add error handling
- [ ] Add loading states
- [ ] Implement session stream

**Estimate**: 12 hours

#### 3.3 Create Reusable Widgets
**Files**:
- `lib/src/features/assistant/presentation/widgets/message_bubble.dart`
- `lib/src/features/assistant/presentation/widgets/thinking_indicator.dart`
- `lib/src/features/assistant/presentation/widgets/thinking_bottomsheet.dart`
- `lib/src/features/assistant/presentation/widgets/chat_input_field.dart`
- `lib/src/features/assistant/presentation/widgets/feedback_buttons.dart`
- `lib/src/features/assistant/presentation/widgets/session_item.dart`

**Tasks**:
- [ ] Create MessageBubble
  - User vs AI styling
  - Markdown rendering
  - Link handling
- [ ] Create ThinkingIndicator
  - Animation
  - Tap to open bottomsheet
- [ ] Create ThinkingBottomsheet
  - Display thinking steps
  - Scrollable content
- [ ] Create ChatInputField
  - Character counter (355 max)
  - Send button state
  - Disable when sending
- [ ] Create FeedbackButtons
  - Like/Dislike buttons
  - One-time selection
  - Visual feedback
- [ ] Create SessionItem
  - Session name
  - Description (2 lines max)
  - Hold to delete

**Estimate**: 8 hours

#### 3.4 Create Chat Page
**File**: `lib/src/features/assistant/presentation/pages/chat_page.dart`

**Tasks**:
- [ ] Create ChatPage UI
- [ ] Add AppBar with session name
  - Editable name
  - Close button
- [ ] Add animation header (first time only)
- [ ] Add messages list
  - ListView.builder
  - Reverse order
  - Pagination
- [ ] Add empty state (welcome message)
- [ ] Add thinking indicator
- [ ] Add input field
- [ ] Connect to AssistantBloc
- [ ] Handle navigation
- [ ] Handle error states

**Estimate**: 8 hours

#### 3.5 Create History Drawer
**File**: `lib/src/features/assistant/presentation/widgets/history_drawer.dart`

**Tasks**:
- [ ] Create HistoryDrawer UI
- [ ] Display max 10 sessions
- [ ] Sort by updatedAt descending
- [ ] Add "New Chat" button
- [ ] Add session items
- [ ] Implement hold to delete
- [ ] Connect to AssistantBloc

**Estimate**: 4 hours

#### 3.6 Create Home Integration Widgets
**Files**:
- `lib/src/features/assistant/presentation/widgets/prompt_item_widget.dart`
- `lib/src/features/assistant/presentation/widgets/mini_conversation_widget.dart`

**Tasks**:
- [ ] Create PromptItemWidget
  - Display prompt title
  - Tap to open chat with prompt
- [ ] Create MiniConversationWidget
  - Floating widget
  - Display last session title
  - Close button
  - Tap to open chat

**Estimate**: 3 hours

#### 3.7 Modify Home Feature
**File**: `lib/src/features/home/presentation/pages/home_page.dart`

**Tasks**:
- [ ] Add floating button for AI Chat
- [ ] Add prompt items section
- [ ] Add mini conversation widget
- [ ] Handle navigation to chat

**Estimate**: 3 hours

#### 3.8 Write Widget Tests
**Files**: `test/features/assistant/presentation/widgets/*_test.dart`

**Tasks**:
- [ ] Test MessageBubble
- [ ] Test ThinkingIndicator
- [ ] Test ChatInputField
- [ ] Test FeedbackButtons
- [ ] Test SessionItem

**Estimate**: 4 hours

#### 3.9 Write BLoC Tests
**File**: `test/features/assistant/presentation/bloc/assistant_bloc_test.dart`

**Tasks**:
- [ ] Test all event handlers
- [ ] Test state transitions
- [ ] Test error scenarios
- [ ] Use bloc_test package

**Estimate**: 5 hours

**Total Presentation Layer**: 51 hours (6 days)

---

### 4. Integration & Routing (1 day)

#### 4.1 Register Dependencies
**File**: `infrastructure/lib/src/di/service_locator.dart`

**Tasks**:
- [ ] Register AIApiClient
- [ ] Register ChatLocalStorage
- [ ] Register AssistantRepository
- [ ] Register all use cases
- [ ] Register AssistantBloc

**Estimate**: 2 hours

#### 4.2 Configure Routes
**File**: `infrastructure/lib/src/routing/app_router.dart`

**Tasks**:
- [ ] Add /ai-chat route
- [ ] Add /ai-chat/:sessionId route
- [ ] Configure route parameters
- [ ] Add deep link handling

**Estimate**: 2 hours

#### 4.3 Update Main App
**File**: `lib/main.dart`

**Tasks**:
- [ ] Initialize assistant dependencies
- [ ] Setup cleanup scheduler (60 days)

**Estimate**: 1 hour

#### 4.4 Integration Testing
**File**: `integration_test/assistant_flow_test.dart`

**Tasks**:
- [ ] Test complete chat flow
- [ ] Test session management
- [ ] Test feedback submission
- [ ] Test navigation

**Estimate**: 3 hours

**Total Integration**: 8 hours (1 day)

---

### 5. Testing & Bug Fixes (2 days)

#### 5.1 Run All Tests
- [ ] Run unit tests
- [ ] Run widget tests
- [ ] Run BLoC tests
- [ ] Run integration tests
- [ ] Check coverage (target: >80%)

**Estimate**: 2 hours

#### 5.2 Manual Testing
- [ ] Test on iOS simulator
- [ ] Test on Android emulator
- [ ] Test on real devices
- [ ] Test all error scenarios
- [ ] Test offline mode
- [ ] Test 60-day cleanup
- [ ] Test multi-device sync

**Estimate**: 5 hours

#### 5.3 Performance Testing
- [ ] Measure message send time
- [ ] Check memory usage
- [ ] Check for memory leaks
- [ ] Verify 60 FPS
- [ ] Test with long conversations

**Estimate**: 3 hours

#### 5.4 Bug Fixes
- [ ] Fix identified bugs
- [ ] Re-test after fixes

**Estimate**: 6 hours

**Total Testing**: 16 hours (2 days)

---

## Effort Estimate Summary

| Phase | Estimate | Days |
|-------|----------|------|
| Domain Layer | 16 hours | 2 |
| Data Layer | 33 hours | 4 |
| Presentation Layer | 51 hours | 6 |
| Integration & Routing | 8 hours | 1 |
| Testing & Bug Fixes | 16 hours | 2 |
| **Total** | **124 hours** | **15 days** |

**Adjusted with buffer**: **16 days** (3 weeks)

---

## Complexity Assessment

**Overall Complexity**: **High**

**Factors**:
- ✅ Clear requirements from design
- ⚠️ **Streaming API complexity** (real-time response parsing)
- ⚠️ **Reasoning display** (thinking process visualization)
- ⚠️ Complex state management (multiple sessions + streaming)
- ⚠️ Real-time chat interactions
- ⚠️ Multi-device synchronization
- ⚠️ Storage management (60 days cleanup)
- ⚠️ Link handling complexity
- ⚠️ **Missing API endpoints** (delete, rename, feedback, prompts)
- ✅ Can leverage existing infrastructure

---

## Reusable Components

### From `common/widgets`
- ✅ LoadingIndicator
- ✅ ErrorView
- ✅ CustomTextField (can extend for chat input)
- ✅ PrimaryButton

### From `infrastructure`
- ✅ Dio client (already configured)
- ✅ Hive setup (already initialized)
- ✅ BLoC base classes
- ✅ Router configuration

### New Reusable Components
- MessageBubble (can be used in other chat features)
- ThinkingIndicator (reusable for AI features)
- ChatInputField (reusable for messaging)
- FeedbackButtons (reusable for ratings)

---

## Technical Considerations

### Performance
- Use const widgets wherever possible
- ListView.builder for message list
- Pagination for history (20 messages at a time)
- Lazy load old messages
- **Optimize streaming response parsing** (avoid blocking UI)
- **Debounce reasoning updates** (avoid too frequent rebuilds)
- Optimize markdown rendering
- Cache images and avatars

### Error Handling
- Network errors → Show retry option
- Timeout (30s) → Show timeout message with retry
- Server errors → Show user-friendly messages
- Validation errors → Show inline messages

### Offline Support
- Show cached sessions when offline
- Queue messages when offline
- Sync when back online
- Show offline indicator

### Storage Management
- Auto-cleanup > 60 days on app start
- Compress old messages
- Index for fast queries
- Monitor storage usage

---

## Dependencies

### New Dependencies to Add

```yaml
dependencies:
  flutter_markdown: ^0.6.18  # For markdown rendering
  url_launcher: ^6.3.1  # For opening links

dev_dependencies:
  mockito: ^5.4.4
  build_runner: ^2.4.13
  bloc_test: ^9.1.7
```

### Installation Steps
```bash
# Add dependencies
flutter pub add flutter_markdown url_launcher
flutter pub add --dev mockito build_runner bloc_test

# Get packages
flutter pub get
```

---

## Risk Mitigation

### Risk 1: Backend AI API Not Ready
**Mitigation**:
- Create mock API client for development
- Use feature flag to enable/disable
- Coordinate with Backend team weekly
- Test with mock responses

### Risk 2: Streaming API Complexity
**Mitigation**:
- Implement robust stream parsing
- Handle partial chunks correctly
- Test with slow/unstable connections
- Add comprehensive error handling
- Fallback to non-streaming if needed

### Risk 3: Missing API Endpoints
**Mitigation**:
- Confirm with Backend team ASAP
- Use Firebase Remote Config for prompts if API not available
- Implement local-only delete/rename if API not ready
- Add feature flags for each capability

### Risk 4: Complex State Management
**Mitigation**:
- Implement streaming if API supports
- Show engaging loading animation
- Set reasonable timeout (30s)
- Provide retry mechanism

### Risk 3: Complex State Management
**Mitigation**:
- Comprehensive BLoC testing
- Clear state transitions
- Use stream-based updates
- Add extensive logging for debugging

### Risk 4: Storage Issues
**Mitigation**:
- Implement auto-cleanup early
- Monitor storage usage
- Test with large datasets
- Add storage limit warnings

### Risk 5: Timeline Overrun
**Mitigation**:
- Daily progress tracking
- Identify blockers early
- Request help when needed
- Reduce scope if necessary (defer nice-to-haves)

---

## Development Workflow

### Day 1-2: Domain Layer
- Morning: Create entities and interfaces
- Afternoon: Write use cases
- Evening: Write unit tests

### Day 3-5: Data Layer
- Day 3: Models and API client
- Day 4: Local storage and repository
- Day 5: Unit tests and bug fixes

### Day 6-10: Presentation Layer
- Day 6: BLoC and basic widgets
- Day 7: Chat page
- Day 8: History drawer and home integration
- Day 9: Polish UI and animations
- Day 10: Widget and BLoC tests

### Day 11: Integration
- Morning: Dependency injection and routing
- Afternoon: Integration tests

### Day 12-13: Testing
- Day 12: Automated and manual testing
- Day 13: Bug fixes and retesting

### Day 14: Buffer
- Final testing
- Documentation
- Code cleanup

---

## Code Review Checkpoints

### Checkpoint 1: After Domain Layer (Day 2)
- Architecture Expert reviews
- Verify no framework dependencies
- Check test coverage

### Checkpoint 2: After Data Layer (Day 5)
- Architecture Expert reviews
- Verify repository implementation
- Check error handling

### Checkpoint 3: After Presentation Layer (Day 10)
- Performance Expert reviews
- Check widget optimization
- Verify BLoC implementation

### Checkpoint 4: Before Merge (Day 13)
- Final code review
- Performance validation
- Security review (link handling)

---

## Definition of Done

- [ ] All acceptance criteria met
- [ ] Unit test coverage > 80%
- [ ] All tests passing
- [ ] No critical/high bugs
- [ ] Code reviewed and approved
- [ ] Performance benchmarks met
- [ ] Documentation updated
- [ ] Ready for QA testing
- [ ] Feature flag configured

---

## Next Steps

1. ⏳ Get approval from Architecture Expert
2. ⏳ Get test plan from QA Engineer
3. ⏳ Coordinate with Backend team for API specs
4. ⏳ Schedule kickoff meeting
5. ⏳ Setup feature branch
6. ⏳ Start Domain Layer implementation

---

**Developer Sign-off**: Ready to start implementation upon approval

**Estimated Start Date**: 2026-02-03  
**Estimated Completion Date**: 2026-02-24 (updated for streaming API complexity)
