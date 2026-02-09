# CP-212: Technical Specifications

## Overview

Implement AI Chat system cho Passio app, bao gồm chat interface, session management, prompt handling, và feedback system.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ ChatPage     │  │HistoryDrawer │  │PromptWidget  │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                  │                  │          │
│         └──────────────────┼──────────────────┘          │
│                            │                             │
│                    ┌───────▼────────┐                    │
│                    │ AssistantBloc  │                    │
│                    └───────┬────────┘                    │
└────────────────────────────┼──────────────────────────────┘
                             │
┌────────────────────────────▼──────────────────────────────┐
│                     Domain Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │SendMessage   │  │GetSessions   │  │SubmitFeedback│   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                  │                  │           │
│         └──────────────────┼──────────────────┘           │
│                            │                              │
│                    ┌───────▼────────┐                     │
│                    │IAssistantRepo  │                     │
│                    └───────┬────────┘                     │
└────────────────────────────┼───────────────────────────────┘
                             │
┌────────────────────────────▼───────────────────────────────┐
│                      Data Layer                            │
│                 ┌──────────────────┐                       │
│                 │AssistantRepository│                      │
│                 └────────┬─────────┘                       │
│          ┌──────────────┴──────────────┐                  │
│          │                              │                  │
│  ┌───────▼────────┐          ┌─────────▼────────┐        │
│  │AIApiClient     │          │ChatLocalStorage  │        │
│  │(Dio)           │          │(Hive)            │        │
│  └────────────────┘          └──────────────────┘        │
└────────────────────────────────────────────────────────────┘
```

## Feature Structure

```
lib/src/features/assistant/
├── data/
│   ├── models/
│   │   ├── chat_message_model.dart
│   │   ├── chat_session_model.dart
│   │   ├── ai_response_model.dart
│   │   ├── thinking_process_model.dart
│   │   └── prompt_item_model.dart
│   ├── repositories/
│   │   └── assistant_repository.dart
│   └── datasources/
│       ├── ai_api_client.dart
│       └── chat_local_storage.dart
├── domain/
│   ├── entities/
│   │   ├── chat_message.dart
│   │   ├── chat_session.dart
│   │   ├── ai_response.dart
│   │   ├── thinking_process.dart
│   │   └── prompt_item.dart
│   ├── repositories/
│   │   └── i_assistant_repository.dart
│   └── usecases/
│       ├── send_message.dart
│       ├── get_sessions.dart
│       ├── create_session.dart
│       ├── delete_session.dart
│       ├── rename_session.dart
│       ├── submit_feedback.dart
│       └── get_prompts.dart
└── presentation/
    ├── bloc/
    │   ├── assistant_bloc.dart
    │   ├── assistant_event.dart
    │   └── assistant_state.dart
    ├── pages/
    │   └── chat_page.dart
    └── widgets/
        ├── message_bubble.dart
        ├── thinking_indicator.dart
        ├── thinking_bottomsheet.dart
        ├── chat_input_field.dart
        ├── history_drawer.dart
        ├── session_item.dart
        ├── prompt_item_widget.dart
        ├── mini_conversation_widget.dart
        └── feedback_buttons.dart
```

## Domain Layer Specifications

### Entities

```dart
// domain/entities/chat_message.dart
class ChatMessage {
  final String id;
  final String sessionId;
  final String content;
  final MessageSender sender;
  final DateTime timestamp;
  final MessageStatus status;
  final FeedbackType? feedback;
  final ThinkingProcess? thinkingProcess;
  
  const ChatMessage({
    required this.id,
    required this.sessionId,
    required this.content,
    required this.sender,
    required this.timestamp,
    required this.status,
    this.feedback,
    this.thinkingProcess,
  });
}

enum MessageSender { user, ai }
enum MessageStatus { sending, sent, failed }
enum FeedbackType { like, dislike }

// domain/entities/chat_session.dart
class ChatSession {
  final String id;
  final String name;
  final DateTime createdAt;
  final DateTime updatedAt;
  final List<ChatMessage> messages;
  
  const ChatSession({
    required this.id,
    required this.name,
    required this.createdAt,
    required this.updatedAt,
    required this.messages,
  });
  
  String get description {
    if (messages.isEmpty) return '';
    return messages.first.content;
  }
}

// domain/entities/thinking_process.dart
class ThinkingProcess {
  final String messageId;
  final List<ThinkingStep> steps;
  
