# Chain of Responsibility 패턴 (책임 연쇄)

> **핵심 의도 한줄 요약**: 요청을 처리할 수 있는 객체들의 체인을 구성하여, 요청을 보낸 쪽과 받는 쪽의 결합을 없애고 요청이 처리될 때까지 체인을 따라 전달한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: HTTP 미들웨어 체인](#7-실무-예제-http-미들웨어-체인)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 개요

Chain of Responsibility 패턴은 **요청을 처리할 수 있는 핸들러 객체들을 체인(사슬)으로 연결**하여, 요청이 처리될 때까지 체인을 따라 전달하는 행위 디자인 패턴이다.

요청을 보내는 클라이언트는 어떤 핸들러가 요청을 처리할지 알 필요가 없다. 각 핸들러는 요청을 직접 처리하거나, 다음 핸들러에게 넘긴다.

### 언제 사용하는가?

- 요청을 처리할 객체를 **런타임에 동적으로 결정**해야 할 때
- 여러 객체 중 하나가 요청을 처리해야 하지만, **어떤 객체인지 미리 알 수 없을 때**
- 요청의 처리 순서를 **유연하게 변경**하고 싶을 때

---

## 2. 문제 상황

### 문제: 로깅 레벨 처리

로깅 시스템을 개발한다고 가정하자. DEBUG, INFO, WARNING, ERROR 레벨의 로그가 있고, 각 레벨에 따라 다른 처리가 필요하다.

```python
# 나쁜 예: if/elif 분기가 끝없이 늘어남
def handle_log(level, message):
    if level == "DEBUG":
        write_to_console(message)
    elif level == "INFO":
        write_to_console(message)
        write_to_file(message)
    elif level == "WARNING":
        write_to_console(message)
        write_to_file(message)
        send_email_to_dev(message)
    elif level == "ERROR":
        write_to_console(message)
        write_to_file(message)
        send_email_to_dev(message)
        send_sms_to_admin(message)
    # 새 레벨이 추가되면? 또 elif를 추가해야 한다...
```

이 코드의 문제점:

- **OCP 위반**: 새로운 로그 레벨이나 처리 방식 추가 시 기존 코드 수정 필요
- **SRP 위반**: 하나의 함수가 모든 레벨의 처리 로직을 담당
- **유연성 부족**: 체인의 순서 변경이나 동적 구성이 불가능

---

## 3. 일상 비유

**고객 불만 에스컬레이션**을 떠올려 보자.

```
고객 불만 접수
    ↓
[상담원] → 간단한 문의는 직접 처리
    ↓ (처리 불가)
[팀장] → 환불/교환 등 권한 내 처리
    ↓ (처리 불가)
[부장] → 대규모 보상, 정책 변경 등 처리
    ↓ (처리 불가)
[이사] → 최종 의사결정
```

- 고객은 누가 처리할지 **알 필요 없다** (단지 불만을 접수할 뿐)
- 각 담당자는 자신의 권한 범위 내에서 처리하거나, 다음 사람에게 **넘긴다**
- 체인의 순서를 바꾸거나 새로운 담당자를 **쉽게 추가**할 수 있다

---

## 4. UML 다이어그램

```
 ┌─────────┐        ┌──────────────────────┐
 │ Client  │───────>│   <<abstract>>       │
 └─────────┘        │     Handler          │
                    ├──────────────────────┤
                    │ - next: Handler      │
                    ├──────────────────────┤
                    │ + set_next(handler)  │
                    │ + handle(request)    │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
   ┌──────────▼───┐  ┌────────▼─────┐  ┌───────▼──────┐
   │ HandlerA     │  │ HandlerB     │  │ HandlerC     │
   ├──────────────┤  ├──────────────┤  ├──────────────┤
   │ + handle()   │  │ + handle()   │  │ + handle()   │
   └──────────────┘  └──────────────┘  └──────────────┘

 요청 흐름: Client → HandlerA → HandlerB → HandlerC
            (처리 가능하면 처리, 아니면 다음으로 전달)
```

---

## 5. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from enum import IntEnum


class LogLevel(IntEnum):
    """로그 레벨 정의"""
    DEBUG = 1
    INFO = 2
    WARNING = 3
    ERROR = 4


class LogHandler(ABC):
    """핸들러 기본 클래스 (추상)"""

    def __init__(self, level: LogLevel) -> None:
        self._level = level
        self._next_handler: LogHandler | None = None

    def set_next(self, handler: LogHandler) -> LogHandler:
        """다음 핸들러를 설정하고, 체이닝을 위해 해당 핸들러를 반환한다."""
        self._next_handler = handler
        return handler

    def handle(self, level: LogLevel, message: str) -> None:
        """자신의 레벨 이하이면 처리하고, 다음 핸들러에게도 전달한다."""
        if level >= self._level:
            self._write(message)
        if self._next_handler is not None:
            self._next_handler.handle(level, message)

    @abstractmethod
    def _write(self, message: str) -> None:
        """실제 로그 출력 로직 (서브클래스에서 구현)"""
        pass


class ConsoleHandler(LogHandler):
    """콘솔 출력 핸들러 (DEBUG 이상)"""

    def __init__(self) -> None:
        super().__init__(LogLevel.DEBUG)

    def _write(self, message: str) -> None:
        print(f"[CONSOLE] {message}")


class FileHandler(LogHandler):
    """파일 기록 핸들러 (INFO 이상)"""

    def __init__(self) -> None:
        super().__init__(LogLevel.INFO)

    def _write(self, message: str) -> None:
        print(f"[FILE] {message}")  # 실제로는 파일에 기록


class EmailHandler(LogHandler):
    """이메일 알림 핸들러 (WARNING 이상)"""

    def __init__(self) -> None:
        super().__init__(LogLevel.WARNING)

    def _write(self, message: str) -> None:
        print(f"[EMAIL] {message}")


class SmsHandler(LogHandler):
    """SMS 알림 핸들러 (ERROR 이상)"""

    def __init__(self) -> None:
        super().__init__(LogLevel.ERROR)

    def _write(self, message: str) -> None:
        print(f"[SMS] {message}")


# === 사용 예시 ===
if __name__ == "__main__":
    # 체인 구성: Console → File → Email → SMS
    console = ConsoleHandler()
    console.set_next(
        FileHandler()
    ).set_next(
        EmailHandler()
    ).set_next(
        SmsHandler()
    )

    print("--- DEBUG 레벨 ---")
    console.handle(LogLevel.DEBUG, "디버그 메시지")

    print("\n--- INFO 레벨 ---")
    console.handle(LogLevel.INFO, "정보 메시지")

    print("\n--- WARNING 레벨 ---")
    console.handle(LogLevel.WARNING, "경고 메시지")

    print("\n--- ERROR 레벨 ---")
    console.handle(LogLevel.ERROR, "에러 메시지")
```

**출력 결과:**

```
--- DEBUG 레벨 ---
[CONSOLE] 디버그 메시지

--- INFO 레벨 ---
[CONSOLE] 정보 메시지
[FILE] 정보 메시지

--- WARNING 레벨 ---
[CONSOLE] 경고 메시지
[FILE] 경고 메시지
[EMAIL] 경고 메시지

--- ERROR 레벨 ---
[CONSOLE] 에러 메시지
[FILE] 에러 메시지
[EMAIL] 에러 메시지
[SMS] 에러 메시지
```

---

## 6. C# 구현

```csharp
using System;

// 로그 레벨 정의
public enum LogLevel
{
    Debug = 1,
    Info = 2,
    Warning = 3,
    Error = 4
}

// 핸들러 추상 클래스
public abstract class LogHandler
{
    protected LogLevel Level;
    private LogHandler? _nextHandler;

    protected LogHandler(LogLevel level)
    {
        Level = level;
    }

    /// <summary>
    /// 다음 핸들러를 설정하고, 체이닝을 위해 해당 핸들러를 반환한다.
    /// </summary>
    public LogHandler SetNext(LogHandler handler)
    {
        _nextHandler = handler;
        return handler;
    }

    /// <summary>
    /// 자신의 레벨 이하이면 처리하고, 다음 핸들러에게도 전달한다.
    /// </summary>
    public void Handle(LogLevel level, string message)
    {
        if (level >= Level)
        {
            Write(message);
        }

        _nextHandler?.Handle(level, message);
    }

    protected abstract void Write(string message);
}

// 콘솔 출력 핸들러
public class ConsoleLogHandler : LogHandler
{
    public ConsoleLogHandler() : base(LogLevel.Debug) { }

    protected override void Write(string message)
    {
        Console.WriteLine($"[CONSOLE] {message}");
    }
}

// 파일 기록 핸들러
public class FileLogHandler : LogHandler
{
    public FileLogHandler() : base(LogLevel.Info) { }

    protected override void Write(string message)
    {
        Console.WriteLine($"[FILE] {message}");
        // 실제로는 파일에 기록
    }
}

// 이메일 알림 핸들러
public class EmailLogHandler : LogHandler
{
    public EmailLogHandler() : base(LogLevel.Warning) { }

    protected override void Write(string message)
    {
        Console.WriteLine($"[EMAIL] {message}");
    }
}

// SMS 알림 핸들러
public class SmsLogHandler : LogHandler
{
    public SmsLogHandler() : base(LogLevel.Error) { }

    protected override void Write(string message)
    {
        Console.WriteLine($"[SMS] {message}");
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        // 체인 구성
        var console = new ConsoleLogHandler();
        console
            .SetNext(new FileLogHandler())
            .SetNext(new EmailLogHandler())
            .SetNext(new SmsLogHandler());

        Console.WriteLine("--- DEBUG 레벨 ---");
        console.Handle(LogLevel.Debug, "디버그 메시지");

        Console.WriteLine("\n--- INFO 레벨 ---");
        console.Handle(LogLevel.Info, "정보 메시지");

        Console.WriteLine("\n--- WARNING 레벨 ---");
        console.Handle(LogLevel.Warning, "경고 메시지");

        Console.WriteLine("\n--- ERROR 레벨 ---");
        console.Handle(LogLevel.Error, "에러 메시지");
    }
}
```

---

## 7. 실무 예제: HTTP 미들웨어 체인

웹 프레임워크에서 흔히 볼 수 있는 미들웨어 패턴은 Chain of Responsibility의 대표적인 실무 적용 사례다.

### Python (Flask 스타일 미들웨어)

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Any


@dataclass
class HttpRequest:
    """HTTP 요청 객체"""
    path: str
    method: str = "GET"
    headers: dict[str, str] = field(default_factory=dict)
    body: Any = None
    user: str | None = None  # 인증 후 설정됨


@dataclass
class HttpResponse:
    """HTTP 응답 객체"""
    status_code: int = 200
    body: str = ""


class Middleware(ABC):
    """미들웨어 기본 클래스"""

    def __init__(self) -> None:
        self._next: Middleware | None = None

    def set_next(self, middleware: Middleware) -> Middleware:
        self._next = middleware
        return middleware

    def handle(self, request: HttpRequest) -> HttpResponse:
        """기본 처리: 다음 미들웨어로 전달"""
        if self._next:
            return self._next.handle(request)
        return HttpResponse(200, "OK")

    @abstractmethod
    def process(self, request: HttpRequest) -> HttpResponse | None:
        """미들웨어 고유 로직. None 반환 시 다음으로 전달."""
        pass


class AuthenticationMiddleware(Middleware):
    """인증 미들웨어: API 키 검증"""

    VALID_API_KEYS = {"key-abc-123", "key-xyz-789"}

    def handle(self, request: HttpRequest) -> HttpResponse:
        result = self.process(request)
        if result:
            return result
        return super().handle(request)

    def process(self, request: HttpRequest) -> HttpResponse | None:
        api_key = request.headers.get("X-API-Key")
        if not api_key:
            return HttpResponse(401, "API 키가 필요합니다.")
        if api_key not in self.VALID_API_KEYS:
            return HttpResponse(403, "유효하지 않은 API 키입니다.")
        request.user = f"user_{api_key[-3:]}"
        print(f"  [AUTH] 인증 성공: {request.user}")
        return None  # 다음 미들웨어로 전달


class RateLimitMiddleware(Middleware):
    """속도 제한 미들웨어"""

    def __init__(self, max_requests: int = 5) -> None:
        super().__init__()
        self._max_requests = max_requests
        self._request_count: dict[str, int] = {}

    def handle(self, request: HttpRequest) -> HttpResponse:
        result = self.process(request)
        if result:
            return result
        return super().handle(request)

    def process(self, request: HttpRequest) -> HttpResponse | None:
        user = request.user or "anonymous"
        self._request_count[user] = self._request_count.get(user, 0) + 1

        if self._request_count[user] > self._max_requests:
            return HttpResponse(429, "요청 한도를 초과했습니다.")

        print(f"  [RATE] {user}: {self._request_count[user]}/{self._max_requests}")
        return None


class LoggingMiddleware(Middleware):
    """로깅 미들웨어"""

    def handle(self, request: HttpRequest) -> HttpResponse:
        self.process(request)
        response = super().handle(request)
        print(f"  [LOG] {request.method} {request.path} -> {response.status_code}")
        return response

    def process(self, request: HttpRequest) -> HttpResponse | None:
        print(f"  [LOG] 요청 수신: {request.method} {request.path}")
        return None


class RouterMiddleware(Middleware):
    """라우팅 미들웨어 (최종 처리)"""

    def handle(self, request: HttpRequest) -> HttpResponse:
        result = self.process(request)
        return result if result else HttpResponse(404, "Not Found")

    def process(self, request: HttpRequest) -> HttpResponse | None:
        if request.path == "/api/users":
            return HttpResponse(200, '{"users": ["Alice", "Bob"]}')
        elif request.path == "/api/health":
            return HttpResponse(200, '{"status": "healthy"}')
        return None


# === 사용 예시 ===
if __name__ == "__main__":
    # 미들웨어 체인 구성
    logging = LoggingMiddleware()
    auth = AuthenticationMiddleware()
    rate_limit = RateLimitMiddleware(max_requests=3)
    router = RouterMiddleware()

    logging.set_next(auth)
    auth.set_next(rate_limit)
    rate_limit.set_next(router)

    pipeline = logging  # 진입점

    # 테스트 1: 인증 없이 요청
    print("=== 테스트 1: 인증 없는 요청 ===")
    req = HttpRequest(path="/api/users")
    res = pipeline.handle(req)
    print(f"  응답: {res.status_code} - {res.body}\n")

    # 테스트 2: 올바른 API 키로 요청
    print("=== 테스트 2: 인증된 요청 ===")
    req = HttpRequest(
        path="/api/users",
        headers={"X-API-Key": "key-abc-123"}
    )
    res = pipeline.handle(req)
    print(f"  응답: {res.status_code} - {res.body}\n")

    # 테스트 3: 존재하지 않는 경로
    print("=== 테스트 3: 404 경로 ===")
    req = HttpRequest(
        path="/api/unknown",
        headers={"X-API-Key": "key-abc-123"}
    )
    res = pipeline.handle(req)
    print(f"  응답: {res.status_code} - {res.body}")
```

### C# (미들웨어 체인)

```csharp
using System;
using System.Collections.Generic;

// HTTP 요청/응답 모델
public class HttpRequest
{
    public string Path { get; set; } = "/";
    public string Method { get; set; } = "GET";
    public Dictionary<string, string> Headers { get; set; } = new();
    public string? User { get; set; }
}

public class HttpResponse
{
    public int StatusCode { get; set; } = 200;
    public string Body { get; set; } = "";

    public HttpResponse(int statusCode, string body)
    {
        StatusCode = statusCode;
        Body = body;
    }
}

// 미들웨어 추상 클래스
public abstract class Middleware
{
    private Middleware? _next;

    public Middleware SetNext(Middleware middleware)
    {
        _next = middleware;
        return middleware;
    }

    public virtual HttpResponse Handle(HttpRequest request)
    {
        if (_next != null)
            return _next.Handle(request);
        return new HttpResponse(200, "OK");
    }
}

// 인증 미들웨어
public class AuthMiddleware : Middleware
{
    private readonly HashSet<string> _validKeys = new() { "key-abc-123", "key-xyz-789" };

    public override HttpResponse Handle(HttpRequest request)
    {
        if (!request.Headers.ContainsKey("X-API-Key"))
            return new HttpResponse(401, "API 키가 필요합니다.");

        var apiKey = request.Headers["X-API-Key"];
        if (!_validKeys.Contains(apiKey))
            return new HttpResponse(403, "유효하지 않은 API 키입니다.");

        request.User = $"user_{apiKey[^3..]}";
        Console.WriteLine($"  [AUTH] 인증 성공: {request.User}");
        return base.Handle(request);
    }
}

// 속도 제한 미들웨어
public class RateLimitMiddleware : Middleware
{
    private readonly int _maxRequests;
    private readonly Dictionary<string, int> _counts = new();

    public RateLimitMiddleware(int maxRequests = 5)
    {
        _maxRequests = maxRequests;
    }

    public override HttpResponse Handle(HttpRequest request)
    {
        var user = request.User ?? "anonymous";
        _counts.TryGetValue(user, out int count);
        _counts[user] = ++count;

        if (count > _maxRequests)
            return new HttpResponse(429, "요청 한도를 초과했습니다.");

        Console.WriteLine($"  [RATE] {user}: {count}/{_maxRequests}");
        return base.Handle(request);
    }
}

// 라우터 미들웨어
public class RouterMiddleware : Middleware
{
    public override HttpResponse Handle(HttpRequest request)
    {
        return request.Path switch
        {
            "/api/users" => new HttpResponse(200, "{\"users\": [\"Alice\", \"Bob\"]}"),
            "/api/health" => new HttpResponse(200, "{\"status\": \"healthy\"}"),
            _ => new HttpResponse(404, "Not Found")
        };
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **느슨한 결합** | 요청 발신자와 수신자 사이의 결합도를 낮춘다 |
| **유연한 체인 구성** | 런타임에 핸들러를 추가/제거/재배치할 수 있다 |
| **단일 책임 원칙** | 각 핸들러가 자신의 역할에만 집중한다 |
| **개방-폐쇄 원칙** | 기존 코드 수정 없이 새로운 핸들러를 추가할 수 있다 |

### 단점

| 단점 | 설명 |
|------|------|
| **처리 보장 없음** | 체인 끝까지 도달해도 아무도 처리하지 않을 수 있다 |
| **디버깅 어려움** | 어떤 핸들러가 요청을 처리했는지 추적하기 어렵다 |
| **성능 이슈** | 체인이 길어지면 모든 핸들러를 거쳐야 할 수 있다 |
| **순서 의존성** | 핸들러 순서가 결과에 영향을 미칠 수 있다 |

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Composite** | 부모 컴포넌트를 통해 요청을 전파하는 데 함께 사용 |
| **Command** | 요청을 객체로 캡슐화하는 점에서 함께 사용 가능 |
| **Decorator** | 구조적으로 유사하나, Decorator는 추가 행동을, CoR은 요청 전달을 목적으로 함 |
| **Observer** | 하나의 이벤트를 여러 핸들러에게 전달하는 점에서 유사 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

1. Chain of Responsibility는 **요청 처리를 체인으로 연결된 핸들러들에게 위임**한다.
2. 각 핸들러는 요청을 **직접 처리**하거나 **다음 핸들러에게 전달**한다.
3. 클라이언트는 어떤 핸들러가 처리할지 **알 필요 없다**.
4. 실무에서 **미들웨어, 로깅, 인증 체인** 등에 널리 사용된다.

### 적용 체크리스트

- [ ] 요청을 처리할 객체가 **여러 개**이고, **런타임에 결정**해야 하는가?
- [ ] 요청 발신자와 수신자를 **분리**하고 싶은가?
- [ ] 처리 체인을 **동적으로 구성**할 필요가 있는가?
- [ ] 각 처리 단계가 **독립적으로 변경/추가/삭제**될 수 있어야 하는가?
- [ ] if/elif 분기가 과도하게 중첩되어 **리팩토링이 필요**한가?

> **2~3년차 개발자를 위한 팁**: Django/Flask의 미들웨어, ASP.NET Core의 Middleware Pipeline, Java Servlet의 Filter Chain이 모두 이 패턴의 실무 적용 사례다. 프레임워크를 사용할 때 "아, 이게 Chain of Responsibility구나!"라고 인식할 수 있으면 설계 의도를 훨씬 잘 이해할 수 있다.
