# Architecture Review - CP-202

**Reviewer**: Architecture Expert  
**Date**: 2026-01-29  
**Status**: ✅ APPROVED

---

## Affected Modules

### New Module
- `features/auth` - **MODIFY** (extend existing auth feature)
  - Add email/password authentication
  - Extend existing Google/Apple auth

### Modified Infrastructure
- `infrastructure/routing` - **UPDATE**
  - Add routes: `/login`, `/register`, `/forgot-password`, `/reset-password`
  
- `infrastructure/di` - **UPDATE**
  - Register new auth dependencies

### Impacted Features
- `features/home` - Navigation after login
- `features/menu` - Account settings
- All features requiring authentication

---

## Architecture Design

### Layer Structure

```
┌─────────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                     │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              AuthBloc (State Management)         │  │
│  │  - Handles all auth events                       │  │
│  │  - Manages auth state                            │  │
│  │  - Coordinates use cases                         │  │
│  └──────────────────────────────────────────────────┘  │
│                          │                              │
│  ┌───────────┬──────────┴──────────┬──────────────┐   │
│  │ LoginPage │ RegisterPage │ ForgotPasswordPage  │   │
│  └───────────┴─────────────────────┴──────────────┘   │
└──────────────────────────┬───────────────────────────────┘
                           │ Uses
┌──────────────────────────▼───────────────────────────────┐
│                     DOMAIN LAYER                         │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              IAuthRepository (Interface)         │  │
│  │  - Contract for auth operations                  │  │
│  │  - No framework dependencies                     │  │
│  └──────────────────────────────────────────────────┘  │
│                          ▲                              │
│  ┌───────────┬──────────┴──────────┬──────────────┐   │
│  │LoginUseCase│RegisterUseCase│ForgotPasswordUseCase│  │
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
│  │         AuthRepository (Implementation)          │  │
│  │  - Coordinates data sources                      │  │
│  │  - Handles caching strategy                      │  │
│  │  - Error handling                                │  │
│  └────────────┬─────────────────────┬────────────────┘  │
│               │                     │                    │
│  ┌────────────▼──────────┐  ┌──────▼────────────────┐  │
│  │  AuthApiClient        │  │ AuthLocalStorage      │  │
│  │  - REST API calls     │  │ - Hive storage        │  │
│  │  - Dio client         │  │ - Secure storage      │  │
│  │  - Error mapping      │  │ - Token management    │  │
│  └───────────────────────┘  └───────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### Data Flow

#### Login Flow
```
User Input (Email/Password)
    │
    ▼
LoginPage dispatches LoginWithEmailRequested event
    │
    ▼
AuthBloc receives event
    │
    ▼
AuthBloc calls LoginWithEmail use case
    │
    ▼
LoginWithEmail validates input
    │
    ▼
LoginWithEmail calls IAuthRepository.loginWithEmail()
    │
    ▼
AuthRepository coordinates:
    ├─► AuthApiClient.login() → Server
    │       │
    │       ▼
    │   Returns AuthSession
    │
    └─► AuthLocalStorage.saveSession() → Local DB
    │
    ▼
AuthBloc emits Authenticated state
    │
    ▼
LoginPage navigates to Home
```

---

## Dependencies

### Internal Dependencies

```
features/auth
    ├─► infrastructure/routing (go_router)
    ├─► infrastructure/state (flutter_bloc)
    ├─► infrastructure/network (Dio)
    ├─► infrastructure/storage (Hive)
    └─► infrastructure/analytics (tracking)

features/home
    ├─► features/auth (check auth status)

features/menu
    ├─► features/auth (logout, account info)
```

### External Dependencies

```yaml
# New dependencies
flutter_secure_storage: ^9.0.0  # Secure token storage
crypto: ^3.0.3                  # Password hashing
email_validator: ^2.1.17        # Email validation