  const ThinkingProcess({
    required this.messageId,
    required this.steps,
  });
}

class ThinkingStep {
  final String title;
  final String content;
  final int order;
  
  const ThinkingStep({
    required this.title,
    required this.content,
    required this.order,
  });
}

// domain/entities/prompt_item.dart
class PromptItem {
  final String id;
  final String title;
  final String content;
  final bool isActive;
  final int order;
  
  const PromptItem({
    required this.id,
    required this.title,
    required this.content,
    required this.isActive,
    required this.order,
  });
}
```

### Repository Interface

```dart
// domain/repositories/i_assistant_repository.dart
abstract class IAssistantRepository {
  /// Send message and get AI response
  Future<ChatMessage> sendMessage({
    required String sessionId,
    required String content,
  });
  
  /// Get all chat sessions (max 10, sorted by updatedAt desc)
  Future<List<ChatSession>> getSessions();
  
  /// Get specific session with messages
  Future<ChatSession> getSession(String sessionId);
  
  /// Create new chat session
  Future<ChatSession> createSession();
  
  /// Delete session
  Future<void> deleteSession(String sessionId);
  
  /// Rename session
  Future<void> renameSession(String sessionId, String newName);
  
  /// Submit feedback for AI response
  Future<void> submitFeedback({
    required String messageId,
    required FeedbackType feedback,
  });
  
  /// Get active prompt items
  Future<List<PromptItem>> getPrompts();
  
  /// Watch session updates
  Stream<List<ChatSession>> watchSessions();
  
  /// Clean up old sessions (> 60 days)
  Future<void> cleanupOldSessions();
}
```

### Use Cases

```dart
// domain/usecases/send_message.dart
class SendMessage {
  final IAssistantRepository repository;
  
  SendMessage(this.repository);
  
  Future<ChatMessage> call({
    required String sessionId,
    required String content,
  }) async {
    // Validate input length
    if (content.trim().isEmpty) {
      throw EmptyMessageException();
    }
    
    if (content.length > 355) {
      throw MessageTooLongException();
    }
    
    return await repository.sendMessage(
      sessionId: sessionId,
      content: content.trim(),
    );
  }
}

// domain/usecases/rename_session.dart
class RenameSession {
  final IAssistantRepository repository;
  
  RenameSession(this.repository);
  
  Future<void> call({
    required String sessionId,
    required String newName,
  }) async {
    // Validate name length
    if (newName.trim().isEmpty) {
      throw EmptyNameException();
    }
    
    if (newName.length > 50) {
      throw NameTooLongException();
    }
    
    return await repository.renameSession(sessionId, newName.trim());
  }
}
```

## Data Layer Specifications

### Models

```dart
// data/models/chat_message_model.dart
@JsonSerializable()
class ChatMessageModel {
  final String id;
  @JsonKey(name: 'session_id')
  final String sessionId;
  final String content;
  final String sender;
  final String timestamp;
  final String status;
  final String? feedback;
  @JsonKey(name: 'thinking_process')
  final ThinkingProcessModel? thinkingProcess;
  
  ChatMessageModel({
    required this.id,
    required this.sessionId,
    required this.content,
    required this.sender,
    required this.timestamp,
    required this.status,
    this.feedback,
    this.thinkingProcess,
  });
  
  factory ChatMessageModel.fromJson(Map<String, dynamic> json) =>
      _$ChatMessageModelFromJson(json);
  
  Map<String, dynamic> toJson() => _$ChatMessageModelToJson(this);
  
  ChatMessage toEntity() {
    return ChatMessage(
      id: id,
      sessionId: sessionId,
      content: content,
      sender: MessageSender.values.firstWhere((e) => e.name == sender),
      timestamp: DateTime.parse(timestamp),
      status: MessageStatus.values.firstWhere((e) => e.name == status),
      feedback: feedback != null 
          ? FeedbackType.values.firstWhere((e) => e.name == feedback)
          : null,
      thinkingProcess: thinkingProcess?.toEntity(),
    );
  }
  
  static ChatMessageModel fromEntity(ChatMessage entity) {
    return ChatMessageModel(
      id: entity.id,
      sessionId: entity.sessionId,
      content: entity.content,
      sender: entity.sender.name,
      timestamp: entity.timestamp.toIso8601String(),
      status: entity.status.name,
      feedback: entity.feedback?.name,
      thinkingProcess: entity.thinkingProcess != null
          ? ThinkingProcessModel.fromEntity(entity.thinkingProcess!)
          : null,
    );
  }
}

