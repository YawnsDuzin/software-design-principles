# Singleton 패턴 (싱글톤)

> **핵심 한줄 요약**: 클래스의 인스턴스가 오직 하나만 존재하도록 보장하고, 그 인스턴스에 대한 전역 접근점을 제공한다.

---

## 목차

1. [의도 (Intent)](#1-의도-intent)
2. [문제 상황 (Problem)](#2-문제-상황-problem)
3. [UML 다이어그램](#3-uml-다이어그램)
4. [Python 구현](#4-python-구현)
5. [C# 구현](#5-c-구현)
6. [장단점](#6-장단점)
7. [주의사항](#7-주의사항)
8. [실무 팁](#8-실무-팁)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 의도 (Intent)

Singleton 패턴은 두 가지 문제를 동시에 해결한다:

1. **인스턴스가 하나만 존재**하도록 보장한다
2. 그 인스턴스에 대한 **전역 접근점**을 제공한다

일반적인 생성자 호출(`new`)로는 항상 새 객체가 만들어지기 때문에, 생성자를 외부에서 호출하지 못하도록 제한하고, 인스턴스를 가져오는 별도의 정적 메서드를 제공하는 방식으로 구현한다.

---

## 2. 문제 상황 (Problem)

### 왜 필요한가?

다음과 같은 상황에서 Singleton이 유용하다:

| 상황 | 설명 |
|------|------|
| **설정 관리 (Configuration)** | 앱 전체에서 하나의 설정 객체를 공유해야 할 때 |
| **로깅 (Logger)** | 모든 모듈이 동일한 로거 인스턴스를 사용해야 할 때 |
| **DB 연결 풀 (Connection Pool)** | 연결 풀이 여러 개 생성되면 리소스가 낭비될 때 |
| **캐시 (Cache)** | 앱 전체에서 하나의 캐시를 공유해야 할 때 |
| **스레드 풀 (Thread Pool)** | 작업 큐를 관리하는 풀이 하나여야 할 때 |

### 실생활 비유

```
회사에는 CEO가 한 명만 있다.
누가 "CEO 좀 불러주세요"라고 해도, 항상 같은 사람이 온다.
CEO가 두 명이면 지시가 충돌하고 혼란이 생긴다.

→ Singleton은 "CEO 역할"을 하는 클래스다.
```

---

## 3. UML 다이어그램

```
┌────────────────────────────┐
│        Singleton           │
├────────────────────────────┤
│ - instance: Singleton      │  ← 유일한 인스턴스 (static)
│ - data: string             │
├────────────────────────────┤
│ - Singleton()              │  ← 생성자를 private으로 숨김
│ + getInstance(): Singleton │  ← 유일한 접근점 (static)
│ + someBusinessLogic()      │
└────────────────────────────┘

클라이언트 코드:
  obj1 = Singleton.getInstance()
  obj2 = Singleton.getInstance()
  assert obj1 is obj2  → True (같은 인스턴스)
```

---

## 4. Python 구현

### 방식 1: `__new__` 메서드 오버라이드 (가장 기본적)

```python
class Singleton:
    """__new__를 오버라이드하여 싱글톤을 구현하는 가장 기본적인 방식"""

    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        # 주의: __init__은 getInstance()할 때마다 호출된다!
        # 따라서 초기화 로직이 반복 실행될 수 있다.
        self.value = None


# 사용 예시
s1 = Singleton()
s2 = Singleton()

s1.value = "hello"
print(s2.value)  # "hello" - 같은 인스턴스이므로
print(s1 is s2)  # True
```

### 방식 2: 메타클래스 방식 (우아하고 확장 가능)

```python
class SingletonMeta(type):
    """메타클래스를 활용한 싱글톤 구현

    메타클래스는 '클래스의 클래스'이다.
    클래스가 인스턴스를 만드는 것처럼, 메타클래스는 클래스를 만든다.
    """

    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]


class AppConfig(metaclass=SingletonMeta):
    """앱 설정 관리자 - 싱글톤"""

    def __init__(self):
        self.settings = {}

    def set(self, key, value):
        self.settings[key] = value

    def get(self, key, default=None):
        return self.settings.get(key, default)


# 사용 예시
config1 = AppConfig()
config1.set("debug", True)
config1.set("db_host", "localhost")

config2 = AppConfig()
print(config2.get("debug"))    # True - 같은 인스턴스
print(config1 is config2)      # True
```

### 방식 3: 모듈 방식 (Python에서 가장 Pythonic한 방법)

```python
# config.py - 모듈 자체가 싱글톤 역할을 한다
# Python 모듈은 최초 import 시 한 번만 실행되고, 이후에는 캐시된 객체를 반환한다.

class _AppConfig:
    """모듈 레벨 싱글톤 (언더스코어로 외부 직접 접근 방지)"""

    def __init__(self):
        self.settings = {}
        self._load_defaults()

    def _load_defaults(self):
        self.settings = {
            "debug": False,
            "log_level": "INFO",
            "max_connections": 10,
        }

    def get(self, key, default=None):
        return self.settings.get(key, default)

    def set(self, key, value):
        self.settings[key] = value


# 모듈 레벨에서 인스턴스 생성 (이것이 싱글톤이다)
config = _AppConfig()


# --- 다른 파일에서 사용 ---
# from config import config
# config.get("debug")  → 항상 같은 인스턴스
```

### 방식 4: 스레드 안전 싱글톤 (멀티스레드 환경)

```python
import threading


class ThreadSafeSingleton:
    """스레드 안전한 싱글톤 구현"""

    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        # Double-Checked Locking Pattern
        if cls._instance is None:             # 1차 체크 (락 없이)
            with cls._lock:                    # 락 획득
                if cls._instance is None:      # 2차 체크 (락 안에서)
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if not hasattr(self, '_initialized'):
            self._initialized = True
            self.data = {}


# 멀티스레드 테스트
def create_singleton(results, index):
    singleton = ThreadSafeSingleton()
    results[index] = id(singleton)


results = [None] * 10
threads = []

for i in range(10):
    t = threading.Thread(target=create_singleton, args=(results, i))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

# 모든 스레드에서 같은 인스턴스인지 확인
print(f"모두 같은 인스턴스: {len(set(results)) == 1}")  # True
```

### 실무 예제: 로거 싱글톤

```python
import logging
from datetime import datetime


class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class AppLogger(metaclass=SingletonMeta):
    """앱 전체에서 사용하는 로거 싱글톤"""

    def __init__(self, log_file="app.log"):
        self.logger = logging.getLogger("AppLogger")
        self.logger.setLevel(logging.DEBUG)

        # 파일 핸들러
        file_handler = logging.FileHandler(log_file)
        file_handler.setLevel(logging.DEBUG)

        # 콘솔 핸들러
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)

        # 포맷터
        formatter = logging.Formatter(
            '%(asctime)s [%(levelname)s] %(message)s'
        )
        file_handler.setFormatter(formatter)
        console_handler.setFormatter(formatter)

        self.logger.addHandler(file_handler)
        self.logger.addHandler(console_handler)

    def info(self, message):
        self.logger.info(message)

    def error(self, message):
        self.logger.error(message)

    def debug(self, message):
        self.logger.debug(message)


# --- 여러 모듈에서 사용 ---
# user_service.py
class UserService:
    def __init__(self):
        self.logger = AppLogger()

    def create_user(self, name):
        self.logger.info(f"사용자 생성: {name}")
        # ... 사용자 생성 로직
        return {"name": name}


# order_service.py
class OrderService:
    def __init__(self):
        self.logger = AppLogger()

    def create_order(self, user_id, item):
        self.logger.info(f"주문 생성: user={user_id}, item={item}")
        # ... 주문 생성 로직
        return {"user_id": user_id, "item": item}


# main.py
user_svc = UserService()
order_svc = OrderService()

# 두 서비스가 같은 로거를 사용하고 있음을 확인
print(user_svc.logger is order_svc.logger)  # True
```

---

## 5. C# 구현

### 방식 1: 기본적인 Singleton (Thread-Unsafe)

```csharp
/// <summary>
/// 가장 기본적인 싱글톤 - 스레드 안전하지 않음 (교육용)
/// </summary>
public class BasicSingleton
{
    private static BasicSingleton _instance;

    // private 생성자 - 외부에서 new 불가
    private BasicSingleton()
    {
        Console.WriteLine("싱글톤 인스턴스 생성됨");
    }

    public static BasicSingleton Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new BasicSingleton();
            }
            return _instance;
        }
    }

    public void DoSomething()
    {
        Console.WriteLine("비즈니스 로직 실행");
    }
}

// 사용
var s1 = BasicSingleton.Instance;
var s2 = BasicSingleton.Instance;
Console.WriteLine(ReferenceEquals(s1, s2)); // True
```

### 방식 2: lock을 이용한 Thread-Safe Singleton

```csharp
/// <summary>
/// lock을 사용한 스레드 안전 싱글톤
/// Double-Checked Locking 패턴 적용
/// </summary>
public class ThreadSafeSingleton
{
    private static ThreadSafeSingleton _instance;
    private static readonly object _lock = new object();

    private ThreadSafeSingleton() { }

    public static ThreadSafeSingleton Instance
    {
        get
        {
            // 1차 검사: 인스턴스가 이미 있으면 lock을 건너뜀 (성능)
            if (_instance == null)
            {
                lock (_lock) // 한 번에 하나의 스레드만 진입
                {
                    // 2차 검사: lock 획득 후 다시 확인
                    if (_instance == null)
                    {
                        _instance = new ThreadSafeSingleton();
                    }
                }
            }
            return _instance;
        }
    }
}
```

### 방식 3: Lazy<T> 사용 (권장 - 가장 깔끔)

```csharp
/// <summary>
/// Lazy<T>를 활용한 싱글톤 - .NET 4.0 이상 권장 방식
/// 스레드 안전 + 지연 초기화 + 간결한 코드
/// </summary>
public class LazySingleton
{
    // Lazy<T>는 기본적으로 스레드 안전하다
    private static readonly Lazy<LazySingleton> _lazy =
        new Lazy<LazySingleton>(() => new LazySingleton());

    private LazySingleton()
    {
        Console.WriteLine("LazySingleton 인스턴스 생성됨");
    }

    public static LazySingleton Instance => _lazy.Value;

    public string GetStatus() => "싱글톤 활성화됨";
}

// 사용
Console.WriteLine(LazySingleton.Instance.GetStatus());
```

### 방식 4: static readonly 방식 (Eager Initialization)

```csharp
/// <summary>
/// static readonly를 활용한 즉시 초기화 싱글톤
/// CLR이 스레드 안전성을 보장한다
/// 단점: 사용하지 않아도 인스턴스가 생성됨
/// </summary>
public class EagerSingleton
{
    // CLR이 static 필드 초기화 시 스레드 안전성을 보장
    private static readonly EagerSingleton _instance = new EagerSingleton();

    // static 생성자가 있으면 beforefieldinit 최적화 방지
    static EagerSingleton() { }

    private EagerSingleton()
    {
        Console.WriteLine("EagerSingleton 인스턴스 생성됨");
    }

    public static EagerSingleton Instance => _instance;
}
```

### 실무 예제: 앱 설정 관리자

```csharp
using System;
using System.Collections.Concurrent;
using System.IO;
using System.Text.Json;

/// <summary>
/// 앱 설정 관리 싱글톤 - 실무 예제
/// </summary>
public sealed class AppConfiguration
{
    private static readonly Lazy<AppConfiguration> _lazy =
        new Lazy<AppConfiguration>(() => new AppConfiguration());

    private readonly ConcurrentDictionary<string, string> _settings;

    private AppConfiguration()
    {
        _settings = new ConcurrentDictionary<string, string>();
        LoadDefaults();
    }

    public static AppConfiguration Instance => _lazy.Value;

    private void LoadDefaults()
    {
        _settings["Environment"] = "Development";
        _settings["LogLevel"] = "Information";
        _settings["MaxRetries"] = "3";
        _settings["ConnectionTimeout"] = "30";
    }

    public void LoadFromFile(string filePath)
    {
        if (!File.Exists(filePath))
            throw new FileNotFoundException($"설정 파일을 찾을 수 없습니다: {filePath}");

        var json = File.ReadAllText(filePath);
        var settings = JsonSerializer.Deserialize<Dictionary<string, string>>(json);

        foreach (var kvp in settings)
        {
            _settings[kvp.Key] = kvp.Value;
        }
    }

    public string Get(string key, string defaultValue = "")
    {
        return _settings.TryGetValue(key, out var value) ? value : defaultValue;
    }

    public void Set(string key, string value)
    {
        _settings[key] = value;
    }

    public int GetInt(string key, int defaultValue = 0)
    {
        var value = Get(key);
        return int.TryParse(value, out var result) ? result : defaultValue;
    }

    public override string ToString()
    {
        return string.Join(Environment.NewLine,
            _settings.Select(kvp => $"  {kvp.Key}: {kvp.Value}"));
    }
}

// --- 사용 예시 ---
public class DatabaseService
{
    private readonly int _timeout;
    private readonly int _maxRetries;

    public DatabaseService()
    {
        var config = AppConfiguration.Instance;
        _timeout = config.GetInt("ConnectionTimeout", 30);
        _maxRetries = config.GetInt("MaxRetries", 3);
    }

    public void Connect()
    {
        Console.WriteLine($"DB 연결 (timeout: {_timeout}s, retries: {_maxRetries})");
    }
}

public class EmailService
{
    public void SendEmail(string to, string subject)
    {
        var config = AppConfiguration.Instance;
        var env = config.Get("Environment");

        if (env == "Development")
        {
            Console.WriteLine($"[DEV] 이메일 전송 시뮬레이션: {to} - {subject}");
        }
        else
        {
            Console.WriteLine($"이메일 전송: {to} - {subject}");
        }
    }
}

// Program.cs
var config = AppConfiguration.Instance;
config.Set("Environment", "Production");

var db = new DatabaseService();
db.Connect();

var email = new EmailService();
email.SendEmail("user@example.com", "안녕하세요");
```

---

## 6. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **인스턴스 유일성 보장** | 클래스의 인스턴스가 하나만 존재함을 보장 |
| **전역 접근** | 어디서든 동일한 인스턴스에 접근 가능 |
| **지연 초기화** | 필요할 때까지 인스턴스 생성을 미룰 수 있음 |
| **리소스 절약** | 하나의 인스턴스만 사용하므로 메모리/연결 절약 |

### 단점

| 단점 | 설명 |
|------|------|
| **전역 상태** | 사실상 전역 변수와 같아서, 코드 간 숨은 의존성을 만듦 |
| **테스트 어려움** | 싱글톤 인스턴스를 교체(mock)하기 어려움 |
| **SRP 위반** | "인스턴스 관리"와 "비즈니스 로직"이라는 두 가지 책임 |
| **멀티스레드 주의** | 스레드 안전성을 직접 구현해야 함 |
| **상속 어려움** | private 생성자로 인해 상속/확장이 어려움 |

---

## 7. 주의사항

### 7.1 테스트가 어려운 이유

```python
# 문제: 싱글톤에 직접 의존하면 테스트에서 교체가 어렵다
class OrderService:
    def calculate_price(self, item):
        # 싱글톤에 직접 접근 - 테스트 시 다른 설정을 넣기 어렵다
        config = AppConfig()
        tax_rate = config.get("tax_rate", 0.1)
        return item.price * (1 + tax_rate)


# 해결: 의존성 주입 (DI) 사용
class OrderService:
    def __init__(self, config):
        # 외부에서 주입받으므로, 테스트 시 Mock 객체를 넣을 수 있다
        self.config = config

    def calculate_price(self, item):
        tax_rate = self.config.get("tax_rate", 0.1)
        return item.price * (1 + tax_rate)


# 테스트 코드
class MockConfig:
    def get(self, key, default=None):
        return {"tax_rate": 0.05}.get(key, default)

service = OrderService(MockConfig())
# → 테스트 가능!
```

### 7.2 전역 상태 문제

```csharp
// 문제: 싱글톤의 상태가 어디서 변경되었는지 추적이 어렵다

// 모듈 A에서
AppConfiguration.Instance.Set("Mode", "Batch");

// 모듈 B에서 (모듈 A의 변경을 모르고 있음)
var mode = AppConfiguration.Instance.Get("Mode");
// mode = "Batch"가 되어 예상치 못한 동작 발생!
```

### 7.3 멀티스레드 경합 (Race Condition)

```
스레드 A                      스레드 B
    │                            │
    ▼                            ▼
instance == null? → YES      instance == null? → YES
    │                            │
    ▼                            ▼
new Singleton()              new Singleton()  ← 두 번 생성됨!
    │                            │
    ▼                            ▼
instance = 객체1             instance = 객체2  ← 하나가 유실됨!

→ 해결: lock(C#) 또는 threading.Lock(Python) 사용
```

---

## 8. 실무 팁

### Tip 1: 싱글톤을 남용하지 마라

```
[싱글톤을 써야 하는가? 판단 기준]

Q1. 인스턴스가 정말 "반드시" 하나여야 하는가?
    → "편해서"가 아니라 "논리적으로 하나여야" 하는지 판단

Q2. DI(의존성 주입) 컨테이너로 대체할 수 있는가?
    → 대부분의 DI 프레임워크는 "Scoped" 또는 "Singleton" 수명 관리를 지원

Q3. 전역 접근이 반드시 필요한가?
    → 그냥 인자로 전달하면 안 되는가?
```

### Tip 2: DI 프레임워크와 함께 사용하기

```csharp
// ASP.NET Core에서는 DI 컨테이너가 싱글톤 수명을 관리한다
// 직접 싱글톤을 구현할 필요가 없다!

// Startup.cs 또는 Program.cs
builder.Services.AddSingleton<IAppConfiguration, AppConfiguration>();
builder.Services.AddSingleton<ILogger, FileLogger>();

// 컨트롤러에서 주입받아 사용
public class OrderController : ControllerBase
{
    private readonly IAppConfiguration _config;

    public OrderController(IAppConfiguration config)
    {
        _config = config; // DI 컨테이너가 싱글톤을 주입
    }
}
```

```python
# Python에서도 DI 라이브러리(dependency-injector 등)를 활용할 수 있다
from dependency_injector import containers, providers

class Container(containers.DeclarativeContainer):
    config = providers.Singleton(AppConfig)
    logger = providers.Singleton(AppLogger)
```

### Tip 3: 싱글톤보다 나은 대안들

| 상황 | 싱글톤 대신 | 이유 |
|------|-----------|------|
| 설정 관리 | DI + 환경 변수 | 테스트 용이, 환경별 분리 가능 |
| 로깅 | DI + 로깅 프레임워크 | 이미 프레임워크가 관리함 |
| DB 연결 풀 | DI + 커넥션 풀 라이브러리 | 라이브러리가 풀을 관리함 |
| 캐시 | DI + 캐시 라이브러리 | Redis 등 외부 캐시 활용 |

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Factory Method** | 팩토리 메서드를 싱글톤으로 구현하는 경우가 많다 |
| **Abstract Factory** | 추상 팩토리를 싱글톤으로 구현할 수 있다 |
| **Builder** | 빌더 객체를 싱글톤으로 구현하기도 한다 |
| **Facade** | 파사드 클래스를 싱글톤으로 구현하는 경우가 있다 |
| **Flyweight** | 플라이웨이트 팩토리가 싱글톤인 경우가 많다 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

```
Singleton 패턴 = 인스턴스 하나 보장 + 전역 접근점 제공

[Python]
- 가장 Pythonic한 방법: 모듈 레벨 인스턴스
- 메타클래스 방식: 재사용 가능하고 우아함
- 멀티스레드: threading.Lock + Double-Checked Locking

[C#]
- 권장: Lazy<T> 방식 (간결 + 스레드 안전)
- 대안: static readonly (즉시 초기화)
- 실무: DI 컨테이너의 AddSingleton 활용

[주의]
- 전역 상태는 숨은 의존성을 만든다
- 테스트가 어려워진다 → DI로 해결
- 남용 금지: "편하니까"가 아니라 "필요하니까" 사용
```

### 학습 체크리스트

- [ ] Singleton 패턴의 의도를 한 문장으로 설명할 수 있다
- [ ] Python에서 3가지 이상의 싱글톤 구현 방식을 알고 있다
- [ ] C#에서 Lazy<T>를 활용한 싱글톤을 구현할 수 있다
- [ ] 스레드 안전한 싱글톤과 안전하지 않은 싱글톤의 차이를 설명할 수 있다
- [ ] Double-Checked Locking이 왜 필요한지 설명할 수 있다
- [ ] 싱글톤의 단점(테스트 어려움, 전역 상태)을 이해하고 대안을 제시할 수 있다
- [ ] DI 프레임워크에서 싱글톤 수명을 관리하는 방법을 알고 있다
- [ ] 싱글톤을 사용해야 할 때와 사용하지 말아야 할 때를 구분할 수 있다

### 다음 학습

싱글톤은 가장 단순한 생성 패턴이다. 다음으로는 객체 생성의 유연성을 극대화하는 [Factory Method 패턴](./02-factory-method.md)을 학습하자.
