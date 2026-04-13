# I - 인터페이스 분리 원칙 (Interface Segregation Principle)

> **SOLID 원칙 4/5**
> **대상 독자**: 2~3년차 개발자
> **핵심 문장**: "클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강제되어서는 안 된다." — Robert C. Martin

---

## 목차

1. [ISP란 무엇인가?](#1-isp란-무엇인가)
2. [왜 ISP가 중요한가?](#2-왜-isp가-중요한가)
3. [ISP 위반 사례 (Before)](#3-isp-위반-사례-before)
4. [ISP 적용 사례 (After)](#4-isp-적용-사례-after)
5. [Python에서의 ISP: ABC와 Protocol](#5-python에서의-isp-abc와-protocol)
6. [실무 팁](#6-실무-팁)
7. [정리 및 체크리스트](#7-정리-및-체크리스트)
8. [다음 단계](#8-다음-단계)

---

## 1. ISP란 무엇인가?

### 정의

> **"하나의 범용 인터페이스보다 여러 개의 구체적인 인터페이스가 낫다."**
>
> 클라이언트(사용하는 쪽)가 필요하지 않은 메서드에 의존하지 않도록, **인터페이스를 역할별로 분리**해야 한다.

### 비유

스마트폰의 기능을 생각해 보세요:

- 전화를 걸고 싶은 사람에게 **카메라 매뉴얼까지 읽으라고 강제하지 않습니다**.
- 사진을 찍고 싶은 사람에게 **NFC 설정을 요구하지 않습니다**.
- 각 기능(전화, 카메라, NFC)은 **독립적인 인터페이스**로 접근할 수 있어야 합니다.

### 핵심 개념

```
뚱뚱한 인터페이스 (Fat Interface)
  → 모든 기능을 하나의 인터페이스에 담음
  → 일부만 필요한 구현체도 전부 구현해야 함

날씬한 인터페이스 (Thin Interface / Role Interface)
  → 역할별로 인터페이스를 분리
  → 필요한 역할만 선택적으로 구현
```

---

## 2. 왜 ISP가 중요한가?

### ISP를 지키면

- **불필요한 의존성 제거**: 사용하지 않는 메서드 변경에 영향받지 않음
- **구현 부담 감소**: 필요한 메서드만 구현하면 됨
- **LSP 준수 지원**: 빈 구현이나 예외를 던지는 메서드가 사라짐
- **코드 이해도 향상**: 인터페이스 이름만으로 역할 파악 가능

### ISP를 무시하면

- **강제 구현**: 필요 없는 메서드를 `pass`나 `NotImplementedError`로 구현
- **LSP 위반 유발**: 빈 구현이 부모의 계약을 깨뜨림
- **변경 파급 효과**: 사용하지 않는 메서드가 바뀌어도 재컴파일/재배포 필요
- **혼란**: 어떤 메서드가 실제로 동작하고 어떤 것이 가짜인지 알 수 없음

---

## 3. ISP 위반 사례 (Before)

### 시나리오: 복합기(All-in-One) 인터페이스

사무실에는 다양한 기기가 있습니다:
- **복합기**: 프린트 + 스캔 + 팩스 + 복사 모두 가능
- **단순 프린터**: 프린트만 가능
- **스캐너**: 스캔만 가능

### ❌ Python - 나쁜 예: 뚱뚱한 인터페이스

```python
from abc import ABC, abstractmethod


class IMultiFunctionDevice(ABC):
    """
    ❌ ISP 위반! 뚱뚱한 인터페이스.
    모든 기기가 이 인터페이스의 모든 메서드를 구현해야 한다.
    """

    @abstractmethod
    def print_document(self, document: str) -> None:
        pass

    @abstractmethod
    def scan_document(self) -> str:
        pass

    @abstractmethod
    def fax_document(self, document: str, number: str) -> None:
        pass

    @abstractmethod
    def copy_document(self) -> None:
        pass

    @abstractmethod
    def staple_pages(self) -> None:
        pass


class AllInOnePrinter(IMultiFunctionDevice):
    """복합기 - 모든 기능 사용 → 문제 없음"""

    def print_document(self, document: str) -> None:
        print(f"[복합기] 인쇄 중: {document}")

    def scan_document(self) -> str:
        return "[복합기] 스캔 완료"

    def fax_document(self, document: str, number: str) -> None:
        print(f"[복합기] {number}로 팩스 전송: {document}")

    def copy_document(self) -> None:
        print("[복합기] 복사 중...")

    def staple_pages(self) -> None:
        print("[복합기] 스테이플러 처리 중...")


class SimplePrinter(IMultiFunctionDevice):
    """
    ❌ 단순 프린터 - 인쇄만 가능한데 모든 메서드를 구현해야 함!
    사용하지 않는 메서드를 억지로 구현해야 하는 불합리함.
    """

    def print_document(self, document: str) -> None:
        print(f"[프린터] 인쇄 중: {document}")

    def scan_document(self) -> str:
        # ❌ 할 수 없는 기능을 예외로 처리 → LSP도 위반!
        raise NotImplementedError("단순 프린터는 스캔을 지원하지 않습니다.")

    def fax_document(self, document: str, number: str) -> None:
        # ❌ 빈 구현 → 호출자는 이게 진짜 동작하는지 알 수 없음
        raise NotImplementedError("단순 프린터는 팩스를 지원하지 않습니다.")

    def copy_document(self) -> None:
        raise NotImplementedError("단순 프린터는 복사를 지원하지 않습니다.")

    def staple_pages(self) -> None:
        raise NotImplementedError("단순 프린터는 스테이플러를 지원하지 않습니다.")


class OldScanner(IMultiFunctionDevice):
    """
    ❌ 구형 스캐너 - 스캔만 가능한데 5개 메서드를 모두 구현해야!
    """

    def print_document(self, document: str) -> None:
        raise NotImplementedError("스캐너는 인쇄를 지원하지 않습니다.")

    def scan_document(self) -> str:
        return "[스캐너] 스캔 완료"

    def fax_document(self, document: str, number: str) -> None:
        raise NotImplementedError("스캐너는 팩스를 지원하지 않습니다.")

    def copy_document(self) -> None:
        raise NotImplementedError("스캐너는 복사를 지원하지 않습니다.")

    def staple_pages(self) -> None:
        raise NotImplementedError("스캐너는 스테이플러를 지원하지 않습니다.")
```

### ❌ C# - 나쁜 예: 뚱뚱한 인터페이스

```csharp
public interface IMultiFunctionDevice
{
    // ❌ 모든 기기가 이 5개 메서드를 모두 구현해야 함
    void PrintDocument(string document);
    string ScanDocument();
    void FaxDocument(string document, string number);
    void CopyDocument();
    void StaplePages();
}

public class SimplePrinter : IMultiFunctionDevice
{
    public void PrintDocument(string document)
        => Console.WriteLine($"[프린터] 인쇄 중: {document}");

    // ❌ 사용하지 않는 메서드를 모두 구현해야 함
    public string ScanDocument()
        => throw new NotSupportedException("스캔 미지원");

    public void FaxDocument(string document, string number)
        => throw new NotSupportedException("팩스 미지원");

    public void CopyDocument()
        => throw new NotSupportedException("복사 미지원");

    public void StaplePages()
        => throw new NotSupportedException("스테이플러 미지원");
}
```

**문제점**:
- `SimplePrinter`는 인쇄만 하면 되는데 5개 메서드를 모두 구현해야 함
- `NotImplementedError`를 던지는 메서드가 있으면 LSP도 위반
- `IMultiFunctionDevice`에 새 메서드가 추가되면 **모든 구현체가 수정**되어야 함
- 호출하는 쪽은 어떤 메서드가 실제로 동작하는지 알 수 없음

---

## 4. ISP 적용 사례 (After)

### ✅ Python - 좋은 예: 역할별 인터페이스 분리

```python
from abc import ABC, abstractmethod


# ──────────────────────────────────────────────
# 역할별 인터페이스 분리
# ──────────────────────────────────────────────
class IPrintable(ABC):
    """인쇄 기능"""

    @abstractmethod
    def print_document(self, document: str) -> None:
        pass


class IScannable(ABC):
    """스캔 기능"""

    @abstractmethod
    def scan_document(self) -> str:
        pass


class IFaxable(ABC):
    """팩스 기능"""

    @abstractmethod
    def fax_document(self, document: str, number: str) -> None:
        pass


class ICopyable(ABC):
    """복사 기능"""

    @abstractmethod
    def copy_document(self) -> None:
        pass


class IStapleable(ABC):
    """스테이플러 기능"""

    @abstractmethod
    def staple_pages(self) -> None:
        pass


# ──────────────────────────────────────────────
# 구현: 필요한 인터페이스만 선택적으로 구현
# ──────────────────────────────────────────────
class AllInOnePrinter(IPrintable, IScannable, IFaxable, ICopyable, IStapleable):
    """복합기 - 모든 기능 구현"""

    def print_document(self, document: str) -> None:
        print(f"[복합기] 인쇄 중: {document}")

    def scan_document(self) -> str:
        return "[복합기] 스캔 완료"

    def fax_document(self, document: str, number: str) -> None:
        print(f"[복합기] {number}로 팩스 전송: {document}")

    def copy_document(self) -> None:
        print("[복합기] 복사 중...")

    def staple_pages(self) -> None:
        print("[복합기] 스테이플러 처리 중...")


class SimplePrinter(IPrintable):
    """
    ✅ 단순 프린터 - 인쇄만 구현!
    불필요한 메서드를 강제로 구현할 필요 없음.
    """

    def print_document(self, document: str) -> None:
        print(f"[프린터] 인쇄 중: {document}")


class OldScanner(IScannable):
    """✅ 구형 스캐너 - 스캔만 구현!"""

    def scan_document(self) -> str:
        return "[스캐너] 스캔 완료"


class PrinterScanner(IPrintable, IScannable):
    """✅ 프린터 + 스캐너 복합기 - 필요한 것만 구현"""

    def print_document(self, document: str) -> None:
        print(f"[프린터/스캐너] 인쇄 중: {document}")

    def scan_document(self) -> str:
        return "[프린터/스캐너] 스캔 완료"


# ──────────────────────────────────────────────
# 사용: 필요한 인터페이스만 요구
# ──────────────────────────────────────────────
def batch_print(printer: IPrintable, documents: list[str]) -> None:
    """인쇄 기능만 필요 - IPrintable만 요구"""
    for doc in documents:
        printer.print_document(doc)


def scan_and_save(scanner: IScannable, save_path: str) -> None:
    """스캔 기능만 필요 - IScannable만 요구"""
    result = scanner.scan_document()
    print(f"스캔 결과를 '{save_path}'에 저장: {result}")


def print_and_scan(device: "IPrintable & IScannable", document: str) -> str:
    """인쇄 + 스캔이 모두 필요한 경우 (Python 3.12+ Intersection Type 미지원이므로 Protocol 사용 권장)"""
    device.print_document(document)
    return device.scan_document()


# 사용 예시
docs = ["보고서.pdf", "이력서.docx", "계약서.pdf"]

# SimplePrinter는 IPrintable만 구현 → batch_print에 사용 가능
simple = SimplePrinter()
batch_print(simple, docs)

# AllInOnePrinter는 모든 인터페이스 구현 → 어디든 사용 가능
allinone = AllInOnePrinter()
batch_print(allinone, docs)
scan_and_save(allinone, "/tmp/scan.pdf")

# OldScanner는 IScannable만 구현 → scan_and_save에만 사용 가능
scanner = OldScanner()
scan_and_save(scanner, "/tmp/old_scan.pdf")
# batch_print(scanner, docs)  # 타입 에러! IScannable은 IPrintable이 아님
```

### ✅ C# - 좋은 예: 역할별 인터페이스 분리

```csharp
// ── 역할별 인터페이스 ──
public interface IPrintable
{
    void PrintDocument(string document);
}

public interface IScannable
{
    string ScanDocument();
}

public interface IFaxable
{
    void FaxDocument(string document, string number);
}

public interface ICopyable
{
    void CopyDocument();
}

public interface IStapleable
{
    void StaplePages();
}

// ── 구현: 필요한 인터페이스만 선택 ──
public class AllInOnePrinter : IPrintable, IScannable, IFaxable, ICopyable, IStapleable
{
    public void PrintDocument(string document) => Console.WriteLine($"[복합기] 인쇄: {document}");
    public string ScanDocument() => "[복합기] 스캔 완료";
    public void FaxDocument(string document, string number) => Console.WriteLine($"[복합기] 팩스: {number}");
    public void CopyDocument() => Console.WriteLine("[복합기] 복사 중...");
    public void StaplePages() => Console.WriteLine("[복합기] 스테이플러 처리");
}

public class SimplePrinter : IPrintable
{
    // ✅ 인쇄만 구현! 깔끔!
    public void PrintDocument(string document) => Console.WriteLine($"[프린터] 인쇄: {document}");
}

public class OldScanner : IScannable
{
    // ✅ 스캔만 구현!
    public string ScanDocument() => "[스캐너] 스캔 완료";
}

public class PrinterScanner : IPrintable, IScannable
{
    // ✅ 인쇄 + 스캔만 구현
    public void PrintDocument(string document) => Console.WriteLine($"[프린터/스캐너] 인쇄: {document}");
    public string ScanDocument() => "[프린터/스캐너] 스캔 완료";
}

// ── 사용: 필요한 인터페이스만 매개변수로 요구 ──
public class DocumentService
{
    public void BatchPrint(IPrintable printer, IEnumerable<string> documents)
    {
        foreach (var doc in documents)
            printer.PrintDocument(doc);
    }

    public string ScanAndSave(IScannable scanner, string savePath)
    {
        var result = scanner.ScanDocument();
        Console.WriteLine($"스캔 결과를 '{savePath}'에 저장: {result}");
        return result;
    }

    // 인쇄 + 스캔 모두 필요한 경우: 제네릭 제약 활용
    public string PrintAndScan<T>(T device, string document)
        where T : IPrintable, IScannable
    {
        device.PrintDocument(document);
        return device.ScanDocument();
    }
}

// ── 사용 예시 ──
var service = new DocumentService();
var docs = new[] { "보고서.pdf", "이력서.docx" };

var simple = new SimplePrinter();
service.BatchPrint(simple, docs);  // ✅ IPrintable이니까 OK

var scanner = new OldScanner();
service.ScanAndSave(scanner, "/tmp/scan.pdf");  // ✅ IScannable이니까 OK

var combo = new PrinterScanner();
service.PrintAndScan(combo, "계약서.pdf");  // ✅ 두 인터페이스 모두 구현

// service.BatchPrint(scanner, docs);  // ❌ 컴파일 에러! IScannable은 IPrintable이 아님
```

---

## 5. Python에서의 ISP: ABC와 Protocol

Python은 C#이나 Java와 달리 명시적 인터페이스 키워드가 없습니다. 대신 두 가지 방법으로 ISP를 적용합니다.

### 방법 1: ABC (Abstract Base Class) - 명시적 상속

```python
from abc import ABC, abstractmethod


class IPrintable(ABC):
    @abstractmethod
    def print_document(self, document: str) -> None:
        pass


class IScannable(ABC):
    @abstractmethod
    def scan_document(self) -> str:
        pass


# 명시적으로 인터페이스를 상속
class MyPrinter(IPrintable):
    def print_document(self, document: str) -> None:
        print(f"인쇄: {document}")
```

**장점**: 명시적, 상속 누락 시 인스턴스화할 때 에러 발생
**단점**: 다중 상속 필요, 외부 라이브러리 클래스에 적용 어려움

### 방법 2: Protocol (Python 3.8+) - 구조적 타이핑 (덕 타이핑)

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class Printable(Protocol):
    """프로토콜: 이 메서드를 가진 객체는 Printable로 간주"""

    def print_document(self, document: str) -> None: ...


@runtime_checkable
class Scannable(Protocol):
    def scan_document(self) -> str: ...


# 상속 없이도 프로토콜을 만족!
class ThirdPartyPrinter:
    """외부 라이브러리의 프린터 - 우리 인터페이스를 모르지만 사용 가능"""

    def print_document(self, document: str) -> None:
        print(f"[외부 프린터] 인쇄: {document}")


def batch_print(printer: Printable, documents: list[str]) -> None:
    for doc in documents:
        printer.print_document(doc)


# ThirdPartyPrinter는 Printable을 상속하지 않았지만,
# print_document 메서드를 갖고 있으므로 Printable로 사용 가능!
external_printer = ThirdPartyPrinter()
batch_print(external_printer, ["문서1.pdf", "문서2.pdf"])  # ✅ 동작!

# runtime_checkable 덕분에 isinstance 체크도 가능
print(isinstance(external_printer, Printable))  # True
```

**장점**: 상속 불필요, 외부 라이브러리에도 적용 가능, 파이썬스러움(Pythonic)
**단점**: 런타임이 아닌 정적 타입 체커(mypy) 의존, 메서드 시그니처만 체크

### 실무 권장: 두 방법의 사용 기준

| 상황 | 권장 방법 | 이유 |
|------|----------|------|
| 내부 코드, 명확한 계약 필요 | ABC | 구현 누락 시 즉시 에러 |
| 외부 라이브러리와 호환 필요 | Protocol | 상속 없이 인터페이스 적합성 판단 |
| 팀 내 컨벤션이 있는 경우 | 컨벤션 따라 | 일관성이 가장 중요 |
| 간단한 콜백/핸들러 | Protocol | 가볍고 유연 |

---

## 6. 실무 팁

### 인터페이스 분리의 기준

```
질문: "이 인터페이스의 메서드 중, 항상 함께 사용되는 것은?"

항상 함께 사용 → 같은 인터페이스에 유지
따로 사용될 수 있음 → 별도 인터페이스로 분리
```

### 과도한 분리 주의

```python
# ❌ 과도한 분리 - 메서드 하나당 인터페이스 하나는 너무 극단적
class IGetName(ABC):
    @abstractmethod
    def get_name(self) -> str: ...

class IGetEmail(ABC):
    @abstractmethod
    def get_email(self) -> str: ...

class IGetAge(ABC):
    @abstractmethod
    def get_age(self) -> int: ...


# ✅ 적절한 분리 - 역할(role) 단위
class IUserIdentity(ABC):
    """사용자 식별 정보"""
    @abstractmethod
    def get_name(self) -> str: ...

    @abstractmethod
    def get_email(self) -> str: ...

class IUserProfile(ABC):
    """사용자 프로필 정보"""
    @abstractmethod
    def get_age(self) -> int: ...

    @abstractmethod
    def get_bio(self) -> str: ...
```

### 실무에서 자주 보는 ISP 적용 패턴

```python
# 1. CQRS 패턴: 읽기/쓰기 분리
class IReader(ABC):
    @abstractmethod
    def find_by_id(self, id: str) -> dict: ...

    @abstractmethod
    def find_all(self) -> list[dict]: ...

class IWriter(ABC):
    @abstractmethod
    def save(self, data: dict) -> None: ...

    @abstractmethod
    def delete(self, id: str) -> None: ...


# 2. 이벤트 핸들링: 발행/구독 분리
class IEventPublisher(ABC):
    @abstractmethod
    def publish(self, event: str, data: dict) -> None: ...

class IEventSubscriber(ABC):
    @abstractmethod
    def subscribe(self, event: str, handler: callable) -> None: ...


# 3. 인증/인가 분리
class IAuthenticatable(ABC):
    """인증: "너는 누구인가?"를 확인"""
    @abstractmethod
    def authenticate(self, credentials: dict) -> bool: ...

class IAuthorizable(ABC):
    """인가: "너는 이걸 할 수 있는가?"를 확인"""
    @abstractmethod
    def authorize(self, user_id: str, resource: str) -> bool: ...
```

### ISP와 SRP의 관계

```
SRP: 구현(클래스)의 책임을 분리
ISP: 인터페이스(계약)의 책임을 분리

둘 다 "분리"를 다루지만 관점이 다르다:
- SRP → "이 클래스가 변경될 이유가 하나인가?"
- ISP → "이 인터페이스의 클라이언트가 불필요한 메서드에 의존하지 않는가?"
```

---

## 7. 정리 및 체크리스트

### 핵심 요약

| 항목 | 설명 |
|------|------|
| **원칙** | 클라이언트가 사용하지 않는 메서드에 의존하지 않아야 한다 |
| **위반 징후** | `NotImplementedError`, 빈 메서드, 뚱뚱한 인터페이스 |
| **해결 방법** | 역할(role) 단위로 인터페이스 분리 |
| **Python 도구** | ABC (명시적), Protocol (구조적) |
| **C# 도구** | interface + 제네릭 제약 |
| **관련 원칙** | SRP (구현 분리), LSP (빈 구현 방지) |

### 셀프 체크리스트

- [ ] 인터페이스의 모든 메서드를 구현체가 의미 있게 사용하는가?
- [ ] `NotImplementedError`나 빈 구현(`pass`)이 있는 메서드가 없는가?
- [ ] 인터페이스에 새 메서드 추가 시 모든 구현체를 수정해야 하지 않는가?
- [ ] 클라이언트(사용하는 쪽)가 필요한 메서드만 볼 수 있는가?
- [ ] 과도한 분리로 인터페이스가 너무 잘게 쪼개지지 않았는가?
- [ ] Python에서 Protocol과 ABC 중 상황에 맞는 것을 선택했는가?

---

## 8. 다음 단계

ISP로 인터페이스를 역할별로 분리하는 방법을 배웠습니다. 이제 마지막 SOLID 원칙인 **DIP**를 통해, 이렇게 분리한 인터페이스를 **어떻게 연결(주입)하는지** 배울 차례입니다.

**다음 문서**: [06-solid-dip.md - D: 의존성 역전 원칙 (Dependency Inversion Principle)](./06-solid-dip.md)

> "고수준 모듈이 저수준 모듈에 직접 의존하면, 저수준이 바뀔 때마다 고수준도 함께 바뀝니다." 추상화를 사이에 두어 의존성 방향을 역전시키는 방법을 알아봅니다.
