# Factory Method 패턴 (팩토리 메서드)

> **핵심 한줄 요약**: 객체 생성을 서브클래스에 위임하여, 어떤 클래스의 인스턴스를 만들지를 서브클래스가 결정하게 한다.

---

## 목차

1. [의도 (Intent)](#1-의도-intent)
2. [문제 상황 (Problem)](#2-문제-상황-problem)
3. [UML 다이어그램](#3-uml-다이어그램)
4. [Simple Factory vs Factory Method](#4-simple-factory-vs-factory-method)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 알림 시스템](#7-실무-예제-알림-시스템)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 의도 (Intent)

Factory Method 패턴은 **객체 생성 인터페이스를 정의하되, 실제로 어떤 클래스를 인스턴스화할지는 서브클래스가 결정**하도록 한다.

핵심 아이디어는 다음과 같다:

- `new ConcreteProduct()` 를 직접 호출하는 대신
- `createProduct()` 라는 메서드를 통해 생성을 **한 단계 추상화**한다
- 서브클래스에서 이 메서드를 **오버라이드**하여 적절한 객체를 반환한다

### 실생활 비유

```
물류 회사에서 배송 수단을 결정하는 상황을 생각해 보자.

초기: 트럭만 있었다 → new Truck() 으로 충분
성장: 선박, 항공이 추가됨 → 매번 코드를 수정해야 할까?

Factory Method를 적용하면:
- "운송 수단을 만들어라" 라는 인터페이스만 정의
- 육상물류팀 → 트럭 생성
- 해상물류팀 → 선박 생성
- 항공물류팀 → 비행기 생성

→ 새로운 운송 수단이 추가되어도 기존 코드를 수정할 필요 없다
```

---

## 2. 문제 상황 (Problem)

### 왜 필요한가?

다음과 같은 코드가 있다고 하자:

```python
# 문제: 타입에 따라 if/elif 분기가 계속 늘어난다
class NotificationService:
    def send(self, notification_type, message):
        if notification_type == "email":
            # 이메일 전송 로직 (20줄)
            print(f"이메일 전송: {message}")
        elif notification_type == "sms":
            # SMS 전송 로직 (20줄)
            print(f"SMS 전송: {message}")
        elif notification_type == "push":
            # 푸시 전송 로직 (20줄)
            print(f"푸시 전송: {message}")
        # 새로운 타입이 추가될 때마다 elif를 추가해야 한다!
        # → OCP(개방-폐쇄 원칙) 위반
```

### 이 코드의 문제점

| 문제 | 설명 |
|------|------|
| **OCP 위반** | 새로운 알림 타입 추가 시 기존 코드를 수정해야 함 |
| **SRP 위반** | 하나의 클래스가 모든 알림 타입의 로직을 담당 |
| **높은 결합도** | 구체 클래스에 직접 의존 |
| **테스트 어려움** | 개별 알림 타입을 독립적으로 테스트하기 어려움 |
| **코드 비대화** | 타입이 늘어날수록 클래스가 거대해짐 |

---

## 3. UML 다이어그램

```
┌─────────────────────────┐
│    Creator (추상 클래스)   │
├─────────────────────────┤
│                         │
├─────────────────────────┤
│ + someOperation()       │──────── 팩토리 메서드를 호출하여
│ + createProduct():      │         Product를 생성하고 사용
│   Product {abstract}    │
└────────┬────────────────┘
         │ 상속
    ┌────┴────────────────┐
    │                     │
┌───┴──────────────┐  ┌──┴───────────────┐
│ ConcreteCreatorA │  │ ConcreteCreatorB │
├──────────────────┤  ├──────────────────┤
│ + createProduct()│  │ + createProduct()│
│   : ProductA     │  │   : ProductB     │
└──────────────────┘  └──────────────────┘

┌─────────────────────────┐
│  Product (인터페이스)      │
├─────────────────────────┤
│ + operation()           │
└────────┬────────────────┘
         │ 구현
    ┌────┴────────────────┐
    │                     │
┌───┴──────────┐   ┌─────┴────────┐
│  ProductA    │   │  ProductB    │
├──────────────┤   ├──────────────┤
│ + operation()│   │ + operation()│
└──────────────┘   └──────────────┘

[흐름]
1. 클라이언트 → ConcreteCreator.someOperation() 호출
2. someOperation() 내부에서 createProduct() 호출
3. ConcreteCreator가 적절한 Product를 생성하여 반환
4. someOperation()은 Product 인터페이스만 알고 있음
```

---

## 4. Simple Factory vs Factory Method

### Simple Factory (단순 팩토리)

Simple Factory는 사실 GoF 패턴이 아니라 **프로그래밍 관용구(idiom)** 이다. 하지만 실무에서 자주 사용되므로 비교를 위해 먼저 살펴보자.

```python
# Simple Factory: 하나의 팩토리 클래스가 조건에 따라 객체를 생성
class NotificationFactory:
    @staticmethod
    def create(notification_type: str) -> 'Notification':
        if notification_type == "email":
            return EmailNotification()
        elif notification_type == "sms":
            return SMSNotification()
        elif notification_type == "push":
            return PushNotification()
        raise ValueError(f"Unknown type: {notification_type}")
```

### Factory Method (팩토리 메서드)

```python
# Factory Method: 서브클래스가 생성할 객체를 결정
from abc import ABC, abstractmethod

class NotificationCreator(ABC):
    @abstractmethod
    def create_notification(self) -> 'Notification':
        """서브클래스가 구현 - 어떤 Notification을 만들지 결정"""
        pass

class EmailNotificationCreator(NotificationCreator):
    def create_notification(self) -> 'Notification':
        return EmailNotification()

class SMSNotificationCreator(NotificationCreator):
    def create_notification(self) -> 'Notification':
        return SMSNotification()
```

### 비교표

| 항목 | Simple Factory | Factory Method |
|------|---------------|----------------|
| **구조** | 하나의 팩토리 클래스에 if/elif | 서브클래스에서 오버라이드 |
| **확장** | 새 타입 추가 시 팩토리 코드 수정 | 새 서브클래스만 추가 |
| **OCP** | 위반 (기존 코드 수정 필요) | 준수 (기존 코드 변경 없음) |
| **복잡도** | 낮음 | 중간 |
| **사용 시점** | 타입이 적고 변경이 드문 경우 | 타입이 자주 추가되는 경우 |
| **GoF 패턴** | 아님 (관용구) | 공식 패턴 |

---

## 5. Python 구현

### 기본 구현: 문서 생성기

```python
from abc import ABC, abstractmethod


# ─── Product 인터페이스 ───
class Document(ABC):
    """문서 인터페이스 (Product)"""

    @abstractmethod
    def create(self) -> str:
        pass

    @abstractmethod
    def save(self, filename: str) -> str:
        pass

    def __str__(self):
        return self.__class__.__name__


# ─── Concrete Products ───
class PDFDocument(Document):
    def create(self) -> str:
        return "PDF 문서를 생성했습니다."

    def save(self, filename: str) -> str:
        return f"{filename}.pdf 로 저장했습니다."


class WordDocument(Document):
    def create(self) -> str:
        return "Word 문서를 생성했습니다."

    def save(self, filename: str) -> str:
        return f"{filename}.docx 로 저장했습니다."


class SpreadsheetDocument(Document):
    def create(self) -> str:
        return "스프레드시트를 생성했습니다."

    def save(self, filename: str) -> str:
        return f"{filename}.xlsx 로 저장했습니다."


# ─── Creator (추상 클래스) ───
class DocumentCreator(ABC):
    """문서 생성자 (Creator)

    팩토리 메서드(create_document)를 정의하고,
    문서 처리 파이프라인(process)에서 이를 사용한다.
    """

    @abstractmethod
    def create_document(self) -> Document:
        """팩토리 메서드: 서브클래스가 어떤 문서를 만들지 결정한다"""
        pass

    def process(self, filename: str) -> list[str]:
        """템플릿 메서드: 문서 처리 파이프라인

        팩토리 메서드를 호출하여 문서를 만들고,
        생성 → 저장의 흐름을 처리한다.
        """
        document = self.create_document()  # 팩토리 메서드 호출
        results = []
        results.append(f"문서 타입: {document}")
        results.append(document.create())
        results.append(document.save(filename))
        return results


# ─── Concrete Creators ───
class PDFCreator(DocumentCreator):
    def create_document(self) -> Document:
        return PDFDocument()


class WordCreator(DocumentCreator):
    def create_document(self) -> Document:
        return WordDocument()


class SpreadsheetCreator(DocumentCreator):
    def create_document(self) -> Document:
        return SpreadsheetDocument()


# ─── 클라이언트 코드 ───
def client_code(creator: DocumentCreator, filename: str):
    """클라이언트는 Creator의 구체 타입을 몰라도 된다."""
    results = creator.process(filename)
    for line in results:
        print(f"  {line}")


if __name__ == "__main__":
    print("=== PDF 문서 생성 ===")
    client_code(PDFCreator(), "report")

    print("\n=== Word 문서 생성 ===")
    client_code(WordCreator(), "letter")

    print("\n=== 스프레드시트 생성 ===")
    client_code(SpreadsheetCreator(), "data")
```

**출력:**
```
=== PDF 문서 생성 ===
  문서 타입: PDFDocument
  PDF 문서를 생성했습니다.
  report.pdf 로 저장했습니다.

=== Word 문서 생성 ===
  문서 타입: WordDocument
  Word 문서를 생성했습니다.
  letter.docx 로 저장했습니다.

=== 스프레드시트 생성 ===
  문서 타입: SpreadsheetDocument
  스프레드시트를 생성했습니다.
  data.xlsx 로 저장했습니다.
```

### 매개변수가 있는 Factory Method

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from enum import Enum


class PaymentType(Enum):
    CREDIT_CARD = "credit_card"
    BANK_TRANSFER = "bank_transfer"
    DIGITAL_WALLET = "digital_wallet"


@dataclass
class PaymentResult:
    success: bool
    transaction_id: str
    message: str


# ─── Product ───
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount: float) -> PaymentResult:
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> PaymentResult:
        pass


class CreditCardProcessor(PaymentProcessor):
    def process(self, amount: float) -> PaymentResult:
        # 실제로는 PG사 API 호출
        return PaymentResult(True, "CC-12345", f"카드 결제 {amount}원 완료")

    def refund(self, transaction_id: str) -> PaymentResult:
        return PaymentResult(True, transaction_id, "카드 결제 취소 완료")


class BankTransferProcessor(PaymentProcessor):
    def process(self, amount: float) -> PaymentResult:
        return PaymentResult(True, "BT-67890", f"계좌이체 {amount}원 완료")

    def refund(self, transaction_id: str) -> PaymentResult:
        return PaymentResult(True, transaction_id, "계좌이체 환불 완료")


class DigitalWalletProcessor(PaymentProcessor):
    def process(self, amount: float) -> PaymentResult:
        return PaymentResult(True, "DW-11111", f"디지털 지갑 {amount}원 결제 완료")

    def refund(self, transaction_id: str) -> PaymentResult:
        return PaymentResult(True, transaction_id, "디지털 지갑 환불 완료")


# ─── Creator: 매개변수화된 팩토리 메서드 ───
class PaymentService(ABC):
    """결제 서비스 - 팩토리 메서드를 활용"""

    @abstractmethod
    def create_processor(self) -> PaymentProcessor:
        pass

    def pay(self, amount: float) -> PaymentResult:
        processor = self.create_processor()
        print(f"[결제 시작] 금액: {amount}원, 수단: {processor.__class__.__name__}")
        result = processor.process(amount)
        if result.success:
            print(f"[결제 성공] {result.message}")
        return result


class CreditCardPaymentService(PaymentService):
    def create_processor(self) -> PaymentProcessor:
        return CreditCardProcessor()


class BankTransferPaymentService(PaymentService):
    def create_processor(self) -> PaymentProcessor:
        return BankTransferProcessor()


class DigitalWalletPaymentService(PaymentService):
    def create_processor(self) -> PaymentProcessor:
        return DigitalWalletProcessor()


# ─── 사용 ───
services = {
    PaymentType.CREDIT_CARD: CreditCardPaymentService(),
    PaymentType.BANK_TRANSFER: BankTransferPaymentService(),
    PaymentType.DIGITAL_WALLET: DigitalWalletPaymentService(),
}

# 사용자가 선택한 결제 수단
user_choice = PaymentType.CREDIT_CARD
service = services[user_choice]
result = service.pay(50000)
```

---

## 6. C# 구현

### 기본 구현

```csharp
using System;

// ─── Product 인터페이스 ───
public interface IDocument
{
    string Create();
    string Save(string filename);
}

// ─── Concrete Products ───
public class PdfDocument : IDocument
{
    public string Create() => "PDF 문서를 생성했습니다.";
    public string Save(string filename) => $"{filename}.pdf 로 저장했습니다.";
}

public class WordDocument : IDocument
{
    public string Create() => "Word 문서를 생성했습니다.";
    public string Save(string filename) => $"{filename}.docx 로 저장했습니다.";
}

public class SpreadsheetDocument : IDocument
{
    public string Create() => "스프레드시트를 생성했습니다.";
    public string Save(string filename) => $"{filename}.xlsx 로 저장했습니다.";
}

// ─── Creator (추상 클래스) ───
public abstract class DocumentCreator
{
    /// <summary>
    /// 팩토리 메서드: 서브클래스가 어떤 문서를 만들지 결정한다
    /// </summary>
    public abstract IDocument CreateDocument();

    /// <summary>
    /// 문서 처리 파이프라인 - 팩토리 메서드를 내부에서 호출
    /// </summary>
    public void Process(string filename)
    {
        // 팩토리 메서드 호출 - 어떤 문서가 올지 모름
        IDocument document = CreateDocument();

        Console.WriteLine($"  문서 타입: {document.GetType().Name}");
        Console.WriteLine($"  {document.Create()}");
        Console.WriteLine($"  {document.Save(filename)}");
    }
}

// ─── Concrete Creators ───
public class PdfCreator : DocumentCreator
{
    public override IDocument CreateDocument() => new PdfDocument();
}

public class WordCreator : DocumentCreator
{
    public override IDocument CreateDocument() => new WordDocument();
}

public class SpreadsheetCreator : DocumentCreator
{
    public override IDocument CreateDocument() => new SpreadsheetDocument();
}

// ─── 클라이언트 코드 ───
public class Program
{
    /// <summary>
    /// 클라이언트는 Creator의 구체 타입을 몰라도 된다
    /// </summary>
    static void ClientCode(DocumentCreator creator, string filename)
    {
        creator.Process(filename);
    }

    static void Main()
    {
        Console.WriteLine("=== PDF 문서 생성 ===");
        ClientCode(new PdfCreator(), "report");

        Console.WriteLine("\n=== Word 문서 생성 ===");
        ClientCode(new WordCreator(), "letter");

        Console.WriteLine("\n=== 스프레드시트 생성 ===");
        ClientCode(new SpreadsheetCreator(), "data");
    }
}
```

### 제네릭을 활용한 Factory Method

```csharp
using System;

/// <summary>
/// 제네릭을 활용하면 Creator 하나로 여러 Product를 커버할 수 있다
/// (단, new() 제약 조건이 필요하다)
/// </summary>
public abstract class GenericDocumentCreator<TDocument>
    where TDocument : IDocument, new()
{
    public IDocument CreateDocument()
    {
        return new TDocument();
    }

    public void Process(string filename)
    {
        var doc = CreateDocument();
        Console.WriteLine($"  {doc.Create()}");
        Console.WriteLine($"  {doc.Save(filename)}");
    }
}

// 사용: 한 줄로 Creator 정의 가능
public class PdfCreatorGeneric : GenericDocumentCreator<PdfDocument> { }
public class WordCreatorGeneric : GenericDocumentCreator<WordDocument> { }
```

---

## 7. 실무 예제: 알림 시스템

### 시나리오

서비스에서 사용자에게 알림을 보내야 한다. 알림 수단은 Email, SMS, Push 세 가지이며, 향후 Slack, 카카오톡 등이 추가될 수 있다.

### Python 구현

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime
from typing import Optional


@dataclass
class NotificationResult:
    """알림 발송 결과"""
    success: bool
    channel: str
    recipient: str
    sent_at: datetime
    message_id: Optional[str] = None
    error: Optional[str] = None


# ─── Product: 알림 채널 인터페이스 ───
class NotificationChannel(ABC):
    """알림 채널 (Product)"""

    @abstractmethod
    def send(self, recipient: str, title: str, body: str) -> NotificationResult:
        pass

    @abstractmethod
    def validate_recipient(self, recipient: str) -> bool:
        pass

    @property
    @abstractmethod
    def channel_name(self) -> str:
        pass


class EmailChannel(NotificationChannel):
    """이메일 알림 채널"""

    @property
    def channel_name(self) -> str:
        return "Email"

    def validate_recipient(self, recipient: str) -> bool:
        return "@" in recipient and "." in recipient

    def send(self, recipient: str, title: str, body: str) -> NotificationResult:
        if not self.validate_recipient(recipient):
            return NotificationResult(
                success=False, channel=self.channel_name,
                recipient=recipient, sent_at=datetime.now(),
                error="유효하지 않은 이메일 주소"
            )
        # 실제로는 SMTP 서버를 통해 전송
        print(f"    [Email] To: {recipient}")
        print(f"    [Email] Subject: {title}")
        print(f"    [Email] Body: {body}")
        return NotificationResult(
            success=True, channel=self.channel_name,
            recipient=recipient, sent_at=datetime.now(),
            message_id="EMAIL-001"
        )


class SMSChannel(NotificationChannel):
    """SMS 알림 채널"""

    @property
    def channel_name(self) -> str:
        return "SMS"

    def validate_recipient(self, recipient: str) -> bool:
        return recipient.startswith("010") and len(recipient) == 11

    def send(self, recipient: str, title: str, body: str) -> NotificationResult:
        if not self.validate_recipient(recipient):
            return NotificationResult(
                success=False, channel=self.channel_name,
                recipient=recipient, sent_at=datetime.now(),
                error="유효하지 않은 전화번호"
            )
        # 실제로는 SMS API 호출
        message = f"{title}: {body}"
        if len(message) > 80:
            message = message[:77] + "..."
        print(f"    [SMS] To: {recipient}")
        print(f"    [SMS] Message: {message}")
        return NotificationResult(
            success=True, channel=self.channel_name,
            recipient=recipient, sent_at=datetime.now(),
            message_id="SMS-001"
        )


class PushChannel(NotificationChannel):
    """푸시 알림 채널"""

    @property
    def channel_name(self) -> str:
        return "Push"

    def validate_recipient(self, recipient: str) -> bool:
        return len(recipient) > 0  # device token 검증

    def send(self, recipient: str, title: str, body: str) -> NotificationResult:
        if not self.validate_recipient(recipient):
            return NotificationResult(
                success=False, channel=self.channel_name,
                recipient=recipient, sent_at=datetime.now(),
                error="유효하지 않은 디바이스 토큰"
            )
        # 실제로는 FCM/APNs API 호출
        print(f"    [Push] Device: {recipient[:20]}...")
        print(f"    [Push] Title: {title}")
        print(f"    [Push] Body: {body}")
        return NotificationResult(
            success=True, channel=self.channel_name,
            recipient=recipient, sent_at=datetime.now(),
            message_id="PUSH-001"
        )


# ─── Creator: 알림 발송 서비스 ───
class NotificationService(ABC):
    """알림 발송 서비스 (Creator)

    팩토리 메서드(create_channel)를 정의하고,
    알림 발송 파이프라인(notify)에서 이를 사용한다.
    """

    @abstractmethod
    def create_channel(self) -> NotificationChannel:
        """팩토리 메서드"""
        pass

    def notify(self, recipient: str, title: str, body: str) -> NotificationResult:
        """알림 발송 파이프라인"""
        channel = self.create_channel()

        print(f"\n  [{channel.channel_name}] 알림 발송 시작")
        print(f"  {'─' * 40}")

        # 수신자 검증
        if not channel.validate_recipient(recipient):
            print(f"  [실패] 유효하지 않은 수신자: {recipient}")
            return NotificationResult(
                success=False, channel=channel.channel_name,
                recipient=recipient, sent_at=datetime.now(),
                error="수신자 검증 실패"
            )

        # 발송
        result = channel.send(recipient, title, body)

        # 결과 로깅
        status = "성공" if result.success else "실패"
        print(f"  [결과] {status} (ID: {result.message_id})")

        return result


# ─── Concrete Creators ───
class EmailNotificationService(NotificationService):
    def create_channel(self) -> NotificationChannel:
        return EmailChannel()


class SMSNotificationService(NotificationService):
    def create_channel(self) -> NotificationChannel:
        return SMSChannel()


class PushNotificationService(NotificationService):
    def create_channel(self) -> NotificationChannel:
        return PushChannel()


# ─── 사용 예시 ───
if __name__ == "__main__":
    print("=" * 50)
    print("알림 시스템 - Factory Method 패턴 적용")
    print("=" * 50)

    # 이메일 알림
    email_service = EmailNotificationService()
    email_service.notify(
        "user@example.com",
        "주문 완료",
        "주문번호 #12345가 접수되었습니다."
    )

    # SMS 알림
    sms_service = SMSNotificationService()
    sms_service.notify(
        "01012345678",
        "배송 시작",
        "상품이 배송을 시작했습니다."
    )

    # 푸시 알림
    push_service = PushNotificationService()
    push_service.notify(
        "device_token_abc123xyz",
        "리뷰 요청",
        "최근 구매하신 상품의 리뷰를 작성해 주세요."
    )
```

### C# 구현

```csharp
using System;

// ─── Product: 알림 채널 인터페이스 ───
public interface INotificationChannel
{
    string ChannelName { get; }
    bool ValidateRecipient(string recipient);
    NotificationResult Send(string recipient, string title, string body);
}

public record NotificationResult(
    bool Success,
    string Channel,
    string Recipient,
    DateTime SentAt,
    string? MessageId = null,
    string? Error = null
);

// ─── Concrete Products ───
public class EmailChannel : INotificationChannel
{
    public string ChannelName => "Email";

    public bool ValidateRecipient(string recipient)
        => recipient.Contains('@') && recipient.Contains('.');

    public NotificationResult Send(string recipient, string title, string body)
    {
        if (!ValidateRecipient(recipient))
            return new NotificationResult(false, ChannelName, recipient,
                DateTime.Now, Error: "유효하지 않은 이메일 주소");

        Console.WriteLine($"    [Email] To: {recipient}");
        Console.WriteLine($"    [Email] Subject: {title}");
        Console.WriteLine($"    [Email] Body: {body}");

        return new NotificationResult(true, ChannelName, recipient,
            DateTime.Now, "EMAIL-001");
    }
}

public class SmsChannel : INotificationChannel
{
    public string ChannelName => "SMS";

    public bool ValidateRecipient(string recipient)
        => recipient.StartsWith("010") && recipient.Length == 11;

    public NotificationResult Send(string recipient, string title, string body)
    {
        if (!ValidateRecipient(recipient))
            return new NotificationResult(false, ChannelName, recipient,
                DateTime.Now, Error: "유효하지 않은 전화번호");

        var message = $"{title}: {body}";
        if (message.Length > 80) message = message[..77] + "...";

        Console.WriteLine($"    [SMS] To: {recipient}");
        Console.WriteLine($"    [SMS] Message: {message}");

        return new NotificationResult(true, ChannelName, recipient,
            DateTime.Now, "SMS-001");
    }
}

public class PushChannel : INotificationChannel
{
    public string ChannelName => "Push";

    public bool ValidateRecipient(string recipient)
        => !string.IsNullOrEmpty(recipient);

    public NotificationResult Send(string recipient, string title, string body)
    {
        if (!ValidateRecipient(recipient))
            return new NotificationResult(false, ChannelName, recipient,
                DateTime.Now, Error: "유효하지 않은 디바이스 토큰");

        Console.WriteLine($"    [Push] Device: {recipient[..Math.Min(20, recipient.Length)]}...");
        Console.WriteLine($"    [Push] Title: {title}");
        Console.WriteLine($"    [Push] Body: {body}");

        return new NotificationResult(true, ChannelName, recipient,
            DateTime.Now, "PUSH-001");
    }
}

// ─── Creator: 알림 발송 서비스 ───
public abstract class NotificationService
{
    /// <summary>팩토리 메서드</summary>
    protected abstract INotificationChannel CreateChannel();

    /// <summary>알림 발송 파이프라인</summary>
    public NotificationResult Notify(string recipient, string title, string body)
    {
        var channel = CreateChannel();

        Console.WriteLine($"\n  [{channel.ChannelName}] 알림 발송 시작");
        Console.WriteLine($"  {new string('─', 40)}");

        if (!channel.ValidateRecipient(recipient))
        {
            Console.WriteLine($"  [실패] 유효하지 않은 수신자: {recipient}");
            return new NotificationResult(false, channel.ChannelName,
                recipient, DateTime.Now, Error: "수신자 검증 실패");
        }

        var result = channel.Send(recipient, title, body);

        var status = result.Success ? "성공" : "실패";
        Console.WriteLine($"  [결과] {status} (ID: {result.MessageId})");

        return result;
    }
}

// ─── Concrete Creators ───
public class EmailNotificationService : NotificationService
{
    protected override INotificationChannel CreateChannel() => new EmailChannel();
}

public class SmsNotificationService : NotificationService
{
    protected override INotificationChannel CreateChannel() => new SmsChannel();
}

public class PushNotificationService : NotificationService
{
    protected override INotificationChannel CreateChannel() => new PushChannel();
}

// ─── 사용 ───
public class Program
{
    static void Main()
    {
        Console.WriteLine(new string('=', 50));
        Console.WriteLine("알림 시스템 - Factory Method 패턴 적용");
        Console.WriteLine(new string('=', 50));

        NotificationService emailService = new EmailNotificationService();
        emailService.Notify("user@example.com", "주문 완료",
            "주문번호 #12345가 접수되었습니다.");

        NotificationService smsService = new SmsNotificationService();
        smsService.Notify("01012345678", "배송 시작",
            "상품이 배송을 시작했습니다.");

        NotificationService pushService = new PushNotificationService();
        pushService.Notify("device_token_abc123xyz", "리뷰 요청",
            "최근 구매하신 상품의 리뷰를 작성해 주세요.");
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **OCP 준수** | 새로운 제품 타입 추가 시 기존 코드를 수정하지 않아도 됨 |
| **SRP 준수** | 제품 생성 코드를 한 곳에 집중 |
| **느슨한 결합** | Creator가 구체 Product를 직접 참조하지 않음 |
| **확장 용이** | 새로운 Creator/Product 쌍을 추가하기만 하면 됨 |
| **테스트 용이** | 각 Product를 독립적으로 테스트 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **클래스 수 증가** | 제품마다 Creator 서브클래스가 필요하여 클래스가 많아짐 |
| **복잡도 증가** | 간단한 경우에는 Simple Factory로 충분할 수 있음 |
| **병렬 계층** | Product 계층과 Creator 계층이 병렬로 늘어남 |

### 판단 기준

```
Factory Method를 적용해야 할 때:
  - 제품 타입이 자주 추가/변경되는 경우
  - 제품 생성 과정이 복잡한 경우
  - 제품 생성을 서브클래스에 위임해야 하는 경우

Simple Factory로 충분할 때:
  - 제품 타입이 3개 이하이고 거의 변하지 않는 경우
  - 빠른 프로토타이핑이 필요한 경우
```

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Abstract Factory** | Factory Method의 집합. 관련 객체 가족을 한꺼번에 생성 |
| **Template Method** | Factory Method는 종종 Template Method 안에서 호출됨 |
| **Prototype** | Factory Method의 대안. 상속 대신 복제를 활용 |
| **Builder** | Factory Method와 함께 사용하여 복잡한 객체를 생성 |
| **Singleton** | 팩토리 클래스를 싱글톤으로 구현하는 경우가 많음 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

```
Factory Method 패턴 = 객체 생성을 서브클래스에 위임

[핵심 구조]
- Product: 생성될 객체의 인터페이스
- ConcreteProduct: Product의 구체 구현
- Creator: 팩토리 메서드를 선언하는 추상 클래스
- ConcreteCreator: 팩토리 메서드를 구현하여 ConcreteProduct 반환

[Simple Factory vs Factory Method]
- Simple Factory: 조건 분기로 객체 생성 (OCP 위반)
- Factory Method: 서브클래스 오버라이드로 객체 생성 (OCP 준수)

[실무 포인트]
- 타입이 자주 추가되는 경우에 적합
- 프레임워크에서 확장 포인트로 자주 사용
- 너무 단순한 경우에는 과도한 설계가 될 수 있음
```

### 학습 체크리스트

- [ ] Factory Method의 의도를 한 문장으로 설명할 수 있다
- [ ] UML 다이어그램의 4가지 역할(Product, ConcreteProduct, Creator, ConcreteCreator)을 이해한다
- [ ] Simple Factory와 Factory Method의 차이를 코드로 설명할 수 있다
- [ ] Python에서 ABC를 활용한 Factory Method를 구현할 수 있다
- [ ] C#에서 abstract class와 override를 활용한 Factory Method를 구현할 수 있다
- [ ] Factory Method가 OCP를 어떻게 지원하는지 설명할 수 있다
- [ ] Factory Method의 장단점을 이해하고, 적용 여부를 판단할 수 있다
- [ ] 실무에서 Factory Method가 적합한 시나리오를 3개 이상 열거할 수 있다

### 다음 학습

Factory Method가 **하나의 제품**을 생성하는 패턴이라면, 다음으로 배울 [Abstract Factory 패턴](./03-abstract-factory.md)은 **관련된 제품 가족 전체**를 생성하는 패턴이다.
