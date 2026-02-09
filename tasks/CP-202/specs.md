# CP-202: Technical Specifications

## Overview

Implement username/password authentication flow cho Passio app, bao gồm registration, login, và forgot password features.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ LoginPage    │  │ RegisterPage │  │ ForgotPwdPage│  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                  │                  │          │
│         └──────────────────┼──────────────────┘          │
│                            │                             │
│                    ┌───────▼────────┐                    │
│                    │   AuthBloc     │                    │
│                    └───────┬────────┘                    │
└────────────────────────────┼──────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────┐
│                     Domain Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ LoginUseCase │  │RegisterUseCase│ │ResetPwdUseCase│  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                  │                  │           │
│         └──────────────────┼──────────────────┘           │
│                            │                              │
│                    ┌───────▼────────┐                     │
│                    │IAuthRepository │                     │
│                    └───────┬────────┘                     │
└────────────────────────────┼───────────────────────────────┘
                             │
┌────────────────────────────▼───────────────────────────────┐
│                      Data Layer                            │
│                 ┌──────────────────┐                       │
│                 │ AuthRepository   │                       │
│                 └────────┬─────────┘                       │
│          ┌──────────────┴──────────────┐                  │
│          │                              │                  │
│  ┌───────▼────────┐          ┌─────────▼────────┐        │
│  │AuthApiClient   │          │AuthLocalStorage  │        │
│  │(Dio)           │          │(Hive + Secure)   │        │
│  └────────────────┘          └──────────────────┘        │
└────────────────────────────────────────────────────────────┘
```

## Feature Structure

```
lib/src/features/auth/
├── data/
│   ├── models/
│   │   ├── user_model.dart
│   │   ├── auth_session_model.dart
│   │   └── login_request_model.dart
│   ├── repositories/
│   │   └── auth_repository.dart
│   └── datasources/
│       ├── auth_api_client.dart
│       └── auth_local_storage.dart
├── domain/
│   ├── entities/
│   │   ├── user.dart
│   │   └── auth_session.dart
│   ├── repositories/
│   │   └── i_auth_repository.dart
│   └── usecases/
│       ├── login_with_email.dart
│       ├── register_with_email.dart
│       ├── forgot_password.dart
│       ├── reset_password.dart
│       └── refresh_token.dart
└── presentation/
    ├── bloc/
    │   ├── auth_bloc.dart
    │   ├── auth_event.dart
    │   └── auth_state.dart
    ├── pages/
    │   ├── login_page.dart
    │   ├── register_page.dart
    │   ├── forgot_password_page.dart
    │   └── reset_password_page.dart
    └── widgets/
        ├── email_input_field.dart
        ├── password_input_field.dart
        ├── password_strength_indicator.dart
        └── auth_button.dart
```

## Domain Layer Specifications

### Entities

```dart
// domain/entities/user.dart
class User {
  final String id;
  final String email;
  final String? name;
  final String? avatar;
  final AuthProvider authProvider;
  final DateTime createdAt;
  
  const User({
    required this.id,
    required this.email,
    this.name,
    this.avatar,
    required this.authProvider,
    required this.createdAt,
  });
}

enum AuthProvider { email, google, apple }

// domain/entities/auth_session.dart
class AuthSession {
  final String userId;
  final String token;
  final String refreshToken;
  final DateTime expiresAt;
  
  const AuthSession({
    required this.userId,
    required this.token,
    required this.refreshToken,
    required this.expiresAt,
  });
  
  bool get isExpired => DateTime.now().isAfter(expiresAt);
}
```

### Repository Interface

```dart
// domain/repositories/i_auth_repository.dart
abstract class IAuthRepository {
  Future<AuthSession> loginWithEmail({
    required String email,
    required String password,
    required bool rememberMe,
  });
  
  Future<AuthSession> registerWithEmail({
    required String email,
    required String password,
  });
  
  Future<void> forgotPassword(String email);
  
  Future<void> resetPassword({
    required String token,
    required String newPassword,
  });
  
  Future<AuthSession> refreshToken(String refreshToken);
  
  Future<void> logout();
  
  Future<User?> getCurrentUser();
  
  Stream<User?> watchAuthState();
}
```

### Use Cases

```dart
// domain/usecases/login_with_email.dart
class LoginWithEmail {
  final IAuthRepository repository;
  
  LoginWithEmail(this.repository);
  
