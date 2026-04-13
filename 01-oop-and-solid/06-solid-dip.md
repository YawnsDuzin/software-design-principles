# D - 의존성 역전 원칙 (Dependency Inversion Principle)

> **SOLID 원칙 5/5**
> **대상 독자**: 2~3년차 개발자
> **핵심 문장**: "고수준 모듈은 저수준 모듈에 의존해서는 안 된다. 둘 다 추상화에 의존해야 한다." — Robert C. Martin

---

## 목차

1. [DIP란 무엇인가?](#1-dip란-무엇인가)
2. [왜 DIP가 중요한가?](#2-왜-dip가-중요한가)
3. [DIP 위반 사례 (Before)](#3-dip-위반-사례-before)
4. [DIP 적용 사례 (After)](#4-dip-적용-사례-after)
5. [DIP vs DI: 원칙과 기법의 차이](#5-dip-vs-di-원칙과-기법의-차이)
6. [DI 컨테이너 소개](#6-di-컨테이너-소개)
7. [실무 적용 가이드](#7-실무-적용-가이드)
8. [정리 및 체크리스트](#8-정리-및-체크리스트)
9. [SOLID 원칙 총정리](#9-solid-원칙-총정리)

---

## 1. DIP란 무엇인가?

### 정의

DIP에는 두 가지 규칙이 있습니다:

> **규칙 1**: 고수준 모듈(High-level Module)은 저수준 모듈(Low-level Module)에 의존해서는 안 된다. **둘 다 추상화(Abstraction)에 의존**해야 한다.
>
> **규칙 2**: 추상화는 세부 구현에 의존해서는 안 된다. **세부 구현이 추상화에 의존**해야 한다.

### 용어 정리

| 용어 | 설명 | 예시 |
|------|------|------|
| **고수준 모듈** | 비즈니스 로직, 정책을 담는 모듈 | OrderService, PaymentService |
| **저수준 모듈** | 기술적 세부 구현을 담는 모듈 | MySQLRepository, SmtpEmailSender |
| **추상화** | 인터페이스 또는 추상 클래스 | IOrderRepository, IEmailSender |

### 비유

**전기 콘센트**를 생각해 보세요:

- **고수준 (사용자 요구)**: "전기를 사용하고 싶다"
- **저수준 (구체적 발전 방식)**: 화력 발전, 수력 발전, 태양광 발전
- **추상화 (표준 인터페이스)**: 220V 콘센트

콘센트(추상화) 덕분에:
- 가전제품(고수준)은 어떤 발전소(저수준)에서 전기가 오는지 몰라도 됩니다
- 발전 방식이 바뀌어도 가전제품을 교체할 필요가 없습니다
- 새로운 가전제품이 나와도 콘센트 규격만 맞으면 사용 가능합니다

### 의존성 방향의 "역전"

```
❌ DIP 위반 (전통적 의존성 방향):
  고수준 모듈 → 저수준 모듈
  OrderService → MySQLOrderRepository

✅ DIP 준수 (의존성 역전):
  고수준 모듈 → 추상화 ← 저수준 모듈
  OrderService → IOrderRepository ← MySQLOrderRepository

"역전"의 의미:
  저수준 모듈이 고수준이 정의한 추상화에 의존하게 되었다!
  의존성의 방향이 뒤집혔다!
```

---

## 2. 왜 DIP가 중요한가?

### DIP를 지키면

- **기술 독립성**: 비즈니스 로직이 특정 기술(DB, 프레임워크)에 묶이지 않음
- **교체 용이**: MySQL에서 PostgreSQL로, SMTP에서 SendGrid로 쉽게 교체
- **테스트 용이**: 저수준 모듈을 Mock으로 교체하여 단위 테스트 가능
- **병렬 개발**: 인터페이스만 합의하면 고수준/저수준 팀이 독립 개발

### DIP를 무시하면

- **강한 결합**: 비즈니스 로직이 특정 DB, 특정 라이브러리에 종속
- **변경 전파**: 저수준 변경이 고수준까지 전파
- **테스트 불가**: 실제 DB, 실제 이메일 서버 없이 테스트 불가능
- **재사용 불가**: 다른 프로젝트에서 비즈니스 로직만 가져올 수 없음

---

## 3. DIP 위반 사례 (Before)

### 시나리오: 주문 처리 시스템

주문 서비스가 MySQL, SMTP, Stripe에 직접 의존하는 상황입니다.

### ❌ Python - 나쁜 예: 구현체에 직접 의존

```python
import mysql.connector
import smtplib
from email.mime.text import MIMEText
import stripe


class MySQLOrderRepository:
    """저수준 모듈: MySQL 데이터베이스 접근"""

    def __init__(self):
        self.connection = mysql.connector.connect(
            host="localhost",
            user="root",
            password="password",
            database="shop"
        )

    def save(self, order: dict) -> int:
        cursor = self.connection.cursor()
        cursor.execute(
            "INSERT INTO orders (product, amount) VALUES (%s, %s)",
            (order["product"], order["amount"])
        )
        self.connection.commit()
        return cursor.lastrowid

    def find_by_id(self, order_id: int) -> dict:
        cursor = self.connection.cursor(dictionary=True)
        cursor.execute("SELECT * FROM orders WHERE id = %s", (order_id,))
        return cursor.fetchone()


class SmtpEmailSender:
    """저수준 모듈: SMTP 이메일 발송"""

    def send(self, to: str, subject: str, body: str) -> None:
        msg = MIMEText(body)
        msg["Subject"] = subject
        msg["From"] = "shop@example.com"
        msg["To"] = to

        with smtplib.SMTP("smtp.example.com", 587) as server:
            server.starttls()
            server.login("user", "pass")
            server.send_message(msg)


class StripePaymentGateway:
    """저수준 모듈: Stripe 결제"""

    def __init__(self):
        stripe.api_key = "sk_test_xxxxx"

    def charge(self, amount: float, token: str) -> dict:
        return stripe.Charge.create(
            amount=int(amount * 100),
            currency="krw",
            source=token,
        )


class OrderService:
    """
    ❌ DIP 위반!
    고수준 모듈(OrderService)이 저수준 모듈(MySQL, SMTP, Stripe)에 직접 의존.

    문제점:
    1. MySQL을 PostgreSQL로 바꾸려면 OrderService를 수정해야 함
    2. SMTP를 SendGrid로 바꾸려면 OrderService를 수정해야 함
    3. 테스트하려면 실제 MySQL, SMTP, Stripe가 필요
    4. OrderService의 비즈니스 로직을 다른 프로젝트에서 재사용 불가
    """

    def __init__(self):
        # ❌ 구현체를 직접 생성! 강한 결합!
        self.repository = MySQLOrderRepository()
        self.email_sender = SmtpEmailSender()
        self.payment_gateway = StripePaymentGateway()

    def place_order(self, product: str, amount: float, payment_token: str, customer_email: str) -> int:
        # 1. 결제
        payment_result = self.payment_gateway.charge(amount, payment_token)
        if not payment_result.get("paid"):
            raise Exception("결제 실패")

        # 2. 주문 저장
        order = {"product": product, "amount": amount, "status": "paid"}
        order_id = self.repository.save(order)

        # 3. 확인 이메일
        self.email_sender.send(
            customer_email,
            "주문 확인",
            f"주문 #{order_id}이 완료되었습니다."
        )

        return order_id
```

### ❌ C# - 나쁜 예: 구현체에 직접 의존

```csharp
public class OrderService
{
    // ❌ 구현체에 직접 의존!
    private readonly MySqlOrderRepository _repository;
    private readonly SmtpEmailSender _emailSender;
    private readonly StripePaymentGateway _paymentGateway;

    public OrderService()
    {
        // ❌ 구현체를 직접 생성!
        _repository = new MySqlOrderRepository();
        _emailSender = new SmtpEmailSender();
        _paymentGateway = new StripePaymentGateway();
    }

    public int PlaceOrder(string product, decimal amount, string paymentToken, string email)
    {
        var paymentResult = _paymentGateway.Charge(amount, paymentToken);
        if (!paymentResult.Paid)
            throw new Exception("결제 실패");

        var orderId = _repository.Save(new Order(product, amount, "paid"));
        _emailSender.Send(email, "주문 확인", $"주문 #{orderId} 완료");

        return orderId;
    }
}
```

**의존 관계도**:
```
OrderService (고수준)
    ├── MySQLOrderRepository (저수준, MySQL 라이브러리)
    ├── SmtpEmailSender      (저수준, SMTP 라이브러리)
    └── StripePaymentGateway (저수준, Stripe SDK)

→ 저수준 중 하나라도 바뀌면 고수준도 수정해야 함!
```

---

## 4. DIP 적용 사례 (After)

### ✅ Python - 좋은 예: 추상화에 의존

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime


# ──────────────────────────────────────────────
# 도메인 모델 (고수준)
# ──────────────────────────────────────────────
@dataclass
class Order:
    product: str
    amount: float
    status: str = "pending"
    id: int | None = None
    created_at: datetime = field(default_factory=datetime.now)


@dataclass
class PaymentResult:
    success: bool
    transaction_id: str
    message: str


# ──────────────────────────────────────────────
# 추상화: 인터페이스 정의 (고수준 모듈이 정의!)
# ──────────────────────────────────────────────
class IOrderRepository(ABC):
    """주문 저장소 인터페이스 - 고수준이 필요한 형태로 정의"""

    @abstractmethod
    def save(self, order: Order) -> int:
        pass

    @abstractmethod
    def find_by_id(self, order_id: int) -> Order | None:
        pass

    @abstractmethod
    def find_by_status(self, status: str) -> list[Order]:
        pass


class INotificationService(ABC):
    """알림 서비스 인터페이스"""

    @abstractmethod
    def send_order_confirmation(self, email: str, order: Order) -> None:
        pass

    @abstractmethod
    def send_order_shipped(self, email: str, order: Order) -> None:
        pass


class IPaymentGateway(ABC):
    """결제 게이트웨이 인터페이스"""

    @abstractmethod
    def charge(self, amount: float, payment_token: str) -> PaymentResult:
        pass

    @abstractmethod
    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        pass


# ──────────────────────────────────────────────
# 고수준 모듈: 비즈니스 로직 (추상화에만 의존!)
# ──────────────────────────────────────────────
class OrderService:
    """
    ✅ DIP 준수!
    고수준 모듈이 추상화(인터페이스)에만 의존.
    MySQL, SMTP, Stripe 등 구체 기술을 전혀 모른다.
    """

    def __init__(
        self,
        repository: IOrderRepository,
        notifier: INotificationService,
        payment: IPaymentGateway,
    ):
        # 추상화에만 의존! 구현체가 무엇인지 모른다.
        self._repository = repository
        self._notifier = notifier
        self._payment = payment

    def place_order(
        self,
        product: str,
        amount: float,
        payment_token: str,
        customer_email: str,
    ) -> Order:
        # 1. 결제 (어떤 결제 시스템인지 모른다)
        result = self._payment.charge(amount, payment_token)
        if not result.success:
            raise ValueError(f"결제 실패: {result.message}")

        # 2. 주문 생성 및 저장 (어떤 DB인지 모른다)
        order = Order(product=product, amount=amount, status="paid")
        order.id = self._repository.save(order)

        # 3. 알림 (이메일인지, SMS인지 모른다)
        self._notifier.send_order_confirmation(customer_email, order)

        print(f"주문 #{order.id} 완료! (결제: {result.transaction_id})")
        return order

    def cancel_order(self, order_id: int, customer_email: str) -> None:
        order = self._repository.find_by_id(order_id)
        if order is None:
            raise ValueError(f"주문 #{order_id}를 찾을 수 없습니다.")

        # 환불 처리
        self._payment.refund(f"txn_{order_id}", order.amount)
        order.status = "cancelled"
        self._repository.save(order)
        print(f"주문 #{order_id} 취소 완료")


# ──────────────────────────────────────────────
# 저수준 모듈: 구체적 구현 (추상화를 구현!)
# ──────────────────────────────────────────────
class MySQLOrderRepository(IOrderRepository):
    """MySQL 구현"""

    def __init__(self, connection_string: str):
        self._conn_str = connection_string
        self._next_id = 1  # 간단한 시뮬레이션

    def save(self, order: Order) -> int:
        order.id = self._next_id
        self._next_id += 1
        print(f"  [MySQL] 주문 저장: #{order.id}")
        return order.id

    def find_by_id(self, order_id: int) -> Order | None:
        print(f"  [MySQL] 주문 조회: #{order_id}")
        return Order("샘플 상품", 10000, "paid", order_id)

    def find_by_status(self, status: str) -> list[Order]:
        print(f"  [MySQL] 상태별 조회: {status}")
        return []


class PostgreSQLOrderRepository(IOrderRepository):
    """PostgreSQL 구현 - MySQL에서 교체해도 OrderService는 변경 없음!"""

    def __init__(self, connection_string: str):
        self._conn_str = connection_string
        self._next_id = 1

    def save(self, order: Order) -> int:
        order.id = self._next_id
        self._next_id += 1
        print(f"  [PostgreSQL] 주문 저장: #{order.id}")
        return order.id

    def find_by_id(self, order_id: int) -> Order | None:
        print(f"  [PostgreSQL] 주문 조회: #{order_id}")
        return Order("샘플 상품", 10000, "paid", order_id)

    def find_by_status(self, status: str) -> list[Order]:
        print(f"  [PostgreSQL] 상태별 조회: {status}")
        return []


class EmailNotificationService(INotificationService):
    """이메일 알림 구현"""

    def send_order_confirmation(self, email: str, order: Order) -> None:
        print(f"  [이메일] {email}으로 주문 확인 메일 발송 (주문 #{order.id})")

    def send_order_shipped(self, email: str, order: Order) -> None:
        print(f"  [이메일] {email}으로 발송 알림 메일 발송 (주문 #{order.id})")


class SlackNotificationService(INotificationService):
    """Slack 알림 구현 - 이메일 대신 Slack으로 교체 가능!"""

    def send_order_confirmation(self, email: str, order: Order) -> None:
        print(f"  [Slack] #{order.id} 주문 확인 알림 전송")

    def send_order_shipped(self, email: str, order: Order) -> None:
        print(f"  [Slack] #{order.id} 발송 알림 전송")


class StripePaymentGateway(IPaymentGateway):
    """Stripe 결제 구현"""

    def charge(self, amount: float, payment_token: str) -> PaymentResult:
        print(f"  [Stripe] {amount:,.0f}원 결제")
        return PaymentResult(success=True, transaction_id="txn_stripe_001", message="OK")

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        print(f"  [Stripe] {amount:,.0f}원 환불")
        return PaymentResult(success=True, transaction_id=transaction_id, message="환불 완료")


class TossPaymentGateway(IPaymentGateway):
    """토스페이 결제 구현 - Stripe에서 교체해도 OrderService 변경 없음!"""

    def charge(self, amount: float, payment_token: str) -> PaymentResult:
        print(f"  [토스] {amount:,.0f}원 결제")
        return PaymentResult(success=True, transaction_id="txn_toss_001", message="OK")

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        print(f"  [토스] {amount:,.0f}원 환불")
        return PaymentResult(success=True, transaction_id=transaction_id, message="환불 완료")


# ──────────────────────────────────────────────
# 테스트용 Mock (DIP 덕분에 쉽게 테스트 가능!)
# ──────────────────────────────────────────────
class InMemoryOrderRepository(IOrderRepository):
    """테스트용 인메모리 저장소"""

    def __init__(self):
        self._orders: dict[int, Order] = {}
        self._next_id = 1

    def save(self, order: Order) -> int:
        order.id = self._next_id
        self._orders[order.id] = order
        self._next_id += 1
        return order.id

    def find_by_id(self, order_id: int) -> Order | None:
        return self._orders.get(order_id)

    def find_by_status(self, status: str) -> list[Order]:
        return [o for o in self._orders.values() if o.status == status]


class FakeNotificationService(INotificationService):
    """테스트용 알림 서비스 - 실제 이메일을 보내지 않음"""

    def __init__(self):
        self.sent_notifications: list[dict] = []

    def send_order_confirmation(self, email: str, order: Order) -> None:
        self.sent_notifications.append({"type": "confirmation", "email": email})

    def send_order_shipped(self, email: str, order: Order) -> None:
        self.sent_notifications.append({"type": "shipped", "email": email})


class FakePaymentGateway(IPaymentGateway):
    """테스트용 결제 - 항상 성공"""

    def charge(self, amount: float, payment_token: str) -> PaymentResult:
        return PaymentResult(True, "txn_fake_001", "OK")

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        return PaymentResult(True, transaction_id, "환불 완료")


# ──────────────────────────────────────────────
# 사용 예시: 조합(Composition Root)
# ──────────────────────────────────────────────
if __name__ == "__main__":
    print("=" * 50)
    print("  프로덕션 환경 (MySQL + Email + Stripe)")
    print("=" * 50)

    production_service = OrderService(
        repository=MySQLOrderRepository("mysql://localhost/shop"),
        notifier=EmailNotificationService(),
        payment=StripePaymentGateway(),
    )
    production_service.place_order("노트북", 1_500_000, "tok_visa", "user@example.com")

    print()
    print("=" * 50)
    print("  변경된 환경 (PostgreSQL + Slack + 토스)")
    print("=" * 50)

    # OrderService 코드는 한 줄도 수정하지 않았다!
    alternative_service = OrderService(
        repository=PostgreSQLOrderRepository("postgres://localhost/shop"),
        notifier=SlackNotificationService(),
        payment=TossPaymentGateway(),
    )
    alternative_service.place_order("태블릿", 800_000, "tok_toss", "user@example.com")

    print()
    print("=" * 50)
    print("  테스트 환경 (InMemory + Fake)")
    print("=" * 50)

    # 실제 DB, 이메일, 결제 없이 비즈니스 로직만 테스트!
    test_service = OrderService(
        repository=InMemoryOrderRepository(),
        notifier=FakeNotificationService(),
        payment=FakePaymentGateway(),
    )
    order = test_service.place_order("테스트 상품", 10_000, "tok_test", "test@test.com")
    print(f"  테스트 결과: 주문 #{order.id}, 상태: {order.status}")
```

### ✅ C# - 좋은 예: 추상화에 의존

```csharp
// ── 도메인 모델 ──
public record Order(string Product, decimal Amount, string Status = "pending")
{
    public int? Id { get; set; }
}

public record PaymentResult(bool Success, string TransactionId, string Message);

// ── 추상화: 인터페이스 정의 ──
public interface IOrderRepository
{
    int Save(Order order);
    Order? FindById(int orderId);
    IReadOnlyList<Order> FindByStatus(string status);
}

public interface INotificationService
{
    void SendOrderConfirmation(string email, Order order);
    void SendOrderShipped(string email, Order order);
}

public interface IPaymentGateway
{
    PaymentResult Charge(decimal amount, string paymentToken);
    PaymentResult Refund(string transactionId, decimal amount);
}

// ── 고수준: 비즈니스 로직 (추상화에만 의존!) ──
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly INotificationService _notifier;
    private readonly IPaymentGateway _payment;

    // ✅ 생성자 주입 (Constructor Injection)
    public OrderService(
        IOrderRepository repository,
        INotificationService notifier,
        IPaymentGateway payment)
    {
        _repository = repository;
        _notifier = notifier;
        _payment = payment;
    }

    public Order PlaceOrder(string product, decimal amount, string paymentToken, string email)
    {
        var result = _payment.Charge(amount, paymentToken);
        if (!result.Success)
            throw new InvalidOperationException($"결제 실패: {result.Message}");

        var order = new Order(product, amount, "paid");
        order.Id = _repository.Save(order);

        _notifier.SendOrderConfirmation(email, order);

        Console.WriteLine($"주문 #{order.Id} 완료!");
        return order;
    }
}

// ── 저수준: 구현체들 ──
public class MySqlOrderRepository : IOrderRepository
{
    private int _nextId = 1;

    public int Save(Order order)
    {
        order.Id = _nextId++;
        Console.WriteLine($"  [MySQL] 주문 저장: #{order.Id}");
        return order.Id.Value;
    }

    public Order? FindById(int orderId)
        => new Order("샘플", 10000, "paid") { Id = orderId };

    public IReadOnlyList<Order> FindByStatus(string status) => Array.Empty<Order>();
}

public class EmailNotificationService : INotificationService
{
    public void SendOrderConfirmation(string email, Order order)
        => Console.WriteLine($"  [이메일] {email}으로 확인 메일 발송");

    public void SendOrderShipped(string email, Order order)
        => Console.WriteLine($"  [이메일] {email}으로 발송 알림 발송");
}

public class StripePaymentGateway : IPaymentGateway
{
    public PaymentResult Charge(decimal amount, string paymentToken)
    {
        Console.WriteLine($"  [Stripe] {amount:#,0}원 결제");
        return new PaymentResult(true, "txn_001", "OK");
    }

    public PaymentResult Refund(string transactionId, decimal amount)
        => new PaymentResult(true, transactionId, "환불 완료");
}

// ── Composition Root (진입점에서 조립) ──
// 프로덕션
var productionService = new OrderService(
    new MySqlOrderRepository(),
    new EmailNotificationService(),
    new StripePaymentGateway()
);
productionService.PlaceOrder("노트북", 1_500_000, "tok_visa", "user@example.com");

// 테스트 (Mock 주입)
var testService = new OrderService(
    new InMemoryOrderRepository(),     // 테스트용
    new FakeNotificationService(),     // 테스트용
    new FakePaymentGateway()           // 테스트용
);
testService.PlaceOrder("테스트 상품", 10_000, "tok_test", "test@test.com");
```

---

## 5. DIP vs DI: 원칙과 기법의 차이

이 두 용어는 자주 혼동되지만 **다른 개념**입니다.

### 명확한 구분

| 구분 | DIP (Dependency Inversion Principle) | DI (Dependency Injection) |
|------|--------------------------------------|---------------------------|
| **성격** | 설계 **원칙** (Principle) | 구현 **기법** (Technique) |
| **질문** | "누가 누구에게 의존하는가?" | "의존성을 어떻게 전달하는가?" |
| **핵심** | 고수준이 저수준이 아닌 추상화에 의존 | 의존 객체를 외부에서 주입 |
| **비유** | "전기는 콘센트 규격을 따라야 한다" (규칙) | "콘센트에 플러그를 꽂는 행위" (실행) |

### DI (의존성 주입)의 3가지 방법

```python
from abc import ABC, abstractmethod


class ILogger(ABC):
    @abstractmethod
    def log(self, message: str) -> None: ...


class ConsoleLogger(ILogger):
    def log(self, message: str) -> None:
        print(f"[LOG] {message}")


# ──────────────────────────────────────────────
# 방법 1: 생성자 주입 (Constructor Injection) ★ 가장 권장
# ──────────────────────────────────────────────
class OrderServiceV1:
    def __init__(self, logger: ILogger):
        self._logger = logger  # 생성 시 주입

    def place_order(self) -> None:
        self._logger.log("주문 처리")


# ──────────────────────────────────────────────
# 방법 2: 메서드 주입 (Method Injection)
# ──────────────────────────────────────────────
class OrderServiceV2:
    def place_order(self, logger: ILogger) -> None:
        logger.log("주문 처리")  # 메서드 호출 시 주입


# ──────────────────────────────────────────────
# 방법 3: 속성 주입 (Property/Setter Injection)
# ──────────────────────────────────────────────
class OrderServiceV3:
    def __init__(self):
        self._logger: ILogger | None = None

    @property
    def logger(self) -> ILogger | None:
        return self._logger

    @logger.setter
    def logger(self, value: ILogger) -> None:
        self._logger = value

    def place_order(self) -> None:
        if self._logger:
            self._logger.log("주문 처리")
```

```csharp
// C#에서의 3가지 DI 방법

// 1. 생성자 주입 ★ 가장 권장
public class OrderServiceV1
{
    private readonly ILogger _logger;

    public OrderServiceV1(ILogger logger) => _logger = logger;

    public void PlaceOrder() => _logger.Log("주문 처리");
}

// 2. 메서드 주입
public class OrderServiceV2
{
    public void PlaceOrder(ILogger logger) => logger.Log("주문 처리");
}

// 3. 속성 주입
public class OrderServiceV3
{
    public ILogger? Logger { get; set; }

    public void PlaceOrder() => Logger?.Log("주문 처리");
}
```

### 왜 생성자 주입이 가장 좋은가?

| 주입 방법 | 장점 | 단점 |
|----------|------|------|
| **생성자 주입** | 불변성 보장, 필수 의존성 명시 | 매개변수가 많아질 수 있음 (SRP 위반 징후) |
| 메서드 주입 | 호출마다 다른 구현체 사용 가능 | 매번 전달해야 하는 번거로움 |
| 속성 주입 | 선택적 의존성에 적합 | null 가능성, 불완전한 객체 상태 |

---

## 6. DI 컨테이너 소개

실무에서는 의존성 조립을 **DI 컨테이너(IoC Container)** 가 자동으로 처리합니다.

### C# - Microsoft.Extensions.DependencyInjection

```csharp
using Microsoft.Extensions.DependencyInjection;

// ── 서비스 등록 (Composition Root) ──
var services = new ServiceCollection();

// 인터페이스 → 구현체 매핑 등록
services.AddScoped<IOrderRepository, MySqlOrderRepository>();
services.AddScoped<INotificationService, EmailNotificationService>();
services.AddScoped<IPaymentGateway, StripePaymentGateway>();
services.AddScoped<OrderService>();  // OrderService 자체도 등록

var provider = services.BuildServiceProvider();

// ── 서비스 사용 (자동으로 의존성 주입!) ──
var orderService = provider.GetRequiredService<OrderService>();
// OrderService의 생성자에 IOrderRepository, INotificationService, IPaymentGateway가
// 자동으로 주입됨!

orderService.PlaceOrder("노트북", 1_500_000, "tok_visa", "user@example.com");


// ── 환경에 따라 다른 구현체 등록 ──
// 개발 환경
if (isDevelopment)
{
    services.AddScoped<IOrderRepository, InMemoryOrderRepository>();
    services.AddScoped<INotificationService, ConsoleNotificationService>();
}
// 프로덕션 환경
else
{
    services.AddScoped<IOrderRepository, MySqlOrderRepository>();
    services.AddScoped<INotificationService, EmailNotificationService>();
}
```

### Python - dependency-injector 라이브러리

```python
# pip install dependency-injector
from dependency_injector import containers, providers


class Container(containers.DeclarativeContainer):
    """DI 컨테이너 - 의존성 설정을 한 곳에서 관리"""

    config = providers.Configuration()

    # 저수준 모듈 등록
    order_repository = providers.Singleton(
        MySQLOrderRepository,
        connection_string=config.db.connection_string,
    )

    notification_service = providers.Factory(
        EmailNotificationService,
    )

    payment_gateway = providers.Factory(
        StripePaymentGateway,
    )

    # 고수준 모듈 - 의존성 자동 주입!
    order_service = providers.Factory(
        OrderService,
        repository=order_repository,
        notifier=notification_service,
        payment=payment_gateway,
    )


# 사용
container = Container()
container.config.db.connection_string.from_value("mysql://localhost/shop")

order_service = container.order_service()
order_service.place_order("노트북", 1_500_000, "tok_visa", "user@example.com")

# 테스트 시 오버라이드
with container.order_repository.override(InMemoryOrderRepository()):
    test_service = container.order_service()
    test_service.place_order("테스트", 10_000, "tok_test", "test@test.com")
```

> **참고**: Python에서는 DI 컨테이너 없이 수동으로 주입하는 것이 더 일반적입니다. 프로젝트 규모가 클 때 DI 컨테이너 도입을 고려하세요.

---

## 7. 실무 적용 가이드

### DIP를 적용해야 하는 경계

| 경계 | 추상화 대상 | 이유 |
|------|-----------|------|
| **데이터 접근** | Repository 인터페이스 | DB 교체 가능성, 테스트 용이 |
| **외부 API** | Gateway/Client 인터페이스 | 외부 서비스 변경 대비, Mock 테스트 |
| **알림/메시지** | Notification 인터페이스 | 채널(이메일/SMS/푸시) 변경 가능 |
| **파일 시스템** | Storage 인터페이스 | 로컬/S3/Azure Blob 교체 가능 |
| **시간/날짜** | Clock 인터페이스 | 테스트에서 시간 고정 가능 |

### DIP를 적용하지 않아도 되는 경우

```python
# 이런 것까지 추상화할 필요는 없다:

# ❌ 과도한 추상화
class IStringFormatter(ABC):
    @abstractmethod
    def format(self, text: str) -> str: ...

# ❌ 표준 라이브러리에 대한 추상화
class IList(ABC):
    @abstractmethod
    def append(self, item): ...

# ✅ 이런 것은 그냥 직접 사용
formatted = f"Hello, {name}!"
items = [1, 2, 3]
```

**추상화할 가치가 있는 기준**:
1. **교체 가능성**이 있는가? (DB, 외부 API, 알림 채널)
2. **테스트할 때** 실제 구현체가 문제가 되는가? (네트워크, 파일 시스템)
3. **변경 빈도**가 높은가?

### 의존성 방향 검증법

```
프로젝트 구조에서 import/using 방향을 확인하세요:

✅ 올바른 방향:
  domain/ (고수준)
    ├── models.py         → 다른 곳에 의존하지 않음
    ├── interfaces.py     → 다른 곳에 의존하지 않음
    └── services.py       → interfaces.py에만 의존

  infrastructure/ (저수준)
    ├── mysql_repo.py     → domain/interfaces.py에 의존 ✅
    ├── smtp_email.py     → domain/interfaces.py에 의존 ✅
    └── stripe_payment.py → domain/interfaces.py에 의존 ✅

❌ 잘못된 방향:
  domain/services.py → infrastructure/mysql_repo.py에 의존!
```

---

## 8. 정리 및 체크리스트

### 핵심 요약

| 항목 | 설명 |
|------|------|
| **원칙** | 고수준 모듈과 저수준 모듈 모두 추상화에 의존 |
| **핵심 도구** | 인터페이스, 추상 클래스, 의존성 주입 |
| **DIP vs DI** | DIP는 설계 원칙, DI는 구현 기법 |
| **DI 방법** | 생성자 주입(권장), 메서드 주입, 속성 주입 |
| **DI 컨테이너** | C#: Microsoft.Extensions.DI, Python: dependency-injector |
| **위반 징후** | import/using이 고수준 → 저수준 방향, new로 직접 생성 |

### 셀프 체크리스트

- [ ] 비즈니스 로직(고수준)이 DB나 외부 API(저수준)에 직접 의존하고 있지 않은가?
- [ ] 추상화(인터페이스)가 고수준 모듈 쪽에 정의되어 있는가?
- [ ] 의존성을 외부에서 주입받고 있는가? (new로 직접 생성하지 않는가?)
- [ ] 테스트 시 Mock/Fake로 쉽게 교체할 수 있는가?
- [ ] 과도한 추상화로 복잡성이 증가하지 않았는가?
- [ ] DIP와 DI의 차이를 명확히 이해하고 있는가?

---

## 9. SOLID 원칙 총정리

5개의 SOLID 원칙을 모두 학습했습니다. 전체를 한 눈에 정리합니다.

### SOLID 전체 요약

| 원칙 | 한 줄 정리 | 핵심 질문 |
|------|-----------|----------|
| **S** - SRP | 클래스는 하나의 변경 이유만 | "이 클래스가 변경되는 이유가 몇 가지인가?" |
| **O** - OCP | 확장에 열고, 수정에 닫기 | "새 기능 추가 시 기존 코드를 수정하는가?" |
| **L** - LSP | 하위 타입은 상위 타입을 대체 | "부모 자리에 자식을 넣어도 문제없는가?" |
| **I** - ISP | 인터페이스를 역할별로 분리 | "사용하지 않는 메서드에 의존하는가?" |
| **D** - DIP | 추상화에 의존, 구현체에 의존하지 않기 | "고수준이 저수준에 직접 의존하는가?" |

### SOLID 원칙 간의 관계

```
SRP (책임 분리)
 └→ 작은 클래스 → ISP (인터페이스도 작게) + OCP (확장 용이)

OCP (확장에 열기)
 └→ 추상화 필요 → DIP (추상화에 의존) + LSP (안전한 대체)

LSP (안전한 상속)
 └→ 계약 준수 → ISP (불필요한 메서드 제거로 계약 축소)

ISP (인터페이스 분리)
 └→ 작은 인터페이스 → DIP (구체적 추상화에 의존)

DIP (추상화에 의존)
 └→ 인터페이스 기반 → OCP (교체로 확장) + ISP (필요한 인터페이스만)
```

### 최종 조언

> **SOLID는 목적이 아니라 수단입니다.**
>
> - 처음부터 완벽한 SOLID를 추구하지 마세요.
> - 코드의 "냄새"를 맡고, 필요할 때 적용하세요.
> - **과도한 추상화**는 과도한 결합만큼 해롭습니다.
> - 팀과의 **합의와 일관성**이 원칙 자체보다 중요합니다.
> - "Rule of Three": 같은 패턴이 3번 반복되면 그때 리팩토링하세요.