// data/models/chat_session_model.dart
@JsonSerializable()
class ChatSessionModel {
  final String id;
  final String name;
  @JsonKey(name: 'created_at')
  final String createdAt;
  @JsonKey(name: 'updated_at')
  final String updatedAt;
  final List<ChatMessageModel> messages;
  
  ChatSessionModel({
    required this.id,
    required this.name,
    required this.createdAt,
    required this.updatedAt,
    required this.messages,
  });
  
  factory ChatSessionModel.fromJson(Map<String, dynamic> json) =>
      _$ChatSessionModelFromJson(json);
  
  Map<String, dynamic> toJson() => _$ChatSessionModelToJson(this);
  
  ChatSession toEntity() {
    return ChatSession(
      id: id,
      name: name,
      createdAt: DateTime.parse(createdAt),
      updatedAt: DateTime.parse(updatedAt),
      messages: messages.map((m) => m.toEntity()).toList(),
    );
  }
}
```

### API Client

```dart
// data/datasources/ai_api_client.dart
class AIApiClient {
  final Dio dio;
  static const String baseUrl = 'https://assistants-api.ecomobi.com';
  static const String devBaseUrl = 'https://dev.assistants-api.ecomobi.com';
  
  AIApiClient(this.dio) {
    // Configure base URL based on environment
    dio.options.baseUrl = baseUrl; // or devBaseUrl for dev
  }
  
  /// Send message with streaming support
  /// Returns a stream of AI response chunks (reasoning + content)
  Stream<Map<String, dynamic>> sendMessageStream({
    required String conversationId,
    required String message,
    bool includeReasoning = true,
    bool includeStructuredData = true,
  }) async* {
    final response = await dio.post(
      '/v1/agents/chat/v1',
      data: {
        'message': message,
        'conversationId': conversationId,
        'options': {
          'streamFormat': 'ai-sdk',
          'includeReasoning': includeReasoning,
          'includeStructuredData': includeStructuredData,
        },
      },
      options: Options(
        responseType: ResponseType.stream,
        headers: {
          'Accept': 'text/event-stream',
        },
      ),
    );
    
    // Parse streaming response
    await for (var chunk in response.data.stream) {
      final data = utf8.decode(chunk);
      final lines = data.split('\n');
      
      for (var line in lines) {
        if (line.trim().isEmpty) continue;
        
        try {
          final json = jsonDecode(line);
          yield json;
        } catch (e) {
          // Skip invalid JSON
          continue;
        }
      }
    }
  }
  
  /// Get all conversations with pagination
  Future<ConversationsResponse> getConversations({
    int page = 1,
    int limit = 20,
  }) async {
    final response = await dio.get(
      '/v1/conversations',
      queryParameters: {
        'page': page,
        'limit': limit,
      },
    );
    
    return ConversationsResponse.fromJson(response.data);
  }
  
  /// Get messages for a specific conversation
  Future<MessagesResponse> getConversationMessages({
    required String conversationId,
    int page = 1,
    int limit = 50,
    String sortOrder = 'desc',
  }) async {
    final response = await dio.get(
      '/v1/conversations/$conversationId/messages',
      queryParameters: {
        'page': page,
        'limit': limit,
        'sortOrder': sortOrder,
      },
    );
    
    return MessagesResponse.fromJson(response.data);
  }
  
  /// Delete conversation (if API supports)
  Future<void> deleteConversation(String conversationId) async {
    // Note: API spec doesn't mention delete endpoint
    // This might need to be implemented or removed
    await dio.delete('/v1/conversations/$conversationId');
  }
  
  /// Rename conversation (if API supports)
  Future<void> renameConversation(String conversationId, String newTitle) async {
    // Note: API spec doesn't mention rename endpoint
    // This might need to be implemented or removed
    await dio.patch(
      '/v1/conversations/$conversationId',
      data: {'title': newTitle},
    );
  }
  
  /// Submit feedback (if API supports)
  Future<void> submitFeedback({
    required String messageId,
    required String feedback,
  }) async {
    // Note: API spec doesn't mention feedback endpoint
    // This might need to be added to API or removed from client
    await dio.post(
      '/v1/feedback',
      data: {
        'messageId': messageId,
        'feedback': feedback,
      },
    );
  }
  
