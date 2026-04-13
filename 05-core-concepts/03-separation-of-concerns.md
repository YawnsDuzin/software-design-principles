# 관심사의 분리 (Separation of Concerns, SoC)

> **"레스토랑에서 셰프가 요리하고, 서버가 서빙하고, 캐셔가 계산하듯 소프트웨어도 각 역할을 분리하라."**

## 목차

1. [정의와 개념](#1-정의와-개념)
2. [비유로 이해하기](#2-비유로-이해하기)
3. [다양한 수준의 SoC](#3-다양한-수준의-soc)
4. [나쁜 예제: 관심사가 혼재된 코드](#4-나쁜-예제-관심사가-혼재된-코드)
5. [좋은 예제: 관심사가 분리된 코드](#5-좋은-예제-관심사가-분리된-코드)
6. [웹 개발에서의 SoC](#6-웹-개발에서의-soc)
7. [Python 코드 예제](#7-python-코드-예제)
8. [C# 코드 예제](#8-c-코드-예제)
9. [SoC 적용 단계별 가이드](#9-soc-적용-단계별-가이드)
10. [실무 팁](#10-실무-팁)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 개념 연결](#12-관련-개념-연결)

---

## 1. 정의와 개념

**관심사의 분리(Separation of Concerns, SoC)** 란 프로그램을 서로 구별된 관심사(Concern)로 분리하여, 각 부분이 하나의 관심사만 다루도록 설계하는 원칙입니다.

### 관심사(Concern)란?

프로그램에서 **특정 목적이나 기능을 다루는 영역**입니다.

```
┌──────────────────────────────────────────┐
│           프로그램의 관심사 예시            │
│                                          │
│  - 데이터 표시 (UI/프레젠테이션)           │
│  - 비즈니스 규칙 (도메인 로직)             │
│  - 데이터 저장 (영속성)                   │
│  - 사용자 인증 (보안)                     │
│  - 로깅 (운영)                           │
│  - 에러 처리 (안정성)                     │
│  - 캐싱 (성능)                           │
│  - 트랜잭션 관리 (데이터 일관성)           │
└──────────────────────────────────────────┘
```

### 핵심 질문

> "이 코드(함수, 클래스, 모듈)가 동시에 여러 관심사를 다루고 있지는 않은가?"

- **YES** -> 관심사가 혼재 -> 분리 필요
- **NO** -> 관심사가 분리됨 -> 좋은 설계

### 왜 중요한가?

| 관점 | SoC 미적용 | SoC 적용 |
|------|-----------|----------|
| 코드 이해 | 여러 관심사가 뒤섞여 복잡 | 각 부분이 독립적으로 이해 가능 |
| 변경 영향 | UI 수정이 DB 로직에 영향 | 각 레이어 독립 변경 가능 |
| 팀 협업 | 동시 작업 시 충돌 빈번 | 각 레이어 담당자가 독립 작업 |
| 재사용 | 특정 로직만 재사용 불가 | 각 관심사를 독립적으로 재사용 |
| 테스트 | 전체를 통합 테스트해야 함 | 각 관심사를 단독 테스트 가능 |

---

## 2. 비유로 이해하기

### 레스토랑 운영

```
┌─────────────────────────────────────────────────────────────┐
│                ✅ SoC가 적용된 레스토랑                        │
│                                                             │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐           │
│   │  주방     │     │   홀     │     │  계산대   │           │
│   │          │     │          │     │          │           │
│   │ 요리만   │ --> │ 서빙만   │ --> │ 결제만   │           │
│   │ 담당     │     │ 담당     │     │ 담당     │           │
│   └──────────┘     └──────────┘     └──────────┘           │
│                                                             │
│   - 셰프가 바뀌어도 서빙에 영향 없음                          │
│   - 결제 시스템 교체해도 주방에 영향 없음                      │
│   - 각 역할이 전문화되어 효율적                               │
├─────────────────────────────────────────────────────────────┤
│                ❌ SoC가 없는 레스토랑                         │
│                                                             │
│   ┌──────────────────────────────────────────────┐          │
│   │              만능 직원                         │          │
│   │                                              │          │
│   │  요리 + 서빙 + 결제 + 청소 + 재료 주문 + ...   │          │
│   │                                              │          │
│   └──────────────────────────────────────────────┘          │
│                                                             │
│   - 한 사람이 모든 것을 함 -> 비효율적                        │
│   - 요리하다 결제하러 가면 요리가 타버림                       │
│   - 한 명이 아프면 전체 운영 마비                             │
└─────────────────────────────────────────────────────────────┘
```

### 현실 세계 비유: 건물 설계

- **전기 배선**: 전기 관심사
- **배관**: 수도 관심사
- **냉난방 덕트**: 온도 관심사
- **구조물(기둥, 벽)**: 구조적 관심사

각각 별도의 전문가가 설계하고 독립적으로 수리할 수 있습니다. 전기 문제가 생겼을 때 배관을 뜯을 필요가 없습니다.

---

## 3. 다양한 수준의 SoC

SoC는 코드의 다양한 수준에서 적용됩니다.

```
┌────────────────────────────────────────────────────────┐
│                    SoC의 적용 수준                      │
│                                                        │
│   아키텍처 수준    ┌──────┬──────┬──────┐              │
│   (계층 분리)      │  UI  │ BIZ  │ DATA │              │
│                    └──────┴──────┴──────┘              │
│                         ▲                              │
│   모듈/패키지 수준  ┌────┴────┐                         │
│   (기능별 분리)    │ users/ │ orders/ │ payments/ │    │
│                    └────────┘                          │
│                         ▲                              │
│   클래스 수준       ┌────┴────┐                         │
│   (책임별 분리)    │ UserRepo │ UserValidator │        │
│                    └──────────┘                        │
│                         ▲                              │
│   함수 수준         ┌────┴────┐                         │
│   (단일 작업)      │ validate_email() │                │
│                    └──────────────────┘                │
└────────────────────────────────────────────────────────┘
```

### 3.1 함수 수준: 하나의 함수는 하나의 관심사

```python
# ❌ 여러 관심사가 혼재된 함수
def process_user_registration(name, email, password):
    # 관심사 1: 입력 검증
    if len(name) < 2:
        raise ValueError("이름이 너무 짧습니다")
    if "@" not in email:
        raise ValueError("유효하지 않은 이메일")
    if len(password) < 8:
        raise ValueError("비밀번호가 너무 짧습니다")

    # 관심사 2: 비밀번호 해싱
    import hashlib
    salt = os.urandom(32)
    hashed = hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)

    # 관심사 3: DB 저장
    connection = sqlite3.connect('users.db')
    cursor = connection.cursor()
    cursor.execute("INSERT INTO users ...", (name, email, hashed))
    connection.commit()

    # 관심사 4: 이메일 전송
    smtp = smtplib.SMTP('smtp.gmail.com', 587)
    smtp.send_message(...)

# ✅ 관심사별로 분리된 함수들
def validate_registration_input(name: str, email: str, password: str) -> None:
    """관심사: 입력 검증만"""
    if len(name) < 2:
        raise ValueError("이름이 너무 짧습니다")
    if "@" not in email:
        raise ValueError("유효하지 않은 이메일")
    if len(password) < 8:
        raise ValueError("비밀번호가 너무 짧습니다")

def hash_password(password: str) -> str:
    """관심사: 비밀번호 해싱만"""
    ...

def save_user(name: str, email: str, password_hash: str) -> int:
    """관심사: DB 저장만"""
    ...

def send_welcome_email(email: str, name: str) -> None:
    """관심사: 이메일 전송만"""
    ...

def register_user(name: str, email: str, password: str) -> int:
    """관심사: 등록 프로세스 조율 (오케스트레이션)"""
    validate_registration_input(name, email, password)
    password_hash = hash_password(password)
    user_id = save_user(name, email, password_hash)
    send_welcome_email(email, name)
    return user_id
```

### 3.2 클래스 수준: SRP와 연관

```python
# ❌ 여러 관심사를 가진 클래스
class User:
    def save(self): ...          # 데이터 접근 관심사
    def validate(self): ...      # 검증 관심사
    def send_email(self): ...    # 알림 관심사
    def calculate_age(self): ... # 비즈니스 로직 관심사

# ✅ 관심사별 클래스 분리
class User:                      # 도메인 모델 (비즈니스 로직)
    def calculate_age(self): ...

class UserRepository:            # 데이터 접근
    def save(self, user): ...

class UserValidator:             # 검증
    def validate(self, user): ...

class UserNotifier:              # 알림
    def send_email(self, user): ...
```

### 3.3 모듈/패키지 수준: 기능별 모듈 분리

```
# ❌ 관심사가 섞인 프로젝트 구조
project/
  models.py          <- 모든 모델이 한 파일에
  views.py           <- 모든 뷰가 한 파일에
  utils.py           <- 모든 유틸이 한 파일에

# ✅ 기능별로 분리된 프로젝트 구조
project/
  users/
    models.py        <- 사용자 모델
    repository.py    <- 사용자 데이터 접근
    service.py       <- 사용자 비즈니스 로직
    validator.py     <- 사용자 검증
  orders/
    models.py        <- 주문 모델
    repository.py    <- 주문 데이터 접근
    service.py       <- 주문 비즈니스 로직
    validator.py     <- 주문 검증
  payments/
    models.py        <- 결제 모델
    gateway.py       <- 결제 처리
    service.py       <- 결제 비즈니스 로직
```

### 3.4 아키텍처 수준: 계층 분리 (Layered Architecture)

```
┌─────────────────────────────────────────────────┐
│              프레젠테이션 계층                     │
│          (UI, API Controller, View)              │
│          관심사: 사용자 상호작용                    │
├─────────────────────────────────────────────────┤
│              애플리케이션 계층                     │
│          (Service, UseCase)                      │
│          관심사: 유스케이스 조율                    │
├─────────────────────────────────────────────────┤
│              도메인 계층                          │
│          (Entity, Value Object)                  │
│          관심사: 비즈니스 규칙                     │
├─────────────────────────────────────────────────┤
│              인프라 계층                          │
│          (Repository, External API)              │
│          관심사: 기술적 세부사항                    │
└─────────────────────────────────────────────────┘

각 계층은 자신의 관심사에만 집중
상위 계층이 하위 계층에 의존 (역방향 의존 금지)
```

---

## 4. 나쁜 예제: 관심사가 혼재된 코드

### 문제 상황: 모든 것이 한 곳에

```
❌ 관심사 혼재

┌──────────────────────────────────────────────┐
│         process_order() 함수 하나에            │
│                                              │
│    [UI 로직]     사용자 입력 읽기              │
│    [검증 로직]    주문 데이터 검증              │
│    [비즈니스]     가격 계산, 할인 적용          │
│    [DB 접근]      SQL 쿼리로 직접 저장         │
│    [외부 API]     결제 게이트웨이 직접 호출     │
│    [이메일]       SMTP로 직접 이메일 전송       │
│    [로깅]         파일에 직접 로그 기록          │
│                                              │
│    총 300줄... 무엇을 고치려면 전체를 이해해야 함│
└──────────────────────────────────────────────┘
```

### Python - 관심사 혼재 예제

```python
# ❌ 나쁜 예: UI + 비즈니스 로직 + DB + 이메일이 한 함수에
import sqlite3
import smtplib
from datetime import datetime


def process_order(request):
    """모든 관심사가 뒤섞인 함수 - 300줄짜리 괴물"""

    # --- 관심사 1: 입력 파싱 (프레젠테이션 관심사) ---
    customer_name = request.get("name", "").strip()
    customer_email = request.get("email", "").strip()
    items = request.get("items", [])
    coupon_code = request.get("coupon", "")

    # --- 관심사 2: 입력 검증 (검증 관심사) ---
    if not customer_name:
        return {"error": "이름을 입력하세요", "status": 400}
    if "@" not in customer_email:
        return {"error": "유효한 이메일을 입력하세요", "status": 400}
    if not items:
        return {"error": "주문 항목이 비어있습니다", "status": 400}

    # --- 관심사 3: 비즈니스 로직 (가격 계산, 할인) ---
    total = 0
    for item in items:
        price = item["price"] * item["quantity"]
        if item["quantity"] >= 10:
            price *= 0.9  # 대량 할인
        total += price

    if coupon_code == "WELCOME10":
        total *= 0.9  # 쿠폰 할인
    elif coupon_code == "VIP20":
        total *= 0.8

    tax = total * 0.1
    shipping = 0 if total >= 50000 else 3000
    grand_total = total + tax + shipping

    # --- 관심사 4: DB 저장 (데이터 접근 관심사) ---
    conn = sqlite3.connect("shop.db")
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO orders (customer_name, email, total, status, created_at) "
        "VALUES (?, ?, ?, ?, ?)",
        (customer_name, customer_email, grand_total, "confirmed", datetime.now())
    )
    order_id = cursor.lastrowid

    for item in items:
        cursor.execute(
            "INSERT INTO order_items (order_id, product_id, quantity, price) "
            "VALUES (?, ?, ?, ?)",
            (order_id, item["product_id"], item["quantity"], item["price"])
        )

        # 재고 차감
        cursor.execute(
            "UPDATE products SET stock = stock - ? WHERE id = ?",
            (item["quantity"], item["product_id"])
        )

    conn.commit()
    conn.close()

    # --- 관심사 5: 이메일 전송 (알림 관심사) ---
    try:
        smtp = smtplib.SMTP("smtp.gmail.com", 587)
        smtp.starttls()
        smtp.login("shop@gmail.com", "password123")
        message = f"""
        Subject: 주문 확인 #{order_id}

        {customer_name}님, 주문이 확인되었습니다.
        총 금액: {grand_total}원
        """
        smtp.sendmail("shop@gmail.com", customer_email, message)
        smtp.quit()
    except Exception as e:
        print(f"이메일 전송 실패: {e}")

    # --- 관심사 6: 로깅 (운영 관심사) ---
    with open("orders.log", "a") as f:
        f.write(f"{datetime.now()} | Order #{order_id} | {customer_email} | {grand_total}\n")

    # --- 관심사 7: 응답 포맷팅 (프레젠테이션 관심사) ---
    return {
        "status": 200,
        "order_id": order_id,
        "total": grand_total,
        "message": f"주문이 완료되었습니다. 주문번호: {order_id}"
    }

# 문제점:
# 1. DB를 바꾸려면? -> 이 함수를 수정해야 함
# 2. 이메일 대신 SMS를 보내려면? -> 이 함수를 수정해야 함
# 3. 할인 정책을 바꾸려면? -> 이 함수를 수정해야 함
# 4. 유닛 테스트하려면? -> 실제 DB, SMTP 서버 필요
# 5. 프론트엔드 개발자가 응답 형식을 바꾸려면? -> 비즈니스 로직도 건드릴 위험
```

### C# - 관심사 혼재 예제

```csharp
// ❌ 나쁜 예: 모든 관심사가 한 메서드에
public class OrderController
{
    public IActionResult ProcessOrder(OrderRequest request)
    {
        // 관심사 1: 입력 검증
        if (string.IsNullOrEmpty(request.Name))
            return BadRequest("이름을 입력하세요");
        if (!request.Email.Contains("@"))
            return BadRequest("유효한 이메일을 입력하세요");

        // 관심사 2: 비즈니스 로직 (가격 계산)
        decimal total = 0;
        foreach (var item in request.Items)
        {
            var price = item.Price * item.Quantity;
            if (item.Quantity >= 10) price *= 0.9m;
            total += price;
        }

        if (request.CouponCode == "WELCOME10") total *= 0.9m;
        var tax = total * 0.1m;
        var shipping = total >= 50000 ? 0 : 3000;
        var grandTotal = total + tax + shipping;

        // 관심사 3: DB 직접 접근
        using var conn = new SqlConnection("Server=...");
        conn.Open();
        using var cmd = new SqlCommand(
            "INSERT INTO Orders (...) VALUES (...)", conn);
        cmd.Parameters.AddWithValue("@total", grandTotal);
        var orderId = (int)cmd.ExecuteScalar();

        // 관심사 4: 이메일 직접 전송
        using var smtp = new SmtpClient("smtp.gmail.com", 587);
        smtp.Send("shop@gmail.com", request.Email,
            $"주문 확인 #{orderId}", $"총 금액: {grandTotal}원");

        // 관심사 5: 로깅
        File.AppendAllText("orders.log",
            $"{DateTime.Now} | Order #{orderId}\n");

        // 관심사 6: 응답 포맷팅
        return Ok(new { orderId, total = grandTotal,
            message = $"주문 완료. 주문번호: {orderId}" });
    }
}
```

---

## 5. 좋은 예제: 관심사가 분리된 코드

### 개선된 구조

```
✅ 관심사 분리된 구조

┌─────────────────────────────────────────────────────┐
│  프레젠테이션 계층                                    │
│  ┌──────────────────────┐                           │
│  │   OrderController     │  입력 파싱, 응답 포맷팅    │
│  └──────────┬───────────┘                           │
├─────────────┼───────────────────────────────────────┤
│  애플리케이션 계층  │                                  │
│  ┌──────────▼───────────┐                           │
│  │  PlaceOrderUseCase   │  프로세스 조율              │
│  └──┬────┬────┬────┬───┘                            │
├─────┼────┼────┼────┼────────────────────────────────┤
│  도메인 계층                                         │
│  ┌──▼──┐┌─▼──┐┌─▼──┐┌▼────┐                        │
│  │검증 ││계산 ││재고 ││주문  │  비즈니스 규칙          │
│  └─────┘└────┘└────┘└─────┘                         │
├─────────────────────────────────────────────────────┤
│  인프라 계층                                         │
│  ┌─────┐  ┌──────┐  ┌──────┐  ┌─────┐              │
│  │ DB  │  │Email │  │Logger│  │결제  │  기술적 구현   │
│  └─────┘  └──────┘  └──────┘  └─────┘              │
└─────────────────────────────────────────────────────┘
```

### Python - 관심사 분리 예제

```python
# ✅ 좋은 예: 관심사별로 분리

from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from decimal import Decimal
from typing import Optional


# ============================================================
# 관심사 1: 도메인 모델 (비즈니스 개념)
# ============================================================
@dataclass
class OrderItem:
    product_id: int
    product_name: str
    quantity: int
    unit_price: Decimal

@dataclass
class Order:
    id: Optional[int] = None
    customer_name: str = ""
    customer_email: str = ""
    items: list[OrderItem] = field(default_factory=list)
    coupon_code: str = ""
    status: str = "pending"

@dataclass
class PricingResult:
    subtotal: Decimal = Decimal("0")
    discount: Decimal = Decimal("0")
    tax: Decimal = Decimal("0")
    shipping: Decimal = Decimal("0")
    total: Decimal = Decimal("0")


# ============================================================
# 관심사 2: 입력 검증 (검증만 담당)
# ============================================================
class OrderValidator:
    """주문 데이터의 유효성 검사만 담당"""

    def validate(self, order: Order) -> list[str]:
        errors = []
        if not order.customer_name:
            errors.append("이름을 입력하세요")
        if not order.customer_email or "@" not in order.customer_email:
            errors.append("유효한 이메일을 입력하세요")
        if not order.items:
            errors.append("주문 항목이 비어있습니다")
        for item in order.items:
            if item.quantity <= 0:
                errors.append(f"{item.product_name}: 수량은 1 이상이어야 합니다")
        return errors


# ============================================================
# 관심사 3: 가격 계산 (비즈니스 로직만 담당)
# ============================================================
class PricingService:
    """가격 계산 비즈니스 로직만 담당"""

    BULK_DISCOUNT_THRESHOLD = 10
    BULK_DISCOUNT_RATE = Decimal("0.1")
    FREE_SHIPPING_THRESHOLD = Decimal("50000")
    SHIPPING_FEE = Decimal("3000")
    TAX_RATE = Decimal("0.1")

    COUPON_DISCOUNTS = {
        "WELCOME10": Decimal("0.1"),
        "VIP20": Decimal("0.2"),
    }

    def calculate(self, items: list[OrderItem], coupon_code: str = "") -> PricingResult:
        subtotal = self._calculate_subtotal(items)
        discount = self._calculate_discount(subtotal, coupon_code)
        discounted = subtotal - discount
        tax = discounted * self.TAX_RATE
        shipping = self._calculate_shipping(discounted)

        return PricingResult(
            subtotal=subtotal,
            discount=discount,
            tax=tax,
            shipping=shipping,
            total=discounted + tax + shipping
        )

    def _calculate_subtotal(self, items: list[OrderItem]) -> Decimal:
        total = Decimal("0")
        for item in items:
            price = item.unit_price * item.quantity
            if item.quantity >= self.BULK_DISCOUNT_THRESHOLD:
                price *= (1 - self.BULK_DISCOUNT_RATE)
            total += price
        return total

    def _calculate_discount(self, subtotal: Decimal, coupon_code: str) -> Decimal:
        rate = self.COUPON_DISCOUNTS.get(coupon_code, Decimal("0"))
        return subtotal * rate

    def _calculate_shipping(self, subtotal: Decimal) -> Decimal:
        return Decimal("0") if subtotal >= self.FREE_SHIPPING_THRESHOLD else self.SHIPPING_FEE


# ============================================================
# 관심사 4: 데이터 접근 (저장/조회만 담당)
# ============================================================
class IOrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order, pricing: PricingResult) -> int:
        pass

    @abstractmethod
    def find_by_id(self, order_id: int) -> Optional[Order]:
        pass

class SqliteOrderRepository(IOrderRepository):
    """SQLite를 사용한 주문 저장소 (DB 접근만 담당)"""

    def __init__(self, db_path: str):
        self._db_path = db_path

    def save(self, order: Order, pricing: PricingResult) -> int:
        print(f"[DB] 주문 저장: {order.customer_name}, 총액: {pricing.total}")
        return 1  # 생성된 주문 ID 반환

    def find_by_id(self, order_id: int) -> Optional[Order]:
        print(f"[DB] 주문 조회: {order_id}")
        return None


# ============================================================
# 관심사 5: 재고 관리 (재고만 담당)
# ============================================================
class IInventoryService(ABC):
    @abstractmethod
    def check_and_reserve(self, items: list[OrderItem]) -> bool:
        pass

    @abstractmethod
    def release(self, items: list[OrderItem]) -> None:
        pass

class InventoryService(IInventoryService):
    def check_and_reserve(self, items: list[OrderItem]) -> bool:
        print(f"[Inventory] {len(items)}개 상품 재고 확인 및 예약")
        return True

    def release(self, items: list[OrderItem]) -> None:
        print(f"[Inventory] {len(items)}개 상품 재고 복구")


# ============================================================
# 관심사 6: 알림 (알림 전송만 담당)
# ============================================================
class INotificationService(ABC):
    @abstractmethod
    def send_order_confirmation(self, order: Order, pricing: PricingResult) -> None:
        pass

class EmailNotificationService(INotificationService):
    def send_order_confirmation(self, order: Order, pricing: PricingResult) -> None:
        print(f"[Email] {order.customer_email}에게 주문 확인 이메일 전송")


# ============================================================
# 관심사 7: 로깅 (로그 기록만 담당)
# ============================================================
class ILogger(ABC):
    @abstractmethod
    def info(self, message: str) -> None:
        pass

    @abstractmethod
    def error(self, message: str) -> None:
        pass

class FileLogger(ILogger):
    def info(self, message: str) -> None:
        print(f"[LOG:INFO] {message}")

    def error(self, message: str) -> None:
        print(f"[LOG:ERROR] {message}")


# ============================================================
# 유스케이스: 각 관심사를 조율 (오케스트레이션)
# ============================================================
class PlaceOrderUseCase:
    """주문 프로세스를 조율하는 유스케이스 - 각 관심사를 조합"""

    def __init__(
        self,
        validator: OrderValidator,
        pricing: PricingService,
        repository: IOrderRepository,
        inventory: IInventoryService,
        notification: INotificationService,
        logger: ILogger
    ):
        self._validator = validator
        self._pricing = pricing
        self._repository = repository
        self._inventory = inventory
        self._notification = notification
        self._logger = logger

    def execute(self, order: Order) -> dict:
        # 검증 관심사에 위임
        errors = self._validator.validate(order)
        if errors:
            return {"success": False, "errors": errors}

        # 가격 계산 관심사에 위임
        pricing = self._pricing.calculate(order.items, order.coupon_code)

        # 재고 관심사에 위임
        if not self._inventory.check_and_reserve(order.items):
            return {"success": False, "errors": ["재고가 부족합니다"]}

        try:
            # 저장 관심사에 위임
            order_id = self._repository.save(order, pricing)
            order.id = order_id
            order.status = "confirmed"

            # 알림 관심사에 위임
            self._notification.send_order_confirmation(order, pricing)

            # 로깅 관심사에 위임
            self._logger.info(f"주문 완료: #{order_id}, 금액: {pricing.total}")

            return {
                "success": True,
                "order_id": order_id,
                "total": str(pricing.total)
            }
        except Exception as e:
            self._inventory.release(order.items)
            self._logger.error(f"주문 실패: {e}")
            return {"success": False, "errors": [str(e)]}


# ============================================================
# 프레젠테이션 계층: 입력/출력 포맷팅만 담당
# ============================================================
class OrderController:
    """HTTP 요청/응답 처리만 담당 (프레젠테이션 관심사)"""

    def __init__(self, use_case: PlaceOrderUseCase):
        self._use_case = use_case

    def handle_request(self, request: dict) -> dict:
        # 관심사: 요청 데이터 -> 도메인 객체 변환
        order = Order(
            customer_name=request.get("name", ""),
            customer_email=request.get("email", ""),
            coupon_code=request.get("coupon", ""),
            items=[
                OrderItem(
                    product_id=item["product_id"],
                    product_name=item.get("name", ""),
                    quantity=item["quantity"],
                    unit_price=Decimal(str(item["price"]))
                )
                for item in request.get("items", [])
            ]
        )

        # 유스케이스에 위임
        result = self._use_case.execute(order)

        # 관심사: 결과 -> HTTP 응답 변환
        if result["success"]:
            return {
                "status": 200,
                "body": {
                    "order_id": result["order_id"],
                    "total": result["total"],
                    "message": f"주문이 완료되었습니다. 주문번호: {result['order_id']}"
                }
            }
        else:
            return {
                "status": 400,
                "body": {"errors": result["errors"]}
            }


# ============================================================
# 조립 (Composition Root)
# ============================================================
if __name__ == "__main__":
    # 각 관심사의 구현체를 생성
    validator = OrderValidator()
    pricing = PricingService()
    repository = SqliteOrderRepository("shop.db")
    inventory = InventoryService()
    notification = EmailNotificationService()
    logger = FileLogger()

    # 유스케이스에 주입
    use_case = PlaceOrderUseCase(
        validator=validator,
        pricing=pricing,
        repository=repository,
        inventory=inventory,
        notification=notification,
        logger=logger
    )

    # 컨트롤러에 주입
    controller = OrderController(use_case)

    # 요청 처리
    request = {
        "name": "홍길동",
        "email": "hong@example.com",
        "coupon": "WELCOME10",
        "items": [
            {"product_id": 1, "name": "키보드", "quantity": 2, "price": 35000},
            {"product_id": 2, "name": "마우스", "quantity": 1, "price": 25000},
        ]
    }

    response = controller.handle_request(request)
    print(f"\n응답: {response}")
```

---

## 6. 웹 개발에서의 SoC

### HTML + CSS + JavaScript 분리

```
웹 프론트엔드의 3가지 관심사:

┌───────────────────────────────────────────────┐
│                                               │
│   HTML (구조)      CSS (스타일)    JS (동작)    │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│   │ <div>    │   │ .button  │   │ onClick  │ │
│   │ <button> │   │ { color: │   │ fetch()  │ │
│   │ <input>  │   │   blue } │   │ validate │ │
│   └──────────┘   └──────────┘   └──────────┘ │
│                                               │
│   "무엇을 보여줄까" "어떻게 보일까" "어떻게 동작할까" │
└───────────────────────────────────────────────┘
```

### 백엔드 아키텍처 패턴에서의 SoC

```
MVC 패턴:
┌─────────┐     ┌────────────┐     ┌───────┐
│  Model  │ <-- │ Controller │ --> │ View  │
│ (데이터) │     │ (제어 흐름)  │     │ (표시) │
└─────────┘     └────────────┘     └───────┘

Clean Architecture:
┌────────────────────────────────────────┐
│          Frameworks & Drivers          │  외부 (웹, DB, UI)
│  ┌──────────────────────────────────┐  │
│  │       Interface Adapters         │  │  어댑터 (컨트롤러, 게이트웨이)
│  │  ┌────────────────────────────┐  │  │
│  │  │     Application Business   │  │  │  유스케이스
│  │  │  ┌──────────────────────┐  │  │  │
│  │  │  │  Enterprise Business │  │  │  │  엔티티 (핵심 비즈니스 규칙)
│  │  │  └──────────────────────┘  │  │  │
│  │  └────────────────────────────┘  │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

### 횡단 관심사 (Cross-Cutting Concerns)

여러 계층에 걸쳐 적용되는 관심사를 **횡단 관심사**라 합니다.

```
                    로깅    보안    캐싱    트랜잭션
                      │      │      │      │
┌─────────────────────┼──────┼──────┼──────┼─────┐
│ 프레젠테이션 계층     │      │      │      │     │
├─────────────────────┼──────┼──────┼──────┼─────┤
│ 비즈니스 계층        │      │      │      │     │
├─────────────────────┼──────┼──────┼──────┼─────┤
│ 데이터 접근 계층     │      │      │      │     │
└─────────────────────┴──────┴──────┴──────┴─────┘

횡단 관심사 처리 방법:
- 데코레이터 패턴 (Python: @decorator, C#: Attribute)
- AOP (Aspect-Oriented Programming)
- 미들웨어 패턴
```

```python
# 횡단 관심사를 데코레이터로 분리
import functools
import time
import logging

logger = logging.getLogger(__name__)

def log_execution(func):
    """로깅 횡단 관심사를 데코레이터로 분리"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"시작: {func.__name__}")
        start = time.time()
        try:
            result = func(*args, **kwargs)
            elapsed = time.time() - start
            logger.info(f"완료: {func.__name__} ({elapsed:.2f}s)")
            return result
        except Exception as e:
            logger.error(f"실패: {func.__name__} - {e}")
            raise
    return wrapper

def require_auth(func):
    """인증 횡단 관심사를 데코레이터로 분리"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # 인증 확인 로직
        if not is_authenticated():
            raise PermissionError("인증이 필요합니다")
        return func(*args, **kwargs)
    return wrapper

# 사용: 비즈니스 로직은 횡단 관심사를 모름
class OrderService:
    @log_execution
    @require_auth
    def place_order(self, order: Order) -> int:
        # 순수하게 비즈니스 로직만 집중
        pricing = self._calculate_pricing(order)
        order_id = self._save_order(order, pricing)
        return order_id
```

---

## 7. Python 코드 예제

### 종합 예제: 블로그 시스템

```python
"""
관심사의 분리를 적용한 블로그 시스템
각 모듈이 하나의 관심사에만 집중
"""
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional


# ============================================================
# 도메인 모델 (관심사: 비즈니스 개념 표현)
# ============================================================
@dataclass
class BlogPost:
    id: Optional[int] = None
    title: str = ""
    content: str = ""
    author_id: int = 0
    slug: str = ""
    tags: list[str] = field(default_factory=list)
    published: bool = False
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: Optional[datetime] = None


# ============================================================
# 관심사: 슬러그 생성
# ============================================================
class SlugGenerator:
    """URL-friendly 슬러그 생성만 담당"""

    def generate(self, title: str) -> str:
        slug = title.lower().strip()
        # 간단한 슬러그 생성 로직
        slug = slug.replace(" ", "-")
        slug = "".join(c for c in slug if c.isalnum() or c == "-")
        return slug

    def ensure_unique(self, slug: str, existing_slugs: list[str]) -> str:
        if slug not in existing_slugs:
            return slug
        counter = 1
        while f"{slug}-{counter}" in existing_slugs:
            counter += 1
        return f"{slug}-{counter}"


# ============================================================
# 관심사: 게시글 검증
# ============================================================
class PostValidator:
    """게시글 유효성 검사만 담당"""

    MIN_TITLE_LENGTH = 5
    MAX_TITLE_LENGTH = 200
    MIN_CONTENT_LENGTH = 50
    MAX_TAGS = 10

    def validate(self, post: BlogPost) -> list[str]:
        errors = []
        errors.extend(self._validate_title(post.title))
        errors.extend(self._validate_content(post.content))
        errors.extend(self._validate_tags(post.tags))
        return errors

    def _validate_title(self, title: str) -> list[str]:
        errors = []
        if len(title) < self.MIN_TITLE_LENGTH:
            errors.append(f"제목은 {self.MIN_TITLE_LENGTH}자 이상이어야 합니다")
        if len(title) > self.MAX_TITLE_LENGTH:
            errors.append(f"제목은 {self.MAX_TITLE_LENGTH}자 이하여야 합니다")
        return errors

    def _validate_content(self, content: str) -> list[str]:
        errors = []
        if len(content) < self.MIN_CONTENT_LENGTH:
            errors.append(f"본문은 {self.MIN_CONTENT_LENGTH}자 이상이어야 합니다")
        return errors

    def _validate_tags(self, tags: list[str]) -> list[str]:
        errors = []
        if len(tags) > self.MAX_TAGS:
            errors.append(f"태그는 {self.MAX_TAGS}개 이하여야 합니다")
        for tag in tags:
            if len(tag) > 30:
                errors.append(f"태그 '{tag}'가 너무 깁니다 (최대 30자)")
        return errors


# ============================================================
# 관심사: 콘텐츠 가공
# ============================================================
class ContentProcessor:
    """게시글 콘텐츠 가공만 담당"""

    def sanitize_html(self, content: str) -> str:
        """위험한 HTML 태그 제거"""
        dangerous_tags = ["<script>", "</script>", "<iframe>", "</iframe>"]
        result = content
        for tag in dangerous_tags:
            result = result.replace(tag, "")
        return result

    def generate_excerpt(self, content: str, max_length: int = 200) -> str:
        """본문에서 요약 추출"""
        if len(content) <= max_length:
            return content
        return content[:max_length].rsplit(" ", 1)[0] + "..."

    def estimate_reading_time(self, content: str) -> int:
        """읽기 시간 추정 (분)"""
        words = len(content.split())
        return max(1, words // 200)  # 분당 200단어 기준


# ============================================================
# 관심사: 데이터 접근
# ============================================================
class IPostRepository(ABC):
    @abstractmethod
    def save(self, post: BlogPost) -> BlogPost: ...

    @abstractmethod
    def find_by_id(self, post_id: int) -> Optional[BlogPost]: ...

    @abstractmethod
    def find_by_slug(self, slug: str) -> Optional[BlogPost]: ...

    @abstractmethod
    def get_all_slugs(self) -> list[str]: ...

class InMemoryPostRepository(IPostRepository):
    """메모리 기반 게시글 저장소 (테스트/개발용)"""

    def __init__(self):
        self._posts: dict[int, BlogPost] = {}
        self._next_id = 1

    def save(self, post: BlogPost) -> BlogPost:
        if post.id is None:
            post.id = self._next_id
            self._next_id += 1
        else:
            post.updated_at = datetime.now()
        self._posts[post.id] = post
        print(f"[DB] 게시글 저장: #{post.id} '{post.title}'")
        return post

    def find_by_id(self, post_id: int) -> Optional[BlogPost]:
        return self._posts.get(post_id)

    def find_by_slug(self, slug: str) -> Optional[BlogPost]:
        for post in self._posts.values():
            if post.slug == slug:
                return post
        return None

    def get_all_slugs(self) -> list[str]:
        return [post.slug for post in self._posts.values()]


# ============================================================
# 관심사: 이벤트/알림
# ============================================================
class IEventPublisher(ABC):
    @abstractmethod
    def publish(self, event: str, data: dict) -> None: ...

class SimpleEventPublisher(IEventPublisher):
    """이벤트 발행만 담당"""

    def __init__(self):
        self._handlers: dict[str, list] = {}

    def subscribe(self, event: str, handler) -> None:
        self._handlers.setdefault(event, []).append(handler)

    def publish(self, event: str, data: dict) -> None:
        print(f"[Event] '{event}' 발행")
        for handler in self._handlers.get(event, []):
            handler(data)


# ============================================================
# 유스케이스: 게시글 작성 (관심사들을 조율)
# ============================================================
class CreatePostUseCase:
    """게시글 작성 프로세스를 조율"""

    def __init__(
        self,
        validator: PostValidator,
        slug_generator: SlugGenerator,
        content_processor: ContentProcessor,
        repository: IPostRepository,
        event_publisher: IEventPublisher
    ):
        self._validator = validator
        self._slug_generator = slug_generator
        self._content_processor = content_processor
        self._repository = repository
        self._event_publisher = event_publisher

    def execute(self, title: str, content: str, author_id: int,
                tags: list[str] = None, publish: bool = False) -> dict:
        # 게시글 생성
        post = BlogPost(
            title=title,
            content=content,
            author_id=author_id,
            tags=tags or [],
            published=publish
        )

        # 검증 관심사에 위임
        errors = self._validator.validate(post)
        if errors:
            return {"success": False, "errors": errors}

        # 슬러그 생성 관심사에 위임
        slug = self._slug_generator.generate(title)
        existing_slugs = self._repository.get_all_slugs()
        post.slug = self._slug_generator.ensure_unique(slug, existing_slugs)

        # 콘텐츠 가공 관심사에 위임
        post.content = self._content_processor.sanitize_html(content)

        # 저장 관심사에 위임
        saved_post = self._repository.save(post)

        # 이벤트 발행 관심사에 위임
        self._event_publisher.publish("post_created", {
            "post_id": saved_post.id,
            "title": saved_post.title,
            "author_id": saved_post.author_id
        })

        return {
            "success": True,
            "post_id": saved_post.id,
            "slug": saved_post.slug,
            "reading_time": self._content_processor.estimate_reading_time(content)
        }


# ============================================================
# 사용 예시
# ============================================================
if __name__ == "__main__":
    # 관심사별 구현체 생성
    validator = PostValidator()
    slug_generator = SlugGenerator()
    content_processor = ContentProcessor()
    repository = InMemoryPostRepository()
    event_publisher = SimpleEventPublisher()

    # 이벤트 구독 (알림 관심사)
    event_publisher.subscribe(
        "post_created",
        lambda data: print(f"[알림] 새 게시글이 작성되었습니다: {data['title']}")
    )

    # 유스케이스 조립
    create_post = CreatePostUseCase(
        validator=validator,
        slug_generator=slug_generator,
        content_processor=content_processor,
        repository=repository,
        event_publisher=event_publisher
    )

    # 게시글 작성
    result = create_post.execute(
        title="관심사의 분리 원칙 이해하기",
        content="관심사의 분리(Separation of Concerns)는 소프트웨어 설계의 핵심 원칙입니다. "
                "이 원칙은 프로그램을 서로 구별된 영역으로 나누어 각 부분이 하나의 관심사만 "
                "다루도록 합니다. 이를 통해 코드의 유지보수성, 재사용성, 테스트 용이성이 크게 향상됩니다.",
        author_id=1,
        tags=["설계원칙", "소프트웨어공학", "SoC"],
        publish=True
    )

    print(f"\n결과: {result}")
```

---

## 8. C# 코드 예제

### 종합 예제: 블로그 시스템

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// ============================================================
// 도메인 모델
// ============================================================
public class BlogPost
{
    public int? Id { get; set; }
    public string Title { get; set; } = "";
    public string Content { get; set; } = "";
    public int AuthorId { get; set; }
    public string Slug { get; set; } = "";
    public List<string> Tags { get; set; } = new();
    public bool Published { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.Now;
    public DateTime? UpdatedAt { get; set; }
}

// ============================================================
// 관심사: 슬러그 생성
// ============================================================
public interface ISlugGenerator
{
    string Generate(string title);
    string EnsureUnique(string slug, IEnumerable<string> existingSlugs);
}

public class SlugGenerator : ISlugGenerator
{
    public string Generate(string title)
    {
        var slug = title.ToLower().Trim().Replace(" ", "-");
        return new string(slug.Where(c => char.IsLetterOrDigit(c) || c == '-').ToArray());
    }

    public string EnsureUnique(string slug, IEnumerable<string> existingSlugs)
    {
        var slugList = existingSlugs.ToList();
        if (!slugList.Contains(slug)) return slug;

        var counter = 1;
        while (slugList.Contains($"{slug}-{counter}")) counter++;
        return $"{slug}-{counter}";
    }
}

// ============================================================
// 관심사: 게시글 검증
// ============================================================
public interface IPostValidator
{
    List<string> Validate(BlogPost post);
}

public class PostValidator : IPostValidator
{
    private const int MinTitleLength = 5;
    private const int MaxTitleLength = 200;
    private const int MinContentLength = 50;
    private const int MaxTags = 10;

    public List<string> Validate(BlogPost post)
    {
        var errors = new List<string>();
        errors.AddRange(ValidateTitle(post.Title));
        errors.AddRange(ValidateContent(post.Content));
        errors.AddRange(ValidateTags(post.Tags));
        return errors;
    }

    private List<string> ValidateTitle(string title)
    {
        var errors = new List<string>();
        if (title.Length < MinTitleLength)
            errors.Add($"제목은 {MinTitleLength}자 이상이어야 합니다");
        if (title.Length > MaxTitleLength)
            errors.Add($"제목은 {MaxTitleLength}자 이하여야 합니다");
        return errors;
    }

    private List<string> ValidateContent(string content)
    {
        var errors = new List<string>();
        if (content.Length < MinContentLength)
            errors.Add($"본문은 {MinContentLength}자 이상이어야 합니다");
        return errors;
    }

    private List<string> ValidateTags(List<string> tags)
    {
        var errors = new List<string>();
        if (tags.Count > MaxTags)
            errors.Add($"태그는 {MaxTags}개 이하여야 합니다");
        foreach (var tag in tags.Where(t => t.Length > 30))
            errors.Add($"태그 '{tag}'가 너무 깁니다 (최대 30자)");
        return errors;
    }
}

// ============================================================
// 관심사: 콘텐츠 가공
// ============================================================
public interface IContentProcessor
{
    string SanitizeHtml(string content);
    string GenerateExcerpt(string content, int maxLength = 200);
    int EstimateReadingTime(string content);
}

public class ContentProcessor : IContentProcessor
{
    private static readonly string[] DangerousTags =
        { "<script>", "</script>", "<iframe>", "</iframe>" };

    public string SanitizeHtml(string content)
    {
        var result = content;
        foreach (var tag in DangerousTags)
            result = result.Replace(tag, "", StringComparison.OrdinalIgnoreCase);
        return result;
    }

    public string GenerateExcerpt(string content, int maxLength = 200)
    {
        if (content.Length <= maxLength) return content;
        var truncated = content[..maxLength];
        var lastSpace = truncated.LastIndexOf(' ');
        return (lastSpace > 0 ? truncated[..lastSpace] : truncated) + "...";
    }

    public int EstimateReadingTime(string content)
    {
        var words = content.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
        return Math.Max(1, words / 200);
    }
}

// ============================================================
// 관심사: 데이터 접근
// ============================================================
public interface IPostRepository
{
    BlogPost Save(BlogPost post);
    BlogPost? FindById(int postId);
    BlogPost? FindBySlug(string slug);
    IEnumerable<string> GetAllSlugs();
}

public class InMemoryPostRepository : IPostRepository
{
    private readonly Dictionary<int, BlogPost> _posts = new();
    private int _nextId = 1;

    public BlogPost Save(BlogPost post)
    {
        if (post.Id == null)
        {
            post.Id = _nextId++;
        }
        else
        {
            post.UpdatedAt = DateTime.Now;
        }
        _posts[post.Id.Value] = post;
        Console.WriteLine($"[DB] 게시글 저장: #{post.Id} '{post.Title}'");
        return post;
    }

    public BlogPost? FindById(int postId)
        => _posts.GetValueOrDefault(postId);

    public BlogPost? FindBySlug(string slug)
        => _posts.Values.FirstOrDefault(p => p.Slug == slug);

    public IEnumerable<string> GetAllSlugs()
        => _posts.Values.Select(p => p.Slug);
}

// ============================================================
// 관심사: 이벤트 시스템
// ============================================================
public interface IEventPublisher
{
    void Publish(string eventName, Dictionary<string, object> data);
}

public class SimpleEventPublisher : IEventPublisher
{
    private readonly Dictionary<string, List<Action<Dictionary<string, object>>>> _handlers = new();

    public void Subscribe(string eventName, Action<Dictionary<string, object>> handler)
    {
        if (!_handlers.ContainsKey(eventName))
            _handlers[eventName] = new List<Action<Dictionary<string, object>>>();
        _handlers[eventName].Add(handler);
    }

    public void Publish(string eventName, Dictionary<string, object> data)
    {
        Console.WriteLine($"[Event] '{eventName}' 발행");
        if (_handlers.TryGetValue(eventName, out var handlers))
            foreach (var handler in handlers)
                handler(data);
    }
}

// ============================================================
// 유스케이스
// ============================================================
public record CreatePostResult(
    bool Success,
    int? PostId = null,
    string? Slug = null,
    int ReadingTime = 0,
    List<string>? Errors = null
);

public class CreatePostUseCase
{
    private readonly IPostValidator _validator;
    private readonly ISlugGenerator _slugGenerator;
    private readonly IContentProcessor _contentProcessor;
    private readonly IPostRepository _repository;
    private readonly IEventPublisher _eventPublisher;

    public CreatePostUseCase(
        IPostValidator validator,
        ISlugGenerator slugGenerator,
        IContentProcessor contentProcessor,
        IPostRepository repository,
        IEventPublisher eventPublisher)
    {
        _validator = validator;
        _slugGenerator = slugGenerator;
        _contentProcessor = contentProcessor;
        _repository = repository;
        _eventPublisher = eventPublisher;
    }

    public CreatePostResult Execute(
        string title, string content, int authorId,
        List<string>? tags = null, bool publish = false)
    {
        var post = new BlogPost
        {
            Title = title,
            Content = content,
            AuthorId = authorId,
            Tags = tags ?? new List<string>(),
            Published = publish
        };

        // 검증 관심사에 위임
        var errors = _validator.Validate(post);
        if (errors.Any())
            return new CreatePostResult(false, Errors: errors);

        // 슬러그 생성 관심사에 위임
        var slug = _slugGenerator.Generate(title);
        post.Slug = _slugGenerator.EnsureUnique(slug, _repository.GetAllSlugs());

        // 콘텐츠 가공 관심사에 위임
        post.Content = _contentProcessor.SanitizeHtml(content);

        // 저장 관심사에 위임
        var savedPost = _repository.Save(post);

        // 이벤트 발행 관심사에 위임
        _eventPublisher.Publish("post_created", new Dictionary<string, object>
        {
            ["post_id"] = savedPost.Id!,
            ["title"] = savedPost.Title,
            ["author_id"] = savedPost.AuthorId
        });

        return new CreatePostResult(
            Success: true,
            PostId: savedPost.Id,
            Slug: savedPost.Slug,
            ReadingTime: _contentProcessor.EstimateReadingTime(content)
        );
    }
}

// ============================================================
// 사용 예시 (Composition Root)
// ============================================================
public class Program
{
    public static void Main()
    {
        // DI Container 대신 수동 조립 (학습용)
        var validator = new PostValidator();
        var slugGenerator = new SlugGenerator();
        var contentProcessor = new ContentProcessor();
        var repository = new InMemoryPostRepository();
        var eventPublisher = new SimpleEventPublisher();

        eventPublisher.Subscribe("post_created", data =>
            Console.WriteLine($"[알림] 새 게시글: {data["title"]}"));

        var createPost = new CreatePostUseCase(
            validator, slugGenerator, contentProcessor,
            repository, eventPublisher);

        var result = createPost.Execute(
            title: "관심사의 분리 원칙 이해하기",
            content: "관심사의 분리(Separation of Concerns)는 소프트웨어 설계의 핵심 원칙입니다. " +
                     "이 원칙은 프로그램을 서로 구별된 영역으로 나누어 각 부분이 하나의 관심사만 " +
                     "다루도록 합니다.",
            authorId: 1,
            tags: new List<string> { "설계원칙", "소프트웨어공학" },
            publish: true
        );

        Console.WriteLine($"\n결과: Success={result.Success}, " +
                          $"PostId={result.PostId}, Slug={result.Slug}");
    }
}
```

---

## 9. SoC 적용 단계별 가이드

### 단계 1: 관심사 식별

기존 코드에서 서로 다른 관심사를 색상으로 표시해보세요.

```python
def process_order(request):
    # [검증] 입력 확인
    if not request.get("name"):        # <- 검증 관심사
        return error("이름 필요")

    # [비즈니스] 가격 계산
    total = sum(i["price"] for i in items)  # <- 비즈니스 관심사

    # [인프라] DB 저장
    db.execute("INSERT INTO ...")       # <- 인프라 관심사

    # [알림] 이메일 발송
    smtp.send(email, "주문 확인")        # <- 알림 관심사
```

### 단계 2: 인터페이스 정의

각 관심사의 경계에 인터페이스를 정의합니다.

```python
class IOrderValidator(ABC):
    @abstractmethod
    def validate(self, order: Order) -> list[str]: ...

class IPricingService(ABC):
    @abstractmethod
    def calculate(self, items: list) -> PricingResult: ...

class IOrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> int: ...

class INotificationService(ABC):
    @abstractmethod
    def notify(self, order: Order) -> None: ...
```

### 단계 3: 구현체 작성

각 인터페이스의 구현체를 별도 클래스로 작성합니다.

```python
class OrderValidator(IOrderValidator):
    def validate(self, order: Order) -> list[str]:
        # 검증 로직만 작성
        ...

class PricingService(IPricingService):
    def calculate(self, items: list) -> PricingResult:
        # 가격 계산 로직만 작성
        ...
```

### 단계 4: 유스케이스에서 조합

```python
class PlaceOrderUseCase:
    def __init__(self, validator, pricing, repository, notification):
        # 각 관심사를 주입받아 조율
        ...
```

### 단계 5: 테스트 작성

```python
# 각 관심사를 독립적으로 테스트
def test_pricing_service():
    pricing = PricingService()
    result = pricing.calculate([...])
    assert result.total == expected_total

def test_order_validator():
    validator = OrderValidator()
    errors = validator.validate(invalid_order)
    assert len(errors) > 0
```

---

## 10. 실무 팁

### SoC 자가 진단

| # | 질문 | 위험 신호 |
|---|------|----------|
| 1 | 하나의 함수/메서드가 100줄 이상인가? | 여러 관심사 혼재 |
| 2 | DB 접근 코드와 비즈니스 로직이 같은 클래스에 있는가? | 계층 미분리 |
| 3 | UI 코드에서 SQL 쿼리를 직접 실행하는가? | 프레젠테이션-데이터 혼재 |
| 4 | import/using 문이 10개 이상인가? | 너무 많은 외부 의존 |
| 5 | 하나의 파일을 변경하면 여러 다른 기능에 영향이 가는가? | 관심사 누출 |
| 6 | 비즈니스 로직 변경 없이 DB를 교체할 수 없는가? | 인프라 관심사 누출 |

### 실무에서 흔한 실수

```python
# 실수 1: 컨트롤러에 비즈니스 로직 작성
class OrderController:
    def create_order(self, request):
        # ❌ 컨트롤러에서 비즈니스 로직을 직접 처리
        if order.total > 100000:
            discount = order.total * 0.1  # 비즈니스 규칙이 컨트롤러에!
        ...

# 실수 2: 도메인 모델에 인프라 코드 포함
class Order:
    def save(self):
        # ❌ 도메인 모델이 DB에 직접 접근
        db.execute("INSERT INTO orders ...")

# 실수 3: 서비스에서 HTTP 응답 포맷 결정
class OrderService:
    def create_order(self, order):
        ...
        # ❌ 서비스가 HTTP 응답 코드를 알고 있음
        return {"status_code": 201, "body": ...}
```

### 적절한 분리 수준 판단

```
너무 적은 분리 (Monolith)          너무 과한 분리 (Over-engineering)
┌──────────────────────┐          ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
│  모든 것이 한 곳에    │          │  │ │  │ │  │ │  │ │  │
│  변경 시 전체 영향    │          │  │ │  │ │  │ │  │ │  │
│                      │          └──┘ └──┘ └──┘ └──┘ └──┘
└──────────────────────┘          50개의 작은 클래스...
                                  오히려 복잡도 증가

            ✅ 적절한 분리
            ┌──────┐ ┌──────┐ ┌──────┐
            │ 검증  │ │ 비즈  │ │ 인프라│
            │      │ │ 니스  │ │      │
            └──────┘ └──────┘ └──────┘
            명확한 경계, 관리 가능한 수

일반적 기준:
- 한 클래스: 50~200줄
- 한 메서드: 5~20줄
- 한 모듈: 관련된 3~10개 클래스
```

---

## 11. 정리 및 체크리스트

### 핵심 요약

```
┌──────────────────────────────────────────────────┐
│            관심사의 분리 핵심 원칙                   │
│                                                  │
│  1. 하나의 모듈은 하나의 관심사만 다루어라           │
│  2. 관심사의 경계에 인터페이스를 두어라              │
│  3. 수직(계층)과 수평(기능) 모두에서 분리하라        │
│  4. 횡단 관심사는 데코레이터/미들웨어로 처리하라     │
│  5. 과도한 분리는 과도한 복잡성을 가져온다           │
└──────────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] 프레젠테이션 계층이 비즈니스 로직을 포함하지 않는가?
- [ ] 비즈니스 로직이 DB 접근 방식에 의존하지 않는가?
- [ ] 각 함수가 하나의 관심사만 처리하는가?
- [ ] 각 클래스가 하나의 관심사에만 집중하는가?
- [ ] 패키지/모듈이 기능별로 분리되어 있는가?
- [ ] 횡단 관심사(로깅, 인증, 캐싱)가 비즈니스 코드와 분리되어 있는가?
- [ ] 관심사 간 통신이 인터페이스를 통해 이루어지는가?
- [ ] 각 관심사를 독립적으로 테스트할 수 있는가?

---

## 12. 관련 개념 연결

| 관련 개념 | 관계 |
|-----------|------|
| [느슨한 결합](./01-loose-coupling.md) | SoC를 적용하면 모듈 간 결합도가 자연스럽게 낮아짐 |
| [높은 응집도](./02-high-cohesion.md) | 하나의 관심사에 집중하면 응집도가 높아짐 |
| [의존성 주입](./04-dependency-injection.md) | 분리된 관심사를 조립하는 핵심 기법 |
| SOLID - SRP | 단일 책임 원칙은 클래스 수준의 SoC |
| MVC 패턴 | 웹 애플리케이션에서의 SoC 적용 |
| Clean Architecture | 아키텍처 수준에서의 SoC |
| 데코레이터 패턴 | 횡단 관심사 처리 기법 |
| AOP | 관심사 분리의 프로그래밍 패러다임 |

---

> **이전 문서**: [높은 응집도 (High Cohesion)](./02-high-cohesion.md)
>
> **다음 문서**: [의존성 주입 (Dependency Injection)](./04-dependency-injection.md)
