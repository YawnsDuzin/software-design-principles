# YAGNI 원칙 (You Ain't Gonna Need It)

> "지금 필요하지 않다면 만들지 마라."
> — Ron Jeffries, XP(Extreme Programming) 공동 창시자

> "미래를 예측하는 가장 좋은 방법은 미래를 만들어 가는 것이다. 단, 필요할 때."
> — 실용주의 프로그래머의 교훈

---

## 목차

1. [YAGNI란 무엇인가?](#1-yagni란-무엇인가)
2. [XP에서 온 원칙](#2-xp에서-온-원칙)
3. [왜 미리 만들면 안 되는가?](#3-왜-미리-만들면-안-되는가)
4. [나쁜 예제 (Before)](#4-나쁜-예제-before)
5. [좋은 예제 (After)](#5-좋은-예제-after)
6. [Python 코드 예제](#6-python-코드-예제)
7. [C# 코드 예제](#7-c-코드-예제)
8. [YAGNI vs 확장성: 균형잡기](#8-yagni-vs-확장성-균형잡기)
9. [YAGNI와 SOLID의 관계](#9-yagni와-solid의-관계)
10. [실무 팁](#10-실무-팁)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 원칙](#12-관련-원칙)

---

## 1. YAGNI란 무엇인가?

YAGNI(You Ain't Gonna Need It)는 **"실제로 필요할 때까지 기능을 구현하지 마라"**는 소프트웨어 개발 원칙입니다. 지금 당장 필요하지 않은 기능을 "나중에 필요할 것 같아서" 미리 만드는 것을 경계합니다.

### 핵심 정의

**현재 요구사항을 해결하는 가장 단순한 코드를 작성하라. 미래의 요구사항을 추측하여 미리 구현하지 마라.**

### 비유로 이해하기

여행 가방을 생각해 보세요. "만약에 비가 오면?", "만약에 산에 가면?", "만약에 수영하면?"... 이 모든 "만약"을 대비해 짐을 싸면, 가방은 들 수 없을 만큼 무거워집니다. 결국 실제로 쓴 건 가방 안의 20%뿐입니다.

코드도 마찬가지입니다. "만약에"를 대비한 코드는 대부분 쓰이지 않지만, 유지보수 비용은 계속 발생합니다.

---

## 2. XP에서 온 원칙

YAGNI는 1990년대 후반 **Extreme Programming(XP)**에서 탄생했습니다. XP의 핵심 가치는 다음과 같습니다:

| XP 가치 | YAGNI와의 관계 |
|---------|----------------|
| 단순성 (Simplicity) | 필요한 것만 만들어 단순함 유지 |
| 피드백 (Feedback) | 빠른 피드백으로 필요한 것을 정확히 파악 |
| 용기 (Courage) | "나중에 추가하면 되지"라고 말할 용기 |
| 소통 (Communication) | 고객과 소통하여 진짜 필요한 것을 파악 |

### XP의 핵심 전제

> **요구사항은 반드시 변한다.**

미래를 예측하여 코드를 작성하는 대신, **변경에 빠르게 대응할 수 있는 유연한 코드**를 작성하는 것이 XP의 접근 방식입니다. YAGNI는 이 철학에서 자연스럽게 나온 원칙입니다.

---

## 3. 왜 미리 만들면 안 되는가?

### 3.1 비용 (Cost)

미리 구현한 기능은 다음 비용을 발생시킵니다:

```
미리 구현한 기능의 비용:
┌──────────────────────────────────────────────┐
│  개발 시간      ████████                      │
│  테스트 비용    ██████                        │
│  유지보수 비용  ████████████████████ (지속적!) │
│  코드 복잡도    ████████████                  │
│  인지 부담      ██████████                    │
│                                              │
│  실제 사용 확률: ▓░░░░░░░░░ (약 10~20%)       │
└──────────────────────────────────────────────┘
```

**Martin Fowler의 분석:**
- 미리 만든 기능의 약 **80%는 사용되지 않거나** 요구사항이 변경됨
- 사용되더라도 **원래 구현과 다른 형태**로 필요한 경우가 대부분

### 3.2 불확실성 (Uncertainty)

```python
# 6개월 전 개발자의 생각:
# "앞으로 결제 시스템에 비트코인이 필요할 거야!"
# → 회사는 비트코인 결제를 도입하지 않았음

# 3개월 전 개발자의 생각:
# "사용자가 10만 명이 넘으면 샤딩이 필요해!"
# → 현재 사용자 500명

# 1개월 전 개발자의 생각:
# "멀티 언어 지원이 필요할 거야!"
# → PM이 결정한 건 다크 모드였음
```

### 3.3 복잡도 증가

사용하지 않는 코드도 복잡도를 증가시킵니다:

- **코드를 읽는 사람**은 사용되지 않는 기능까지 이해해야 함
- **리팩토링할 때** 사용되지 않는 코드까지 수정해야 할 수 있음
- **테스트할 때** 사용되지 않는 코드 경로까지 테스트해야 함
- **버그가 숨어들** 수 있는 표면적이 넓어짐

---

## 4. 나쁜 예제 (Before)

### 4.1 "나중에 필요할 것 같은" 기능 미리 구현

```python
# ❌ Bad: 현재 요구사항은 "CSV 내보내기"인데,
# "나중에 다른 포맷도 필요할 것 같아서" 전체 구조를 미리 만듦

from abc import ABC, abstractmethod

class ExportFormat(ABC):
    @abstractmethod
    def export(self, data: list[dict]) -> str:
        pass

    @abstractmethod
    def get_content_type(self) -> str:
        pass

    @abstractmethod
    def get_file_extension(self) -> str:
        pass

class CsvExporter(ExportFormat):
    def export(self, data):
        # CSV 내보내기 (실제 필요한 것)
        pass
    def get_content_type(self):
        return "text/csv"
    def get_file_extension(self):
        return ".csv"

class JsonExporter(ExportFormat):
    def export(self, data):
        # 아직 아무도 요청하지 않은 JSON 내보내기
        pass
    def get_content_type(self):
        return "application/json"
    def get_file_extension(self):
        return ".json"

class XmlExporter(ExportFormat):
    def export(self, data):
        # 아직 아무도 요청하지 않은 XML 내보내기
        pass
    def get_content_type(self):
        return "application/xml"
    def get_file_extension(self):
        return ".xml"

class PdfExporter(ExportFormat):
    def export(self, data):
        # 아직 아무도 요청하지 않은 PDF 내보내기
        pass
    def get_content_type(self):
        return "application/pdf"
    def get_file_extension(self):
        return ".pdf"

class ExportFormatFactory:
    _formats = {
        "csv": CsvExporter,
        "json": JsonExporter,
        "xml": XmlExporter,
        "pdf": PdfExporter,
    }

    @staticmethod
    def create(format_type: str) -> ExportFormat:
        exporter_class = ExportFormatFactory._formats.get(format_type)
        if not exporter_class:
            raise ValueError(f"Unknown format: {format_type}")
        return exporter_class()

# 결과: CSV 하나만 필요한데 5개의 클래스 + 1개의 팩토리를 만들었음
```

### 4.2 과도한 설정 가능성 (Configurability)

```csharp
// ❌ Bad: 현재는 "로그를 콘솔에 출력"만 필요한데,
// "나중을 위해" 모든 것을 설정 가능하게 만듦
public class SuperFlexibleLogger
{
    public LogLevel MinLevel { get; set; }
    public LogLevel MaxLevel { get; set; }
    public string DateFormat { get; set; }
    public string MessageTemplate { get; set; }
    public bool IncludeStackTrace { get; set; }
    public bool IncludeThreadId { get; set; }
    public bool IncludeProcessId { get; set; }
    public bool UseUtc { get; set; }
    public int MaxMessageLength { get; set; }
    public string FieldSeparator { get; set; }
    public bool AsyncMode { get; set; }
    public int BufferSize { get; set; }
    public TimeSpan FlushInterval { get; set; }
    public bool EnableColorOutput { get; set; }
    public Dictionary<LogLevel, ConsoleColor> ColorMap { get; set; }
    public List<ILogFilter> Filters { get; set; }
    public List<ILogEnricher> Enrichers { get; set; }
    public ILogSerializer Serializer { get; set; }
    public RetryPolicy RetryPolicy { get; set; }
    // ... 20개 이상의 설정 옵션

    public void Log(LogLevel level, string message)
    {
        // 모든 설정을 처리하는 100줄짜리 메서드
    }
}
```

### 4.3 불필요한 확장 포인트

```python
# ❌ Bad: "확장성"을 위해 플러그인 시스템까지 미리 구축
class PluginManager:
    """현재 플러그인이 하나도 없는데 만든 플러그인 시스템"""
    def __init__(self):
        self._plugins = {}
        self._hooks = defaultdict(list)

    def register_plugin(self, name, plugin):
        self._plugins[name] = plugin

    def register_hook(self, event, callback):
        self._hooks[event].append(callback)

    def trigger_hook(self, event, *args, **kwargs):
        for callback in self._hooks[event]:
            callback(*args, **kwargs)

    def get_plugin(self, name):
        return self._plugins.get(name)

class EventBus:
    """현재 이벤트가 하나도 없는데 만든 이벤트 버스"""
    pass

class MiddlewarePipeline:
    """현재 미들웨어가 하나도 없는데 만든 파이프라인"""
    pass

# 이 모든 "인프라"가 있지만, 실제 비즈니스 로직은 아직 없음
```

---

## 5. 좋은 예제 (After)

### 5.1 현재 요구사항에 집중한 구현

```python
# ✅ Good: 필요한 것만 구현 (CSV 내보내기)
import csv
import io

def export_to_csv(data: list[dict]) -> str:
    """데이터를 CSV 형식으로 내보냅니다."""
    if not data:
        return ""

    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=data[0].keys())
    writer.writeheader()
    writer.writerows(data)
    return output.getvalue()

# 나중에 JSON이 정말 필요하면 그때 추가하면 된다.
# 그때는 실제 요구사항을 알고 있으므로 더 올바른 설계가 가능하다.
```

### 5.2 필요할 때 리팩토링

```csharp
// ✅ Good: 처음에는 단순하게
public class Logger
{
    public void Log(string message)
    {
        Console.WriteLine($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {message}");
    }

    public void LogError(string message, Exception? ex = null)
    {
        Console.WriteLine($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] ERROR: {message}");
        if (ex != null)
            Console.WriteLine($"  Exception: {ex.Message}");
    }
}

// 나중에 파일 로깅이 정말 필요해지면?
// 그때 ILogger 인터페이스를 추출하면 된다.
// IDE의 "Extract Interface" 리팩토링 기능으로 5분이면 충분하다.
```

### 5.3 점진적 확장

```python
# ✅ Good: 기능이 정말 필요해졌을 때의 자연스러운 진화

# === 1단계: 최초 요구사항 - CSV만 필요 ===
def export_to_csv(data: list[dict]) -> str:
    # ... CSV 구현 ...
    pass

# === 2단계: JSON이 정말 필요해짐 (요구사항 확정) ===
def export_to_csv(data: list[dict]) -> str:
    # ... CSV 구현 ...
    pass

def export_to_json(data: list[dict]) -> str:
    # ... JSON 구현 ...
    pass

# === 3단계: 3번째 포맷이 추가됨 → 이제 패턴이 보인다! 추상화 시점 ===
# Rule of Three에 따라 이제 리팩토링
from typing import Protocol

class Exporter(Protocol):
    def export(self, data: list[dict]) -> str: ...
    def content_type(self) -> str: ...

class CsvExporter:
    def export(self, data: list[dict]) -> str:
        # ...
        pass
    def content_type(self) -> str:
        return "text/csv"

class JsonExporter:
    def export(self, data: list[dict]) -> str:
        # ...
        pass
    def content_type(self) -> str:
        return "application/json"

class ExcelExporter:
    def export(self, data: list[dict]) -> str:
        # ...
        pass
    def content_type(self) -> str:
        return "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
```

---

## 6. Python 코드 예제

### 6.1 사용자 권한 시스템

```python
# ❌ Bad: "나중에 복잡한 권한이 필요할 거야"
# 현재 요구사항: 관리자/일반 사용자 구분만 필요

class Permission:
    def __init__(self, name, description, category):
        self.name = name
        self.description = description
        self.category = category

class Role:
    def __init__(self, name):
        self.name = name
        self.permissions = []
        self.parent_roles = []  # 역할 상속

    def add_permission(self, permission):
        self.permissions.append(permission)

    def add_parent_role(self, role):
        self.parent_roles.append(role)

    def get_all_permissions(self):
        """상속된 권한까지 모두 수집"""
        perms = set(p.name for p in self.permissions)
        for parent in self.parent_roles:
            perms.update(parent.get_all_permissions())
        return perms

class PermissionManager:
    def __init__(self):
        self.roles = {}
        self.user_roles = {}
        self.permission_cache = {}

    def create_role(self, name):
        self.roles[name] = Role(name)

    def assign_role(self, user_id, role_name):
        if user_id not in self.user_roles:
            self.user_roles[user_id] = []
        self.user_roles[user_id].append(role_name)
        self._invalidate_cache(user_id)

    def has_permission(self, user_id, permission_name):
        if user_id in self.permission_cache:
            return permission_name in self.permission_cache[user_id]
        # ... 복잡한 캐시 + 상속 해결 로직

    def _invalidate_cache(self, user_id):
        self.permission_cache.pop(user_id, None)

# 이 모든 구조를 만들었지만, 실제로는 is_admin 하나면 충분했음
```

```python
# ✅ Good: 현재 필요한 것만 (관리자/일반 구분)
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str
    is_admin: bool = False

def require_admin(func):
    """관리자 권한이 필요한 함수를 위한 데코레이터"""
    @functools.wraps(func)
    def wrapper(user: User, *args, **kwargs):
        if not user.is_admin:
            raise PermissionError("관리자 권한이 필요합니다.")
        return func(user, *args, **kwargs)
    return wrapper

@require_admin
def delete_user(admin: User, target_user_id: int):
    """사용자 삭제 (관리자만 가능)"""
    # 삭제 로직...
    pass

# 나중에 정말 세분화된 권한이 필요하면?
# 그때 is_admin을 role로 변경하면 된다.
# 현재 코드가 단순하므로 리팩토링도 쉽다!
```

### 6.2 API 응답 포맷

```python
# ❌ Bad: 현재 JSON만 필요한데 "나중을 위해" 콘텐츠 협상까지 구현
class ContentNegotiator:
    """아직 JSON만 쓰는데 만든 콘텐츠 협상기"""
    def __init__(self):
        self._serializers = {}

    def register(self, content_type, serializer):
        self._serializers[content_type] = serializer

    def negotiate(self, accept_header):
        for content_type in self._parse_accept(accept_header):
            if content_type in self._serializers:
                return self._serializers[content_type]
        raise NotAcceptable()

    def _parse_accept(self, header):
        # Accept 헤더 파싱 로직 (q-factor 지원 등)...
        pass
```

```python
# ✅ Good: JSON만 반환
from flask import jsonify

@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    user = user_service.get_by_id(user_id)
    if not user:
        return jsonify({"error": "User not found"}), 404
    return jsonify(user.to_dict())

# 깔끔하고 명확하다. XML이 필요해지면 그때 추가하자.
```

### 6.3 데이터베이스 접근

```python
# ❌ Bad: "나중에 DB를 바꿀 수 있으니" 추상화 레이어를 미리 구축
class DatabaseAdapter(ABC):
    @abstractmethod
    def connect(self): pass
    @abstractmethod
    def execute(self, query, params): pass
    @abstractmethod
    def fetch_one(self, query, params): pass
    @abstractmethod
    def fetch_all(self, query, params): pass
    @abstractmethod
    def begin_transaction(self): pass
    @abstractmethod
    def commit(self): pass
    @abstractmethod
    def rollback(self): pass

class PostgresAdapter(DatabaseAdapter):
    # PostgreSQL 구현 (현재 사용 중)
    pass

class MySqlAdapter(DatabaseAdapter):
    # MySQL 구현 (아무도 요청하지 않음)
    pass

class SqliteAdapter(DatabaseAdapter):
    # SQLite 구현 (아무도 요청하지 않음)
    pass

class DatabaseFactory:
    @staticmethod
    def create(db_type: str) -> DatabaseAdapter:
        # ...
        pass
```

```python
# ✅ Good: 현재 사용하는 DB에 집중
# SQLAlchemy나 Django ORM을 쓰면 DB 교체도 나중에 쉽게 가능
import sqlalchemy as sa

engine = sa.create_engine("postgresql://localhost/mydb")

class UserRepository:
    def __init__(self, session):
        self.session = session

    def find_by_id(self, user_id: int) -> User | None:
        return self.session.query(User).get(user_id)

    def find_by_email(self, email: str) -> User | None:
        return self.session.query(User).filter_by(email=email).first()

    def save(self, user: User):
        self.session.add(user)
        self.session.flush()

# ORM이 이미 DB 추상화를 제공한다.
# 직접 추상화 레이어를 만들 필요가 없다.
```

---

## 7. C# 코드 예제

### 7.1 알림 시스템

```csharp
// ❌ Bad: "나중에 SMS, 카카오톡, Slack도 필요할 거야"
// 현재 요구사항: 이메일 알림만 필요

public interface INotificationChannel
{
    Task SendAsync(Notification notification);
    bool CanHandle(NotificationType type);
    int Priority { get; }
}

public interface INotificationRouter
{
    Task RouteAsync(Notification notification);
}

public interface INotificationTemplateEngine
{
    string Render(string template, Dictionary<string, object> variables);
}

public interface INotificationRetryPolicy
{
    int MaxRetries { get; }
    TimeSpan GetDelay(int attemptNumber);
}

public class EmailChannel : INotificationChannel { /* ... */ }
public class SmsChannel : INotificationChannel { /* ... */ }  // 아직 불필요
public class SlackChannel : INotificationChannel { /* ... */ }  // 아직 불필요

public class NotificationRouter : INotificationRouter
{
    private readonly IEnumerable<INotificationChannel> _channels;
    // ... 복잡한 라우팅 로직
}

// 인터페이스 4개 + 구현 클래스 4개 = 8개 파일
// 실제로 쓰는 건 이메일 하나...
```

```csharp
// ✅ Good: 이메일 알림만 구현
public class EmailNotificationService
{
    private readonly SmtpClient _smtpClient;
    private readonly ILogger<EmailNotificationService> _logger;

    public EmailNotificationService(
        SmtpClient smtpClient,
        ILogger<EmailNotificationService> logger)
    {
        _smtpClient = smtpClient;
        _logger = logger;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        try
        {
            var message = new MailMessage(_fromAddress, to, subject, body);
            await _smtpClient.SendMailAsync(message);
            _logger.LogInformation("이메일 전송 완료: {To}", to);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "이메일 전송 실패: {To}", to);
            throw;
        }
    }
}

// 깔끔하고 이해하기 쉽다.
// SMS가 정말 필요해지면? 그때 INotificationService 인터페이스를 추출하면 된다.
```

### 7.2 캐싱

```csharp
// ❌ Bad: "성능을 위해" 아직 필요 없는 캐시를 미리 구현
public interface ICacheProvider
{
    Task<T?> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value, CacheOptions options);
    Task RemoveAsync(string key);
    Task<bool> ExistsAsync(string key);
    Task FlushAsync();
}

public class CacheOptions
{
    public TimeSpan? AbsoluteExpiration { get; set; }
    public TimeSpan? SlidingExpiration { get; set; }
    public CachePriority Priority { get; set; }
    public long? MaxSize { get; set; }
    public string? Region { get; set; }
}

public class MemoryCacheProvider : ICacheProvider { /* ... */ }
public class RedisCacheProvider : ICacheProvider { /* ... */ }
public class DistributedCacheProvider : ICacheProvider { /* ... */ }

public class CacheInterceptor  // AOP 캐시 인터셉터까지!
{
    public T GetOrSet<T>(string key, Func<T> factory, CacheOptions options)
    {
        // ...
    }
}

// 현재 사용자 100명인 서비스에 Redis + 분산 캐시???
```

```csharp
// ✅ Good: 캐시가 정말 필요해지면 그때 추가
// 일단은 직접 DB를 조회 (현재 성능에 문제가 없다면)
public class ProductService
{
    private readonly ProductRepository _repository;

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _repository.FindByIdAsync(id);
    }
}

// 만약 성능 이슈가 실제로 발생하면?
// .NET의 IMemoryCache를 간단히 추가하면 된다.
public class ProductService
{
    private readonly ProductRepository _repository;
    private readonly IMemoryCache _cache;

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _cache.GetOrCreateAsync(
            $"product:{id}",
            async entry =>
            {
                entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
                return await _repository.FindByIdAsync(id);
            });
    }
}
```

### 7.3 API 버전관리

```csharp
// ❌ Bad: API v1도 완성 안 됐는데 버전 관리 시스템부터 구축
public interface IApiVersionStrategy
{
    string GetVersion(HttpRequest request);
}

public class UrlVersionStrategy : IApiVersionStrategy { }
public class HeaderVersionStrategy : IApiVersionStrategy { }
public class QueryStringVersionStrategy : IApiVersionStrategy { }

public class ApiVersionRouter
{
    private readonly Dictionary<string, IApiVersionStrategy> _strategies;
    // ... 복잡한 버전 라우팅 로직
}
```

```csharp
// ✅ Good: v1 먼저 완성하자
[ApiController]
[Route("api/products")]
public class ProductController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _service.GetByIdAsync(id);
        return product != null ? Ok(product) : NotFound();
    }
}

// v2가 정말 필요해지면?
// 그때 [Route("api/v1/products")]로 변경하고
// v2 컨트롤러를 추가하면 된다.
// 또는 ASP.NET Core의 API Versioning 패키지를 사용하면 된다.
```

---

## 8. YAGNI vs 확장성: 균형잡기

YAGNI를 따르면서도 완전히 경직된 코드를 작성하는 것은 아닙니다. **핵심은 "확장 가능하게 만들되, 미리 확장하지는 마라"입니다.**

### 미리 하면 좋은 것 vs 나중에 해도 되는 것

| 미리 해야 하는 것 | 나중에 해도 되는 것 |
|---|---|
| 깔끔한 코드 구조 | 추상화 계층 추가 |
| 의미 있는 네이밍 | 인터페이스 추출 |
| 적절한 함수 크기 | 디자인 패턴 적용 |
| 테스트 작성 | 성능 최적화 |
| 의존성 주입 (DI) | 캐싱 |
| 버전 관리 (Git) | 분산 시스템 |

### 확장 가능한 단순 코드 작성법

```python
# ✅ Good: 단순하지만 나중에 확장하기 쉬운 코드

class OrderService:
    """현재는 단순하지만, 의존성 주입으로 테스트와 교체가 쉬움"""

    def __init__(self, repository, email_service):
        # 의존성 주입: 나중에 교체가 쉬움
        self._repository = repository
        self._email_service = email_service

    def create_order(self, user_id: int, items: list[dict]) -> Order:
        """주문 생성 - 현재 필요한 기능만"""
        order = Order(user_id=user_id, items=items)
        self._repository.save(order)
        self._email_service.send_order_confirmation(order)
        return order

    # cancel_order, refund_order 등은 요구사항이 생기면 그때 추가
```

---

## 9. YAGNI와 SOLID의 관계

### YAGNI와 SRP (단일 책임 원칙)

**조화:** 클래스가 하나의 책임만 가지면 나중에 수정/교체가 쉬움 → YAGNI를 실천하기 좋은 구조

### YAGNI와 OCP (개방-폐쇄 원칙)

**긴장:** OCP는 "확장에 열려있고, 수정에 닫혀있어야 한다"고 말함. 하지만 미래의 확장을 위해 미리 추상화하면 YAGNI 위반.

**해결:** 현재 필요한 만큼만 OCP를 적용하라.

```python
# 첫 번째 구현: 할인 타입이 하나일 때 → OCP 불필요
def calculate_discount(order):
    if order.is_first_purchase:
        return order.total * 0.1
    return 0

# 두 번째 할인 타입 추가: 아직은 if-else로 충분
def calculate_discount(order):
    if order.is_first_purchase:
        return order.total * 0.1
    elif order.coupon_code:
        return order.total * 0.05
    return 0

# 세 번째 할인 타입: 이제 전략 패턴이 의미 있다 (Rule of Three!)
class DiscountStrategy(Protocol):
    def calculate(self, order) -> float: ...

class FirstPurchaseDiscount:
    def calculate(self, order) -> float:
        return order.total * 0.1 if order.is_first_purchase else 0

class CouponDiscount:
    def calculate(self, order) -> float:
        return order.total * 0.05 if order.coupon_code else 0

class VipDiscount:
    def calculate(self, order) -> float:
        return order.total * 0.15 if order.is_vip else 0
```

### YAGNI와 DIP (의존성 역전 원칙)

**조화:** 의존성 주입을 사용하면 나중에 구현을 교체하기 쉬우므로, 미리 복잡한 추상화 없이도 유연성을 확보할 수 있음.

```csharp
// ✅ 의존성 주입만 해두면, 인터페이스 없이도 나중에 교체 가능
public class OrderController
{
    // 지금은 구체 클래스를 직접 주입
    // 나중에 인터페이스가 필요하면 그때 추출
    private readonly EmailNotificationService _notificationService;

    public OrderController(EmailNotificationService notificationService)
    {
        _notificationService = notificationService;
    }
}
```

---

## 10. 실무 팁

### 10.1 YAGNI 의사결정 플로우차트

```
이 기능이 지금 당장 필요한가?
├── Yes → 구현하라
└── No
    ├── 요구사항이 확정되었는가?
    │   ├── Yes → 다음 스프린트에 구현
    │   └── No → 구현하지 마라 (YAGNI!)
    └── 구현하지 않으면 나중에 큰 비용이 드는가?
        ├── Yes → 아키텍처 수준의 결정이면 팀과 논의
        └── No → 구현하지 마라 (YAGNI!)
```

### 10.2 YAGNI를 위반하게 만드는 유혹들

| 유혹 | 현실 |
|------|------|
| "나중에 필요할 거야" | 80%는 필요하지 않음 |
| "지금 만들면 나중에 시간 절약" | 유지보수 비용이 더 큼 |
| "재미있을 것 같아" | 재미와 필요는 다름 |
| "이력서에 쓸 수 있어" | 팀에 피해를 줌 |
| "Best Practice래!" | 컨텍스트를 무시한 적용 |

### 10.3 YAGNI를 지키면서도 미래를 대비하는 방법

1. **깨끗한 코드를 유지하라**
   - 깨끗한 코드는 리팩토링하기 쉽다
   - 리팩토링이 쉬우면 나중에 기능을 추가하기도 쉽다

2. **테스트를 작성하라**
   - 테스트가 있으면 리팩토링할 때 안전망이 된다
   - 안전망이 있으면 "나중에 바꾸면 되지"라고 자신 있게 말할 수 있다

3. **의존성 주입을 사용하라**
   - 인터페이스 없이도 나중에 교체가 가능한 구조
   - 복잡한 추상화 없이 유연성 확보

4. **작은 커밋, 자주 배포**
   - 작은 변경은 리스크가 적다
   - 문제가 생기면 쉽게 되돌릴 수 있다

### 10.4 YAGNI 예외: 미리 고려해야 하는 것들

다음은 YAGNI 예외에 해당하며, 초기에 고려하는 것이 좋습니다:

- **보안**: 나중에 추가하기 매우 어려움
- **접근성(a11y)**: 나중에 추가하면 전면 수정 필요
- **국제화(i18n)**: 문자열 하드코딩을 나중에 바꾸기 어려움
- **로깅/모니터링**: 문제 진단을 위해 초기부터 필요
- **데이터 마이그레이션 전략**: 스키마 변경은 돌이키기 어려움

---

## 11. 정리 및 체크리스트

### 핵심 요약

| 개념 | 설명 |
|------|------|
| YAGNI | 지금 필요하지 않으면 만들지 마라 |
| 비용 | 미사용 코드도 유지보수 비용 발생 |
| 불확실성 | 미래 요구사항은 예측 불가 |
| 점진적 진화 | 필요할 때 추가, 패턴 보일 때 추상화 |
| 균형 | 단순하되 확장 가능한 구조 유지 |

### YAGNI 적용 체크리스트

- [ ] 이 기능은 현재 요구사항에 포함되어 있는가?
- [ ] "나중에 필요할 것 같아서"가 구현의 유일한 이유는 아닌가?
- [ ] 이 추상화 없이 현재 문제를 해결할 수 있는가?
- [ ] 이 설정 옵션을 실제로 누가 사용하는가?
- [ ] 지금 구현하지 않으면 나중에 구현이 불가능해지는가?
- [ ] Rule of Three를 적용했는가? (패턴이 3번 반복된 후 추상화)
- [ ] 깨끗한 코드와 테스트로 나중의 리팩토링이 안전한가?

---

## 12. 관련 원칙

| 원칙 | 관계 |
|------|------|
| [DRY](./01-dry.md) | Rule of Three를 통해 YAGNI와 균형 |
| [KISS](./02-kiss.md) | YAGNI의 자연스러운 동반자 - 단순하게 유지 |
| [Clean Code](./04-clean-code.md) | 깨끗한 코드는 나중에 리팩토링하기 쉬움 |
| OCP (개방-폐쇄 원칙) | 확장 포인트는 필요할 때 추가 |
| Lean | "낭비를 제거하라" - YAGNI의 Lean 버전 |
| Agile | 점진적 개발, 빠른 피드백 |

---

> **이전 문서:** [KISS 원칙 (Keep It Simple, Stupid)](./02-kiss.md)
> **다음 문서:** [Clean Code 원칙 (클린 코드)](./04-clean-code.md)