  /// Get prompts (if API supports)
  Future<List<PromptItemModel>> getPrompts() async {
    // Note: API spec doesn't mention prompts endpoint
    // This might need to be added to API or fetched from remote config
    final response = await dio.get('/v1/prompts');
    
    return (response.data['data'] as List)
        .map((json) => PromptItemModel.fromJson(json))
        .toList();
  }
}

// Response models for API
class ConversationsResponse {
  final bool success;
  final List<ConversationModel> data;
  final PaginationModel pagination;
  
  ConversationsResponse({
    required this.success,
    required this.data,
    required this.pagination,
  });
  
  factory ConversationsResponse.fromJson(Map<String, dynamic> json) {
    return ConversationsResponse(
      success: json['success'],
      data: (json['data'] as List)
          .map((e) => ConversationModel.fromJson(e))
          .toList(),
      pagination: PaginationModel.fromJson(json['pagination']),
    );
  }
}

class ConversationModel {
  final String id;
  final String title;
  final String createdAt;
  final String updatedAt;
  final int messageCount;
  
  ConversationModel({
    required this.id,
    required this.title,
    required this.createdAt,
    required this.updatedAt,
    required this.messageCount,
  });
  
  factory ConversationModel.fromJson(Map<String, dynamic> json) {
    return ConversationModel(
      id: json['id'],
      title: json['title'],
      createdAt: json['createdAt'],
      updatedAt: json['updatedAt'],
      messageCount: json['messageCount'],
    );
  }
}

class MessagesResponse {
  final bool success;
  final List<MessageModel> data;
  final PaginationModel pagination;
  
  MessagesResponse({
    required this.success,
    required this.data,
    required this.pagination,
  });
  
  factory MessagesResponse.fromJson(Map<String, dynamic> json) {
    return MessagesResponse(
      success: json['success'],
      data: (json['data'] as List)
          .map((e) => MessageModel.fromJson(e))
          .toList(),
      pagination: PaginationModel.fromJson(json['pagination']),
    );
  }
}

class MessageModel {
  final String id;
  final String role; // 'user' or 'assistant'
  final String content;
  final String createdAt;
  final Map<String, dynamic>? metadata;
  
  MessageModel({
    required this.id,
    required this.role,
    required this.content,
    required this.createdAt,
    this.metadata,
  });
  
  factory MessageModel.fromJson(Map<String, dynamic> json) {
    return MessageModel(
      id: json['id'],
      role: json['role'],
      content: json['content'],
      createdAt: json['createdAt'],
      metadata: json['metadata'],
    );
  }
}

class PaginationModel {
  final int page;
  final int total;
  final int? totalPages;
  final bool hasNextPage;
  
  PaginationModel({
    required this.page,
    required this.total,
    this.totalPages,
    required this.hasNextPage,
  });
  
  factory PaginationModel.fromJson(Map<String, dynamic> json) {
    return PaginationModel(
      page: json['page'],
      total: json['total'],
      totalPages: json['totalPages'],
      hasNextPage: json['hasNextPage'],
    );
  }
}
```

### Local Storage

```dart
// data/datasources/chat_local_storage.dart
class ChatLocalStorage {
  final HiveInterface hive;
  
  static const _sessionsBoxName = 'chat_sessions';
  static const _messagesBoxName = 'chat_messages';
  static const _promptsBoxName = 'chat_prompts';
  
  ChatLocalStorage(this.hive);
  
  Future<void> saveSession(ChatSessionModel session) async {
    final box = await hive.openBox<Map>(_sessionsBoxName);
    await box.put(session.id, session.toJson());
  }
  
  Future<List<ChatSessionModel>> getSessions() async {
    final box = await hive.openBox<Map>(_sessionsBoxName);
    
    final sessions = box.values
        .map((json) => ChatSessionModel.fromJson(Map<String, dynamic>.from(json)))
        .toList();
    
    // Sort by updatedAt descending
    sessions.sort((a, b) => 
        DateTime.parse(b.updatedAt).compareTo(DateTime.parse(a.updatedAt)));
    
    // Return max 10
    return sessions.take(10).toList();
  }
  
