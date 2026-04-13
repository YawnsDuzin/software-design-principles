# Adapter 패턴 (어댑터)

> **핵심 의도 한줄 요약:** 호환되지 않는 인터페이스를 가진 클래스들이 함께 동작할 수 있도록 중간에서 변환해주는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [객체 어댑터 vs 클래스 어댑터](#5-객체-어댑터-vs-클래스-어댑터)
6. [패턴 적용 전 (Before)](#6-패턴-적용-전-before)
7. [패턴 적용 후 (After)](#7-패턴-적용-후-after)
8. [Python 구현](#8-python-구현)
9. [C# 구현](#9-c-구현)
10. [실무 예제: 서드파티 결제 시스템 어댑터](#10-실무-예제-서드파티-결제-시스템-어댑터)
11. [장단점](#11-장단점)
12. [관련 패턴](#12-관련-패턴)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)

---

## 1. 개요

Adapter 패턴은 **GoF 구조 패턴** 중 하나로, 기존 클래스의 인터페이스를 클라이언트가 기대하는 다른 인터페이스로 변환합니다. 이를 통해 인터페이스 호환성 문제 때문에 함께 쓸 수 없었던 클래스들을 협력하게 만듭니다.

실무에서 가장 흔하게 마주치는 상황:
- 외부 라이브러리를 우리 시스템에 통합할 때
- 레거시 코드를 새로운 아키텍처에 연동할 때
- 서드파티 API의 응답 형식을 우리 도메인 모델에 맞출 때

---

## 2. 왜 필요한가? - 문제 상황

### 상황 1: 외부 라이브러리 통합

우리 시스템은 `ILogger` 인터페이스를 사용하는데, 새로 도입한 로깅 라이브러리는 완전히 다른 메서드를 제공합니다.

```
우리 시스템: logger.log(level, message)
외부 라이브러리: externalLogger.writeLog(msg, severity)
```

### 상황 2: 레거시 시스템 연동

새로 개발한 주문 시스템이 `IPaymentProcessor`를 기대하는데, 기존 결제 모듈은 `OldPaymentModule.processTransaction()`이라는 다른 인터페이스를 가지고 있습니다.

### 상황 3: API 응답 변환

외부 API에서 XML로 응답을 주는데, 우리 시스템은 JSON 기반의 객체만 처리할 수 있습니다.

**공통점:** 우리가 기대하는 인터페이스와 실제 제공되는 인터페이스가 다르다!

---

## 3. 비유로 이해하기

### 전원 어댑터 (110V -> 220V)

```
한국에서 사용하는 노트북 충전기 (220V)를 미국 (110V)에서 사용하고 싶다면?

[한국 노트북 충전기] ---> [전원 어댑터] ---> [미국 콘센트]
     (220V 기대)         (110V→220V 변환)      (110V 제공)

어댑터가 없으면? -> 호환 불가, 충전기 사용 불가
어댑터가 있으면? -> 충전기를 수정하지 않고도 미국 콘센트 사용 가능!
```

핵심: **양쪽 모두 변경하지 않고**, 중간에 어댑터를 두어 호환성을 확보합니다.

---

## 4. UML 다이어그램

### 객체 어댑터 (Object Adapter) - 컴포지션 사용

```
┌─────────────┐         ┌──────────────────┐
│   Client     │         │  <<interface>>   │
│              │────────>│     Target       │
│              │         │──────────────────│
└─────────────┘         │ + request()      │
                         └──────────────────┘
                                  △
                                  │ implements
                         ┌──────────────────┐       ┌──────────────────┐
                         │    Adapter        │       │    Adaptee       │
                         │──────────────────│       │──────────────────│
                         │ - adaptee        │──────>│ + specificReq()  │
                         │──────────────────│       │                  │
                         │ + request()      │       └──────────────────┘
                         │   adaptee.       │
                         │   specificReq()  │
                         └──────────────────┘
```

### 클래스 어댑터 (Class Adapter) - 다중 상속 사용

```
┌──────────────────┐         ┌──────────────────┐
│  <<interface>>   │         │    Adaptee       │
│     Target       │         │──────────────────│
│──────────────────│         │ + specificReq()  │
│ + request()      │         │                  │
└──────────────────┘         └──────────────────┘
          △                           △
          │ implements                │ extends
          │     ┌─────────────┐       │
          └─────│   Adapter   │───────┘
                │─────────────│
                │ + request() │
                │  (calls     │
                │  specificReq│
                │  from super)│
                └─────────────┘
```

---

## 5. 객체 어댑터 vs 클래스 어댑터

| 구분 | 객체 어댑터 (Object Adapter) | 클래스 어댑터 (Class Adapter) |
|------|-----|-----|
| **방식** | 컴포지션 (has-a) | 다중 상속 (is-a) |
| **유연성** | 높음 (런타임에 adaptee 교체 가능) | 낮음 (컴파일 타임에 결정) |
| **Adaptee 서브클래스** | 서브클래스도 어댑트 가능 | 특정 Adaptee에만 동작 |
| **오버라이드** | Adaptee 동작 변경 어려움 | Adaptee 메서드 오버라이드 가능 |
| **언어 지원** | 모든 언어 | 다중 상속 지원 언어만 |
| **권장도** | **실무에서 권장** | Python에서는 가능, C#/Java 불가 |

> **실무 팁:** 대부분의 경우 객체 어댑터를 사용합니다. 컴포지션이 상속보다 유연하고 유지보수가 쉽습니다.

---

## 6. 패턴 적용 전 (Before)

### 문제: 클라이언트 코드가 외부 라이브러리에 직접 의존

```python
# ❌ Before: 외부 라이브러리에 직접 의존하는 코드
class AnalyticsService:
    def track_event(self, event_name: str, properties: dict):
        # 외부 라이브러리 직접 호출 - 강결합!
        from third_party_analytics import ThirdPartyTracker
        tracker = ThirdPartyTracker()
        tracker.send_event_data(
            event_id=event_name,
            payload=properties,
            format_type="json"
        )

class OrderService:
    def __init__(self):
        self.analytics = AnalyticsService()

    def create_order(self, order):
        # 주문 로직...
        # 외부 라이브러리 API가 바뀌면 여기도 수정해야 함!
        self.analytics.track_event("order_created", {"order_id": order.id})
```

**문제점:**
- `ThirdPartyTracker`의 API가 변경되면 모든 사용 코드를 수정해야 함
- 다른 분석 도구로 교체하기 어려움
- 단위 테스트에서 외부 의존성을 제거하기 어려움

---

## 7. 패턴 적용 후 (After)

### 해결: 어댑터로 인터페이스 통일

```python
# ✅ After: 어댑터 패턴으로 인터페이스 통일
from abc import ABC, abstractmethod

class AnalyticsTracker(ABC):
    """우리 시스템이 기대하는 인터페이스 (Target)"""
    @abstractmethod
    def track(self, event: str, data: dict) -> None:
        pass

class ThirdPartyAnalyticsAdapter(AnalyticsTracker):
    """어댑터: 외부 라이브러리를 우리 인터페이스에 맞춤"""
    def __init__(self, tracker):
        self._tracker = tracker  # Adaptee를 컴포지션으로 보유

    def track(self, event: str, data: dict) -> None:
        # 외부 라이브러리의 인터페이스로 변환
        self._tracker.send_event_data(
            event_id=event,
            payload=data,
            format_type="json"
        )

class OrderService:
    def __init__(self, analytics: AnalyticsTracker):  # 인터페이스에 의존
        self.analytics = analytics

    def create_order(self, order):
        # 주문 로직...
        self.analytics.track("order_created", {"order_id": order.id})
```

**장점:**
- 외부 라이브러리 변경이 어댑터 내부에만 영향
- 다른 분석 도구로 쉽게 교체 가능
- 테스트에서 Mock 어댑터 사용 가능

---

## 8. Python 구현

### 컴포지션 기반 객체 어댑터

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol


# ===== Target: 우리 시스템이 기대하는 인터페이스 =====
class NotificationSender(Protocol):
    """알림 발송 인터페이스 (Target)"""
    def send(self, recipient: str, title: str, body: str) -> bool:
        ...


# ===== Adaptee 1: 외부 이메일 라이브러리 =====
class ExternalEmailService:
    """외부 이메일 발송 라이브러리 (Adaptee)
    - 우리가 수정할 수 없는 코드
    - 인터페이스가 우리 시스템과 다름
    """
    def send_email(self, to_address: str, subject: str,
                   html_content: str, priority: int = 3) -> dict:
        print(f"[ExternalEmail] 발송: {to_address}, 제목: {subject}")
        return {"status": "sent", "message_id": "ext-email-001"}


# ===== Adaptee 2: 외부 SMS 라이브러리 =====
class ExternalSmsGateway:
    """외부 SMS 발송 라이브러리 (Adaptee)
    - 또 다른 인터페이스
    """
    def transmit(self, phone_number: str, text: str) -> int:
        print(f"[ExternalSMS] 발송: {phone_number}, 내용: {text}")
        return 200  # HTTP 상태 코드 반환


# ===== Adapter 1: 이메일 어댑터 =====
class EmailAdapter:
    """이메일 서비스를 우리 인터페이스에 맞춤 (Adapter)"""

    def __init__(self, email_service: ExternalEmailService):
        self._service = email_service

    def send(self, recipient: str, title: str, body: str) -> bool:
        """우리 인터페이스 -> 외부 라이브러리 인터페이스 변환"""
        result = self._service.send_email(
            to_address=recipient,
            subject=title,
            html_content=f"<p>{body}</p>",
            priority=3
        )
        return result["status"] == "sent"


# ===== Adapter 2: SMS 어댑터 =====
class SmsAdapter:
    """SMS 게이트웨이를 우리 인터페이스에 맞춤 (Adapter)"""

    def __init__(self, sms_gateway: ExternalSmsGateway):
        self._gateway = sms_gateway

    def send(self, recipient: str, title: str, body: str) -> bool:
        """우리 인터페이스 -> 외부 라이브러리 인터페이스 변환"""
        # SMS는 제목과 본문을 합침
        message = f"[{title}] {body}"
        status_code = self._gateway.transmit(
            phone_number=recipient,
            text=message
        )
        return status_code == 200


# ===== 클라이언트 코드 =====
class NotificationManager:
    """클라이언트: Target 인터페이스만 알면 됨"""

    def __init__(self):
        self._senders: list[NotificationSender] = []

    def register_sender(self, sender: NotificationSender):
        self._senders.append(sender)

    def notify_all(self, recipient: str, title: str, body: str):
        for sender in self._senders:
            success = sender.send(recipient, title, body)
            print(f"  -> 발송 결과: {'성공' if success else '실패'}")


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 외부 라이브러리 인스턴스 (Adaptee)
    email_service = ExternalEmailService()
    sms_gateway = ExternalSmsGateway()

    # 어댑터로 감싸기
    email_adapter = EmailAdapter(email_service)
    sms_adapter = SmsAdapter(sms_gateway)

    # 클라이언트는 통일된 인터페이스로 사용
    manager = NotificationManager()
    manager.register_sender(email_adapter)
    manager.register_sender(sms_adapter)

    print("=== 알림 발송 ===")
    manager.notify_all(
        recipient="user@example.com / 010-1234-5678",
        title="주문 확인",
        body="주문이 성공적으로 접수되었습니다."
    )
```

**실행 결과:**
```
=== 알림 발송 ===
[ExternalEmail] 발송: user@example.com / 010-1234-5678, 제목: 주문 확인
  -> 발송 결과: 성공
[ExternalSMS] 발송: user@example.com / 010-1234-5678, 내용: [주문 확인] 주문이 성공적으로 접수되었습니다.
  -> 발송 결과: 성공
```

### Python 클래스 어댑터 (다중 상속 활용)

```python
from abc import ABC, abstractmethod


class Target(ABC):
    @abstractmethod
    def request(self) -> str:
        pass


class Adaptee:
    def specific_request(self) -> str:
        return "Adaptee의 특수한 동작"


# 클래스 어댑터: 다중 상속으로 Target과 Adaptee를 모두 상속
class ClassAdapter(Target, Adaptee):
    def request(self) -> str:
        # Adaptee의 메서드를 직접 호출
        return f"ClassAdapter 변환: {self.specific_request()}"


# 비교: 객체 어댑터
class ObjectAdapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee

    def request(self) -> str:
        return f"ObjectAdapter 변환: {self._adaptee.specific_request()}"
```

---

## 9. C# 구현

### Interface 기반 어댑터

```csharp
using System;
using System.Collections.Generic;

// ===== Target: 우리 시스템이 기대하는 인터페이스 =====
public interface INotificationSender
{
    bool Send(string recipient, string title, string body);
}

// ===== Adaptee 1: 외부 이메일 라이브러리 =====
public class ExternalEmailService
{
    // 우리가 수정할 수 없는 외부 라이브러리
    public EmailResult SendEmail(string toAddress, string subject,
                                  string htmlContent, int priority = 3)
    {
        Console.WriteLine($"[ExternalEmail] 발송: {toAddress}, 제목: {subject}");
        return new EmailResult { Status = "sent", MessageId = "ext-email-001" };
    }
}

public class EmailResult
{
    public string Status { get; set; }
    public string MessageId { get; set; }
}

// ===== Adaptee 2: 외부 SMS 라이브러리 =====
public class ExternalSmsGateway
{
    public int Transmit(string phoneNumber, string text)
    {
        Console.WriteLine($"[ExternalSMS] 발송: {phoneNumber}, 내용: {text}");
        return 200;
    }
}

// ===== Adapter 1: 이메일 어댑터 =====
public class EmailAdapter : INotificationSender
{
    private readonly ExternalEmailService _service;

    public EmailAdapter(ExternalEmailService service)
    {
        _service = service;
    }

    public bool Send(string recipient, string title, string body)
    {
        // 인터페이스 변환: 우리 형식 → 외부 라이브러리 형식
        var result = _service.SendEmail(
            toAddress: recipient,
            subject: title,
            htmlContent: $"<p>{body}</p>",
            priority: 3
        );
        return result.Status == "sent";
    }
}

// ===== Adapter 2: SMS 어댑터 =====
public class SmsAdapter : INotificationSender
{
    private readonly ExternalSmsGateway _gateway;

    public SmsAdapter(ExternalSmsGateway gateway)
    {
        _gateway = gateway;
    }

    public bool Send(string recipient, string title, string body)
    {
        string message = $"[{title}] {body}";
        int statusCode = _gateway.Transmit(
            phoneNumber: recipient,
            text: message
        );
        return statusCode == 200;
    }
}

// ===== 클라이언트 코드 =====
public class NotificationManager
{
    private readonly List<INotificationSender> _senders = new();

    public void RegisterSender(INotificationSender sender)
    {
        _senders.Add(sender);
    }

    public void NotifyAll(string recipient, string title, string body)
    {
        foreach (var sender in _senders)
        {
            bool success = sender.Send(recipient, title, body);
            Console.WriteLine($"  -> 발송 결과: {(success ? "성공" : "실패")}");
        }
    }
}

// ===== 사용 예시 =====
public class Program
{
    public static void Main()
    {
        // Adaptee 인스턴스
        var emailService = new ExternalEmailService();
        var smsGateway = new ExternalSmsGateway();

        // 어댑터로 감싸기
        var emailAdapter = new EmailAdapter(emailService);
        var smsAdapter = new SmsAdapter(smsGateway);

        // 통일된 인터페이스로 사용
        var manager = new NotificationManager();
        manager.RegisterSender(emailAdapter);
        manager.RegisterSender(smsAdapter);

        Console.WriteLine("=== 알림 발송 ===");
        manager.NotifyAll("user@example.com", "주문 확인", "주문이 접수되었습니다.");
    }
}
```

---

## 10. 실무 예제: 서드파티 결제 시스템 어댑터

### Python - 여러 PG사 통합

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from enum import Enum
from typing import Optional


class PaymentStatus(Enum):
    SUCCESS = "success"
    FAILED = "failed"
    PENDING = "pending"


@dataclass
class PaymentResult:
    """우리 시스템의 통일된 결제 결과"""
    status: PaymentStatus
    transaction_id: str
    amount: int
    message: str


# ===== Target: 우리 결제 인터페이스 =====
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, order_id: str, amount: int,
                        card_number: str) -> PaymentResult:
        pass

    @abstractmethod
    def refund(self, transaction_id: str, amount: int) -> PaymentResult:
        pass


# ===== Adaptee 1: A사 PG (레거시 XML 기반) =====
class LegacyPGCompanyA:
    """A사 PG - XML 기반 레거시 시스템"""

    def execute_transaction(self, xml_data: str) -> str:
        print(f"[A사 PG] XML 요청 처리 중...")
        return '<response><code>0000</code><tid>TXN-A-001</tid></response>'

    def cancel_transaction(self, tid: str, cancel_amount: str) -> str:
        print(f"[A사 PG] 거래 취소: {tid}")
        return '<response><code>0000</code></response>'


# ===== Adaptee 2: B사 PG (REST API 기반) =====
class ModernPGCompanyB:
    """B사 PG - REST API 기반 현대적 시스템"""

    def charge(self, merchant_id: str, payment_info: dict) -> dict:
        print(f"[B사 PG] REST API 결제 처리 중...")
        return {
            "result_code": "SUCCESS",
            "transaction_id": "TXN-B-001",
            "approved_amount": payment_info.get("amount", 0)
        }

    def cancel(self, merchant_id: str, transaction_id: str,
               amount: int) -> dict:
        print(f"[B사 PG] REST API 환불 처리 중...")
        return {"result_code": "SUCCESS"}


# ===== Adapter 1: A사 PG 어댑터 =====
class PGCompanyAAdapter(PaymentProcessor):
    """레거시 A사 PG를 우리 인터페이스에 맞춤"""

    def __init__(self, pg: LegacyPGCompanyA):
        self._pg = pg

    def _build_xml(self, order_id: str, amount: int,
                   card_number: str) -> str:
        """우리 데이터를 XML로 변환"""
        return (
            f'<request>'
            f'<orderId>{order_id}</orderId>'
            f'<amount>{amount}</amount>'
            f'<cardNo>{card_number}</cardNo>'
            f'</request>'
        )

    def _parse_response(self, xml_response: str) -> tuple[str, str]:
        """XML 응답을 파싱 (간소화)"""
        # 실제로는 XML 파서 사용
        code = "0000" if "0000" in xml_response else "9999"
        tid = "TXN-A-001"  # 실제로는 XML에서 추출
        return code, tid

    def process_payment(self, order_id: str, amount: int,
                        card_number: str) -> PaymentResult:
        # 1. 우리 형식 -> A사 XML 형식으로 변환
        xml_data = self._build_xml(order_id, amount, card_number)

        # 2. A사 PG 호출
        xml_response = self._pg.execute_transaction(xml_data)

        # 3. A사 응답 -> 우리 형식으로 변환
        code, tid = self._parse_response(xml_response)

        return PaymentResult(
            status=PaymentStatus.SUCCESS if code == "0000"
                   else PaymentStatus.FAILED,
            transaction_id=tid,
            amount=amount,
            message="결제 성공" if code == "0000" else "결제 실패"
        )

    def refund(self, transaction_id: str, amount: int) -> PaymentResult:
        xml_response = self._pg.cancel_transaction(
            tid=transaction_id, cancel_amount=str(amount)
        )
        code, _ = self._parse_response(xml_response)
        return PaymentResult(
            status=PaymentStatus.SUCCESS if code == "0000"
                   else PaymentStatus.FAILED,
            transaction_id=transaction_id,
            amount=amount,
            message="환불 성공" if code == "0000" else "환불 실패"
        )


# ===== Adapter 2: B사 PG 어댑터 =====
class PGCompanyBAdapter(PaymentProcessor):
    """현대적 B사 PG를 우리 인터페이스에 맞춤"""

    MERCHANT_ID = "SHOP-001"

    def __init__(self, pg: ModernPGCompanyB):
        self._pg = pg

    def process_payment(self, order_id: str, amount: int,
                        card_number: str) -> PaymentResult:
        # B사 형식으로 변환
        payment_info = {
            "order_id": order_id,
            "amount": amount,
            "card_number": card_number,
            "currency": "KRW"
        }

        result = self._pg.charge(self.MERCHANT_ID, payment_info)

        return PaymentResult(
            status=PaymentStatus.SUCCESS
                   if result["result_code"] == "SUCCESS"
                   else PaymentStatus.FAILED,
            transaction_id=result["transaction_id"],
            amount=result["approved_amount"],
            message="결제 성공"
        )

    def refund(self, transaction_id: str, amount: int) -> PaymentResult:
        result = self._pg.cancel(
            self.MERCHANT_ID, transaction_id, amount
        )
        return PaymentResult(
            status=PaymentStatus.SUCCESS
                   if result["result_code"] == "SUCCESS"
                   else PaymentStatus.FAILED,
            transaction_id=transaction_id,
            amount=amount,
            message="환불 성공"
        )


# ===== 클라이언트: 주문 서비스 =====
class OrderService:
    """PG사가 뭔지 모르고, PaymentProcessor만 알면 됨"""

    def __init__(self, payment: PaymentProcessor):
        self._payment = payment

    def checkout(self, order_id: str, amount: int, card_number: str):
        print(f"\n주문 처리 시작: {order_id}")
        result = self._payment.process_payment(order_id, amount, card_number)
        print(f"결과: {result}")

        if result.status == PaymentStatus.SUCCESS:
            print("-> 주문 완료!")
        else:
            print("-> 주문 실패!")
        return result


# ===== 실행 =====
if __name__ == "__main__":
    # A사 PG 사용
    print("===== A사 PG (레거시) =====")
    pg_a = LegacyPGCompanyA()
    adapter_a = PGCompanyAAdapter(pg_a)
    order_service_a = OrderService(adapter_a)
    order_service_a.checkout("ORD-001", 50000, "4111-1111-1111-1111")

    # B사 PG로 교체 - OrderService 코드 변경 없음!
    print("\n===== B사 PG (현대적) =====")
    pg_b = ModernPGCompanyB()
    adapter_b = PGCompanyBAdapter(pg_b)
    order_service_b = OrderService(adapter_b)
    order_service_b.checkout("ORD-002", 75000, "5222-2222-2222-2222")
```

### C# - DI 컨테이너와 함께 사용

```csharp
using System;

// Target
public interface IPaymentProcessor
{
    PaymentResult ProcessPayment(string orderId, int amount, string cardNumber);
    PaymentResult Refund(string transactionId, int amount);
}

// Adaptee: 외부 PG
public class ExternalPGService
{
    public PGResponse Charge(PGRequest request)
    {
        Console.WriteLine($"[외부 PG] 결제 처리: {request.Amount}원");
        return new PGResponse { Code = "00", TxId = "PG-TXN-001" };
    }
}

// Adapter
public class ExternalPGAdapter : IPaymentProcessor
{
    private readonly ExternalPGService _pg;

    public ExternalPGAdapter(ExternalPGService pg)
    {
        _pg = pg;
    }

    public PaymentResult ProcessPayment(string orderId, int amount,
                                         string cardNumber)
    {
        // 형식 변환
        var pgRequest = new PGRequest
        {
            MerchantOrderId = orderId,
            Amount = amount,
            CardNo = cardNumber,
            Currency = "KRW"
        };

        var pgResponse = _pg.Charge(pgRequest);

        return new PaymentResult
        {
            Status = pgResponse.Code == "00"
                ? PaymentStatus.Success
                : PaymentStatus.Failed,
            TransactionId = pgResponse.TxId,
            Amount = amount,
            Message = pgResponse.Code == "00" ? "결제 성공" : "결제 실패"
        };
    }

    public PaymentResult Refund(string transactionId, int amount)
    {
        // 환불 로직...
        return new PaymentResult
        {
            Status = PaymentStatus.Success,
            TransactionId = transactionId,
            Amount = amount,
            Message = "환불 성공"
        };
    }
}

// DI 등록 (ASP.NET Core 예시)
// services.AddSingleton<ExternalPGService>();
// services.AddSingleton<IPaymentProcessor, ExternalPGAdapter>();
```

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **단일 책임 원칙** | 인터페이스 변환 로직을 비즈니스 로직과 분리 |
| **개방/폐쇄 원칙** | 기존 코드 수정 없이 새로운 어댑터 추가 가능 |
| **재사용성** | 기존 클래스를 수정하지 않고 재사용 |
| **테스트 용이** | 어댑터를 Mock으로 교체하여 단위 테스트 가능 |
| **점진적 마이그레이션** | 레거시 시스템을 단계적으로 교체할 때 유용 |

### 단점

| 단점 | 설명 |
|------|------|
| **코드 복잡성 증가** | 새로운 인터페이스와 클래스 추가 필요 |
| **간접 호출 오버헤드** | 어댑터를 거치는 추가 레이어 |
| **과도한 사용 주의** | 설계 자체를 개선하는 것이 나을 수 있음 |

---

## 12. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Bridge** | Adapter는 사후에 호환성을 맞추고, Bridge는 사전에 추상화와 구현을 분리 |
| **Decorator** | Adapter는 인터페이스를 변환하고, Decorator는 인터페이스를 유지하면서 기능 추가 |
| **Facade** | Adapter는 1:1 변환, Facade는 복잡한 서브시스템 전체를 단순화 |
| **Proxy** | Adapter는 다른 인터페이스 제공, Proxy는 같은 인터페이스 제공 |

---

## 13. 정리 및 체크리스트

### 핵심 정리

```
Adapter 패턴 = "인터페이스 번역기"

When: 기존 클래스를 수정할 수 없지만, 우리 시스템과 함께 사용해야 할 때
How:  중간에 어댑터 클래스를 두어 인터페이스를 변환
Why:  양쪽 코드를 모두 변경하지 않고 호환성 확보
```

### 적용 체크리스트

- [ ] 호환되지 않는 두 인터페이스가 존재하는가?
- [ ] Adaptee(기존 클래스)를 직접 수정할 수 없는가? (외부 라이브러리, 레거시 등)
- [ ] 어댑터가 단순 변환만 하는가? (복잡한 로직이 필요하면 다른 패턴 고려)
- [ ] 객체 어댑터(컴포지션)를 우선 고려했는가?
- [ ] Target 인터페이스가 명확하게 정의되어 있는가?

### 실무 적용 시나리오

1. **외부 라이브러리 래핑:** PG사, SMS 발송, 클라우드 서비스 등
2. **레거시 시스템 마이그레이션:** 기존 코드를 새 인터페이스에 맞춤
3. **데이터 포맷 변환:** XML -> JSON, CSV -> 도메인 객체
4. **테스트 더블:** 외부 의존성을 테스트용으로 교체

> **기억하세요:** "기존 코드를 건드리지 않고, 중간에서 통역해주는 것이 어댑터의 역할입니다."