  Future<AuthSession> call({
    required String email,
    required String password,
    bool rememberMe = false,
  }) async {
    // Validate email
    if (!_isValidEmail(email)) {
      throw InvalidEmailException();
    }
    
    // Validate password
    if (password.length < 8) {
      throw WeakPasswordException();
    }
    
    return await repository.loginWithEmail(
      email: email,
      password: password,
      rememberMe: rememberMe,
    );
  }
  
  bool _isValidEmail(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
}
```

## Data Layer Specifications

### Models

```dart
// data/models/user_model.dart
@JsonSerializable()
class UserModel {
  final String id;
  final String email;
  final String? name;
  final String? avatar;
  @JsonKey(name: 'auth_provider')
  final String authProvider;
  @JsonKey(name: 'created_at')
  final String createdAt;
  
  UserModel({
    required this.id,
    required this.email,
    this.name,
    this.avatar,
    required this.authProvider,
    required this.createdAt,
  });
  
  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
  
  Map<String, dynamic> toJson() => _$UserModelToJson(this);
  
  User toEntity() {
    return User(
      id: id,
      email: email,
      name: name,
      avatar: avatar,
      authProvider: AuthProvider.values.firstWhere(
        (e) => e.name == authProvider,
      ),
      createdAt: DateTime.parse(createdAt),
    );
  }
}
```

### API Client

```dart
// data/datasources/auth_api_client.dart
class AuthApiClient {
  final Dio dio;
  
  AuthApiClient(this.dio);
  
  Future<AuthSessionModel> login({
    required String email,
    required String password,
    required String deviceId,
    required bool rememberMe,
  }) async {
    final hashedPassword = _hashPassword(password);
    
    final response = await dio.post(
      '/api/auth/login',
      data: {
        'email': email,
        'password': hashedPassword,
        'deviceId': deviceId,
        'rememberMe': rememberMe,
      },
    );
    
    return AuthSessionModel.fromJson(response.data['data']);
  }
  
  String _hashPassword(String password) {
    // Use crypto package to hash password
    final bytes = utf8.encode(password);
    final hash = sha256.convert(bytes);
    return hash.toString();
  }
}
```

### Local Storage

```dart
// data/datasources/auth_local_storage.dart
class AuthLocalStorage {
  final HiveInterface hive;
  final FlutterSecureStorage secureStorage;
  
  static const _authBoxName = 'auth_box';
  static const _tokenKey = 'auth_token';
  static const _refreshTokenKey = 'refresh_token';
  static const _userKey = 'current_user';
  
  AuthLocalStorage(this.hive, this.secureStorage);
  
  Future<void> saveSession(AuthSessionModel session) async {
    // Save tokens securely
    await secureStorage.write(key: _tokenKey, value: session.token);
    await secureStorage.write(key: _refreshTokenKey, value: session.refreshToken);
    
    // Save session metadata in Hive
    final box = await hive.openBox(_authBoxName);
    await box.put('session', session.toJson());
  }
  
  Future<AuthSessionModel?> getSession() async {
    final token = await secureStorage.read(key: _tokenKey);
    final refreshToken = await secureStorage.read(key: _refreshTokenKey);
    
    if (token == null || refreshToken == null) return null;
    
    final box = await hive.openBox(_authBoxName);
    final sessionData = box.get('session');
    
    if (sessionData == null) return null;
    
    return AuthSessionModel.fromJson(sessionData);
  }
  
  Future<void> clearSession() async {
    await secureStorage.delete(key: _tokenKey);
    await secureStorage.delete(key: _refreshTokenKey);
    
    final box = await hive.openBox(_authBoxName);
    await box.clear();
  }
}
```

## Presentation Layer Specifications

### BLoC Events

```dart
// presentation/bloc/auth_event.dart
abstract class AuthEvent {}

class LoginWithEmailRequested extends AuthEvent {
  final String email;
  final String password;
  final bool rememberMe;
  
  LoginWithEmailRequested({
    required this.email,
    required this.password,
    this.rememberMe = false,
  });
}

class RegisterWithEmailRequested extends AuthEvent {
  final String email;
  final String password;
  
  RegisterWithEmailRequested({
    required this.email,
    required this.password,
  });
}

class ForgotPasswordRequested extends AuthEvent {
  final String email;
  
  ForgotPasswordRequested(this.email);
}

class LogoutRequested extends AuthEvent {}

class CheckAuthStatus extends AuthEvent {}
```

### BLoC States

```dart
// presentation/bloc/auth_state.dart
abstract class AuthState {}

class AuthInitial extends AuthState {}

class AuthLoading extends AuthState {}

class Authenticated extends AuthState {
  final User user;
  