  Future<ChatSessionModel?> getSession(String sessionId) async {
    final box = await hive.openBox<Map>(_sessionsBoxName);
    final json = box.get(sessionId);
    
    if (json == null) return null;
    
    return ChatSessionModel.fromJson(Map<String, dynamic>.from(json));
  }
  
  Future<void> deleteSession(String sessionId) async {
    final box = await hive.openBox<Map>(_sessionsBoxName);
    await box.delete(sessionId);
  }
  
  Future<void> saveMessage(ChatMessageModel message) async {
    final box = await hive.openBox<Map>(_messagesBoxName);
    await box.put(message.id, message.toJson());
  }
  
  Future<void> cleanupOldSessions() async {
    final box = await hive.openBox<Map>(_sessionsBoxName);
    final now = DateTime.now();
    final cutoffDate = now.subtract(Duration(days: 60));
    
    final keysToDelete = <String>[];
    
    for (var entry in box.toMap().entries) {
      final session = ChatSessionModel.fromJson(
          Map<String, dynamic>.from(entry.value));
      
      if (DateTime.parse(session.updatedAt).isBefore(cutoffDate)) {
        keysToDelete.add(entry.key as String);
      }
    }
    
    await box.deleteAll(keysToDelete);
  }
}
```

## Presentation Layer Specifications

### BLoC Events

```dart
// presentation/bloc/assistant_event.dart
abstract class AssistantEvent {}

class LoadSessions extends AssistantEvent {}

class LoadSession extends AssistantEvent {
  final String sessionId;
  LoadSession(this.sessionId);
}

class CreateNewSession extends AssistantEvent {}

class SendMessageRequested extends AssistantEvent {
  final String content;
  final String? sessionId;
  
  SendMessageRequested(this.content, {this.sessionId});
}

class DeleteSessionRequested extends AssistantEvent {
  final String sessionId;
  DeleteSessionRequested(this.sessionId);
}

class RenameSessionRequested extends AssistantEvent {
  final String sessionId;
  final String newName;
  
  RenameSessionRequested(this.sessionId, this.newName);
}

class SubmitFeedbackRequested extends AssistantEvent {
  final String messageId;
  final FeedbackType feedback;
  
  SubmitFeedbackRequested(this.messageId, this.feedback);
}

class LoadPrompts extends AssistantEvent {}

class HideMiniConversation extends AssistantEvent {}
```

### BLoC States

```dart
// presentation/bloc/assistant_state.dart
abstract class AssistantState {}

class AssistantInitial extends AssistantState {}

class AssistantLoading extends AssistantState {}

class SessionsLoaded extends AssistantState {
  final List<ChatSession> sessions;
  final ChatSession? activeSession;
  final bool showMiniConversation;
  
  SessionsLoaded({
    required this.sessions,
    this.activeSession,
    this.showMiniConversation = false,
  });
}

class MessageSending extends AssistantState {
  final ChatSession session;
  final ChatMessage pendingMessage;
  
  MessageSending(this.session, this.pendingMessage);
}

class MessageReceived extends AssistantState {
  final ChatSession session;
  final ChatMessage aiMessage;
  
  MessageReceived(this.session, this.aiMessage);
}

class AssistantError extends AssistantState {
  final String message;
  final AssistantErrorType type;
  
  AssistantError(this.message, this.type);
}

enum AssistantErrorType {
  networkError,
  timeout,
  serverError,
  validationError,
  unknown,
}

class PromptsLoaded extends AssistantState {
  final List<PromptItem> prompts;
  
  PromptsLoaded(this.prompts);
}
```

### UI Specifications

#### Chat Page

```dart
// presentation/pages/chat_page.dart
class ChatPage extends StatelessWidget {
  static const routeName = '/ai-chat';
  final String? sessionId;
  
