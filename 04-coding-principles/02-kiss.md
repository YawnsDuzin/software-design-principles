# KISS 원칙 (Keep It Simple, Stupid)

> "단순함이 최고의 정교함이다." (Simplicity is the ultimate sophistication.)
> — Leonardo da Vinci

> "어떤 바보라도 컴퓨터가 이해할 수 있는 코드를 작성할 수 있다. 좋은 프로그래머는 사람이 이해할 수 있는 코드를 작성한다."
> — Martin Fowler

---

## 목차

1. [KISS란 무엇인가?](#1-kiss란-무엇인가)
2. [왜 단순해야 하는가?](#2-왜-단순해야-하는가)
3. [복잡도의 종류](#3-복잡도의-종류)
4. [나쁜 예제 (Before)](#4-나쁜-예제-before)
5. [좋은 예제 (After)](#5-좋은-예제-after)
6. [Python 코드 예제](#6-python-코드-예제)
7. [C# 코드 예제](#7-c-코드-예제)
8. [단순함과 쉬움의 차이](#8-단순함과-쉬움의-차이)
9. [실무 팁](#9-실무-팁)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)
11. [관련 원칙](#11-관련-원칙)

---

## 1. KISS란 무엇인가?

KISS(Keep It Simple, Stupid)는 미국 해군에서 유래한 설계 원칙으로, 1960년대 록히드 스컹크 웍스의 수석 엔지니어 Kelly Johnson이 제안했습니다. 핵심 메시지는 **"불필요한 복잡성을 피하라"**입니다.

### 핵심 정의

**시스템은 복잡하게 만들기보다 단순하게 만들 때 가장 잘 작동한다.**

코드에 적용하면: **가장 단순한 방법으로 문제를 해결하라.** 더 복잡한 방법이 있더라도, 단순한 방법이 충분히 동작한다면 단순한 방법을 선택하라.

### 비유로 이해하기

스위스 군용 칼을 생각해 보세요. 칼, 가위, 드라이버, 병따개... 온갖 도구가 들어있지만, 실제로 칼이 필요할 때 사용하기가 불편합니다. 반면 잘 만들어진 요리용 칼은 한 가지를 정말 잘 합니다. 좋은 코드도 마찬가지입니다. **한 가지를 명확하게 잘 하는 코드**가 좋은 코드입니다.

---

## 2. 왜 단순해야 하는가?

### 2.1 유지보수성

코드는 **작성되는 시간보다 읽히는 시간이 10배** 이상 많습니다. 복잡한 코드는 읽는 데 오래 걸리고, 수정할 때 실수를 유발합니다.

```
작성 시간:  ██ (1)
읽기 시간:  ████████████████████ (10+)
디버깅 시간: ████████████████████████████████████████ (20+)
```

### 2.2 가독성

6개월 후의 나는 오늘의 나와 다른 사람입니다. "내가 이 코드를 왜 이렇게 짰지?"라는 말이 나오지 않아야 합니다.

### 2.3 디버깅

> "디버깅은 코드를 작성하는 것보다 두 배 어렵다. 따라서 가능한 한 가장 영리하게 코드를 작성한다면, 정의상 그것을 디버깅할 만큼 똑똑하지 않다."
> — Brian Kernighan

### 2.4 협업

팀에서 일할 때, 다른 사람이 이해할 수 없는 코드는 병목이 됩니다. "천재적인 코드"보다 "누구나 이해할 수 있는 코드"가 팀에 더 가치 있습니다.

---

## 3. 복잡도의 종류

### 3.1 본질적 복잡도 (Essential Complexity)

문제 자체에서 오는 복잡도로, **제거할 수 없습니다.**

예시:
- 세금 계산 규칙이 복잡하다 (법률이 복잡하니까)
- 글로벌 서비스의 타임존 처리 (시간대가 복잡하니까)
- 동시성 제어 (동시 접근은 본질적으로 어렵다)

### 3.2 우발적 복잡도 (Accidental Complexity)

해결 방법에서 오는 복잡도로, **줄이거나 제거할 수 있습니다.**

예시:
- 과도한 디자인 패턴 적용
- 불필요하게 복잡한 아키텍처
- "혹시 모를" 확장성을 위한 추상화
- 한 줄에 모든 것을 담으려는 시도

```
KISS의 목표: 본질적 복잡도는 받아들이되, 우발적 복잡도를 최소화하는 것
```

### 비유

요리를 생각해 보세요.
- **본질적 복잡도**: 스테이크를 굽기 위해 고기의 온도를 관리해야 하는 것
- **우발적 복잡도**: 간단한 스테이크에 불필요한 소스 10가지를 만드는 것

---

## 4. 나쁜 예제 (Before)

### 4.1 과도한 추상화

```python
# ❌ Bad: 간단한 문자열 처리에 과도한 추상화
from abc import ABC, abstractmethod

class StringProcessor(ABC):
    @abstractmethod
    def process(self, text: str) -> str:
        pass

class UpperCaseProcessor(StringProcessor):
    def process(self, text: str) -> str:
        return text.upper()

class TrimProcessor(StringProcessor):
    def process(self, text: str) -> str:
        return text.strip()

class ProcessorChain:
    def __init__(self):
        self._processors: list[StringProcessor] = []

    def add_processor(self, processor: StringProcessor):
        self._processors.append(processor)
        return self

    def execute(self, text: str) -> str:
        result = text
        for processor in self._processors:
            result = processor.process(result)
        return result

# 사용
chain = ProcessorChain()
chain.add_processor(TrimProcessor())
chain.add_processor(UpperCaseProcessor())
result = chain.execute("  hello world  ")
```

```python
# ✅ Good: 간단한 작업은 간단하게
result = "  hello world  ".strip().upper()
```

### 4.2 불필요한 디자인 패턴 적용

```csharp
// ❌ Bad: 단순한 설정 읽기에 빌더 패턴 + 팩토리 패턴
public class DatabaseConfig
{
    public string Host { get; set; }
    public int Port { get; set; }
    public string Database { get; set; }
}

public class DatabaseConfigBuilder
{
    private readonly DatabaseConfig _config = new();

    public DatabaseConfigBuilder WithHost(string host)
    {
        _config.Host = host;
        return this;
    }

    public DatabaseConfigBuilder WithPort(int port)
    {
        _config.Port = port;
        return this;
    }

    public DatabaseConfigBuilder WithDatabase(string database)
    {
        _config.Database = database;
        return this;
    }

    public DatabaseConfig Build() => _config;
}

public class DatabaseConfigFactory
{
    public static DatabaseConfigBuilder CreateBuilder()
    {
        return new DatabaseConfigBuilder();
    }
}

// 사용: 7줄이나 필요
var config = DatabaseConfigFactory
    .CreateBuilder()
    .WithHost("localhost")
    .WithPort(5432)
    .WithDatabase("mydb")
    .Build();
```

```csharp
// ✅ Good: 오브젝트 이니셜라이저로 충분
var config = new DatabaseConfig
{
    Host = "localhost",
    Port = 5432,
    Database = "mydb"
};
```

### 4.3 한 줄에 모든 걸 담으려는 코드

```python
# ❌ Bad: "파이썬스럽게" 보이려는 과도한 한 줄 코드
result = {k: v for k, v in sorted(
    {name: sum(score for _, score in grades) / len(grades)
     for name, grades in itertools.groupby(
         sorted(data, key=lambda x: x[0]),
         key=lambda x: x[0])
    }.items(),
    key=lambda x: x[1],
    reverse=True
) if v >= 60}
```

```python
# ✅ Good: 의도가 명확한 단계별 코드
# 1. 학생별로 그룹화
students = defaultdict(list)
for name, score in data:
    students[name].append(score)

# 2. 평균 계산
averages = {
    name: sum(scores) / len(scores)
    for name, scores in students.items()
}

# 3. 60점 이상만 필터링
passing = {
    name: avg for name, avg in averages.items()
    if avg >= 60
}

# 4. 점수순 정렬
result = dict(sorted(passing.items(), key=lambda x: x[1], reverse=True))
```

### 4.4 과도한 제네릭/메타프로그래밍

```csharp
// ❌ Bad: 과도한 제네릭 - 이 코드가 뭘 하는지 이해하려면 얼마나 걸릴까?
public class GenericProcessor<TInput, TOutput, TContext, TValidator>
    where TInput : IProcessable<TContext>
    where TOutput : IResult, new()
    where TContext : IContext<TInput>
    where TValidator : IValidator<TInput, TContext>
{
    private readonly TValidator _validator;
    private readonly Func<TInput, TContext, TOutput> _transformer;

    public GenericProcessor(
        TValidator validator,
        Func<TInput, TContext, TOutput> transformer)
    {
        _validator = validator;
        _transformer = transformer;
    }

    public TOutput Process(TInput input, TContext context)
    {
        if (!_validator.Validate(input, context))
            return new TOutput { Success = false };
        return _transformer(input, context);
    }
}
```

```csharp
// ✅ Good: 구체적이고 이해하기 쉬운 코드
public class OrderProcessor
{
    private readonly OrderValidator _validator;

    public OrderProcessor(OrderValidator validator)
    {
        _validator = validator;
    }

    public OrderResult Process(Order order)
    {
        var errors = _validator.Validate(order);
        if (errors.Any())
            return OrderResult.Failure(errors);

        // 주문 처리 로직...
        return OrderResult.Success(order.Id);
    }
}
```

---

## 5. 좋은 예제 (After)

### 5.1 읽기 쉬운 코드

```python
# ✅ Good: 의도를 명확하게 표현하는 코드
def get_eligible_voters(citizens: list[Citizen]) -> list[Citizen]:
    """투표 자격이 있는 시민 목록을 반환합니다."""
    voting_age = 18
    return [
        citizen for citizen in citizens
        if citizen.age >= voting_age and citizen.is_registered
    ]
```

### 5.2 적절한 수준의 추상화

```csharp
// ✅ Good: 필요한 만큼만 추상화
public interface IEmailSender
{
    Task SendAsync(string to, string subject, string body);
}

// 구현이 하나라면 인터페이스도 하나면 충분
public class SmtpEmailSender : IEmailSender
{
    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_host, _port);
        var message = new MailMessage(_from, to, subject, body);
        await client.SendMailAsync(message);
    }
}
```

### 5.3 명확한 변수/함수 이름

```python
# ❌ Bad: 축약된 이름
def calc(d, r, t):
    return d * (1 + r) ** t

# ✅ Good: 의미가 드러나는 이름
def calculate_compound_interest(
    principal: float,
    annual_rate: float,
    years: int
) -> float:
    """복리 이자를 계산합니다."""
    return principal * (1 + annual_rate) ** years
```

---

## 6. Python 코드 예제

### 6.1 조건문 단순화

```python
# ❌ Bad: 복잡한 중첩 조건문
def get_shipping_cost(order):
    if order.country == "KR":
        if order.is_premium_member:
            if order.total >= 50000:
                return 0
            else:
                return 2500
        else:
            if order.total >= 50000:
                return 0
            else:
                if order.total >= 30000:
                    return 2500
                else:
                    return 3500
    else:
        if order.is_premium_member:
            return 10000
        else:
            return 20000
```

```python
# ✅ Good: Early Return + 가드 절로 단순화
def get_shipping_cost(order) -> int:
    """배송비를 계산합니다."""
    # 해외 배송
    if order.country != "KR":
        return 10000 if order.is_premium_member else 20000

    # 국내 - 무료 배송 기준
    if order.total >= 50000:
        return 0

    # 국내 - 프리미엄 회원
    if order.is_premium_member:
        return 2500

    # 국내 - 일반 회원
    return 2500 if order.total >= 30000 else 3500
```

### 6.2 클래스 설계의 단순화

```python
# ❌ Bad: 과도하게 설계된 로거
class LogLevel:
    DEBUG = 0
    INFO = 1
    WARNING = 2
    ERROR = 3

class LogFormatter(ABC):
    @abstractmethod
    def format(self, level, message): pass

class JsonFormatter(LogFormatter):
    def format(self, level, message):
        return json.dumps({"level": level, "message": message})

class TextFormatter(LogFormatter):
    def format(self, level, message):
        return f"[{level}] {message}"

class LogDestination(ABC):
    @abstractmethod
    def write(self, formatted_message): pass

class ConsoleDestination(LogDestination):
    def write(self, formatted_message):
        print(formatted_message)

class FileDestination(LogDestination):
    def __init__(self, path):
        self.path = path
    def write(self, formatted_message):
        with open(self.path, "a") as f:
            f.write(formatted_message + "\n")

class Logger:
    def __init__(self, formatter: LogFormatter, destination: LogDestination):
        self.formatter = formatter
        self.destination = destination
        self.min_level = LogLevel.INFO

    def log(self, level: int, message: str):
        if level >= self.min_level:
            formatted = self.formatter.format(level, message)
            self.destination.write(formatted)

# 사용하려면 이 모든 것을 알아야 함
logger = Logger(TextFormatter(), ConsoleDestination())
logger.log(LogLevel.INFO, "시작합니다")
```

```python
# ✅ Good: 필요한 것만 있는 단순한 로거
# (프로젝트 초기에는 이것으로 충분하다!)
import logging

# Python 내장 logging 모듈 활용
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# 사용
logger.info("시작합니다")
logger.error("오류가 발생했습니다: %s", error)
```

### 6.3 데이터 처리의 단순화

```python
# ❌ Bad: 불필요하게 복잡한 데이터 변환
class DataTransformPipeline:
    def __init__(self):
        self.steps = []

    def add_step(self, func):
        self.steps.append(func)
        return self

    def execute(self, data):
        for step in self.steps:
            data = step(data)
        return data

pipeline = DataTransformPipeline()
pipeline.add_step(lambda users: [u for u in users if u["active"]])
pipeline.add_step(lambda users: [
    {"name": u["name"], "email": u["email"]} for u in users
])
pipeline.add_step(lambda users: sorted(users, key=lambda u: u["name"]))
result = pipeline.execute(raw_users)
```

```python
# ✅ Good: 직관적인 순차 처리
def get_active_user_contacts(users: list[dict]) -> list[dict]:
    """활성 사용자의 연락처 정보를 이름순으로 반환합니다."""
    active_users = [u for u in users if u["active"]]

    contacts = [
        {"name": u["name"], "email": u["email"]}
        for u in active_users
    ]

    return sorted(contacts, key=lambda u: u["name"])
```

---

## 7. C# 코드 예제

### 7.1 LINQ의 적절한 사용

```csharp
// ❌ Bad: LINQ 체인이 너무 길어 의도 파악이 어려움
var result = orders
    .Where(o => o.Status != OrderStatus.Cancelled)
    .Where(o => o.CreatedAt >= DateTime.Now.AddMonths(-6))
    .GroupBy(o => o.CustomerId)
    .Select(g => new {
        CustomerId = g.Key,
        TotalAmount = g.Sum(o => o.Items.Sum(i => i.Price * i.Quantity)),
        OrderCount = g.Count(),
        AverageAmount = g.Average(o => o.Items.Sum(i => i.Price * i.Quantity)),
        LastOrderDate = g.Max(o => o.CreatedAt)
    })
    .Where(c => c.TotalAmount > 1000000)
    .OrderByDescending(c => c.TotalAmount)
    .Take(10)
    .ToList();
```

```csharp
// ✅ Good: 의미 있는 단위로 분리
var sixMonthsAgo = DateTime.Now.AddMonths(-6);

// 1. 유효한 주문 필터링
var validOrders = orders
    .Where(o => o.Status != OrderStatus.Cancelled)
    .Where(o => o.CreatedAt >= sixMonthsAgo);

// 2. 고객별 통계 계산
var customerStats = validOrders
    .GroupBy(o => o.CustomerId)
    .Select(g => new CustomerOrderSummary
    {
        CustomerId = g.Key,
        TotalAmount = g.Sum(o => o.GetTotalPrice()),
        OrderCount = g.Count(),
        AverageAmount = g.Average(o => o.GetTotalPrice()),
        LastOrderDate = g.Max(o => o.CreatedAt)
    });

// 3. 상위 고객 추출
var topCustomers = customerStats
    .Where(c => c.TotalAmount > 1_000_000)
    .OrderByDescending(c => c.TotalAmount)
    .Take(10)
    .ToList();
```

### 7.2 예외 처리의 단순화

```csharp
// ❌ Bad: 과도한 try-catch 남용
public UserDto GetUser(int id)
{
    try
    {
        try
        {
            var user = _repository.FindById(id);
            try
            {
                var dto = _mapper.Map<UserDto>(user);
                try
                {
                    _cache.Set($"user:{id}", dto);
                }
                catch (CacheException ex)
                {
                    _logger.LogWarning(ex, "캐시 저장 실패");
                }
                return dto;
            }
            catch (MappingException ex)
            {
                _logger.LogError(ex, "매핑 실패");
                throw;
            }
        }
        catch (RepositoryException ex)
        {
            _logger.LogError(ex, "DB 조회 실패");
            throw new ServiceException("사용자 조회 실패", ex);
        }
    }
    catch (Exception ex) when (ex is not ServiceException)
    {
        _logger.LogError(ex, "알 수 없는 오류");
        throw new ServiceException("알 수 없는 오류", ex);
    }
}
```

```csharp
// ✅ Good: 깔끔한 예외 처리
public UserDto GetUser(int id)
{
    var user = _repository.FindById(id)
        ?? throw new NotFoundException($"User {id} not found");

    var dto = _mapper.Map<UserDto>(user);

    // 캐시는 실패해도 괜찮음 - 비핵심 작업
    TryCacheUser(id, dto);

    return dto;
}

private void TryCacheUser(int id, UserDto dto)
{
    try
    {
        _cache.Set($"user:{id}", dto);
    }
    catch (Exception ex)
    {
        _logger.LogWarning(ex, "캐시 저장 실패 - 무시합니다.");
    }
}
```

### 7.3 설정 패턴의 단순화

```csharp
// ❌ Bad: 과도한 설정 추상화
public interface IConfigProvider { }
public interface IConfigSection { }
public interface IConfigValidator { }
public class ConfigProviderFactory { }
public class ConfigSectionBuilder { }
public class ConfigValidationPipeline { }
// ... 설정 하나 읽으려고 6개의 클래스/인터페이스가 필요함

// ✅ Good: .NET의 Options 패턴 활용
public class SmtpSettings
{
    public string Host { get; set; } = "localhost";
    public int Port { get; set; } = 587;
    public string Username { get; set; } = "";
    public string Password { get; set; } = "";
}

// Startup.cs / Program.cs
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));

// 사용
public class EmailService
{
    private readonly SmtpSettings _settings;

    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }
}
```

---

## 8. 단순함과 쉬움의 차이

이 두 가지는 자주 혼동되지만, 매우 다른 개념입니다.

### 쉬움 (Easy)

- **친숙함**에 기반
- 내가 이미 아는 것이 쉬운 것
- 주관적이고 상대적
- 예: jQuery를 아는 사람에게 jQuery가 쉬움

### 단순함 (Simple)

- **복잡하지 않음**에 기반
- 하나의 역할, 하나의 개념, 하나의 목적
- 객관적이고 측정 가능
- 예: 순수 함수는 단순함 (입력 -> 출력만 있으므로)

```
                쉬움           어려움
    ┌───────────┬───────────┐
단순│  이상적!   │  학습 필요  │
    │  (목표)    │  하지만 OK │
    ├───────────┼───────────┤
복잡│  위험!     │  최악      │
    │ (나중에    │  (유지보수  │
    │  폭탄)    │  불가능)   │
    └───────────┴───────────┘
```

### 실무 사례

```python
# 쉽지만 복잡한 코드 (위험!)
# - 전역 변수를 여기저기서 수정 (작성은 쉬움, 추적은 어려움)
user_data = {}

def update_user():
    global user_data
    user_data["name"] = "Kim"  # 쉽지만, 어디서 변경되는지 추적 어려움

# 처음엔 어렵지만 단순한 코드 (좋음!)
# - 불변 데이터와 순수 함수 (학습 곡선이 있지만 추적이 쉬움)
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str

def update_user_name(user: User, new_name: str) -> User:
    """새 이름으로 새 User를 반환합니다 (원본 불변)."""
    return User(name=new_name, email=user.email)
```

**KISS는 "쉬운 코드를 작성하라"가 아니라 "단순한 코드를 작성하라"입니다.**

---

## 9. 실무 팁

### 9.1 코드 리뷰에서 KISS 적용하기

코드 리뷰 시 다음 질문을 던져보세요:

1. **"이것보다 더 단순한 방법이 있나?"**
2. **"이 추상화가 정말 지금 필요한가?"**
3. **"6개월 후에 이 코드를 이해할 수 있나?"**
4. **"주니어 개발자가 이 코드를 이해할 수 있나?"**

### 9.2 복잡도 감소 전략

| 전략 | 설명 | 예시 |
|------|------|------|
| Early Return | 중첩 줄이기 | `if (!valid) return;` |
| 가드 절 | 예외 케이스를 먼저 처리 | 유효성 검사를 상단에 |
| 메서드 추출 | 긴 메서드를 분리 | 100줄 -> 20줄 x 5 |
| 표준 라이브러리 활용 | 직접 만들지 않기 | `logging`, `json` 등 |
| 명확한 네이밍 | 주석 없이도 이해 | `get_active_users()` |

### 9.3 KISS를 위한 일상 습관

1. **먼저 동작하게 만들고, 그 다음에 개선하라**
   - "Make it work, make it right, make it fast" — Kent Beck

2. **설명이 필요한 코드는 복잡한 코드**
   - 코드를 설명하는 주석 대신, 설명이 필요 없는 코드를 작성하라

3. **"만약에..."를 경계하라**
   - "만약에 트래픽이 10배 늘면?" → 그때 대응하라 (YAGNI와 연결)

4. **팀의 수준에 맞춰라**
   - 팀원 모두가 이해할 수 있는 수준의 코드를 작성하라
   - 고급 기법은 팀이 함께 학습한 후에 적용하라

### 9.4 복잡도 냄새(Complexity Smells)

다음 신호가 보이면 KISS 위반을 의심하세요:

- 한 함수가 **20줄 이상**
- 들여쓰기가 **3단계 이상**
- 매개변수가 **4개 이상**
- 한 클래스에 **메서드가 15개 이상**
- 주석으로 **코드를 설명해야 하는** 경우
- 새로운 팀원이 **이해하는데 30분 이상** 걸리는 코드

---

## 10. 정리 및 체크리스트

### 핵심 요약

| 개념 | 설명 |
|------|------|
| KISS | 불필요한 복잡성을 피하라 |
| 본질적 복잡도 | 문제 자체의 복잡도 (제거 불가) |
| 우발적 복잡도 | 해결 방법의 복잡도 (제거 가능) |
| 단순함 vs 쉬움 | 단순함은 객관적, 쉬움은 주관적 |

### KISS 적용 체크리스트

- [ ] 이 코드보다 더 단순한 방법이 있는가?
- [ ] 표준 라이브러리나 프레임워크 기능으로 대체할 수 있는가?
- [ ] 불필요한 추상화 계층이 있지 않은가?
- [ ] 조건문의 중첩이 3단계를 넘지 않는가?
- [ ] 함수가 하나의 일만 하고 있는가?
- [ ] 변수/함수 이름만으로 의도를 파악할 수 있는가?
- [ ] 6개월 후의 내가 이 코드를 이해할 수 있는가?
- [ ] 주니어 개발자가 30분 내에 이해할 수 있는가?
- [ ] "혹시 모를" 상황을 위한 코드가 아닌가? (YAGNI)

---

## 11. 관련 원칙

| 원칙 | 관계 |
|------|------|
| [DRY](./01-dry.md) | 단순화 과정에서 중복이 발생할 수 있음 - 균형 필요 |
| [YAGNI](./03-yagni.md) | KISS의 자연스러운 동반자 - 필요한 것만 만들어라 |
| [Clean Code](./04-clean-code.md) | 클린 코드는 단순한 코드의 구체적 실천법 |
| SRP (단일 책임 원칙) | 한 가지 일만 하는 것이 단순함의 핵심 |
| Occam's Razor | "더 적은 가정을 하는 설명이 좋은 설명이다" |

---

> **이전 문서:** [DRY 원칙](./01-dry.md)
> **다음 문서:** [YAGNI 원칙 (You Ain't Gonna Need It)](./03-yagni.md)
