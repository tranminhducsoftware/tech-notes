# Clean Architecture – Tổ chức code cho .NET

> Mục tiêu: Rõ ràng *chỗ đặt* **Entity / ValueObject / Enum / DTO / Utils / Handler**, quy tắc phụ thuộc, naming, và layout áp dụng tốt cho monolith và microservice nhỏ.

---

## 1) Tổng quan solution

```less
src/
├─ MyApp.Domain/                // Trung tâm - chỉ business model & rules
├─ MyApp.Application/           // Use cases, CQRS, validation, contracts nội bộ
├─ MyApp.Infrastructure/        // EF Core, Redis, Kafka, Email, Files, 3rd party
├─ MyApp.Presentation/          // API/Minimal API/SignalR/GRPC (controllers, filters)
├─ MyApp.Contracts/             // (tuỳ chọn) DTO public cho API (request/response)
├─ MyApp.SharedKernel/          // (tuỳ chọn) base types, spec, primitives chia sẻ
tests/
├─ MyApp.UnitTests/
├─ MyApp.IntegrationTests/
```

### Quy tắc phụ thuộc (bắt buộc)

- `Presentation` → **Application**
- `Infrastructure` → **Application** & **Domain**
- `Application` → **Domain**
- `Domain` → **không tham chiếu** ai (độc lập hoàn toàn)

> Không để `Presentation` hoặc `Infrastructure` tham chiếu **ngược** nhau. Mọi thứ đi **qua Application**.

---

## 2) MyApp.Domain (Entities/ValueObjects/Enums/Domain logic)

```less
MyApp.Domain/
├─ Entities/
│ ├─ User.cs
│ ├─ Order.cs
│ └─ ...
├─ ValueObjects/
│ ├─ Money.cs
│ ├─ Email.cs
│ └─ ...
├─ Enums/
│ ├─ OrderStatus.cs
│ └─ ...
├─ Events/
│ ├─ OrderPlaced.cs
│ └─ ...
├─ Services/ # Domain Services (pure, không IO)
│ └─ PricingService.cs
├─ Specifications/ # (tuỳ chọn) Ardalis.Specification
└─ Abstractions/ # (rất hạn chế, ví dụ IClock cho test)
└─ IClock.cs
```

**Nguyên tắc:**
- **Entity/ValueObject/Enum** là “nguồn sự thật” business → luôn ở **Domain**.
- **Không** DataAnnotations EF, **không** phụ thuộc hạ tầng.
- **Invariants** đảm bảo trong constructor/factory; **ValueObject** bất biến.
- **Domain Events** chỉ mô tả sự kiện; App layer sẽ lắng nghe/điều phối.

---

## 3) MyApp.Application (CQRS, DTO nội bộ, Validation, Abstractions)

```less
MyApp.Application/
├─ Abstractions/ # "Ports" do App định nghĩa để Infra implement
│ ├─ Persistence/
│ │ ├─ IUnitOfWork.cs
│ │ └─ IUserRepository.cs
│ ├─ Messaging/IEventBus.cs
│ ├─ Caching/ICacheService.cs
│ └─ Security/ICurrentUser.cs
├─ Behaviors/ # MediatR pipeline: Validation/Transaction/Logging/Retry
│ ├─ ValidationBehavior.cs
│ ├─ TransactionBehavior.cs
│ ├─ PerformanceBehavior.cs
│ └─ RetryBehavior.cs
├─ Common/
│ ├─ Exceptions/ (NotFound, Conflict, ValidationFailed…)
│ ├─ Mappings/ (Mapster/AutoMapper profiles: Domain ⇄ DTO/Contracts)
│ ├─ Policies/ (ví dụ RefreshTokenPolicy)
│ ├─ Results/ (Result<T>, Error, PaginatedList<T>)
│ └─ Constants/
├─ Features/ # Feature-sliced theo bounded context
│ ├─ Auth/
│ │ ├─ Commands/
│ │ │ ├─ LoginCommand.cs
│ │ │ ├─ LoginCommandHandler.cs
│ │ │ └─ ...
│ │ ├─ Queries/
│ │ ├─ Dtos/ # DTO nội bộ (không nhất thiết trùng API contracts)
│ │ ├─ Validators/ # FluentValidation
│ │ └─ Policies/
│ ├─ Products/
│ │ ├─ Commands | Queries | Dtos | Validators
│ │ └─ ...
│ └─ ...
└─ DependencyInjection.cs # AddMediatR, AddValidators, Behaviors,...
```