  const ChatPage({Key? key, this.sessionId}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => getIt<AssistantBloc>()
        ..add(sessionId != null 
            ? LoadSession(sessionId!) 
            : CreateNewSession()),
      child: Scaffold(
        appBar: _buildAppBar(),
        drawer: HistoryDrawer(),
        body: Column(
          children: [
            // Animation header (first time only)
            _buildAnimationHeader(),
            
            // Messages list
            Expanded(
              child: BlocBuilder<AssistantBloc, AssistantState>(
                builder: (context, state) {
                  if (state is SessionsLoaded && state.activeSession != null) {
                    return _buildMessagesList(state.activeSession!);
                  }
                  return _buildEmptyState();
                },
              ),
            ),
            
            // Input field
            ChatInputField(),
          ],
        ),
      ),
    );
  }
  
  Widget _buildMessagesList(ChatSession session) {
    if (session.messages.isEmpty) {
      return Center(
        child: Text('Passio xin chào\nBạn đang cần hỗ trợ về vấn đề gì ạ?'),
      );
    }
    
    return ListView.builder(
      reverse: true,
      itemCount: session.messages.length,
      itemBuilder: (context, index) {
        final message = session.messages[index];
        return MessageBubble(message: message);
      },
    );
  }
}
```

#### Message Bubble

```dart
// presentation/widgets/message_bubble.dart
class MessageBubble extends StatelessWidget {
  final ChatMessage message;
  
  const MessageBubble({Key? key, required this.message}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    final isUser = message.sender == MessageSender.user;
    
    return Align(
      alignment: isUser ? Alignment.centerRight : Alignment.centerLeft,
      child: Container(
        margin: EdgeInsets.symmetric(vertical: 8, horizontal: 16),
        padding: EdgeInsets.all(12),
        decoration: BoxDecoration(
          color: isUser ? Colors.blue : Colors.grey[200],
          borderRadius: BorderRadius.circular(16),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Message content
            Text(
              message.content,
              style: TextStyle(
                color: isUser ? Colors.white : Colors.black,
              ),
            ),
            
            // Thinking indicator (AI only)
            if (!isUser && message.thinkingProcess != null)
              ThinkingIndicator(
                thinkingProcess: message.thinkingProcess!,
              ),
            
            // Feedback buttons (AI only)
            if (!isUser)
              FeedbackButtons(
                messageId: message.id,
                currentFeedback: message.feedback,
              ),
          ],
        ),
      ),
    );
  }
}
```

## Security Specifications

### Input Validation
- Max 355 characters per message
- Trim whitespace
- Sanitize HTML/scripts
- Validate session IDs

### Link Handling
- Validate URLs before opening
- In-app links: Use deep linking
- External links: Open in WebView
- Block malicious URLs

### Data Protection
- Encrypt sensitive data in Hive
- Secure API communication (HTTPS)
- Token-based authentication

## Performance Requirements

- Message send response time: < 3 seconds (normal)
- Message send response time: < 30 seconds (max with timeout)
- UI should be responsive (60 FPS)
- No memory leaks
- Offline support: Show cached sessions
- Pagination: Load 20 messages at a time

## Testing Requirements

- Unit test coverage > 90% for business logic
- Widget tests for all UI components
- Integration tests for complete flows
- Performance testing for message rendering
- Offline scenario testing

## Dependencies

```yaml
dependencies:
  # Existing
  flutter_bloc: ^8.1.6
  dio: ^5.7.0
  hive: ^2.2.3
  go_router: ^14.4.1
  intl: (existing)
  uuid: ^4.5.1
  
  # New (if needed)
  flutter_markdown: ^0.6.18  # For markdown rendering
  url_launcher: ^6.3.1  # For opening links
```

# Hệ Thống AI Chat - Tài Liệu Đặc Tả API (Full Specification)

## 1. Chat API v1 (Streaming & Reasoning)

Sử dụng khi cần tương tác trực tiếp với AI, hỗ trợ hiển thị quá trình suy luận (reasoning) và trả về các thành phần UI có cấu trúc.

### Thông tin kết nối

* 
**Endpoint:** `/v1/agents/chat/v1` 


* 
**Method:** `POST` 


* 
**Auth:** `Bearer Token` 


* 
**Servers:** * Dev: `https://dev.assistants-api.ecomobi.com` 


* Prod: `https://assistants-api.ecomobi.com` 





### Request Body

```json
{
  [cite_start]"message": "Ecomobi rank là gì?", // Câu hỏi của user [cite: 35]
  [cite_start]"conversationId": "conv_abc123", // ID để track conversation (tự tạo nếu trống) [cite: 35]
  "options": {
    [cite_start]"streamFormat": "ai-sdk", // Mặc định [cite: 37]
    [cite_start]"includeReasoning": true, // Hiển thị Phase 1 reasoning [cite: 37, 46]
    [cite_start]"includeStructuredData": true // Trả về structured UI components [cite: 37]
  }
}

```

### Response Sample (Streamed Data)