  Authenticated(this.user);
}

class Unauthenticated extends AuthState {}

class AuthError extends AuthState {
  final String message;
  final AuthErrorType type;
  
  AuthError(this.message, this.type);
}

enum AuthErrorType {
  invalidCredentials,
  networkError,
  weakPassword,
  emailAlreadyExists,
  unknown,
}

class PasswordResetEmailSent extends AuthState {
  final String email;
  
  PasswordResetEmailSent(this.email);
}
```

### UI Specifications

#### Login Page

```dart
// presentation/pages/login_page.dart
class LoginPage extends StatelessWidget {
  static const routeName = '/login';
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: BlocConsumer<AuthBloc, AuthState>(
          listener: (context, state) {
            if (state is Authenticated) {
              context.go('/home');
            } else if (state is AuthError) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text(state.message)),
              );
            }
          },
          builder: (context, state) {
            return SingleChildScrollView(
              padding: EdgeInsets.all(24),
              child: Column(
                children: [
                  // Logo
                  SizedBox(height: 60),
                  Image.asset('assets/images/logo.png', height: 80),
                  SizedBox(height: 40),
                  
                  // Title
                  Text(
                    'Welcome Back',
                    style: Theme.of(context).textTheme.headlineMedium,
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Login to continue',
                    style: Theme.of(context).textTheme.bodyMedium,
                  ),
                  SizedBox(height: 40),
                  
                  // Email Input
                  EmailInputField(),
                  SizedBox(height: 16),
                  
                  // Password Input
                  PasswordInputField(),
                  SizedBox(height: 8),
                  
                  // Remember Me & Forgot Password
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      RememberMeCheckbox(),
                      TextButton(
                        onPressed: () => context.push('/forgot-password'),
                        child: Text('Forgot Password?'),
                      ),
                    ],
                  ),
                  SizedBox(height: 24),
                  
                  // Login Button
                  AuthButton(
                    text: 'Login',
                    isLoading: state is AuthLoading,
                    onPressed: () {
                      // Dispatch login event
                    },
                  ),
                  SizedBox(height: 24),
                  
                  // Divider
                  Row(
                    children: [
                      Expanded(child: Divider()),
                      Padding(
                        padding: EdgeInsets.symmetric(horizontal: 16),
                        child: Text('OR'),
                      ),
                      Expanded(child: Divider()),
                    ],
                  ),
                  SizedBox(height: 24),
                  
                  // Social Login Buttons
                  GoogleSignInButton(),
                  SizedBox(height: 12),
                  AppleSignInButton(),
                  SizedBox(height: 24),
                  
                  // Sign Up Link
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text("Don't have an account? "),
                      TextButton(
                        onPressed: () => context.push('/register'),
                        child: Text('Sign Up'),
                      ),
                    ],
                  ),
                ],
              ),
            );
          },
        ),
      ),
    );
  }
}
```

## Security Specifications

### Password Requirements
- Minimum 8 characters
- At least 1 uppercase letter
- At least 1 lowercase letter
- At least 1 number
- Optional: 1 special character

### Password Hashing
- Use SHA-256 for client-side hashing
- Server should use bcrypt for storage

### Token Management
- JWT tokens với 1 hour expiration
- Refresh tokens với 30 days expiration
- Auto-refresh when token < 5 minutes to expire

### Rate Limiting
- Max 5 login attempts per 5 minutes
- Account lockout for 15 minutes after 5 failed attempts

## Performance Requirements

- Login response time: < 2 seconds
- Registration response time: < 3 seconds
- UI should be responsive (60 FPS)
- No memory leaks
- Offline support: Show cached user data

## Testing Requirements

- Unit test coverage > 90% for business logic
- Widget tests for all UI components
- Integration tests for complete flows
- Security testing for password handling
- Performance testing for login flow

## Dependencies

```yaml
dependencies:
  # Existing
  flutter_bloc: ^8.1.6
  dio: ^5.7.0
  hive: ^2.2.3
  go_router: ^14.4.1
  
  # New
  flutter_secure_storage: ^9.0.0
  crypto: ^3.0.3
  email_validator: ^2.1.17
```

## Migration Plan

1. Add new dependencies
2. Implement domain layer
3. Implement data layer
4. Implement presentation layer
5. Update routing configuration
6. Add to existing auth flow (alongside Google/Apple)
7. Testing
8. Release

## Rollback Plan

- Feature flag để enable/disable email login
- Nếu có issues, disable feature flag
- Users vẫn có thể dùng Google/Apple login