# Existing (already in project)
flutter_bloc: ^8.1.6
dio: ^5.7.0
hive: ^2.2.3
go_router: ^14.4.1
```

**Dependency Rule Compliance**: ✅
- Presentation → Domain → Data
- Domain has NO dependencies on other layers
- Infrastructure is shared across features

---

## Technical Risks

### 🔴 High Risk: Security Vulnerabilities

**Risk**: Password handling, token storage có thể bị exploit

**Mitigation**:
- ✅ Use SHA-256 for client-side password hashing
- ✅ Use flutter_secure_storage for token storage
- ✅ Implement rate limiting
- ✅ Add account lockout mechanism
- ✅ Security review before release
- ✅ Penetration testing

### 🟡 Medium Risk: Complex State Management

**Risk**: Auth state phải sync giữa multiple screens và features

**Mitigation**:
- ✅ Centralized AuthBloc
- ✅ Stream-based auth state watching
- ✅ Clear state transitions
- ✅ Comprehensive BLoC testing

### 🟡 Medium Risk: Token Refresh Complexity

**Risk**: Token expiration handling có thể gây UX issues

**Mitigation**:
- ✅ Auto-refresh khi token < 5 mins to expire
- ✅ Retry failed requests after refresh
- ✅ Clear error messages khi refresh fails
- ✅ Graceful logout on token invalid

### 🟢 Low Risk: API Integration

**Risk**: Backend API có thể chưa ready hoặc có breaking changes

**Mitigation**:
- ✅ Mock API client for testing
- ✅ API contract documentation
- ✅ Coordinate với Backend team
- ✅ Feature flag để enable/disable

---

## Design Decisions

### Decision 1: Extend Existing Auth Feature vs New Feature

**Options**:
1. Create new `features/email_auth` module
2. Extend existing `features/auth` module

**Decision**: **Extend existing auth module**

**Rationale**:
- Auth logic should be centralized
- Avoid duplication of auth state management
- Easier to maintain single AuthBloc
- Consistent auth experience across providers

### Decision 2: Password Hashing Strategy

**Options**:
1. Hash on client-side only
2. Hash on server-side only
3. Hash on both client and server

**Decision**: **Hash on both client and server**

**Rationale**:
- Client-side: Prevent password exposure in transit
- Server-side: Secure storage with bcrypt
- Defense in depth approach

### Decision 3: Token Storage

**Options**:
1. Hive only
2. SharedPreferences
3. flutter_secure_storage

**Decision**: **flutter_secure_storage for tokens + Hive for metadata**

**Rationale**:
- Tokens need encryption at rest
- flutter_secure_storage uses Keychain (iOS) / KeyStore (Android)
- Hive for non-sensitive session metadata
- Best security practice

### Decision 4: State Management

**Options**:
1. Separate BLoC for each auth screen
2. Single AuthBloc for all auth operations

**Decision**: **Single AuthBloc**

**Rationale**:
- Centralized auth state
- Easier to coordinate between screens
- Consistent with existing architecture
- Simpler dependency injection

---

## Performance Considerations

### Network Optimization
- ✅ Cache user data locally
- ✅ Offline mode: Show cached user
- ✅ Retry logic for failed requests
- ✅ Request timeout: 10 seconds

### UI Performance
- ✅ Use const widgets
- ✅ Debounce email validation (300ms)
- ✅ Password strength calculation in isolate
- ✅ Lazy load user avatar

### Memory Management
- ✅ Dispose TextEditingControllers
- ✅ Cancel stream subscriptions
- ✅ Clear sensitive data from memory after use

---

## Scalability

### Horizontal Scalability
- ✅ Stateless design (no in-memory session)
- ✅ Token-based auth (no server-side session)
- ✅ Can scale to millions of users

### Feature Extensibility
- ✅ Easy to add new auth providers (Facebook, Twitter)
- ✅ Easy to add 2FA later
- ✅ Easy to add biometric auth
- ✅ Repository pattern allows swapping implementations

---

## Testing Strategy

### Unit Tests
- Domain layer: 95% coverage target
- Use cases: Test all validation rules
- Repository: Mock API and storage

### Integration Tests
- Complete login flow
- Complete registration flow
- Token refresh flow
- Offline scenarios

### Widget Tests
- All input fields
- Form validation
- Error states
- Loading states

### E2E Tests
- Login → Home navigation
- Register → Email verification → Login
- Forgot password → Reset → Login

---

## Recommendations

### Must Have (P0)
- ✅ Implement all security measures
- ✅ Comprehensive error handling
- ✅ Token auto-refresh
- ✅ Rate limiting

### Should Have (P1)
- ✅ Email verification flow
- ✅ Password strength indicator
- ✅ Remember me functionality
- ✅ Offline support

### Nice to Have (P2)
- ⚠️ Biometric auth (defer to next phase)
- ⚠️ Social login with Facebook (defer)
- ⚠️ 2FA (defer to next phase)

### Won't Have (P3)
- ❌ Username-based login (email only)
- ❌ Phone number auth
- ❌ Magic link login

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

**Architecture Expert**: ✅ APPROVED

**Comments**:
Design follows Clean Architecture principles correctly. Security measures are comprehensive. Token management strategy is solid. Ready for implementation.

**Conditions**:
- Security review required before release
- Coordinate with Backend team for API readiness
- Performance testing required for login flow

**Next Steps**:
1. Senior Developer creates implementation plan
2. QA Engineer creates test plan
3. Schedule review meeting
