# Architecture Review - CP-212

**Reviewer**: Architecture Expert  
**Date**: 2026-01-29  
**Status**: 🔄 PENDING REVIEW

---

## Affected Modules

### New Module
- `features/assistant` - **CREATE/EXTEND**
  - AI Chat interface
  - Session management
  - Prompt handling
  - Feedback system

### Modified Infrastructure
- `infrastructure/routing` - **UPDATE**
  - Add routes: `/ai-chat`, `/ai-chat/:sessionId`
  
- `infrastructure/di` - **UPDATE**
  - Register AI chat dependencies
  - Register API clients for AI service

### Impacted Features
- `features/home` - **MODIFY**
  - Add floating button for AI Chat
  - Add prompt items display
  - Add mini conversation widget
- All features may integrate with AI assistant via deep links

---

## Architecture Design

### Layer Structure

```
┌─────────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                     │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │          AssistantBloc (State Management)        │  │
│  │  - Manages chat sessions                         │  │
│  │  - Handles message flow                          │  │
│  │  - Coordinates use cases                         │  │
│  └──────────────────────────────────────────────────┘  │
│                          │                              │
│  ┌───────────┬──────────┴──────────┬──────────────┐   │
│  │ ChatPage  │ HistoryDrawer │ PromptWidget      │   │
│  └───────────┴─────────────────────┴──────────────┘   │
└──────────────────────────┬───────────────────────────────┘
                           │ Uses
┌──────────────────────────▼───────────────────────────────┐
│                     DOMAIN LAYER                         │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         IAssistantRepository (Interface)         │  │
│  │  - Contract for AI chat operations               │  │
│  │  - No framework dependencies                     │  │
│  └──────────────────────────────────────────────────┘  │
│                          ▲                              │
│  ┌───────────┬──────────┴──────────┬──────────────┐   │
│  │SendMessage│GetSessions │SubmitFeedback │       │   │
│  │  - Business logic                                │  │
│  │  - Validation rules                              │  │
│  │  - Domain rules                                  │  │
│  └───────────┴─────────────────────┴──────────────┘   │
└──────────────────────────┬───────────────────────────────┘
                           │ Implements
┌──────────────────────────▼───────────────────────────────┐
│                      DATA LAYER                          │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │      AssistantRepository (Implementation)        │  │
│  │  - Coordinates data sources                      │  │
│  │  - Handles caching strategy                      │  │
│  │  - Error handling                                │  │
│  └────────────┬─────────────────────┬────────────────┘  │
│               │                     │                    │
│  ┌────────────▼──────────┐  ┌──────▼────────────────┐  │
│  │  AIApiClient          │  │ ChatLocalStorage      │  │
│  │  - REST API calls     │  │ - Hive storage        │  │
│  │  - Dio client         │  │ - Session cache       │  │
│  │  - Streaming support  │  │ - Message history     │  │
│  └───────────────────────┘  └───────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### Data Flow

#### Send Message Flow
```
User Input (Message)
    │
    ▼
ChatPage dispatches SendMessageRequested event
    │
    ▼
AssistantBloc receives event
    │
    ▼
AssistantBloc calls SendMessage use case
    │
    ▼
SendMessage validates input (max 355 chars)
    │
    ▼
SendMessage calls IAssistantRepository.sendMessage()
    │
    ▼
AssistantRepository coordinates:
    ├─► AIApiClient.sendMessage() → AI Server
    │       │
    │       ▼
    │   Returns AIResponse (with thinking process)
    │
    └─► ChatLocalStorage.saveMessage() → Local DB
    │
    ▼
AssistantBloc emits MessageReceived state
    │
    ▼
ChatPage displays AI response with thinking indicator
```

#### Session Management Flow
```
Open Chat
    │
    ▼
Check if active session exists
    │
    ├─► Yes: Load existing session
    │       └─► Display conversation history
    │
    └─► No: Create new session
            └─► Show welcome message
```

---

## Dependencies

### Internal Dependencies

```
features/assistant
    ├─► infrastructure/routing (go_router)
    ├─► infrastructure/state (flutter_bloc)
    ├─► infrastructure/network (Dio)
    ├─► infrastructure/storage (Hive)
    └─► infrastructure/analytics (tracking)

features/home
    ├─► features/assistant (navigate to chat)
```

### External Dependencies

```yaml
# Existing (already in project)
flutter_bloc: ^8.1.6
dio: ^5.7.0
hive: ^2.2.3
go_router: ^14.4.1
intl: (for date formatting)
uuid: ^4.5.1 (for session IDs)

