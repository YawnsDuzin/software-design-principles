# 느슨한 결합 (Loose Coupling)

> **"좋은 소프트웨어는 레고 블록과 같다. 각 블록을 독립적으로 교체할 수 있어야 한다."**

## 목차

1. [정의와 개념](#1-정의와-개념)
2. [비유로 이해하기](#2-비유로-이해하기)
3. [결합도의 종류](#3-결합도의-종류)
4. [나쁜 예제: 높은 결합도](#4-나쁜-예제-높은-결합도)
5. [좋은 예제: 낮은 결합도](#5-좋은-예제-낮은-결합도)
6. [결합도를 낮추는 기법들](#6-결합도를-낮추는-기법들)
7. [Python 코드 예제](#7-python-코드-예제)
8. [C# 코드 예제](#8-c-코드-예제)
9. [테스트 용이성과의 관계](#9-테스트-용이성과의-관계)
10. [실무에서 결합도 측정하기](#10-실무에서-결합도-측정하기)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 개념 연결](#12-관련-개념-연결)

---

## 1. 정의와 개념

**느슨한 결합(Loose Coupling)** 이란 모듈이나 클래스 간의 의존성을 최소화하여, 하나의 모듈을 변경해도 다른 모듈에 미치는 영향이 적도록 설계하는 원칙입니다.

### 핵심 질문

> "클래스 A를 수정했을 때, 클래스 B도 반드시 수정해야 하는가?"

- **YES** -> 강한 결합 (Tight Coupling) -> 나쁜 설계
- **NO** -> 느슨한 결합 (Loose Coupling) -> 좋은 설계

### 왜 중요한가?

| 문제 상황 | 높은 결합도일 때 | 낮은 결합도일 때 |
|-----------|-----------------|-----------------|
| DB 교체 (MySQL -> PostgreSQL) | 수십 개 파일 수정 | 구현체 1개만 교체 |
| 외부 API 변경 | 연쇄적 수정 필요 | 어댑터만 수정 |
| 유닛 테스트 작성 | Mock 불가, 통합 테스트만 가능 | Mock 주입으로 단위 테스트 용이 |
| 기능 추가 | 기존 코드 대폭 수정 | 새 구현체 추가만으로 해결 |

---

## 2. 비유로 이해하기

### 레고 블록 vs 퍼즐 조각

```
┌─────────────────────────────────────────────────────────┐
│                   느슨한 결합 = 레고 블록                   │
│                                                         │
│   ┌───┐  ┌───┐  ┌───┐       표준화된 연결부(인터페이스)로  │
│   │ A │──│ B │──│ C │       블록을 자유롭게 교체 가능       │
│   └───┘  └───┘  └───┘                                   │
│                                                         │
│   ┌───┐  ┌───┐  ┌───┐       B를 B'로 교체해도             │
│   │ A │──│B' │──│ C │       A와 C는 영향 없음              │
│   └───┘  └───┘  └───┘                                   │
├─────────────────────────────────────────────────────────┤
│                   강한 결합 = 퍼즐 조각                     │
│                                                         │
│   ┌──┬──┬──┐                 각 조각이 특정 위치에만       │
│   │A ┤B ┤C │                 맞도록 설계됨                 │
│   └──┴──┴──┘                                            │
│                                                         │
│   ┌──┬──┐                   B를 빼면 전체가 무너짐         │
│   │A ┤  ┤C │  <- 빈 공간!                                │
│   └──┴──┘                                               │
└─────────────────────────────────────────────────────────┘
```

### 현실 세계 비유: 전원 콘센트

- **느슨한 결합**: 표준 콘센트에 어떤 가전제품이든 꽂을 수 있음
- **강한 결합**: 가전제품 전선이 벽 안에 직접 납땜되어 있음 (교체 불가)

---

## 3. 결합도의 종류

결합도는 낮은 것(좋은 것)부터 높은 것(나쁜 것) 순으로 다음과 같이 분류됩니다.

```
좋음 ◀──────────────────────────────────────────────▶ 나쁨
 │                                                    │
 ▼                                                    ▼
데이터    스탬프     제어      공통      내용
결합      결합       결합      결합      결합
(Data)   (Stamp)  (Control) (Common) (Content)
```

### 3.1 데이터 결합 (Data Coupling) - 가장 좋음

모듈 간에 **필요한 데이터만** 매개변수로 전달합니다.

```python
# 데이터 결합: 필요한 값만 전달
def calculate_tax(price: float, tax_rate: float) -> float:
    return price * tax_rate
```

### 3.2 스탬프 결합 (Stamp Coupling)

모듈 간에 **데이터 구조체(객체)** 를 전달하지만, 수신 모듈은 그 중 일부만 사용합니다.

```python
# 스탬프 결합: User 객체를 전달하지만 이름만 사용
def send_greeting(user: User) -> str:
    return f"안녕하세요, {user.name}님!"  # user.email, user.age 등은 사용 안 함
```

### 3.3 제어 결합 (Control Coupling)

한 모듈이 다른 모듈의 **제어 흐름을 결정하는 플래그**를 전달합니다.

```python
# 제어 결합: flag가 내부 동작을 제어
def format_output(data: str, use_json: bool) -> str:
    if use_json:
        return json.dumps({"data": data})
    else:
        return f"<xml>{data}</xml>"
```

### 3.4 공통 결합 (Common Coupling)

여러 모듈이 **전역 변수나 공유 상태**를 참조합니다.

```python
# 공통 결합: 전역 변수 공유
global_config = {}  # 여러 모듈이 이 전역 변수를 읽고 씀

def module_a():
    global_config["mode"] = "debug"

def module_b():
    if global_config["mode"] == "debug":  # module_a에 의존
        print("디버그 모드")
```

### 3.5 내용 결합 (Content Coupling) - 가장 나쁨

한 모듈이 다른 모듈의 **내부 구현에 직접 접근**합니다.

```python
# 내용 결합: 다른 모듈의 내부 구현에 직접 접근
class OrderProcessor:
    def __init__(self):
        self._internal_state = "ready"  # 비공개 의도

class ReportGenerator:
    def generate(self, processor: OrderProcessor):
        # 다른 클래스의 내부 상태에 직접 접근 (매우 나쁨!)
        if processor._internal_state == "ready":
            pass
```

---

## 4. 나쁜 예제: 높은 결합도

### 문제 상황: 주문 처리 시스템

```
❌ 높은 결합도 구조

┌─────────────────────────────────────┐
│           OrderService              │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ MySqlDatabase db = new ...    │  │  <- 구체 클래스 직접 생성
│  │ SmtpEmailSender email = new   │  │  <- 구체 클래스 직접 생성
│  │ StripePayment pay = new ...   │  │  <- 구체 클래스 직접 생성
│  └───────────────────────────────┘  │
│                                     │
│  processOrder() {                   │
│    db.save(order);        // MySQL에 종속  │
│    pay.charge(amount);    // Stripe에 종속  │
│    email.send(receipt);   // SMTP에 종속    │
│  }                                  │
└─────────────────────────────────────┘

문제: DB를 PostgreSQL로 바꾸려면? -> OrderService 수정 필요
     결제를 PayPal로 바꾸려면? -> OrderService 수정 필요
     이메일을 SMS로 바꾸려면? -> OrderService 수정 필요
```

### Python - 높은 결합도 예제

```python
# ❌ 나쁜 예: 구체 클래스에 직접 의존
class MySqlDatabase:
    def save(self, data: dict) -> None:
        print(f"[MySQL] 데이터 저장: {data}")

class SmtpEmailSender:
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[SMTP] {to}에게 이메일 전송: {subject}")

class StripePayment:
    def charge(self, amount: float) -> bool:
        print(f"[Stripe] {amount}원 결제 처리")
        return True


class OrderService:
    """모든 의존성을 내부에서 직접 생성 - 높은 결합도!"""

    def __init__(self):
        # 구체 클래스를 직접 생성 (강한 결합)
        self.db = MySqlDatabase()
        self.email = SmtpEmailSender()
        self.payment = StripePayment()

    def process_order(self, order: dict) -> None:
        # MySQL에 종속
        self.db.save(order)

        # Stripe에 종속
        self.payment.charge(order["amount"])

        # SMTP에 종속
        self.email.send(
            order["customer_email"],
            "주문 확인",
            f"주문 {order['id']}이 처리되었습니다."
        )


# 사용
service = OrderService()
service.process_order({"id": 1, "amount": 50000, "customer_email": "user@test.com"})

# 문제점:
# 1. MySqlDatabase를 PostgresDatabase로 변경하려면 OrderService 코드 수정 필요
# 2. 테스트 시 실제 DB, 이메일, 결제 서비스가 필요
# 3. OrderService를 재사용할 수 없음
```

### C# - 높은 결합도 예제

```csharp
// ❌ 나쁜 예: 구체 클래스에 직접 의존
public class MySqlDatabase
{
    public void Save(Dictionary<string, object> data)
    {
        Console.WriteLine($"[MySQL] 데이터 저장: {data["id"]}");
    }
}

public class SmtpEmailSender
{
    public void Send(string to, string subject, string body)
    {
        Console.WriteLine($"[SMTP] {to}에게 이메일 전송: {subject}");
    }
}

public class StripePayment
{
    public bool Charge(decimal amount)
    {
        Console.WriteLine($"[Stripe] {amount}원 결제 처리");
        return true;
    }
}

public class OrderService
{
    // 구체 클래스를 직접 생성 (강한 결합!)
    private readonly MySqlDatabase _db = new MySqlDatabase();
    private readonly SmtpEmailSender _email = new SmtpEmailSender();
    private readonly StripePayment _payment = new StripePayment();

    public void ProcessOrder(Order order)
    {
        _db.Save(order.ToDict());       // MySQL에 종속
        _payment.Charge(order.Amount);   // Stripe에 종속
        _email.Send(                     // SMTP에 종속
            order.CustomerEmail,
            "주문 확인",
            $"주문 {order.Id}이 처리되었습니다."
        );
    }
}

// 문제점:
// 1. DB 변경 시 OrderService 수정 필요
// 2. 유닛 테스트 불가능 (실제 서비스 연결 필요)
// 3. OCP(개방-폐쇄 원칙) 위반
```

---

## 5. 좋은 예제: 낮은 결합도

### 개선된 구조

```
✅ 낮은 결합도 구조

┌──────────────────────────────────────────┐
│              OrderService                │
│                                          │
│  IDatabase db          (인터페이스)       │
│  INotificationSender sender (인터페이스)  │
│  IPaymentGateway payment   (인터페이스)   │
│                                          │
│  processOrder() {                        │
│    db.save(order);        // 어떤 DB든 OK    │
│    payment.charge(amount);// 어떤 결제든 OK   │
│    sender.send(receipt);  // 어떤 알림이든 OK  │
│  }                                       │
└──────────┬───────────────────────────────┘
           │ 인터페이스에만 의존
           │
     ┌─────┼──────────────┐
     ▼     ▼              ▼
┌────────┐ ┌──────────┐ ┌────────────┐
│ MySQL  │ │ Postgres │ │ MongoDB    │   <- 구현체는 자유롭게 교체
└────────┘ └──────────┘ └────────────┘
```

### Python - 낮은 결합도 예제

```python
# ✅ 좋은 예: 인터페이스(추상 클래스)에 의존
from abc import ABC, abstractmethod


# --- 인터페이스 정의 ---
class IDatabase(ABC):
    @abstractmethod
    def save(self, data: dict) -> None:
        pass

class INotificationSender(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> None:
        pass

class IPaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float) -> bool:
        pass


# --- 구현체들 (필요에 따라 교체 가능) ---
class MySqlDatabase(IDatabase):
    def save(self, data: dict) -> None:
        print(f"[MySQL] 데이터 저장: {data}")

class PostgresDatabase(IDatabase):
    def save(self, data: dict) -> None:
        print(f"[PostgreSQL] 데이터 저장: {data}")

class SmtpEmailSender(INotificationSender):
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[SMTP] {to}에게 이메일 전송: {subject}")

class SlackNotifier(INotificationSender):
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[Slack] #{to} 채널에 알림: {subject}")

class StripePayment(IPaymentGateway):
    def charge(self, amount: float) -> bool:
        print(f"[Stripe] {amount}원 결제 처리")
        return True

class PayPalPayment(IPaymentGateway):
    def charge(self, amount: float) -> bool:
        print(f"[PayPal] {amount}원 결제 처리")
        return True


# --- 핵심 서비스: 인터페이스에만 의존 ---
class OrderService:
    """의존성을 외부에서 주입받아 느슨한 결합 달성"""

    def __init__(
        self,
        db: IDatabase,
        notification: INotificationSender,
        payment: IPaymentGateway
    ):
        self._db = db
        self._notification = notification
        self._payment = payment

    def process_order(self, order: dict) -> None:
        self._db.save(order)
        self._payment.charge(order["amount"])
        self._notification.send(
            order["customer_email"],
            "주문 확인",
            f"주문 {order['id']}이 처리되었습니다."
        )


# --- 사용 예시 ---
# 설정 1: MySQL + SMTP + Stripe
service_v1 = OrderService(
    db=MySqlDatabase(),
    notification=SmtpEmailSender(),
    payment=StripePayment()
)

# 설정 2: PostgreSQL + Slack + PayPal (OrderService 코드 변경 없이 교체!)
service_v2 = OrderService(
    db=PostgresDatabase(),
    notification=SlackNotifier(),
    payment=PayPalPayment()
)

order = {"id": 1, "amount": 50000, "customer_email": "user@test.com"}
service_v1.process_order(order)
service_v2.process_order(order)
```

### C# - 낮은 결합도 예제

```csharp
// ✅ 좋은 예: 인터페이스에 의존

// --- 인터페이스 정의 ---
public interface IDatabase
{
    void Save(Dictionary<string, object> data);
}

public interface INotificationSender
{
    void Send(string to, string subject, string body);
}

public interface IPaymentGateway
{
    bool Charge(decimal amount);
}

// --- 구현체들 ---
public class MySqlDatabase : IDatabase
{
    public void Save(Dictionary<string, object> data)
        => Console.WriteLine($"[MySQL] 데이터 저장: {data["id"]}");
}

public class PostgresDatabase : IDatabase
{
    public void Save(Dictionary<string, object> data)
        => Console.WriteLine($"[PostgreSQL] 데이터 저장: {data["id"]}");
}

public class SmtpEmailSender : INotificationSender
{
    public void Send(string to, string subject, string body)
        => Console.WriteLine($"[SMTP] {to}에게 이메일 전송: {subject}");
}

public class SlackNotifier : INotificationSender
{
    public void Send(string to, string subject, string body)
        => Console.WriteLine($"[Slack] #{to} 채널에 알림: {subject}");
}

public class StripePayment : IPaymentGateway
{
    public bool Charge(decimal amount)
    {
        Console.WriteLine($"[Stripe] {amount}원 결제 처리");
        return true;
    }
}

// --- 핵심 서비스: 인터페이스에만 의존 ---
public class OrderService
{
    private readonly IDatabase _db;
    private readonly INotificationSender _notification;
    private readonly IPaymentGateway _payment;

    // 생성자를 통해 의존성 주입 (Loose Coupling!)
    public OrderService(
        IDatabase db,
        INotificationSender notification,
        IPaymentGateway payment)
    {
        _db = db;
        _notification = notification;
        _payment = payment;
    }

    public void ProcessOrder(Order order)
    {
        _db.Save(order.ToDict());
        _payment.Charge(order.Amount);
        _notification.Send(
            order.CustomerEmail,
            "주문 확인",
            $"주문 {order.Id}이 처리되었습니다."
        );
    }
}

// --- 사용 예시 ---
// 설정 1: MySQL + SMTP + Stripe
var serviceV1 = new OrderService(
    new MySqlDatabase(),
    new SmtpEmailSender(),
    new StripePayment()
);

// 설정 2: PostgreSQL + Slack + PayPal (코드 변경 없이!)
var serviceV2 = new OrderService(
    new PostgresDatabase(),
    new SlackNotifier(),
    new StripePayment()
);
```

---

## 6. 결합도를 낮추는 기법들

### 6.1 인터페이스 / 추상 클래스

```
구체 클래스에 의존 (강결합)        인터페이스에 의존 (약결합)

  OrderService                    OrderService
       │                               │
       ▼                               ▼
  MySqlDatabase                   IDatabase (인터페이스)
                                  ▲         ▲
                                  │         │
                             MySQL DB   Postgres DB
```

### 6.2 의존성 주입 (Dependency Injection)

객체가 스스로 의존성을 생성하지 않고, 외부에서 주입받습니다.

```python
# Before: 내부에서 생성 (강결합)
class Service:
    def __init__(self):
        self.repo = MySqlRepository()  # 직접 생성

# After: 외부에서 주입 (약결합)
class Service:
    def __init__(self, repo: IRepository):  # 주입받음
        self.repo = repo
```

### 6.3 이벤트 / 옵저버 패턴

모듈 간 직접 호출 대신 이벤트를 통해 통신합니다.

```python
# 이벤트 기반 느슨한 결합
from typing import Callable

class EventBus:
    def __init__(self):
        self._handlers: dict[str, list[Callable]] = {}

    def subscribe(self, event: str, handler: Callable) -> None:
        self._handlers.setdefault(event, []).append(handler)

    def publish(self, event: str, data: dict) -> None:
        for handler in self._handlers.get(event, []):
            handler(data)

# 사용: 모듈들이 서로를 모름 (완전한 분리)
bus = EventBus()

# 모듈 A: 주문 처리
def process_order(order):
    # 주문 처리 로직...
    bus.publish("order_completed", order)

# 모듈 B: 이메일 전송 (모듈 A를 전혀 모름)
bus.subscribe("order_completed", lambda order: print(f"이메일 전송: {order['id']}"))

# 모듈 C: 재고 업데이트 (모듈 A, B를 전혀 모름)
bus.subscribe("order_completed", lambda order: print(f"재고 차감: {order['id']}"))
```

### 6.4 메시지 큐

시스템 간 통신을 메시지 큐를 통해 비동기로 처리합니다.

```
┌──────────┐     ┌─────────────┐     ┌──────────┐
│ Producer │ --> │ Message Queue│ --> │ Consumer │
│ (주문)    │     │ (RabbitMQ)  │     │ (결제)    │
└──────────┘     └─────────────┘     └──────────┘
                                     ┌──────────┐
                                 --> │ Consumer │
                                     │ (알림)    │
                                     └──────────┘

Producer와 Consumer는 서로의 존재를 모름 -> 완전한 느슨한 결합
```

---

## 7. Python 코드 예제

### 종합 예제: 알림 시스템

```python
"""
느슨한 결합을 적용한 알림 시스템 종합 예제
"""
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol


# --- 도메인 모델 ---
@dataclass
class Notification:
    recipient: str
    title: str
    message: str
    priority: str = "normal"


# --- Protocol을 이용한 구조적 타이핑 (Python 3.8+) ---
class NotificationChannel(Protocol):
    """알림 채널 프로토콜 - 덕 타이핑 기반 느슨한 결합"""
    def send(self, notification: Notification) -> bool: ...

class NotificationFormatter(Protocol):
    """알림 포매터 프로토콜"""
    def format(self, notification: Notification) -> str: ...

class NotificationLogger(Protocol):
    """알림 로거 프로토콜"""
    def log(self, message: str) -> None: ...


# --- 구현체들 ---
class EmailChannel:
    """이메일 채널 구현"""
    def send(self, notification: Notification) -> bool:
        print(f"[Email] To: {notification.recipient} | {notification.title}")
        return True

class SmsChannel:
    """SMS 채널 구현"""
    def send(self, notification: Notification) -> bool:
        print(f"[SMS] To: {notification.recipient} | {notification.message[:50]}")
        return True

class PushChannel:
    """푸시 알림 채널 구현"""
    def send(self, notification: Notification) -> bool:
        print(f"[Push] To: {notification.recipient} | {notification.title}")
        return True

class SimpleFormatter:
    """단순 포맷터"""
    def format(self, notification: Notification) -> str:
        return f"[{notification.priority.upper()}] {notification.title}: {notification.message}"

class HtmlFormatter:
    """HTML 포맷터"""
    def format(self, notification: Notification) -> str:
        return f"<h1>{notification.title}</h1><p>{notification.message}</p>"

class ConsoleLogger:
    """콘솔 로거"""
    def log(self, message: str) -> None:
        print(f"[LOG] {message}")


# --- 핵심 서비스 (인터페이스에만 의존) ---
class NotificationService:
    """
    알림 서비스 - 느슨한 결합의 핵심 예시

    이 클래스는 어떤 채널, 포맷터, 로거를 사용하는지 알지 못합니다.
    오직 Protocol에 맞는 인터페이스만 기대합니다.
    """

    def __init__(
        self,
        channels: list[NotificationChannel],
        formatter: NotificationFormatter,
        logger: NotificationLogger
    ):
        self._channels = channels
        self._formatter = formatter
        self._logger = logger

    def notify(self, notification: Notification) -> dict[str, bool]:
        formatted_message = self._formatter.format(notification)
        self._logger.log(f"알림 발송 시작: {notification.title}")

        results = {}
        for channel in self._channels:
            channel_name = type(channel).__name__
            try:
                success = channel.send(notification)
                results[channel_name] = success
                self._logger.log(f"  {channel_name}: {'성공' if success else '실패'}")
            except Exception as e:
                results[channel_name] = False
                self._logger.log(f"  {channel_name}: 오류 - {e}")

        return results


# --- 사용 예시 ---
if __name__ == "__main__":
    # 설정 1: 이메일 + SMS, 단순 포맷
    service_basic = NotificationService(
        channels=[EmailChannel(), SmsChannel()],
        formatter=SimpleFormatter(),
        logger=ConsoleLogger()
    )

    # 설정 2: 모든 채널, HTML 포맷 (코드 변경 없이 설정만 변경)
    service_full = NotificationService(
        channels=[EmailChannel(), SmsChannel(), PushChannel()],
        formatter=HtmlFormatter(),
        logger=ConsoleLogger()
    )

    notification = Notification(
        recipient="user@example.com",
        title="주문 완료",
        message="주문 #1234가 성공적으로 처리되었습니다.",
        priority="high"
    )

    print("=== Basic Service ===")
    service_basic.notify(notification)

    print("\n=== Full Service ===")
    service_full.notify(notification)
```

---

## 8. C# 코드 예제

### 종합 예제: 알림 시스템

```csharp
using System;
using System.Collections.Generic;

// --- 도메인 모델 ---
public record Notification(
    string Recipient,
    string Title,
    string Message,
    string Priority = "normal"
);

// --- 인터페이스 정의 ---
public interface INotificationChannel
{
    string Name { get; }
    bool Send(Notification notification);
}

public interface INotificationFormatter
{
    string Format(Notification notification);
}

public interface INotificationLogger
{
    void Log(string message);
}

// --- 구현체들 ---
public class EmailChannel : INotificationChannel
{
    public string Name => "Email";

    public bool Send(Notification notification)
    {
        Console.WriteLine($"[Email] To: {notification.Recipient} | {notification.Title}");
        return true;
    }
}

public class SmsChannel : INotificationChannel
{
    public string Name => "SMS";

    public bool Send(Notification notification)
    {
        var shortMessage = notification.Message.Length > 50
            ? notification.Message[..50]
            : notification.Message;
        Console.WriteLine($"[SMS] To: {notification.Recipient} | {shortMessage}");
        return true;
    }
}

public class PushChannel : INotificationChannel
{
    public string Name => "Push";

    public bool Send(Notification notification)
    {
        Console.WriteLine($"[Push] To: {notification.Recipient} | {notification.Title}");
        return true;
    }
}

public class SimpleFormatter : INotificationFormatter
{
    public string Format(Notification notification)
        => $"[{notification.Priority.ToUpper()}] {notification.Title}: {notification.Message}";
}

public class HtmlFormatter : INotificationFormatter
{
    public string Format(Notification notification)
        => $"<h1>{notification.Title}</h1><p>{notification.Message}</p>";
}

public class ConsoleLogger : INotificationLogger
{
    public void Log(string message)
        => Console.WriteLine($"[LOG] {message}");
}

// --- 핵심 서비스 (인터페이스에만 의존) ---
public class NotificationService
{
    private readonly IEnumerable<INotificationChannel> _channels;
    private readonly INotificationFormatter _formatter;
    private readonly INotificationLogger _logger;

    // 생성자 주입: 모든 의존성을 외부에서 받음
    public NotificationService(
        IEnumerable<INotificationChannel> channels,
        INotificationFormatter formatter,
        INotificationLogger logger)
    {
        _channels = channels;
        _formatter = formatter;
        _logger = logger;
    }

    public Dictionary<string, bool> Notify(Notification notification)
    {
        var formatted = _formatter.Format(notification);
        _logger.Log($"알림 발송 시작: {notification.Title}");

        var results = new Dictionary<string, bool>();

        foreach (var channel in _channels)
        {
            try
            {
                var success = channel.Send(notification);
                results[channel.Name] = success;
                _logger.Log($"  {channel.Name}: {(success ? "성공" : "실패")}");
            }
            catch (Exception ex)
            {
                results[channel.Name] = false;
                _logger.Log($"  {channel.Name}: 오류 - {ex.Message}");
            }
        }

        return results;
    }
}

// --- 사용 예시 ---
public class Program
{
    public static void Main()
    {
        // 설정 1: 이메일 + SMS
        var serviceBasic = new NotificationService(
            channels: new INotificationChannel[] { new EmailChannel(), new SmsChannel() },
            formatter: new SimpleFormatter(),
            logger: new ConsoleLogger()
        );

        // 설정 2: 모든 채널 + HTML 포맷 (코드 변경 없이!)
        var serviceFull = new NotificationService(
            channels: new INotificationChannel[]
            {
                new EmailChannel(),
                new SmsChannel(),
                new PushChannel()
            },
            formatter: new HtmlFormatter(),
            logger: new ConsoleLogger()
        );

        var notification = new Notification(
            Recipient: "user@example.com",
            Title: "주문 완료",
            Message: "주문 #1234가 성공적으로 처리되었습니다.",
            Priority: "high"
        );

        Console.WriteLine("=== Basic Service ===");
        serviceBasic.Notify(notification);

        Console.WriteLine("\n=== Full Service ===");
        serviceFull.Notify(notification);
    }
}
```

---

## 9. 테스트 용이성과의 관계

느슨한 결합의 가장 큰 실무적 이점 중 하나는 **테스트 용이성**입니다.

### 높은 결합도 -> 테스트 어려움

```python
# ❌ 테스트하기 어려운 코드 (높은 결합도)
class OrderService:
    def __init__(self):
        self.db = MySqlDatabase()  # 실제 DB 필요!
        self.payment = StripePayment()  # 실제 결제 서비스 필요!

    def process_order(self, order):
        self.db.save(order)          # DB 없으면 테스트 불가
        self.payment.charge(order)   # Stripe 없으면 테스트 불가
```

### 낮은 결합도 -> 테스트 용이

```python
# ✅ 테스트하기 쉬운 코드 (느슨한 결합)
class OrderService:
    def __init__(self, db: IDatabase, payment: IPaymentGateway):
        self._db = db
        self._payment = payment

# 테스트: Mock 객체 주입
class MockDatabase(IDatabase):
    def __init__(self):
        self.saved_data = []

    def save(self, data: dict) -> None:
        self.saved_data.append(data)  # 실제 DB 없이 저장 확인

class MockPayment(IPaymentGateway):
    def __init__(self, should_succeed: bool = True):
        self._should_succeed = should_succeed
        self.charged_amounts = []

    def charge(self, amount: float) -> bool:
        self.charged_amounts.append(amount)
        return self._should_succeed


# 유닛 테스트
def test_process_order_saves_to_database():
    mock_db = MockDatabase()
    mock_payment = MockPayment()
    service = OrderService(db=mock_db, payment=mock_payment)

    service.process_order({"id": 1, "amount": 50000})

    assert len(mock_db.saved_data) == 1
    assert mock_db.saved_data[0]["id"] == 1

def test_process_order_charges_correct_amount():
    mock_db = MockDatabase()
    mock_payment = MockPayment()
    service = OrderService(db=mock_db, payment=mock_payment)

    service.process_order({"id": 1, "amount": 50000})

    assert mock_payment.charged_amounts == [50000]

def test_process_order_handles_payment_failure():
    mock_db = MockDatabase()
    mock_payment = MockPayment(should_succeed=False)
    service = OrderService(db=mock_db, payment=mock_payment)

    # 결제 실패 시나리오를 쉽게 테스트 가능
    service.process_order({"id": 1, "amount": 50000})
    assert mock_payment.charged_amounts == [50000]
```

### C# - Mock을 이용한 테스트

```csharp
// Moq 라이브러리를 사용한 유닛 테스트
using Moq;
using Xunit;

public class OrderServiceTests
{
    [Fact]
    public void ProcessOrder_Should_SaveToDatabase()
    {
        // Arrange
        var mockDb = new Mock<IDatabase>();
        var mockPayment = new Mock<IPaymentGateway>();
        var mockNotification = new Mock<INotificationSender>();
        var service = new OrderService(
            mockDb.Object, mockNotification.Object, mockPayment.Object);

        var order = new Order { Id = 1, Amount = 50000m };

        // Act
        service.ProcessOrder(order);

        // Assert - DB에 저장되었는지 확인
        mockDb.Verify(db => db.Save(It.IsAny<Dictionary<string, object>>()), Times.Once);
    }

    [Fact]
    public void ProcessOrder_Should_ChargeCorrectAmount()
    {
        // Arrange
        var mockDb = new Mock<IDatabase>();
        var mockPayment = new Mock<IPaymentGateway>();
        var mockNotification = new Mock<INotificationSender>();
        mockPayment.Setup(p => p.Charge(It.IsAny<decimal>())).Returns(true);

        var service = new OrderService(
            mockDb.Object, mockNotification.Object, mockPayment.Object);

        // Act
        service.ProcessOrder(new Order { Id = 1, Amount = 30000m });

        // Assert - 정확한 금액이 청구되었는지 확인
        mockPayment.Verify(p => p.Charge(30000m), Times.Once);
    }
}
```

---

## 10. 실무에서 결합도 측정하기

### 결합도 자가 진단 체크리스트

다음 질문에 "예"가 많을수록 결합도가 높습니다.

| # | 질문 | 예/아니오 |
|---|------|----------|
| 1 | 클래스 내부에서 `new` 키워드로 의존 객체를 직접 생성하는가? | |
| 2 | 구체 클래스 타입을 직접 참조하는가? (인터페이스 대신) | |
| 3 | 하나의 클래스를 변경하면 다른 클래스도 반드시 변경해야 하는가? | |
| 4 | 클래스를 단독으로 유닛 테스트할 수 없는가? | |
| 5 | 전역 변수나 공유 상태를 사용하는가? | |
| 6 | 다른 클래스의 private/내부 필드에 접근하는가? | |
| 7 | import/using 문이 10개 이상인가? | |

### 코드 메트릭

```
결합도 관련 주요 메트릭:

1. Afferent Coupling (Ca): 이 모듈에 의존하는 외부 모듈 수
   - 높으면: 변경 시 영향 범위가 큼 (신중하게 변경)

2. Efferent Coupling (Ce): 이 모듈이 의존하는 외부 모듈 수
   - 높으면: 외부 변경에 취약

3. Instability (I) = Ce / (Ca + Ce)
   - 0에 가까움: 안정적 (변경하기 어려움)
   - 1에 가까움: 불안정 (변경하기 쉬움)
```

### IDE 도구 활용

- **Python**: `pylint`, `radon`, `import-linter`
- **C#**: Visual Studio의 "Code Metrics", `NDepend`
- **공통**: SonarQube의 결합도 분석

---

## 11. 정리 및 체크리스트

### 핵심 요약

```
┌───────────────────────────────────────────┐
│           느슨한 결합 핵심 원칙              │
│                                           │
│  1. 구체 클래스가 아닌 인터페이스에 의존하라   │
│  2. 객체 생성을 외부에 위임하라 (DI)         │
│  3. 모듈 간 직접 호출보다 간접 통신을 선호하라 │
│  4. "변경의 파급 효과"를 항상 고려하라        │
└───────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] 클래스가 인터페이스/추상 클래스에 의존하는가?
- [ ] 의존성을 생성자 주입으로 받고 있는가?
- [ ] 클래스를 Mock으로 교체하여 단독 테스트할 수 있는가?
- [ ] 구현체 교체 시 해당 클래스의 코드 수정이 불필요한가?
- [ ] 전역 상태 공유를 피하고 있는가?
- [ ] 이벤트/메시지 기반으로 모듈 간 통신을 하고 있는가?
- [ ] import/using 문의 수가 적절한가?

### 흔한 실수

| 실수 | 해결 방법 |
|------|----------|
| `new`로 직접 생성 | DI 컨테이너 사용 |
| 구체 타입 매개변수 | 인터페이스 타입으로 변경 |
| 전역 변수 남용 | 의존성 주입으로 전달 |
| 순환 의존성 | 인터페이스 분리, 이벤트 도입 |

---

## 12. 관련 개념 연결

| 관련 개념 | 관계 |
|-----------|------|
| [높은 응집도](./02-high-cohesion.md) | 결합도와 응집도는 반비례 관계. 응집도를 높이면 자연스럽게 결합도가 낮아짐 |
| [관심사의 분리](./03-separation-of-concerns.md) | 관심사를 분리하면 모듈 간 결합도가 낮아짐 |
| [의존성 주입](./04-dependency-injection.md) | 느슨한 결합을 달성하는 핵심 기법 |
| SOLID - DIP | 의존성 역전 원칙은 느슨한 결합의 이론적 기반 |
| SOLID - OCP | 느슨한 결합이 되어야 확장에 열리고 수정에 닫힌 구조 가능 |
| 전략 패턴 | 인터페이스를 통한 알고리즘 교체로 느슨한 결합 구현 |
| 옵저버 패턴 | 이벤트 기반 통신으로 발행자-구독자 간 느슨한 결합 |

---

> **다음 문서**: [높은 응집도 (High Cohesion)](./02-high-cohesion.md)
