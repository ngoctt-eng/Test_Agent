---
description: Quy trình phát triển feature từ spec đến deployment
---

# Development Flow

## Mục đích
Quy trình chuẩn để phát triển feature mới, đảm bảo chất lượng code và tuân thủ architecture standards.

## Vai trò tham gia
- **Architecture Expert**: Hướng dẫn thiết kế
- **Senior Developer**: Implement feature
- **Performance Expert**: Review performance
- **QA Engineer**: Testing và quality assurance

---

## Quy trình Development

### Phase 1: Setup & Planning
**Người thực hiện**: Senior Developer

// turbo
**Bước 1.1: Tạo feature branch**
```bash
git checkout develop
git pull origin develop
git checkout -b feature/{TASK_ID}-{feature-name}
```

**Bước 1.2: Review documentation**
- [ ] Đọc `tasks/{TASK_ID}/requirements.md`
- [ ] Đọc `tasks/{TASK_ID}/architecture_review.md`
- [ ] Đọc `tasks/{TASK_ID}/development_plan.md`
- [ ] Hiểu rõ acceptance criteria

**Bước 1.3: Setup feature structure**
```bash
# Tạo cấu trúc thư mục
mkdir -p lib/src/features/{feature_name}/{data,domain,presentation}
mkdir -p lib/src/features/{feature_name}/data/{models,repositories,datasources}
mkdir -p lib/src/features/{feature_name}/domain/{entities,repositories,usecases}
mkdir -p lib/src/features/{feature_name}/presentation/{bloc,pages,widgets}
```

---

### Phase 2: Domain Layer Implementation
**Người thực hiện**: Senior Developer  
**Review bởi**: Architecture Expert

**Bước 2.1: Tạo Entities**
```dart
// lib/src/features/campaign/domain/entities/campaign.dart
class Campaign {
  final String id;
  final String name;
  final DateTime startDate;
  final DateTime endDate;
  final CampaignStatus status;
  
  const Campaign({
    required this.id,
    required this.name,
    required this.startDate,
    required this.endDate,
    required this.status,
  });
}
```

**Bước 2.2: Define Repository Interfaces**
```dart
// lib/src/features/campaign/domain/repositories/campaign_repository.dart
abstract class ICampaignRepository {
  Future<List<Campaign>> getCampaigns();
  Future<Campaign> getCampaignById(String id);
  Future<void> createCampaign(Campaign campaign);
}
```

**Bước 2.3: Write Use Cases**
```dart
// lib/src/features/campaign/domain/usecases/get_campaigns.dart
class GetCampaigns {
  final ICampaignRepository repository;
  
  GetCampaigns(this.repository);
  
  Future<List<Campaign>> call() async {
    return await repository.getCampaigns();
  }
}
```

**Bước 2.4: Write Unit Tests**
```dart
// test/features/campaign/domain/usecases/get_campaigns_test.dart
void main() {
  late GetCampaigns usecase;
  late MockICampaignRepository mockRepository;
  
  setUp(() {
    mockRepository = MockICampaignRepository();
    usecase = GetCampaigns(mockRepository);
  });
  
  test('should get campaigns from repository', () async {
    // Test implementation
  });
}
```

**Checklist Phase 2:**
- [ ] Entities created with immutable properties
- [ ] Repository interfaces defined
- [ ] Use cases implemented
- [ ] Unit tests written (coverage > 90%)
- [ ] No framework dependencies in domain layer

---

### Phase 3: Data Layer Implementation
**Người thực hiện**: Senior Developer  
**Review bởi**: Architecture Expert

**Bước 3.1: Create Models**
```dart
// lib/src/features/campaign/data/models/campaign_model.dart
@JsonSerializable()
class CampaignModel {
  final String id;
  final String name;
  @JsonKey(name: 'start_date')
  final String startDate;
  
  CampaignModel({required this.id, required this.name, required this.startDate});
  
  factory CampaignModel.fromJson(Map<String, dynamic> json) =>
      _$CampaignModelFromJson(json);
  
  Map<String, dynamic> toJson() => _$CampaignModelToJson(this);
  
  Campaign toEntity() => Campaign(
    id: id,
    name: name,
    startDate: DateTime.parse(startDate),
  );
}
```

// turbo
**Bước 3.2: Generate JSON Serialization**
```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

**Bước 3.3: Implement API Client**
```dart
// lib/src/features/campaign/data/datasources/campaign_api_client.dart
class CampaignApiClient {
  final Dio dio;
  