**Handlers ở đâu?**  
- Trong **Features/\<Feature\>/Commands|Queries** (CQRS).  
- **Validators** đặt cạnh command/query.

**DTO ở đâu?**  
- **Application/Features/\<Feature\>/Dtos** dùng nội bộ use case.  
- Nếu cần API public ổn định → dùng project **MyApp.Contracts** cho request/response.

**“Utils” trong Application?**  
- Tránh thư mục “Utils”. Đặt **service/primitive** có tên ngữ nghĩa trong **Abstractions** hoặc **Common**.

---

## 4) MyApp.Infrastructure (adapters: EF/Redis/Kafka/Email/Files…)

```yaml
MyApp.Infrastructure/
├─ Persistence/
│ ├─ MyAppDbContext.cs
│ ├─ Configurations/ # 1 file/entity: IEntityTypeConfiguration<T>
│ │ ├─ UserConfiguration.cs
│ │ └─ OrderConfiguration.cs
│ ├─ Repositories/
│ │ ├─ UserRepository.cs # implements IUserRepository
│ │ └─ ...
│ ├─ Interceptors/
│ │ ├─ AuditableEntityInterceptor.cs
│ │ └─ CommandLoggingInterceptor.cs
│ └─ Migrations/
├─ Caching/
│ ├─ RedisCacheService.cs
│ └─ ...
├─ Messaging/
│ ├─ KafkaEventBus.cs
│ └─ EmailSender.cs
├─ Security/
│ ├─ JwtTokenGenerator.cs
│ └─ PasswordHasher.cs
├─ Files/
│ └─ S3FileStorage.cs
└─ DependencyInjection.cs # AddDbContext, add repos, add external clients
```

**Nguyên tắc:**
- **EF Config** để ở `Persistence/Configurations` (mỗi entity 1 file).
- **Repositories** implement **interfaces ở Application.Abstractions**.
- Không đặt DTO/Handler ở đây.  
- **Options/Settings** bind từ `appsettings` (class đặt ở Infra hoặc SharedKernel).

---

## 5) MyApp.Presentation (API/Minimal API/GRPC/SignalR)

```yaml
MyApp.Presentation/
├─ Controllers/ or Endpoints/ # Minimal APIs chia theo feature
├─ Filters/ # Exception/Validation/Authorization
├─ Middlewares/ # Logging, CorrelationId, GlobalErrors
├─ Auth/ # JWT, policies, authorization handlers
├─ Swagger/
└─ DependencyInjection.cs # CORS, Auth, RateLimit, Versioning, Swagger
```

**Nguyên tắc:**
- **Controller/Endpoint** chỉ: bind request → `IMediator.Send()` → map response.  
- **Không** viết business logic ở Controller.  
- Map Domain ⇄ DTO ở **Application.Common.Mappings** (hoặc Contracts).

---

## 6) MyApp.Contracts (tuỳ chọn – API contracts public)

```yaml
MyApp.Contracts/
├─ Auth/
│ ├─ LoginRequest.cs
│ ├─ LoginResponse.cs
│ └─ ...
├─ Products/
│ ├─ CreateProductRequest.cs
│ ├─ ProductResponse.cs
│ └─ ...
└─ Common/
├─ PagingRequest.cs
├─ PagingResponse.cs
└─ ProblemDetails.cs
```

- Đảm bảo **versioning** & ổn định đối ngoại.
- Controller nên nhận/trả **Contracts** thay vì DTO nội bộ.

---

## 7) MyApp.SharedKernel (tuỳ chọn – base types chia sẻ)

```markdown
MyApp.SharedKernel/
├─ Primitives/
│ ├─ Entity.cs
│ ├─ AggregateRoot.cs
│ ├─ ValueObject.cs
│ └─ Guard.cs
├─ Specifications/
├─ Results/
└─ Time/IClock.cs (nếu cần)
```

> Đừng nhét rule business cụ thể vào SharedKernel.

---

## 8) Quy ước quan trọng

### Entities & ValueObjects
- **Không** dùng DataAnnotations EF; mọi mapping để ở Infra.
- **Invariants** enforce trong constructor/factory.
- **ValueObject**: bất biến, equality theo *components*.

