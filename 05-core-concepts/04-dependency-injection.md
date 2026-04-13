# 의존성 주입 (Dependency Injection, DI)

> **"자동차 공장이 엔진을 직접 만들지 않고 전문 엔진 공급업체에서 납품받듯, 객체도 필요한 의존성을 외부에서 주입받아야 한다."**

## 목차

1. [정의와 개념](#1-정의와-개념)
2. [비유로 이해하기](#2-비유로-이해하기)
3. [DI의 3가지 방식](#3-di의-3가지-방식)
4. [나쁜 예제: DI 없는 코드](#4-나쁜-예제-di-없는-코드)
5. [좋은 예제: DI 적용 코드](#5-좋은-예제-di-적용-코드)
6. [IoC 컨테이너 소개](#6-ioc-컨테이너-소개)
7. [Service Lifetime 개념](#7-service-lifetime-개념)
8. [Python 코드 예제](#8-python-코드-예제)
9. [C# 코드 예제](#9-c-코드-예제)
10. [DI와 테스트: Mock 객체 주입](#10-di와-테스트-mock-객체-주입)
11. [DIP와 DI의 관계](#11-dip와-di의-관계)
12. [실무 팁](#12-실무-팁)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)
14. [관련 개념 연결](#14-관련-개념-연결)

---

## 1. 정의와 개념

**의존성 주입(Dependency Injection, DI)** 이란 객체가 필요로 하는 의존성(다른 객체)을 스스로 생성하지 않고, 외부에서 주입(전달)받는 디자인 패턴입니다.

### 핵심 아이디어

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│   DI 없이 (직접 생성)          DI 적용 (외부에서 주입)    │
│                                                        │
│   class Car:                   class Car:               │
│     def __init__(self):          def __init__(self,     │
│       self.engine = V8Engine()     engine: IEngine):    │
│       # 직접 생성!                   self.engine = engine│
│       # V8에 종속됨                  # 어떤 엔진이든 OK!  │
│                                                        │
│   car = Car()                  car = Car(ElectricEngine())│
│   # V8 엔진만 가능              # 원하는 엔진을 주입     │
└────────────────────────────────────────────────────────┘
```

### 왜 중요한가?

| 측면 | DI 없이 | DI 적용 |
|------|---------|---------|
| 유연성 | 구체 클래스에 종속 | 인터페이스만 충족하면 교체 가능 |
| 테스트 | 실제 객체가 있어야 테스트 | Mock 주입으로 단위 테스트 가능 |
| 재사용 | 특정 구현에 묶여 재사용 어려움 | 다양한 조합으로 재사용 가능 |
| 유지보수 | 변경 시 사용하는 곳 모두 수정 | 주입 지점만 변경 |
| 결합도 | 높음 (구체 클래스 의존) | 낮음 (인터페이스 의존) |

---

## 2. 비유로 이해하기

### 자동차 공장 비유

```
┌──────────────────────────────────────────────────────────────┐
│              ❌ DI 없는 자동차 공장                             │
│                                                              │
│   ┌──────────────────────────────────────────┐               │
│   │            자동차 공장                      │               │
│   │                                          │               │
│   │  ┌──────────┐  ┌──────────┐  ┌────────┐ │               │
│   │  │ 엔진 제조 │  │ 타이어 제조│  │ 시트 제조│ │               │
│   │  │ 라인     │  │ 라인     │  │ 라인   │ │               │
│   │  └──────────┘  └──────────┘  └────────┘ │               │
│   │                                          │               │
│   │  -> 모든 부품을 직접 제조 (비효율적)         │               │
│   │  -> 엔진 종류를 바꾸려면 공장 전체 수정      │               │
│   └──────────────────────────────────────────┘               │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│              ✅ DI가 적용된 자동차 공장                         │
│                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│   │ 엔진     │  │ 타이어   │  │ 시트     │  <- 외부 공급업체   │
│   │ 공급업체  │  │ 공급업체  │  │ 공급업체  │                    │
│   └─────┬────┘  └─────┬────┘  └─────┬────┘                  │
│         │             │             │                        │
│         ▼ 납품        ▼ 납품        ▼ 납품 (= 주입)           │
│   ┌──────────────────────────────────────────┐               │
│   │            자동차 공장                      │               │
│   │                                          │               │
│   │  -> 부품을 조립만 함 (효율적)               │               │
│   │  -> 엔진 공급업체만 바꾸면 됨               │               │
│   └──────────────────────────────────────────┘               │
└──────────────────────────────────────────────────────────────┘
```

### 커피 주문 비유

- **DI 없이**: 커피숍에서 원두 재배부터 로스팅, 추출까지 전부 직접 함
- **DI 적용**: 원두 공급업체에서 원두를 납품받아(주입) 추출만 집중

---

## 3. DI의 3가지 방식

### 한눈에 비교

```
┌─────────────────────────────────────────────────────────────┐
│                   DI의 3가지 방식 비교                        │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐│
│  │ 생성자 주입       │  │ 메서드 주입      │  │ 속성 주입     ││
│  │ (Constructor)    │  │ (Method)        │  │ (Property)   ││
│  │                  │  │                 │  │              ││
│  │ class A:         │  │ class A:        │  │ class A:     ││
│  │   def __init__   │  │   def do_work   │  │   service =  ││
│  │     (self, dep): │  │     (self, dep):│  │     None     ││
│  │     self.dep=dep │  │     dep.run()   │  │              ││
│  │                  │  │                 │  │ a.service =  ││
│  │                  │  │                 │  │   MyService()││
│  ├──────────────────┤  ├─────────────────┤  ├──────────────┤│
│  │ ✅ 권장!         │  │ 선택적 의존성에   │  │ 선택적이지만  ││
│  │ 필수 의존성 보장   │  │ 적합            │  │ 불완전 상태   ││
│  │ 불변성 보장       │  │ 호출마다 다른    │  │ 가능성 있음   ││
│  │                  │  │ 의존성 전달 가능  │  │              ││
│  └─────────────────┘  └─────────────────┘  └──────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 3.1 생성자 주입 (Constructor Injection) - 권장

객체 생성 시 생성자를 통해 의존성을 전달합니다.

```python
# Python - 생성자 주입
class OrderService:
    def __init__(self, repository: IOrderRepository, payment: IPaymentGateway):
        self._repository = repository  # 필수 의존성: 생성 시 반드시 전달
        self._payment = payment        # 필수 의존성: 생성 시 반드시 전달

    def place_order(self, order: Order) -> int:
        order_id = self._repository.save(order)
        self._payment.charge(order.amount)
        return order_id
```

```csharp
// C# - 생성자 주입
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IPaymentGateway _payment;

    // 생성자에서 의존성을 받아 readonly 필드에 저장
    public OrderService(IOrderRepository repository, IPaymentGateway payment)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _payment = payment ?? throw new ArgumentNullException(nameof(payment));
    }

    public int PlaceOrder(Order order)
    {
        var orderId = _repository.Save(order);
        _payment.Charge(order.Amount);
        return orderId;
    }
}
```

**장점**:
- 필수 의존성을 보장 (없으면 객체 생성 불가)
- 불변성 확보 (`readonly`/생성 후 변경 불가)
- 객체가 항상 유효한 상태로 생성됨

### 3.2 메서드 주입 (Method Injection)

특정 메서드 호출 시 의존성을 매개변수로 전달합니다.

```python
# Python - 메서드 주입
class ReportGenerator:
    def generate(self, data: list, formatter: IReportFormatter) -> str:
        """매 호출마다 다른 포맷터를 사용할 수 있음"""
        return formatter.format(data)

# 사용: 호출마다 다른 의존성 전달
generator = ReportGenerator()
html_report = generator.generate(data, HtmlFormatter())
pdf_report = generator.generate(data, PdfFormatter())
csv_report = generator.generate(data, CsvFormatter())
```

```csharp
// C# - 메서드 주입
public class ReportGenerator
{
    public string Generate(List<ReportData> data, IReportFormatter formatter)
    {
        return formatter.Format(data);
    }
}

// 사용
var generator = new ReportGenerator();
var html = generator.Generate(data, new HtmlFormatter());
var pdf = generator.Generate(data, new PdfFormatter());
```

**적합한 경우**:
- 호출마다 다른 의존성이 필요할 때
- 의존성이 선택적일 때
- 전략 패턴 구현 시

### 3.3 속성 주입 (Property/Setter Injection)

속성(setter)을 통해 의존성을 나중에 설정합니다.

```python
# Python - 속성 주입
class NotificationService:
    def __init__(self):
        self.logger: ILogger | None = None  # 선택적 의존성

    def notify(self, message: str) -> None:
        if self.logger:
            self.logger.log(f"알림 전송: {message}")
        print(f"알림: {message}")

# 사용
service = NotificationService()
service.logger = FileLogger()  # 나중에 주입
service.notify("안녕하세요")
```

```csharp
// C# - 속성 주입
public class NotificationService
{
    // 선택적 의존성 (없어도 동작 가능)
    public ILogger? Logger { get; set; }

    public void Notify(string message)
    {
        Logger?.Log($"알림 전송: {message}");
        Console.WriteLine($"알림: {message}");
    }
}

// 사용
var service = new NotificationService();
service.Logger = new FileLogger();  // 나중에 주입
```

**주의사항**:
- 객체가 불완전한 상태로 사용될 수 있음
- null 체크가 필요함
- 선택적(optional) 의존성에만 사용 권장

---

## 4. 나쁜 예제: DI 없는 코드

### Python - DI 없는 예제

```python
# ❌ 나쁜 예: 클래스 내부에서 의존성을 직접 생성

class MySqlDatabase:
    def __init__(self):
        self.host = "localhost"
        self.port = 3306
        print(f"[MySQL] {self.host}:{self.port}에 연결")

    def query(self, sql: str) -> list:
        print(f"[MySQL] 쿼리 실행: {sql}")
        return [{"id": 1, "name": "상품A"}]

    def execute(self, sql: str, params: tuple) -> int:
        print(f"[MySQL] 실행: {sql}")
        return 1


class StripePaymentProcessor:
    def __init__(self):
        self.api_key = "sk_test_..."
        print("[Stripe] API 키 설정 완료")

    def charge(self, amount: float, card_token: str) -> dict:
        print(f"[Stripe] {amount}원 결제")
        return {"status": "success", "transaction_id": "txn_123"}


class SmtpEmailSender:
    def __init__(self):
        self.smtp_host = "smtp.gmail.com"
        self.smtp_port = 587
        print(f"[SMTP] {self.smtp_host}:{self.smtp_port} 설정")

    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[SMTP] {to}에게 '{subject}' 전송")


class OrderService:
    """❌ 모든 의존성을 내부에서 직접 생성하는 서비스"""

    def __init__(self):
        # 구체 클래스를 직접 생성! (강한 결합)
        self.db = MySqlDatabase()              # MySQL에 종속
        self.payment = StripePaymentProcessor() # Stripe에 종속
        self.email = SmtpEmailSender()          # SMTP에 종속

    def create_order(self, customer_email: str, items: list, card_token: str) -> dict:
        # DB에 직접 종속된 쿼리
        products = self.db.query(
            f"SELECT * FROM products WHERE id IN ({','.join(str(i['id']) for i in items)})"
        )

        # 가격 계산
        total = sum(p["price"] * p["quantity"] for p in items)

        # 주문 저장 (MySQL SQL 직접 사용)
        order_id = self.db.execute(
            "INSERT INTO orders (email, total, status) VALUES (?, ?, ?)",
            (customer_email, total, "pending")
        )

        # Stripe 결제 (Stripe API 직접 사용)
        payment_result = self.payment.charge(total, card_token)

        # SMTP 이메일 (SMTP 직접 사용)
        self.email.send(
            customer_email,
            "주문 확인",
            f"주문 #{order_id}이 처리되었습니다. 금액: {total}원"
        )

        return {"order_id": order_id, "total": total, "payment": payment_result}


# 사용
service = OrderService()  # 생성만 해도 MySQL, Stripe, SMTP 모두 초기화됨!

# 문제점:
# 1. DB를 PostgreSQL로 바꾸려면? -> OrderService 코드 수정 필요
# 2. Stripe 대신 PayPal을 쓰려면? -> OrderService 코드 수정 필요
# 3. 유닛 테스트를 하려면? -> 실제 MySQL, Stripe, SMTP 서버 필요!
# 4. 테스트에서 결제를 시뮬레이션할 수 없음
```

### C# - DI 없는 예제

```csharp
// ❌ 나쁜 예: 내부에서 직접 생성
public class OrderService
{
    // 구체 클래스에 직접 의존!
    private readonly MySqlDatabase _db = new MySqlDatabase();
    private readonly StripePaymentProcessor _payment = new StripePaymentProcessor();
    private readonly SmtpEmailSender _email = new SmtpEmailSender();

    public OrderResult CreateOrder(string email, List<OrderItem> items, string cardToken)
    {
        // MySQL에 종속된 쿼리
        var products = _db.Query("SELECT * FROM products WHERE ...");

        // Stripe에 종속된 결제
        var paymentResult = _payment.Charge(total, cardToken);

        // SMTP에 종속된 이메일
        _email.Send(email, "주문 확인", $"주문이 처리되었습니다.");

        return new OrderResult(orderId, total);
    }
}

// 테스트 불가능!
// var service = new OrderService(); <- MySQL, Stripe, SMTP 모두 필요
```

---

## 5. 좋은 예제: DI 적용 코드

### 구조 비교

```
❌ DI 없이                          ✅ DI 적용

OrderService                       OrderService
  │ new                               │ 인터페이스 의존
  ├── MySqlDatabase                   ├── IDatabase
  ├── StripePayment                   ├── IPaymentGateway
  └── SmtpEmail                       └── INotification

  (내부에서 생성)                      (외부에서 주입)
  (교체 불가)                         (자유롭게 교체)
  (테스트 불가)                       (Mock 주입으로 테스트)
```

### Python - DI 적용 예제

```python
# ✅ 좋은 예: 의존성 주입 적용
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional


# --- 인터페이스 정의 ---
class IDatabase(ABC):
    @abstractmethod
    def query(self, sql: str) -> list: ...

    @abstractmethod
    def execute(self, sql: str, params: tuple) -> int: ...

class IPaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, token: str) -> dict: ...

class INotificationService(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> None: ...

class ILogger(ABC):
    @abstractmethod
    def info(self, message: str) -> None: ...

    @abstractmethod
    def error(self, message: str) -> None: ...


# --- 구현체들 ---
class MySqlDatabase(IDatabase):
    def __init__(self, connection_string: str):
        self._conn_str = connection_string

    def query(self, sql: str) -> list:
        print(f"[MySQL] 쿼리: {sql}")
        return []

    def execute(self, sql: str, params: tuple) -> int:
        print(f"[MySQL] 실행: {sql}")
        return 1

class PostgresDatabase(IDatabase):
    def __init__(self, connection_string: str):
        self._conn_str = connection_string

    def query(self, sql: str) -> list:
        print(f"[PostgreSQL] 쿼리: {sql}")
        return []

    def execute(self, sql: str, params: tuple) -> int:
        print(f"[PostgreSQL] 실행: {sql}")
        return 1

class StripePayment(IPaymentGateway):
    def __init__(self, api_key: str):
        self._api_key = api_key

    def charge(self, amount: float, token: str) -> dict:
        print(f"[Stripe] {amount}원 결제")
        return {"status": "success", "id": "txn_123"}

class PayPalPayment(IPaymentGateway):
    def __init__(self, client_id: str, secret: str):
        self._client_id = client_id
        self._secret = secret

    def charge(self, amount: float, token: str) -> dict:
        print(f"[PayPal] {amount}원 결제")
        return {"status": "success", "id": "pp_456"}

class EmailNotification(INotificationService):
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[Email] {to}에게 '{subject}' 전송")

class SmsNotification(INotificationService):
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"[SMS] {to}에게 메시지 전송")

class ConsoleLogger(ILogger):
    def info(self, message: str) -> None:
        print(f"[INFO] {message}")

    def error(self, message: str) -> None:
        print(f"[ERROR] {message}")


# --- 핵심 서비스: 인터페이스에만 의존, 의존성은 외부에서 주입 ---
class OrderService:
    """✅ 모든 의존성을 생성자를 통해 주입받는 서비스"""

    def __init__(
        self,
        db: IDatabase,                     # 어떤 DB든 가능
        payment: IPaymentGateway,           # 어떤 결제수단이든 가능
        notification: INotificationService, # 어떤 알림이든 가능
        logger: ILogger                     # 어떤 로거든 가능
    ):
        self._db = db
        self._payment = payment
        self._notification = notification
        self._logger = logger

    def create_order(self, customer_email: str, items: list, card_token: str) -> dict:
        self._logger.info(f"주문 처리 시작: {customer_email}")

        try:
            # DB 종류를 몰라도 됨
            total = sum(item["price"] * item["quantity"] for item in items)
            order_id = self._db.execute(
                "INSERT INTO orders (email, total) VALUES (?, ?)",
                (customer_email, total)
            )

            # 결제 수단을 몰라도 됨
            payment_result = self._payment.charge(total, card_token)

            # 알림 방식을 몰라도 됨
            self._notification.send(
                customer_email,
                "주문 확인",
                f"주문 #{order_id}, 금액: {total}원"
            )

            self._logger.info(f"주문 완료: #{order_id}")
            return {"order_id": order_id, "total": total, "payment": payment_result}

        except Exception as e:
            self._logger.error(f"주문 실패: {e}")
            raise


# --- 사용: 원하는 조합으로 자유롭게 구성 ---
if __name__ == "__main__":
    print("=== 설정 1: MySQL + Stripe + Email ===")
    service_v1 = OrderService(
        db=MySqlDatabase("mysql://localhost/shop"),
        payment=StripePayment("sk_test_123"),
        notification=EmailNotification(),
        logger=ConsoleLogger()
    )
    service_v1.create_order("user@test.com", [{"price": 10000, "quantity": 2}], "tok_visa")

    print("\n=== 설정 2: PostgreSQL + PayPal + SMS ===")
    service_v2 = OrderService(
        db=PostgresDatabase("postgresql://localhost/shop"),
        payment=PayPalPayment("client_id", "secret"),
        notification=SmsNotification(),
        logger=ConsoleLogger()
    )
    service_v2.create_order("user@test.com", [{"price": 10000, "quantity": 2}], "pp_tok")

    # OrderService 코드는 한 줄도 변경하지 않았음!
```

### C# - DI 적용 예제

```csharp
// ✅ 좋은 예: 의존성 주입 적용

// --- 인터페이스 ---
public interface IDatabase
{
    List<Dictionary<string, object>> Query(string sql);
    int Execute(string sql, params object[] parameters);
}

public interface IPaymentGateway
{
    PaymentResult Charge(decimal amount, string token);
}

public interface INotificationService
{
    void Send(string to, string subject, string body);
}

public interface ILogger
{
    void Info(string message);
    void Error(string message);
}

// --- 핵심 서비스 ---
public class OrderService
{
    private readonly IDatabase _db;
    private readonly IPaymentGateway _payment;
    private readonly INotificationService _notification;
    private readonly ILogger _logger;

    // 생성자 주입: 모든 의존성을 외부에서 받음
    public OrderService(
        IDatabase db,
        IPaymentGateway payment,
        INotificationService notification,
        ILogger logger)
    {
        _db = db ?? throw new ArgumentNullException(nameof(db));
        _payment = payment ?? throw new ArgumentNullException(nameof(payment));
        _notification = notification ?? throw new ArgumentNullException(nameof(notification));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public OrderResult CreateOrder(string email, List<OrderItem> items, string cardToken)
    {
        _logger.Info($"주문 처리 시작: {email}");

        try
        {
            var total = items.Sum(i => i.Price * i.Quantity);
            var orderId = _db.Execute(
                "INSERT INTO orders (email, total) VALUES (@p0, @p1)", email, total);

            var paymentResult = _payment.Charge(total, cardToken);

            _notification.Send(email, "주문 확인",
                $"주문 #{orderId}, 금액: {total}원");

            _logger.Info($"주문 완료: #{orderId}");
            return new OrderResult(orderId, total, paymentResult);
        }
        catch (Exception ex)
        {
            _logger.Error($"주문 실패: {ex.Message}");
            throw;
        }
    }
}
```

---

## 6. IoC 컨테이너 소개

**IoC(Inversion of Control) 컨테이너**는 의존성의 등록, 해석, 생성을 자동으로 관리해주는 프레임워크입니다.

### 수동 DI vs IoC 컨테이너

```
수동 DI (Poor Man's DI):
  모든 의존성을 직접 생성하고 주입

  db = MySqlDatabase(conn_str)
  payment = StripePayment(api_key)
  notification = EmailNotification()
  logger = ConsoleLogger()
  service = OrderService(db, payment, notification, logger)

IoC 컨테이너:
  의존성을 등록하면 자동으로 해석하고 주입

  container.register(IDatabase, MySqlDatabase)
  container.register(IPaymentGateway, StripePayment)
  service = container.resolve(OrderService)  # 자동으로 의존성 주입!
```

### Python: dependency-injector 라이브러리

```python
# pip install dependency-injector
from dependency_injector import containers, providers
from dependency_injector.wiring import Provide, inject


# --- 인터페이스와 구현체 (앞에서 정의한 것 사용) ---

# --- DI 컨테이너 정의 ---
class Container(containers.DeclarativeContainer):
    """애플리케이션 DI 컨테이너"""

    # 설정
    config = providers.Configuration()

    # 인프라 계층
    database = providers.Singleton(
        MySqlDatabase,
        connection_string=config.db.connection_string
    )

    payment_gateway = providers.Factory(
        StripePayment,
        api_key=config.payment.api_key
    )

    notification_service = providers.Factory(
        EmailNotification
    )

    logger = providers.Singleton(
        ConsoleLogger
    )

    # 애플리케이션 계층
    order_service = providers.Factory(
        OrderService,
        db=database,
        payment=payment_gateway,
        notification=notification_service,
        logger=logger
    )


# --- 사용 예시 ---
if __name__ == "__main__":
    # 컨테이너 생성 및 설정
    container = Container()
    container.config.db.connection_string.from_value("mysql://localhost/shop")
    container.config.payment.api_key.from_value("sk_test_123")

    # 컨테이너에서 서비스 가져오기 (의존성 자동 주입!)
    order_service = container.order_service()
    order_service.create_order(
        "user@test.com",
        [{"price": 10000, "quantity": 2}],
        "tok_visa"
    )

    # 테스트 시: 특정 의존성만 오버라이드
    with container.database.override(MockDatabase()):
        test_service = container.order_service()
        # MockDatabase가 주입된 OrderService로 테스트!
```

### C#: Microsoft.Extensions.DependencyInjection

```csharp
using Microsoft.Extensions.DependencyInjection;

// --- DI 컨테이너 설정 ---
public class Program
{
    public static void Main()
    {
        // 서비스 등록
        var services = new ServiceCollection();

        // 인프라 계층 등록
        services.AddSingleton<IDatabase>(sp =>
            new MySqlDatabase("Server=localhost;Database=shop"));
        services.AddTransient<IPaymentGateway>(sp =>
            new StripePayment("sk_test_123"));
        services.AddTransient<INotificationService, EmailNotification>();
        services.AddSingleton<ILogger, ConsoleLogger>();

        // 애플리케이션 계층 등록
        services.AddTransient<OrderService>();

        // 서비스 프로바이더 빌드
        var serviceProvider = services.BuildServiceProvider();

        // 서비스 해석 (의존성 자동 주입!)
        var orderService = serviceProvider.GetRequiredService<OrderService>();

        orderService.CreateOrder(
            "user@test.com",
            new List<OrderItem> { new(1, "상품A", 2, 10000m) },
            "tok_visa"
        );
    }
}

// ASP.NET Core에서의 DI 설정
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // 한 곳에서 모든 의존성을 등록 (Composition Root)
        services.AddSingleton<IDatabase, MySqlDatabase>();
        services.AddTransient<IPaymentGateway, StripePayment>();
        services.AddTransient<INotificationService, EmailNotification>();
        services.AddSingleton<ILogger, ConsoleLogger>();
        services.AddScoped<OrderService>();

        // 컨트롤러에서는 생성자 주입으로 자동 주입됨
    }
}

// 컨트롤러 (ASP.NET Core)
[ApiController]
[Route("api/orders")]
public class OrderController : ControllerBase
{
    private readonly OrderService _orderService;

    // ASP.NET Core가 자동으로 OrderService를 주입!
    public OrderController(OrderService orderService)
    {
        _orderService = orderService;
    }

    [HttpPost]
    public IActionResult CreateOrder([FromBody] CreateOrderRequest request)
    {
        var result = _orderService.CreateOrder(
            request.Email, request.Items, request.CardToken);
        return Ok(result);
    }
}
```

---

## 7. Service Lifetime 개념

DI 컨테이너에 서비스를 등록할 때 **생명주기(Lifetime)** 를 지정합니다.

### 3가지 생명주기

```
┌──────────────────────────────────────────────────────────────┐
│                    Service Lifetime 비교                      │
│                                                              │
│  Transient (일회용)        Scoped (요청 단위)     Singleton    │
│                                                (앱 전체 하나)│
│  요청 1: [A1]             요청 1: [A1]           앱 시작: [A] │
│  요청 2: [A2]             요청 1: [A1] (같은 것)  요청 1: [A]  │
│  요청 3: [A3]             요청 2: [A2]           요청 2: [A]  │
│                           요청 2: [A2] (같은 것)  요청 3: [A]  │
│                                                              │
│  매번 새 인스턴스          요청 내에서 같은 인스턴스  항상 같은    │
│                           다른 요청은 다른 인스턴스  인스턴스     │
├──────────────────────────────────────────────────────────────┤
│  사용 시기:                                                   │
│                                                              │
│  Transient:  경량 서비스, 상태 없는 서비스                      │
│              예: Validator, Formatter, Calculator              │
│                                                              │
│  Scoped:     HTTP 요청 단위 공유, DB 컨텍스트                   │
│              예: DbContext, UnitOfWork, UserSession             │
│                                                              │
│  Singleton:  비용이 큰 초기화, 공유 리소스                       │
│              예: Logger, Configuration, Cache, HttpClient       │
└──────────────────────────────────────────────────────────────┘
```

### C# 예제

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Transient: 요청할 때마다 새 인스턴스 생성
    services.AddTransient<IOrderValidator, OrderValidator>();
    services.AddTransient<IPricingService, PricingService>();

    // Scoped: HTTP 요청 하나 내에서 같은 인스턴스 공유
    services.AddScoped<IDbContext, AppDbContext>();
    services.AddScoped<IUnitOfWork, UnitOfWork>();
    services.AddScoped<OrderService>();

    // Singleton: 애플리케이션 전체에서 하나의 인스턴스만 존재
    services.AddSingleton<ILogger, FileLogger>();
    services.AddSingleton<IConfiguration, AppConfiguration>();
    services.AddSingleton<ICache, RedisCache>();
}
```

### Python 예제 (dependency-injector)

```python
from dependency_injector import containers, providers

class Container(containers.DeclarativeContainer):
    # Singleton: 한 번만 생성, 계속 재사용
    logger = providers.Singleton(ConsoleLogger)
    config = providers.Singleton(AppConfiguration, path="config.yaml")

    # Factory (= Transient): 매번 새로 생성
    validator = providers.Factory(OrderValidator)
    pricing = providers.Factory(PricingService)

    # ThreadLocalSingleton (= Scoped에 가까움): 스레드 내에서 하나
    db_session = providers.ThreadLocalSingleton(DatabaseSession)
```

### 생명주기 주의사항

```
⚠️ 흔한 실수: Captive Dependency (포로 의존성)

Singleton이 Scoped/Transient 서비스에 의존하면 안 됨!

❌ 잘못된 예:
Singleton(CacheService) -> Scoped(DbContext)

문제: CacheService는 앱 전체에서 하나인데,
     DbContext는 요청마다 달라야 함.
     CacheService가 첫 번째 DbContext를 계속 들고 있음!

✅ 올바른 의존 방향:
Transient -> Scoped -> Singleton (왼쪽이 오른쪽에 의존 OK)
Singleton -> Scoped -> Transient (오른쪽이 왼쪽에 의존 NG!)
```

---

## 8. Python 코드 예제

### 종합 예제: 파일 처리 파이프라인

```python
"""
의존성 주입을 적용한 파일 처리 파이프라인
각 단계를 교체 가능하게 설계
"""
from abc import ABC, abstractmethod
from dataclasses import dataclass
from pathlib import Path
from typing import Protocol


# ============================================================
# 도메인 모델
# ============================================================
@dataclass
class FileInfo:
    name: str
    path: str
    size: int
    content: str = ""

@dataclass
class ProcessingResult:
    success: bool
    original_file: str
    output_path: str = ""
    message: str = ""
    error: str = ""


# ============================================================
# 인터페이스 (Protocol 기반)
# ============================================================
class FileReader(Protocol):
    """파일 읽기 인터페이스"""
    def read(self, path: str) -> FileInfo: ...

class ContentTransformer(Protocol):
    """콘텐츠 변환 인터페이스"""
    def transform(self, content: str) -> str: ...

class FileWriter(Protocol):
    """파일 쓰기 인터페이스"""
    def write(self, path: str, content: str) -> None: ...

class ProcessingNotifier(Protocol):
    """처리 결과 알림 인터페이스"""
    def notify(self, result: ProcessingResult) -> None: ...

class ProcessingLogger(Protocol):
    """로깅 인터페이스"""
    def log(self, level: str, message: str) -> None: ...


# ============================================================
# 구현체들 (자유롭게 교체 가능)
# ============================================================

# --- 파일 리더 구현체 ---
class LocalFileReader:
    """로컬 파일 시스템에서 읽기"""
    def read(self, path: str) -> FileInfo:
        print(f"[LocalFS] 파일 읽기: {path}")
        return FileInfo(
            name=Path(path).name,
            path=path,
            size=1024,
            content="Hello, World! This is sample content."
        )

class S3FileReader:
    """AWS S3에서 읽기"""
    def __init__(self, bucket: str, region: str = "ap-northeast-2"):
        self._bucket = bucket
        self._region = region

    def read(self, path: str) -> FileInfo:
        print(f"[S3] s3://{self._bucket}/{path} 에서 읽기")
        return FileInfo(name=Path(path).name, path=path, size=2048, content="S3 content")


# --- 콘텐츠 변환기 구현체 ---
class UpperCaseTransformer:
    """대문자 변환"""
    def transform(self, content: str) -> str:
        return content.upper()

class MarkdownToHtmlTransformer:
    """마크다운을 HTML로 변환"""
    def transform(self, content: str) -> str:
        html = content.replace("# ", "<h1>").replace("\n", "</h1>\n")
        return f"<html><body>{html}</body></html>"

class CompressingTransformer:
    """콘텐츠 압축 (시뮬레이션)"""
    def transform(self, content: str) -> str:
        return f"[COMPRESSED:{len(content)}bytes]{content[:50]}..."


# --- 파일 라이터 구현체 ---
class LocalFileWriter:
    """로컬 파일 시스템에 쓰기"""
    def write(self, path: str, content: str) -> None:
        print(f"[LocalFS] 파일 쓰기: {path} ({len(content)} bytes)")

class S3FileWriter:
    """AWS S3에 쓰기"""
    def __init__(self, bucket: str):
        self._bucket = bucket

    def write(self, path: str, content: str) -> None:
        print(f"[S3] s3://{self._bucket}/{path} 에 쓰기 ({len(content)} bytes)")


# --- 알림 구현체 ---
class SlackNotifier:
    """Slack 알림"""
    def __init__(self, webhook_url: str):
        self._webhook = webhook_url

    def notify(self, result: ProcessingResult) -> None:
        status = "성공" if result.success else "실패"
        print(f"[Slack] 파일 처리 {status}: {result.original_file}")

class EmailNotifier:
    """이메일 알림"""
    def __init__(self, recipients: list[str]):
        self._recipients = recipients

    def notify(self, result: ProcessingResult) -> None:
        status = "성공" if result.success else "실패"
        print(f"[Email] {self._recipients}에게 처리 {status} 알림")


# --- 로거 구현체 ---
class ConsoleLogger:
    def log(self, level: str, message: str) -> None:
        print(f"[{level.upper()}] {message}")


# ============================================================
# 핵심 서비스: 모든 의존성을 주입받음
# ============================================================
class FileProcessingPipeline:
    """
    파일 처리 파이프라인
    모든 구성 요소를 외부에서 주입받아 유연하게 구성 가능
    """

    def __init__(
        self,
        reader: FileReader,
        transformers: list[ContentTransformer],
        writer: FileWriter,
        notifier: ProcessingNotifier,
        logger: ProcessingLogger
    ):
        self._reader = reader
        self._transformers = transformers
        self._writer = writer
        self._notifier = notifier
        self._logger = logger

    def process(self, input_path: str, output_path: str) -> ProcessingResult:
        """파일 처리 파이프라인 실행"""
        self._logger.log("info", f"파이프라인 시작: {input_path}")

        try:
            # 1. 파일 읽기 (어떤 소스든 가능)
            file_info = self._reader.read(input_path)
            self._logger.log("info", f"파일 읽기 완료: {file_info.size} bytes")

            # 2. 변환 체인 적용 (변환기 조합 자유)
            content = file_info.content
            for transformer in self._transformers:
                content = transformer.transform(content)
                self._logger.log("info",
                    f"변환 적용: {type(transformer).__name__}")

            # 3. 결과 쓰기 (어떤 대상이든 가능)
            self._writer.write(output_path, content)
            self._logger.log("info", f"파일 쓰기 완료: {output_path}")

            result = ProcessingResult(
                success=True,
                original_file=input_path,
                output_path=output_path,
                message="처리 완료"
            )

        except Exception as e:
            self._logger.log("error", f"처리 실패: {e}")
            result = ProcessingResult(
                success=False,
                original_file=input_path,
                error=str(e)
            )

        # 4. 결과 알림 (어떤 채널이든 가능)
        self._notifier.notify(result)
        return result


# ============================================================
# 사용 예시: 다양한 구성
# ============================================================
if __name__ == "__main__":
    logger = ConsoleLogger()

    # 구성 1: 로컬 -> 대문자 변환 -> 로컬, Slack 알림
    print("=== Pipeline 1: Local to Local ===")
    pipeline_local = FileProcessingPipeline(
        reader=LocalFileReader(),
        transformers=[UpperCaseTransformer()],
        writer=LocalFileWriter(),
        notifier=SlackNotifier("https://hooks.slack.com/..."),
        logger=logger
    )
    pipeline_local.process("input.txt", "output.txt")

    # 구성 2: S3 -> Markdown변환 + 압축 -> S3, Email 알림
    print("\n=== Pipeline 2: S3 to S3 ===")
    pipeline_s3 = FileProcessingPipeline(
        reader=S3FileReader(bucket="my-input-bucket"),
        transformers=[
            MarkdownToHtmlTransformer(),  # 변환기 체인!
            CompressingTransformer()
        ],
        writer=S3FileWriter(bucket="my-output-bucket"),
        notifier=EmailNotifier(["admin@example.com"]),
        logger=logger
    )
    pipeline_s3.process("docs/readme.md", "output/readme.html")

    # FileProcessingPipeline 코드는 전혀 변경하지 않음!
    # 구성만 바꾸면 완전히 다른 파이프라인이 됨
```

---

## 9. C# 코드 예제

### 종합 예제: 파일 처리 파이프라인 + ASP.NET Core DI

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Extensions.DependencyInjection;

// ============================================================
// 도메인 모델
// ============================================================
public record FileInfo(string Name, string Path, long Size, string Content = "");
public record ProcessingResult(
    bool Success,
    string OriginalFile,
    string OutputPath = "",
    string Message = "",
    string Error = ""
);

// ============================================================
// 인터페이스
// ============================================================
public interface IFileReader
{
    FileInfo Read(string path);
}

public interface IContentTransformer
{
    string Transform(string content);
}

public interface IFileWriter
{
    void Write(string path, string content);
}

public interface IProcessingNotifier
{
    void Notify(ProcessingResult result);
}

public interface IProcessingLogger
{
    void Log(string level, string message);
}

// ============================================================
// 구현체들
// ============================================================
public class LocalFileReader : IFileReader
{
    public FileInfo Read(string path)
    {
        Console.WriteLine($"[LocalFS] 파일 읽기: {path}");
        return new FileInfo(
            System.IO.Path.GetFileName(path), path, 1024,
            "Hello, World! This is sample content.");
    }
}

public class S3FileReader : IFileReader
{
    private readonly string _bucket;

    public S3FileReader(string bucket)
    {
        _bucket = bucket;
    }

    public FileInfo Read(string path)
    {
        Console.WriteLine($"[S3] s3://{_bucket}/{path} 에서 읽기");
        return new FileInfo(System.IO.Path.GetFileName(path), path, 2048, "S3 content");
    }
}

public class UpperCaseTransformer : IContentTransformer
{
    public string Transform(string content) => content.ToUpper();
}

public class MarkdownToHtmlTransformer : IContentTransformer
{
    public string Transform(string content)
        => $"<html><body>{content.Replace("# ", "<h1>")}</body></html>";
}

public class LocalFileWriter : IFileWriter
{
    public void Write(string path, string content)
        => Console.WriteLine($"[LocalFS] 파일 쓰기: {path} ({content.Length} bytes)");
}

public class SlackNotifier : IProcessingNotifier
{
    private readonly string _webhookUrl;

    public SlackNotifier(string webhookUrl)
    {
        _webhookUrl = webhookUrl;
    }

    public void Notify(ProcessingResult result)
    {
        var status = result.Success ? "성공" : "실패";
        Console.WriteLine($"[Slack] 파일 처리 {status}: {result.OriginalFile}");
    }
}

public class ConsoleProcessingLogger : IProcessingLogger
{
    public void Log(string level, string message)
        => Console.WriteLine($"[{level.ToUpper()}] {message}");
}

// ============================================================
// 핵심 서비스
// ============================================================
public class FileProcessingPipeline
{
    private readonly IFileReader _reader;
    private readonly IEnumerable<IContentTransformer> _transformers;
    private readonly IFileWriter _writer;
    private readonly IProcessingNotifier _notifier;
    private readonly IProcessingLogger _logger;

    public FileProcessingPipeline(
        IFileReader reader,
        IEnumerable<IContentTransformer> transformers,
        IFileWriter writer,
        IProcessingNotifier notifier,
        IProcessingLogger logger)
    {
        _reader = reader;
        _transformers = transformers;
        _writer = writer;
        _notifier = notifier;
        _logger = logger;
    }

    public ProcessingResult Process(string inputPath, string outputPath)
    {
        _logger.Log("info", $"파이프라인 시작: {inputPath}");

        try
        {
            var fileInfo = _reader.Read(inputPath);
            _logger.Log("info", $"파일 읽기 완료: {fileInfo.Size} bytes");

            var content = fileInfo.Content;
            foreach (var transformer in _transformers)
            {
                content = transformer.Transform(content);
                _logger.Log("info", $"변환 적용: {transformer.GetType().Name}");
            }

            _writer.Write(outputPath, content);
            _logger.Log("info", $"파일 쓰기 완료: {outputPath}");

            var result = new ProcessingResult(true, inputPath, outputPath, "처리 완료");
            _notifier.Notify(result);
            return result;
        }
        catch (Exception ex)
        {
            _logger.Log("error", $"처리 실패: {ex.Message}");
            var result = new ProcessingResult(false, inputPath, Error: ex.Message);
            _notifier.Notify(result);
            return result;
        }
    }
}

// ============================================================
// DI 컨테이너를 이용한 구성
// ============================================================
public class Program
{
    public static void Main()
    {
        // --- 구성 1: 로컬 파일 처리 ---
        Console.WriteLine("=== Pipeline 1: Local to Local ===");
        var localServices = new ServiceCollection();
        localServices.AddTransient<IFileReader, LocalFileReader>();
        localServices.AddTransient<IContentTransformer, UpperCaseTransformer>();
        localServices.AddTransient<IFileWriter, LocalFileWriter>();
        localServices.AddTransient<IProcessingNotifier>(sp =>
            new SlackNotifier("https://hooks.slack.com/..."));
        localServices.AddSingleton<IProcessingLogger, ConsoleProcessingLogger>();
        localServices.AddTransient<FileProcessingPipeline>();

        var localProvider = localServices.BuildServiceProvider();
        var localPipeline = localProvider.GetRequiredService<FileProcessingPipeline>();
        localPipeline.Process("input.txt", "output.txt");

        // --- 구성 2: S3 파일 처리 ---
        Console.WriteLine("\n=== Pipeline 2: S3 to Local ===");
        var s3Services = new ServiceCollection();
        s3Services.AddTransient<IFileReader>(sp =>
            new S3FileReader("my-bucket"));
        // 여러 변환기 등록 (IEnumerable로 주입됨)
        s3Services.AddTransient<IContentTransformer, MarkdownToHtmlTransformer>();
        s3Services.AddTransient<IFileWriter, LocalFileWriter>();
        s3Services.AddTransient<IProcessingNotifier>(sp =>
            new SlackNotifier("https://hooks.slack.com/..."));
        s3Services.AddSingleton<IProcessingLogger, ConsoleProcessingLogger>();
        s3Services.AddTransient<FileProcessingPipeline>();

        var s3Provider = s3Services.BuildServiceProvider();
        var s3Pipeline = s3Provider.GetRequiredService<FileProcessingPipeline>();
        s3Pipeline.Process("docs/readme.md", "output/readme.html");
    }
}

// ============================================================
// ASP.NET Core에서의 DI 등록 예시
// ============================================================
/*
// Program.cs (ASP.NET Core 6+)
var builder = WebApplication.CreateBuilder(args);

// DI 등록
builder.Services.AddSingleton<IProcessingLogger, ConsoleProcessingLogger>();
builder.Services.AddTransient<IFileReader, LocalFileReader>();
builder.Services.AddTransient<IContentTransformer, UpperCaseTransformer>();
builder.Services.AddTransient<IFileWriter, LocalFileWriter>();
builder.Services.AddTransient<IProcessingNotifier>(sp =>
    new SlackNotifier(builder.Configuration["Slack:WebhookUrl"]));
builder.Services.AddScoped<FileProcessingPipeline>();

var app = builder.Build();

// 컨트롤러에서 자동으로 주입됨
[ApiController]
[Route("api/files")]
public class FileController : ControllerBase
{
    private readonly FileProcessingPipeline _pipeline;

    public FileController(FileProcessingPipeline pipeline)
    {
        _pipeline = pipeline;  // DI 컨테이너가 자동 주입
    }

    [HttpPost("process")]
    public IActionResult ProcessFile([FromBody] ProcessRequest request)
    {
        var result = _pipeline.Process(request.InputPath, request.OutputPath);
        return result.Success ? Ok(result) : BadRequest(result);
    }
}
*/
```

---

## 10. DI와 테스트: Mock 객체 주입

DI의 가장 강력한 이점 중 하나는 **테스트 용이성**입니다.

### Python - Mock을 이용한 테스트

```python
"""DI 덕분에 Mock 주입으로 쉽게 유닛 테스트 가능"""
import pytest


# --- Mock 구현체 ---
class MockFileReader:
    """테스트용 파일 리더"""
    def __init__(self, content: str = "test content"):
        self._content = content
        self.read_called = False
        self.last_path = ""

    def read(self, path: str) -> FileInfo:
        self.read_called = True
        self.last_path = path
        return FileInfo(name="test.txt", path=path, size=100, content=self._content)


class MockFileWriter:
    """테스트용 파일 라이터"""
    def __init__(self):
        self.written_files: dict[str, str] = {}

    def write(self, path: str, content: str) -> None:
        self.written_files[path] = content


class MockNotifier:
    """테스트용 알림"""
    def __init__(self):
        self.notifications: list[ProcessingResult] = []

    def notify(self, result: ProcessingResult) -> None:
        self.notifications.append(result)


class MockLogger:
    """테스트용 로거"""
    def __init__(self):
        self.logs: list[tuple[str, str]] = []

    def log(self, level: str, message: str) -> None:
        self.logs.append((level, message))


# --- 테스트 ---
class TestFileProcessingPipeline:
    def setup_method(self):
        """각 테스트 전에 Mock 객체들을 준비"""
        self.mock_reader = MockFileReader(content="hello world")
        self.mock_writer = MockFileWriter()
        self.mock_notifier = MockNotifier()
        self.mock_logger = MockLogger()

    def _create_pipeline(self, transformers=None):
        """테스트용 파이프라인 생성 (DI로 Mock 주입!)"""
        return FileProcessingPipeline(
            reader=self.mock_reader,
            transformers=transformers or [],
            writer=self.mock_writer,
            notifier=self.mock_notifier,
            logger=self.mock_logger
        )

    def test_successful_processing(self):
        """정상적인 파일 처리 테스트"""
        pipeline = self._create_pipeline([UpperCaseTransformer()])

        result = pipeline.process("input.txt", "output.txt")

        # 검증
        assert result.success is True
        assert result.original_file == "input.txt"
        assert result.output_path == "output.txt"

    def test_file_is_read_from_correct_path(self):
        """올바른 경로에서 파일을 읽는지 테스트"""
        pipeline = self._create_pipeline()

        pipeline.process("data/input.csv", "output.csv")

        assert self.mock_reader.read_called is True
        assert self.mock_reader.last_path == "data/input.csv"

    def test_transformed_content_is_written(self):
        """변환된 콘텐츠가 올바르게 기록되는지 테스트"""
        pipeline = self._create_pipeline([UpperCaseTransformer()])

        pipeline.process("input.txt", "output.txt")

        assert "output.txt" in self.mock_writer.written_files
        assert self.mock_writer.written_files["output.txt"] == "HELLO WORLD"

    def test_multiple_transformers_applied_in_order(self):
        """여러 변환기가 순서대로 적용되는지 테스트"""
        class AddPrefixTransformer:
            def transform(self, content: str) -> str:
                return f"PREFIX:{content}"

        pipeline = self._create_pipeline([
            UpperCaseTransformer(),     # 먼저 대문자로
            AddPrefixTransformer()      # 그 다음 접두사 추가
        ])

        pipeline.process("input.txt", "output.txt")

        assert self.mock_writer.written_files["output.txt"] == "PREFIX:HELLO WORLD"

    def test_notification_sent_on_success(self):
        """성공 시 알림이 전송되는지 테스트"""
        pipeline = self._create_pipeline()

        pipeline.process("input.txt", "output.txt")

        assert len(self.mock_notifier.notifications) == 1
        assert self.mock_notifier.notifications[0].success is True

    def test_notification_sent_on_failure(self):
        """실패 시에도 알림이 전송되는지 테스트"""
        class FailingReader:
            def read(self, path: str):
                raise IOError("파일을 읽을 수 없습니다")

        pipeline = FileProcessingPipeline(
            reader=FailingReader(),  # 실패하는 Mock 주입!
            transformers=[],
            writer=self.mock_writer,
            notifier=self.mock_notifier,
            logger=self.mock_logger
        )

        result = pipeline.process("missing.txt", "output.txt")

        assert result.success is False
        assert "파일을 읽을 수 없습니다" in result.error
        assert len(self.mock_notifier.notifications) == 1
```

### C# - Moq를 이용한 테스트

```csharp
using Moq;
using Xunit;
using System.Collections.Generic;

public class FileProcessingPipelineTests
{
    private readonly Mock<IFileReader> _mockReader;
    private readonly Mock<IFileWriter> _mockWriter;
    private readonly Mock<IProcessingNotifier> _mockNotifier;
    private readonly Mock<IProcessingLogger> _mockLogger;

    public FileProcessingPipelineTests()
    {
        _mockReader = new Mock<IFileReader>();
        _mockWriter = new Mock<IFileWriter>();
        _mockNotifier = new Mock<IProcessingNotifier>();
        _mockLogger = new Mock<IProcessingLogger>();

        // 기본 Mock 동작 설정
        _mockReader.Setup(r => r.Read(It.IsAny<string>()))
            .Returns(new FileInfo("test.txt", "input.txt", 100, "hello world"));
    }

    private FileProcessingPipeline CreatePipeline(
        IEnumerable<IContentTransformer>? transformers = null)
    {
        return new FileProcessingPipeline(
            _mockReader.Object,
            transformers ?? new List<IContentTransformer>(),
            _mockWriter.Object,
            _mockNotifier.Object,
            _mockLogger.Object
        );
    }

    [Fact]
    public void Process_SuccessfulProcessing_ReturnsSuccessResult()
    {
        var pipeline = CreatePipeline();

        var result = pipeline.Process("input.txt", "output.txt");

        Assert.True(result.Success);
        Assert.Equal("input.txt", result.OriginalFile);
        Assert.Equal("output.txt", result.OutputPath);
    }

    [Fact]
    public void Process_ReadsFromCorrectPath()
    {
        var pipeline = CreatePipeline();

        pipeline.Process("data/input.csv", "output.csv");

        _mockReader.Verify(r => r.Read("data/input.csv"), Times.Once);
    }

    [Fact]
    public void Process_WritesTransformedContent()
    {
        var pipeline = CreatePipeline(
            new List<IContentTransformer> { new UpperCaseTransformer() });

        pipeline.Process("input.txt", "output.txt");

        _mockWriter.Verify(w =>
            w.Write("output.txt", "HELLO WORLD"), Times.Once);
    }

    [Fact]
    public void Process_SendsNotificationOnSuccess()
    {
        var pipeline = CreatePipeline();

        pipeline.Process("input.txt", "output.txt");

        _mockNotifier.Verify(n =>
            n.Notify(It.Is<ProcessingResult>(r => r.Success)), Times.Once);
    }

    [Fact]
    public void Process_SendsNotificationOnFailure()
    {
        _mockReader.Setup(r => r.Read(It.IsAny<string>()))
            .Throws(new System.IO.IOException("파일 없음"));

        var pipeline = CreatePipeline();

        var result = pipeline.Process("missing.txt", "output.txt");

        Assert.False(result.Success);
        _mockNotifier.Verify(n =>
            n.Notify(It.Is<ProcessingResult>(r => !r.Success)), Times.Once);
    }
}
```

---

## 11. DIP와 DI의 관계

**DIP(Dependency Inversion Principle)** 와 **DI(Dependency Injection)** 는 서로 밀접하지만 다른 개념입니다.

```
┌──────────────────────────────────────────────────────────────┐
│                    DIP vs DI 비교                              │
│                                                              │
│  DIP (의존성 역전 원칙)              DI (의존성 주입)            │
│  ─────────────────                  ────────────             │
│  SOLID 원칙 중 하나                  디자인 패턴               │
│  "무엇을 의존해야 하는가"             "어떻게 의존성을 전달하는가"│
│  (추상화에 의존하라)                 (외부에서 주입하라)          │
│                                                              │
│  원칙(Principle)                    구현 기법(Technique)       │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                                                         │ │
│  │   DIP: 고수준 모듈은 저수준 모듈에 의존하면 안 된다.       │ │
│  │        둘 다 추상화에 의존해야 한다.                       │ │
│  │                                                         │ │
│  │   DI: DIP를 실현하는 구체적인 방법.                       │ │
│  │       인터페이스를 통해 의존성을 외부에서 주입.              │ │
│  │                                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  관계: DIP(원칙) --구현--> DI(기법) --도구--> IoC Container    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 관계 요약

```python
# 1. DIP 위반 (추상화에 의존하지 않음)
class OrderService:
    def __init__(self):
        self.db = MySqlDatabase()  # 구체 클래스에 의존

# 2. DIP 준수 (추상화에 의존)
class OrderService:
    def __init__(self):
        self.db: IDatabase = MySqlDatabase()  # 인터페이스 타입이지만...
        # 여전히 내부에서 생성! DI는 아님

# 3. DIP + DI 모두 적용 (추상화 의존 + 외부 주입)
class OrderService:
    def __init__(self, db: IDatabase):  # 인터페이스에 의존 + 외부에서 주입
        self.db = db
```

---

## 12. 실무 팁

### DI 적용 가이드

| 상황 | 권장 방식 |
|------|----------|
| 필수 의존성 | 생성자 주입 |
| 선택적 의존성 | 속성 주입 (기본값 제공) |
| 호출마다 다른 의존성 | 메서드 주입 |
| 많은 의존성 (4개 이상) | 파사드 패턴으로 그룹화 검토 |

### 흔한 실수와 해결

```python
# ❌ 실수 1: Service Locator 안티패턴
class OrderService:
    def __init__(self, container):
        # 컨테이너를 통째로 받아서 필요한 것을 꺼냄 (나쁨!)
        self.db = container.resolve(IDatabase)
        self.payment = container.resolve(IPaymentGateway)
        # 문제: 실제 의존성이 무엇인지 알 수 없음

# ✅ 해결: 명시적 생성자 주입
class OrderService:
    def __init__(self, db: IDatabase, payment: IPaymentGateway):
        # 필요한 의존성이 명확하게 드러남
        self.db = db
        self.payment = payment
```

```python
# ❌ 실수 2: 과도한 추상화
class IStringFormatter(ABC):
    @abstractmethod
    def format(self, text: str) -> str: ...

class UpperCaseFormatter(IStringFormatter):
    def format(self, text: str) -> str:
        return text.upper()

# 단순한 문자열 변환에 인터페이스는 과도함
# str.upper()를 직접 호출하는 것이 나음

# ✅ DI가 필요한 경우:
# - 외부 시스템과의 통신 (DB, API, 파일시스템)
# - 테스트에서 교체가 필요한 경우
# - 런타임에 구현체가 바뀔 수 있는 경우
```

```python
# ❌ 실수 3: 생성자에서 복잡한 로직 실행
class OrderService:
    def __init__(self, db: IDatabase):
        self.db = db
        self.db.connect()  # 생성자에서 연결! (나쁨)
        self._warm_cache()  # 생성자에서 캐시 워밍! (나쁨)

# ✅ 해결: 생성자는 할당만, 초기화는 별도로
class OrderService:
    def __init__(self, db: IDatabase):
        self.db = db  # 할당만

    def initialize(self):
        self.db.connect()  # 명시적 초기화 메서드
```

### 의존성 개수 기준

```
생성자 매개변수 수:

1~3개: 정상 범위
4~5개: 주의 - 클래스의 책임이 너무 많은 건 아닌지 확인
6개 이상: 위험 - 클래스 분리 또는 파사드 패턴 적용 고려

# 의존성이 너무 많을 때의 해결책
class OrderService:
    def __init__(self, db, payment, email, sms, logger, cache, validator, ...):
        # 7개 이상 -> 분리 필요!
        ...

# 해결: 관련 의존성을 그룹화
class NotificationGroup:
    def __init__(self, email: IEmail, sms: ISms):
        self.email = email
        self.sms = sms

class OrderService:
    def __init__(self, db: IDatabase, payment: IPayment, notifications: NotificationGroup):
        # 3개로 줄임
        ...
```

---

## 13. 정리 및 체크리스트

### 핵심 요약

```
┌───────────────────────────────────────────────────┐
│             의존성 주입 핵심 원칙                    │
│                                                   │
│  1. 구체 클래스가 아닌 인터페이스에 의존하라 (DIP)    │
│  2. 의존성을 내부에서 생성하지 말고 외부에서 주입하라  │
│  3. 필수 의존성은 생성자 주입을 사용하라              │
│  4. IoC 컨테이너로 의존성 관리를 자동화하라           │
│  5. 테스트에서는 Mock을 주입하여 단위 테스트하라      │
└───────────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] 클래스 내부에서 `new`로 의존 객체를 생성하고 있지 않은가?
- [ ] 모든 의존성이 인터페이스/추상 클래스 타입인가?
- [ ] 필수 의존성은 생성자를 통해 주입받고 있는가?
- [ ] 생성자 매개변수가 5개 이하인가?
- [ ] IoC 컨테이너를 사용하고 있는가?
- [ ] Service Lifetime이 적절하게 설정되어 있는가?
- [ ] Captive Dependency 문제가 없는가?
- [ ] Service Locator 패턴을 사용하고 있지 않은가?
- [ ] Mock 주입으로 유닛 테스트를 작성할 수 있는가?
- [ ] Composition Root에서 모든 의존성을 등록하고 있는가?

### DI 방식 선택 가이드

| 상황 | 추천 방식 | 이유 |
|------|----------|------|
| 서비스가 반드시 필요 | 생성자 주입 | 누락 방지, 불변성 보장 |
| 선택적 기능 (로깅 등) | 속성 주입 | 없어도 동작 가능 |
| 호출마다 다른 전략 | 메서드 주입 | 유연한 전략 교체 |
| 순환 의존성 해결 | 속성 주입 | 생성자 순환 회피 |

---

## 14. 관련 개념 연결

| 관련 개념 | 관계 |
|-----------|------|
| [느슨한 결합](./01-loose-coupling.md) | DI는 느슨한 결합을 달성하는 핵심 기법 |
| [높은 응집도](./02-high-cohesion.md) | DI로 분리된 작은 클래스들은 자연스럽게 높은 응집도를 가짐 |
| [관심사의 분리](./03-separation-of-concerns.md) | 분리된 관심사를 DI로 조립 |
| SOLID - DIP | 의존성 역전 원칙은 DI의 이론적 기반 |
| SOLID - OCP | DI를 통해 구현체 교체만으로 기능 확장 가능 |
| 전략 패턴 | 메서드 주입을 통한 알고리즘 교체 |
| 팩토리 패턴 | 복잡한 객체 생성을 캡슐화 |
| Composition Root | DI에서 모든 의존성을 등록하는 진입점 |

---

> **이전 문서**: [관심사의 분리 (Separation of Concerns)](./03-separation-of-concerns.md)
