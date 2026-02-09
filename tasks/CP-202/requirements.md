# CP-202: Username/Password Login Feature

## Business Context

Hiện tại ứng dụng Passio chỉ hỗ trợ đăng nhập qua Google và Apple Sign-in. Nhiều người dùng yêu cầu có thêm phương thức đăng nhập truyền thống bằng username/password để:
- Không phụ thuộc vào tài khoản Google/Apple
- Dễ dàng quản lý nhiều tài khoản
- Phù hợp với người dùng doanh nghiệp

## User Stories

### US-1: Đăng ký tài khoản mới
**As a** new user  
**I want to** create an account with username and password  
**So that** I can access the app without Google/Apple account

### US-2: Đăng nhập với username/password
**As a** registered user  
**I want to** login with my username and password  
**So that** I can access my account

### US-3: Quên mật khẩu
**As a** user who forgot password  
**I want to** reset my password via email  
**So that** I can regain access to my account

## Acceptance Criteria

### AC-1: Registration Flow
- [ ] User có thể nhập username (email format)
- [ ] User có thể nhập password (min 8 chars, có chữ hoa, chữ thường, số)
- [ ] User có thể nhập confirm password
- [ ] Hiển thị password strength indicator
- [ ] Validate email format real-time
- [ ] Validate password match real-time
- [ ] Show error messages rõ ràng
- [ ] Sau khi đăng ký thành công, tự động login
- [ ] Gửi email verification

### AC-2: Login Flow
- [ ] User có thể nhập username (email)
- [ ] User có thể nhập password
- [ ] Có checkbox "Remember me"
- [ ] Có link "Forgot password"
- [ ] Show/hide password toggle
- [ ] Loading state khi đang login
- [ ] Error handling cho invalid credentials
- [ ] Redirect to home screen sau khi login thành công
- [ ] Save session token locally

### AC-3: Forgot Password Flow
- [ ] User có thể nhập email để reset password
- [ ] Gửi email với reset link
- [ ] Link có expiration time (24 hours)
- [ ] User có thể set new password
- [ ] Validate new password strength
- [ ] Success message sau khi reset
- [ ] Auto login sau khi reset thành công

### AC-4: Security Requirements
- [ ] Password được hash trước khi gửi lên server
- [ ] Session token được encrypt
- [ ] Token auto-refresh khi sắp expire
- [ ] Logout khi token invalid
- [ ] Rate limiting cho login attempts (max 5 attempts/5 minutes)
- [ ] Account lockout sau 5 failed attempts

## UI/UX Requirements

### Login Screen
- Email input field với icon
- Password input field với show/hide toggle
- "Remember me" checkbox
- "Forgot password?" link
- "Login" button (primary)
- "Sign up" link
- Divider với text "OR"
- Google Sign-in button
- Apple Sign-in button

### Registration Screen
- Email input field
- Password input field với strength indicator
- Confirm password input field
- Terms & Conditions checkbox
- "Sign up" button (primary)
- "Already have account? Login" link

### Forgot Password Screen
- Email input field
- "Send reset link" button
- "Back to login" link
- Success message screen

## Technical Requirements

### API Endpoints

```
POST /api/auth/register
Request:
{
  "email": "user@example.com",
  "password": "hashedPassword123",
  "deviceId": "uuid"
}
Response:
{
  "success": true,
  "data": {
    "userId": "123",
    "token": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {
      "email": "user@example.com",
      "name": null
    }
  }
}

POST /api/auth/login
Request:
{
  "email": "user@example.com",
  "password": "hashedPassword123",
  "deviceId": "uuid",
  "rememberMe": true
}
Response:
{
  "success": true,
  "data": {
    "userId": "123",
    "token": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {...}
  }
}

POST /api/auth/forgot-password
Request:
{
  "email": "user@example.com"
}
Response:
{
  "success": true,
  "message": "Reset link sent to email"
}

POST /api/auth/reset-password
Request:
{
  "token": "reset_token",
  "newPassword": "hashedNewPassword123"
}
Response:
{
  "success": true,
  "message": "Password reset successfully"
}

POST /api/auth/refresh-token
Request:
{
  "refreshToken": "refresh_token"
}
Response:
{
  "success": true,
  "data": {
    "token": "new_jwt_token",
    "refreshToken": "new_refresh_token"
  }
}
```

### Data Models

**User**:
- id: String
- email: String
- name: String?
- avatar: String?
- authProvider: enum (email, google, apple)
- createdAt: DateTime
- updatedAt: DateTime

**AuthSession**:
- userId: String
- token: String
- refreshToken: String
- deviceId: String
- expiresAt: DateTime

### Local Storage

Sử dụng Hive để lưu:
- User credentials (encrypted)
- Session token
- Refresh token
- "Remember me" preference

### Third-party Integrations

- **crypto**: Password hashing
- **flutter_secure_storage**: Secure token storage
- **email_validator**: Email validation

## Out of Scope

- ❌ Social login với Facebook, Twitter (chỉ giữ Google/Apple)
- ❌ Two-factor authentication (2FA)
- ❌ Biometric authentication (Face ID, Touch ID)
- ❌ Change password trong app (sẽ làm ở phase sau)
- ❌ Account deletion

## Success Metrics

- 30% users sử dụng email/password login trong tháng đầu
- < 5% failed login rate
- < 2s average login time
- > 90% email verification rate

## Timeline

- Spec Review: 2 days
- Development: 8 days
- Testing: 3 days
- Total: 13 days (2.5 weeks)

## Dependencies

- Backend API endpoints phải ready trước
- Email service (SendGrid/AWS SES) phải được setup
- Firebase Auth có thể tích hợp email/password provider

## Risks

- **High**: Security vulnerabilities nếu implement sai
- **Medium**: UX phức tạp hơn so với social login
- **Low**: Email delivery issues

## Notes

- Cần coordinate với Backend team để đảm bảo API ready on time
- Cần security review trước khi release
- Consider sử dụng Firebase Auth Email/Password provider để giảm complexity