### DTO
- **Application DTO**: phục vụ use case, có thể khác API contract.
- **Contracts**: hợp đồng đối ngoại tách hẳn project để public & versioning.

### Enums
- Enum business → **Domain/Enums**.  
- API nên **serialize as string** (tránh rò giá trị số).

### Handlers (MediatR)
- Đặt theo **Feature**: `Features/<Feature>/Commands|Queries`.
- **Không** gọi 3rd party trực tiếp → qua **Application.Abstractions**.

### Mapping
- Profiles ở **Application.Common.Mappings**.
- Controller **không** map thủ công.

### Validation
- **FluentValidation** cạnh command/query.
- Đăng ký **ValidationBehavior** toàn cục.

### Persistence
- EF config theo **entity-per-file** trong `Configurations`.
- Interceptors: Auditable/Logging/Outbox/SoftDelete.

### Config/Options
- Class options ở **Infra** (nếu chỉ dùng Infra) hoặc **SharedKernel** (nếu dùng chung).
- Bind `IOptions<T>` tại **Presentation**.

### Error handling
- Exception types ở **Application.Common.Exceptions** (NotFound/Conflict/Validation…).
- Middleware/Filter ở **Presentation** chuẩn hoá **ProblemDetails**.

### Naming
- Command: `VerbNounCommand`, Query: `GetNounQuery`.
- Repository: `I<User>Repository` (Application) → `UserRepository` (Infra).
- Indexing feature theo **bounded context** (Auth, Billing, Catalog…), **không** theo “technical layer”.

---

## 9) Module hoá khi app lớn (t tuỳ khẩu vị)

Thay vì 1 Application khổng lồ, chia module slice hoàn chỉnh:

```markdown
MyApp.Catalog.Domain
MyApp.Catalog.Application
MyApp.Catalog.Infrastructure
MyApp.Catalog.API
(… các bounded context khác …)
```

> Mỗi module giữ đúng luật phụ thuộc; SharedKernel dùng thật tiết kiệm.

---

## 10) Checklist best practices

- [ ] **Luật phụ thuộc** đúng như trên (verify bằng ArchUnitNet/NetArchTest).
- [ ] **CQRS + MediatR**: Commands/Queries theo Feature; dùng **Pipeline Behaviors**.
- [ ] **Validation**: FluentValidation + ValidationBehavior.
- [ ] **Transaction**: TransactionBehavior (UnitOfWork).
- [ ] **Logging/Tracing**: Middleware + Enrichers (Serilog) + OpenTelemetry.
- [ ] **Mapping**: profiles tập trung, testable.
- [ ] **EF Config**: 1 entity/1 config file; Interceptors cho audit/log/outbox.
- [ ] **Configuration**: Options pattern + validation.
- [ ] **Error handling**: Global exception → ProblemDetails.
- [ ] **Contracts** tách project nếu API public/đa client.

---

## 11) Anti-patterns cần tránh

- Domain biết EF/SQL/Redis (leak hạ tầng).
- Controller viết business logic, gọi thẳng DbContext.
- Thư mục **Utils** “túi rác”.
- Dùng Domain Enums trực tiếp làm API response (khó versioning/breaking).
- DTO trùng lặp lung tung giữa Application và API → khó bảo trì.
- Thiếu Pipeline Behaviors → cross-cutting rải rác trong handlers.

---

## 12) Mẫu cây thư mục rút gọn (copy/paste)

```yaml
MyApp.Domain/
Entities/.cs
ValueObjects/.cs
Enums/.cs
Events/.cs
Services/.cs
Specifications/.cs

MyApp.Application/
Abstractions/(Persistence|Messaging|Security|Caching)/.cs
Behaviors/.cs
Common/(Mappings|Results|Policies|Exceptions|Constants)/.cs
Features/
Auth/
Commands/.cs
Queries/.cs
Dtos/.cs
Validators/.cs
Products/
Commands|Queries|Dtos|Validators/.cs
DependencyInjection.cs

MyApp.Infrastructure/
Persistence/(DbContext|Configurations|Repositories|Interceptors)/.cs
Caching/.cs
Messaging/.cs
Security/.cs
Files/*.cs
DependencyInjection.cs

MyApp.Presentation/
Controllers/.cs
Filters/.cs
Middlewares/.cs
Auth/.cs
Swagger/*.cs
DependencyInjection.cs

MyApp.Contracts/ (tuỳ chọn)
<Feature>/[Request|Response].cs
```