Dữ liệu được trả về theo từng chunk với các kiểu `type` khác nhau:

```json
// 1. Phân đoạn suy luận (Reasoning)
[cite_start]{"type":"reasoning","content":"Đang phân tích câu hỏi của bạn và tìm kiếm tài liệu liên quan..."} [cite: 48]

// 2. Phân đoạn nội dung (Content Delta)
[cite_start]{"type":"content","delta":"Ecomobi "} [cite: 51, 52]
[cite_start]{"type":"content","delta":"rank "} [cite: 53]
[cite_start]{"type":"content","delta":"là hệ thống..."} [cite: 60]

// 3. Kết thúc (Done)
[cite_start]{"type":"done", "finishReason": "stop", "conversationId":"conv_abc123"} [cite: 142]

```

---

## 2. Conversations API (Quản lý Danh sách)

Lấy danh sách tất cả các cuộc hội thoại của người dùng hiện tại.

### Thông tin kết nối

* 
**Endpoint:** `/v1/conversations` 


* 
**Method:** `GET` 


* 
**Params:** `page` (mặc định 1), `limit` (mặc định 20, max 100).



### Response Sample

```json
{
  "success": true,
  "data": [
    {
      [cite_start]"id": "conv_abc123", // Unique ID [cite: 315]
      [cite_start]"title": "Hỏi về Ecomobi rank và commission", // Tự động gen [cite: 315]
      [cite_start]"createdAt": "2025-11-10T08:30:00.000Z", [cite: 265]
      [cite_start]"updatedAt": "2025-11-10T09:45:00.000Z", [cite: 267]
      [cite_start]"messageCount": 12 [cite: 272]
    }
  ],
  "pagination": {
    "page": 1,
    "total": 45,
    "totalPages": 3,
    [cite_start]"hasNextPage": true [cite: 297, 301, 305, 306]
  }
}

```

---

## 3. Get Conversation Messages API (Chi tiết Hội thoại)

Truy xuất toàn bộ tin nhắn trong một cuộc hội thoại cụ thể với hỗ trợ phân trang.

### Thông tin kết nối

* 
**Endpoint:** `/v1/conversations/:id/messages` 


* 
**Method:** `GET` 


* 
**Params:** `page` (mặc định 1), `limit` (mặc định 50), `sortOrder` (asc/desc).



### Response Sample

```json
{
  "success": true,
  "data": [
    {
      [cite_start]"id": "msg_001", [cite: 180]
      [cite_start]"role": "user", [cite: 182]
      [cite_start]"content": "Ecomobi rank là gì?", [cite: 184]
      [cite_start]"createdAt": "2025-11-10T08:30:15.000Z", [cite: 186]
      [cite_start]"metadata": null [cite: 188]
    },
    {
      [cite_start]"id": "msg_002", [cite: 192]
      [cite_start]"role": "assistant", [cite: 194]
      [cite_start]"content": "Ecomobi rank là hệ thống xếp hạng publisher...", [cite: 196]
      "metadata": {
        [cite_start]"agentId": "ecomobi-agent", [cite: 202]
        [cite_start]"toolsUsed": ["retrieveDocuments"] [cite: 204]
      }
    }
  ],
  "pagination": {
    "page": 1,
    "total": 120,
    [cite_start]"hasNextPage": true [cite: 211, 214, 224]
  }
}

```

---

## 4. Bảng mã lỗi (Error Handling)

Cần xử lý các mã lỗi sau để đảm bảo trải nghiệm người dùng:

| Mã lỗi | Trạng thái | Mô tả |
| --- | --- | --- |
| **200** | OK | Request thành công.

 |
| **400** | Bad Request | Thiếu `message` hoặc dữ liệu không hợp lệ.

 |
| **401** | Unauthorized | Token hết hạn hoặc không có quyền truy cập.

 |
| **403** | Access Denied | Cố gắng truy cập hội thoại của người dùng khác.

 |
| **500** | Internal Error | Lỗi hệ thống từ server.

 |

## Migration Plan

1. Create feature structure
2. Implement domain layer
3. Implement data layer
4. Implement presentation layer
5. Integrate with Home feature
6. Update routing configuration
7. Testing
8. Release

## Rollback Plan

- Feature flag để enable/disable AI Chat
- Nếu có issues, disable feature flag
- Users vẫn có thể sử dụng các features khác bình thường