# New dependencies (if needed)
markdown: ^7.0.0  # For rendering markdown in AI responses
url_launcher: ^6.3.1  # For opening external links
flutter_markdown: (check if already exists)
```

**Dependency Rule Compliance**: ✅
- Presentation → Domain → Data
- Domain has NO dependencies on other layers
- Infrastructure is shared across features

---

## Technical Risks

### 🔴 High Risk: AI Response Latency

**Risk**: AI responses có thể chậm, gây UX issues

**Mitigation**:
- ✅ Implement streaming response (if API supports)
- ✅ Show typing indicator với animation
- ✅ Timeout sau 30 seconds
- ✅ Retry mechanism với exponential backoff
- ✅ Cache responses locally

### 🟡 Medium Risk: Complex State Management

**Risk**: Chat state phải sync giữa multiple sessions và UI components

**Mitigation**:
- ✅ Centralized AssistantBloc
- ✅ Stream-based message watching
- ✅ Clear state transitions
- ✅ Comprehensive BLoC testing
- ✅ Optimistic UI updates

### 🟡 Medium Risk: Data Synchronization

**Risk**: Multi-device sync có thể gây conflicts

**Mitigation**:
- ✅ Server as source of truth
- ✅ Conflict resolution strategy (last-write-wins)
- ✅ Sync on app foreground
- ✅ Clear sync indicators

### 🟡 Medium Risk: Storage Limitations

**Risk**: 60 days history có thể chiếm nhiều storage

**Mitigation**:
- ✅ Auto-cleanup messages > 60 days
- ✅ Compress old messages
- ✅ Pagination for history loading
- ✅ Monitor storage usage

### 🟢 Low Risk: Link Handling

**Risk**: In-app vs external links cần xử lý khác nhau

**Mitigation**:
- ✅ URL parsing utility
- ✅ Deep link handler
- ✅ WebView for external links
- ✅ Security validation for URLs

---

## Design Decisions

### Decision 1: Create New Feature vs Extend Existing

**Options**:
1. Create new `features/ai_chat` module
2. Extend existing `features/assistant` module
3. Add to `features/chatting` module

**Decision**: **Create new `features/assistant` module** (or extend if exists)

**Rationale**:
- AI assistant is distinct from regular chat
- Separate concerns from messaging
- Easier to maintain and test
- Can reuse for other AI features later

### Decision 2: Message Storage Strategy

**Options**:
1. Store all messages locally + sync with server
2. Server-only storage with local cache
3. Hybrid: Recent messages local, old messages server

**Decision**: **Hybrid approach**

**Rationale**:
- Fast access to recent conversations
- Reduce local storage usage
- Server backup for multi-device
- Offline support for recent chats

### Decision 3: Thinking Process Display

**Options**:
1. Show thinking inline in message
2. Show in separate bottomsheet
3. Don't show thinking process

**Decision**: **Bottomsheet after response complete**

**Rationale**:
- Doesn't clutter main chat UI
- User can choose to view details
- Follows requirements specification
- Better UX for long thinking processes

### Decision 4: Session Naming

**Options**:
1. Auto-generate from first message
2. User manually names
3. Timestamp-based naming

**Decision**: **Auto-generate with manual edit option**

**Rationale**:
- Reduces friction for users
- Contextual names are more useful
- Flexibility to customize
- Follows requirements

### Decision 5: Prompt Items Integration

**Options**:
1. Fetch from API dynamically
2. Hardcode in app
3. Remote config (Firebase)

**Decision**: **Fetch from API with cache**

**Rationale**:
- Admin can update without app release
- Personalization possible
- A/B testing capability
- Cache for offline access

---

## Performance Considerations

### Network Optimization
- ✅ Cache AI responses locally
- ✅ Debounce send button (prevent spam)
- ✅ Request compression
- ✅ Timeout: 30 seconds
- ✅ Retry logic for failed requests

### UI Performance
- ✅ Use const widgets
- ✅ ListView.builder for message list
- ✅ Lazy load history (pagination)
- ✅ Optimize markdown rendering
- ✅ Image caching for avatars

### Memory Management
- ✅ Dispose TextEditingControllers
- ✅ Cancel stream subscriptions
- ✅ Clear old messages from memory
- ✅ Limit in-memory message count (e.g., 100)

### Storage Optimization
- ✅ Auto-cleanup > 60 days
- ✅ Compress message content
- ✅ Index for fast queries
- ✅ Limit session count (max 10 in history)

---

## Scalability

### Horizontal Scalability
- ✅ Stateless design
- ✅ API-based architecture
- ✅ Can handle millions of users

### Feature Extensibility
- ✅ Easy to add new AI capabilities
- ✅ Easy to add voice input
- ✅ Easy to add image attachments
- ✅ Repository pattern allows swapping AI providers

---

## Testing Strategy

### Unit Tests
- Domain layer: 95% coverage target
- Use cases: Test all validation rules
- Repository: Mock API and storage

### Integration Tests
- Complete chat flow
- Session management
- Feedback submission
- Offline scenarios

### Widget Tests
- Message bubbles
- Input field
- History drawer
- Prompt items

### E2E Tests
- Send message → Receive response
- Create session → View history
- Submit feedback
- Navigate from Home → Chat

---

## Recommendations

### Must Have (P0)
- ✅ Core chat functionality
- ✅ Session management
- ✅ Message history (60 days)
- ✅ Thinking process display
- ✅ Feedback system (Like/Dislike)
- ✅ Error handling with retry
- ✅ Offline support

### Should Have (P1)
- ✅ Prompt items from admin
- ✅ Mini conversation widget
- ✅ Link handling (in-app vs external)
- ✅ Multi-device sync
- ✅ Animation header (first time)

### Nice to Have (P2)
- ⚠️ Voice input (defer to next phase)
- ⚠️ Image attachments (defer)
- ⚠️ Export conversation (defer)

### Won't Have (P3)
- ❌ Video attachments
- ❌ Group chat with AI
- ❌ AI voice responses

---

## Architecture Compliance Checklist

- [x] ✅ Follows Clean Architecture principles
- [x] ✅ Proper layer separation (Presentation → Domain → Data)
- [x] ✅ Domain layer has no framework dependencies
- [x] ✅ Repository pattern implemented correctly
- [x] ✅ No circular dependencies
- [x] ✅ Dependency injection configured
- [x] ✅ Consistent with existing codebase structure
- [x] ✅ Scalable and maintainable design

---

## Approval

**Architecture Expert**: 🔄 PENDING REVIEW

**Comments**:
_To be filled after review_

**Conditions**:
- Coordinate with Backend team for AI API specifications
- Performance testing required for message rendering
- Security review for link handling

**Next Steps**:
1. Senior Developer creates technical specifications
2. Senior Developer creates development plan
3. QA Engineer creates test plan
4. Schedule review meeting