  CampaignApiClient(this.dio);
  
  Future<List<CampaignModel>> fetchCampaigns() async {
    final response = await dio.get('/api/campaigns');
    return (response.data['data'] as List)
        .map((json) => CampaignModel.fromJson(json))
        .toList();
  }
}
```

**Bước 3.4: Implement Repository**
```dart
// lib/src/features/campaign/data/repositories/campaign_repository.dart
class CampaignRepository implements ICampaignRepository {
  final CampaignApiClient apiClient;
  final CampaignLocalStorage localStorage;
  
  CampaignRepository(this.apiClient, this.localStorage);
  
  @override
  Future<List<Campaign>> getCampaigns() async {
    try {
      final models = await apiClient.fetchCampaigns();
      await localStorage.saveCampaigns(models);
      return models.map((m) => m.toEntity()).toList();
    } catch (e) {
      final cached = await localStorage.getCampaigns();
      return cached.map((m) => m.toEntity()).toList();
    }
  }
}
```

**Checklist Phase 3:**
- [ ] Models with JSON serialization
- [ ] API client implemented
- [ ] Local storage implemented
- [ ] Repository implemented with error handling
- [ ] Unit tests for repository
- [ ] Offline support (cache fallback)

---

### Phase 4: Presentation Layer Implementation
**Người thực hiện**: Senior Developer  
**Review bởi**: Performance Expert

**Bước 4.1: Define BLoC Events & States**
```dart
// lib/src/features/campaign/presentation/bloc/campaign_event.dart
abstract class CampaignEvent {}
class LoadCampaigns extends CampaignEvent {}

// lib/src/features/campaign/presentation/bloc/campaign_state.dart
abstract class CampaignState {}
class CampaignInitial extends CampaignState {}
class CampaignLoading extends CampaignState {}
class CampaignLoaded extends CampaignState {
  final List<Campaign> campaigns;
  CampaignLoaded(this.campaigns);
}
```

**Bước 4.2: Implement BLoC**
```dart
// lib/src/features/campaign/presentation/bloc/campaign_bloc.dart
class CampaignBloc extends Bloc<CampaignEvent, CampaignState> {
  final GetCampaigns getCampaigns;
  
  CampaignBloc({required this.getCampaigns}) : super(CampaignInitial()) {
    on<LoadCampaigns>(_onLoadCampaigns);
  }
  
  Future<void> _onLoadCampaigns(
    LoadCampaigns event,
    Emitter<CampaignState> emit,
  ) async {
    emit(CampaignLoading());
    try {
      final campaigns = await getCampaigns();
      emit(CampaignLoaded(campaigns));
    } catch (e) {
      emit(CampaignError(e.toString()));
    }
  }
}
```

**Bước 4.3: Create UI Pages**
```dart
// lib/src/features/campaign/presentation/pages/campaign_list_page.dart
class CampaignListPage extends StatelessWidget {
  const CampaignListPage({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => getIt<CampaignBloc>()..add(LoadCampaigns()),
      child: Scaffold(
        appBar: AppBar(title: const Text('Campaigns')),
        body: BlocBuilder<CampaignBloc, CampaignState>(
          builder: (context, state) {
            if (state is CampaignLoading) {
              return const Center(child: CircularProgressIndicator());
            }
            if (state is CampaignLoaded) {
              return _buildList(state.campaigns);
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }
  
  Widget _buildList(List<Campaign> campaigns) {
    return ListView.builder(
      itemCount: campaigns.length,
      itemBuilder: (context, index) {
        return CampaignCard(campaign: campaigns[index]);
      },
    );
  }
}
```

**Bước 4.4: Performance Optimization**
- [ ] Sử dụng `const` constructors
- [ ] ListView.builder cho danh sách dài
- [ ] RepaintBoundary cho animations
- [ ] Optimize images với cacheWidth/cacheHeight
- [ ] Selective BLoC listening với buildWhen

**Checklist Phase 4:**
- [ ] BLoC events/states defined
- [ ] BLoC implemented
- [ ] UI pages created
- [ ] Widgets extracted và reusable
- [ ] Performance optimized
- [ ] BLoC tests written
- [ ] Widget tests written

---

### Phase 5: Integration & Routing
**Người thực hiện**: Senior Developer

**Bước 5.1: Register Dependencies**
```dart
// infrastructure/lib/src/di/service_locator.dart
void setupCampaignDependencies() {
  // Data sources
  getIt.registerLazySingleton<CampaignApiClient>(
    () => CampaignApiClient(getIt<Dio>()),
  );
  
  // Repositories
  getIt.registerLazySingleton<ICampaignRepository>(
    () => CampaignRepository(getIt(), getIt()),
  );
  
  // Use cases
  getIt.registerLazySingleton(() => GetCampaigns(getIt()));
  
  // BLoC
  getIt.registerFactory(() => CampaignBloc(getCampaigns: getIt()));
}
```

**Bước 5.2: Configure Routes**
```dart
// infrastructure/lib/src/routing/app_router.dart
final campaignRoutes = [
  GoRoute(
    path: '/campaigns',
    name: 'campaigns',
    builder: (context, state) => const CampaignListPage(),
  ),
  GoRoute(
    path: '/campaigns/:id',
    name: 'campaign-detail',
    builder: (context, state) {
      final id = state.pathParameters['id']!;
      return CampaignDetailPage(campaignId: id);
    },
  ),
];
```

**Checklist Phase 5:**
- [ ] Dependencies registered
- [ ] Routes configured
- [ ] Navigation tested
- [ ] Deep linking works

---

### Phase 6: Testing
**Người thực hiện**: QA Engineer

// turbo
**Bước 6.1: Run All Tests**
```bash
# Unit tests
flutter test

# Coverage report
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
```

**Bước 6.2: Integration Testing**
```dart
// integration_test/campaign_flow_test.dart
void main() {
  testWidgets('complete campaign flow', (tester) async {
    await tester.pumpWidget(MyApp());
    
    // Navigate to campaigns
    await tester.tap(find.text('Campaigns'));
    await tester.pumpAndSettle();
    
    // Verify list loads
    expect(find.byType(CampaignCard), findsWidgets);
  });
}
```

**Bước 6.3: Manual Testing**
- [ ] Test trên iOS device
- [ ] Test trên Android device
- [ ] Test offline mode
- [ ] Test error scenarios
- [ ] Performance testing

**Checklist Phase 6:**
- [ ] All unit tests pass
- [ ] Test coverage > 80%
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] No critical bugs

---

### Phase 7: Code Review
**Người thực hiện**: Architecture Expert, Performance Expert

// turbo
**Bước 7.1: Create Pull Request**
```bash
git add .
git commit -m "feat({TASK_ID}): implement campaign feature"
git push origin feature/{TASK_ID}-campaign
```

**Bước 7.2: Code Review Checklist**

**Architecture Review:**
- [ ] Follows Clean Architecture
- [ ] Proper layer separation
- [ ] No circular dependencies
- [ ] Repository pattern correct

**Code Quality:**
- [ ] Naming conventions followed
- [ ] Code is readable
- [ ] No code duplication
- [ ] Proper error handling

**Performance:**
- [ ] No jank (60 FPS)
- [ ] Proper widget optimization
- [ ] Memory leaks prevented
- [ ] Images optimized

**Testing:**
- [ ] Test coverage > 80%
- [ ] All tests pass
- [ ] Edge cases covered

**Bước 7.3: Address Review Comments**
- Fix issues raised in review
- Update tests if needed
- Request re-review

---

### Phase 8: Merge & Deploy

// turbo
**Bước 8.1: Merge to Develop**
```bash
git checkout develop
git pull origin develop
git merge feature/{TASK_ID}-campaign
git push origin develop
```

// turbo
**Bước 8.2: Run CI/CD Pipeline**
```bash
# Bitbucket Pipelines will automatically:
# - Run tests
# - Build app
# - Deploy to staging
```

**Bước 8.3: Staging Testing**
- [ ] QA tests on staging
- [ ] Stakeholder review
- [ ] Performance validation

**Bước 8.4: Production Deployment**
- [ ] Merge develop → main
- [ ] Tag release
- [ ] Deploy to production
- [ ] Monitor for issues

---

## Checklist Tổng hợp

### Development Complete
- [x] Domain layer implemented
- [x] Data layer implemented
- [x] Presentation layer implemented
- [x] Integration complete
- [x] All tests pass
- [x] Code reviewed
- [x] Performance validated

### Quality Gates
- [x] Test coverage > 80%
- [x] No critical/high bugs
- [x] Performance benchmarks met
- [x] Code review approved
- [x] Architecture compliant

### Deployment Ready
- [x] Merged to develop
- [x] CI/CD pipeline passed
- [x] Staging tested
- [x] Documentation updated
- [x] Ready for production

---

**Remember**: Ship quality code, not just working code! 🚀
