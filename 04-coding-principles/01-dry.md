# DRY 원칙 (Don't Repeat Yourself)

> "모든 지식은 시스템 내에서 단일하고, 모호하지 않고, 권위 있는 표현을 가져야 한다."
> — Andrew Hunt & David Thomas, *The Pragmatic Programmer* (1999)

---

## 목차

1. [DRY란 무엇인가?](#1-dry란-무엇인가)
2. [단순한 코드 중복이 아니다](#2-단순한-코드-중복이-아니다)
3. [중복의 종류](#3-중복의-종류)
4. [나쁜 예제 (Before)](#4-나쁜-예제-before)
5. [좋은 예제 (After)](#5-좋은-예제-after)
6. [Python 코드 예제](#6-python-코드-예제)
7. [C# 코드 예제](#7-c-코드-예제)
8. [DRY vs WET - 과도한 DRY의 위험성](#8-dry-vs-wet---과도한-dry의-위험성)
9. [Rule of Three](#9-rule-of-three)
10. [실무 팁: 언제 중복을 허용해야 하는가?](#10-실무-팁-언제-중복을-허용해야-하는가)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 원칙](#12-관련-원칙)

---

## 1. DRY란 무엇인가?

DRY(Don't Repeat Yourself)는 소프트웨어 개발에서 가장 기본적이고 강력한 원칙 중 하나입니다. 1999년 Andy Hunt와 Dave Thomas가 저서 *The Pragmatic Programmer*에서 처음 소개했습니다.

### 핵심 정의

**"시스템 내에서 모든 지식의 조각은 단일하고, 모호하지 않으며, 권위 있는 표현을 가져야 한다."**

이 말은 단순히 "코드를 복사-붙여넣기 하지 마라"보다 훨씬 깊은 의미를 갖습니다.

### 비유로 이해하기

도서관을 생각해 보세요. 만약 같은 책이 여러 곳에 분산되어 있다면, 개정판이 나왔을 때 모든 사본을 찾아서 교체해야 합니다. 하지만 한 곳에만 있다면? 한 번만 교체하면 됩니다. DRY는 바로 이 "진실의 단일 소스(Single Source of Truth)"를 만드는 것입니다.

---

## 2. 단순한 코드 중복이 아니다

많은 개발자들이 DRY를 "같은 코드를 두 번 쓰지 마라"로만 이해합니다. 하지만 DRY가 진정으로 말하는 것은 **지식의 중복**을 피하라는 것입니다.

```
같은 코드라도 서로 다른 이유로 변경된다면 → 중복이 아닐 수 있다
다른 코드라도 같은 지식을 표현한다면 → 중복이다
```

예를 들어, 두 함수가 우연히 같은 로직을 갖고 있지만, 서로 다른 도메인에 속하고 서로 다른 이유로 변경된다면, 이를 억지로 합치는 것은 오히려 해로울 수 있습니다.

---

## 3. 중복의 종류

### 3.1 코드 중복 (가장 명확한 형태)

동일한 코드 블록이 여러 곳에 복사-붙여넣기 되어 있는 경우입니다. 가장 쉽게 발견할 수 있고, 가장 먼저 제거해야 하는 중복입니다.

```python
# 동일한 검증 로직이 여러 곳에 존재
def create_user(email):
    if "@" not in email or "." not in email:
        raise ValueError("유효하지 않은 이메일입니다.")
    # ...

def update_user(email):
    if "@" not in email or "." not in email:
        raise ValueError("유효하지 않은 이메일입니다.")
    # ...
```

### 3.2 로직 중복 (코드는 다르지만 같은 역할)

코드의 형태는 다르지만 동일한 비즈니스 로직을 수행하는 경우입니다. 코드 검색으로는 찾기 어렵습니다.

```python
# 두 함수 모두 "할인 가격 계산"이라는 같은 지식을 표현
def get_discount_price_v1(price, rate):
    return price - (price * rate / 100)

def get_discount_price_v2(price, rate):
    return price * (1 - rate / 100)
```

### 3.3 지식 중복 (매직 넘버, 상수 중복)

동일한 비즈니스 규칙이나 값이 여러 곳에 하드코딩되어 있는 경우입니다.

```python
# "최대 재시도 횟수는 3"이라는 지식이 여러 곳에 흩어져 있음
def retry_api_call():
    for i in range(3):  # 왜 3인가?
        # ...

def retry_db_connection():
    max_attempts = 3  # 같은 3인가, 다른 3인가?
    # ...
```

---

## 4. 나쁜 예제 (Before)

### 4.1 복붙 코드

```python
# ❌ Bad: 주문 처리에서 세금 계산이 반복됨
class OrderService:
    def calculate_domestic_order(self, items):
        subtotal = sum(item.price * item.quantity for item in items)
        tax = subtotal * 0.1  # 세금 10%
        shipping = 3000 if subtotal < 50000 else 0
        return subtotal + tax + shipping

    def calculate_international_order(self, items):
        subtotal = sum(item.price * item.quantity for item in items)
        tax = subtotal * 0.1  # 세금 10% (동일 로직 반복!)
        shipping = 15000
        customs = subtotal * 0.05
        return subtotal + tax + shipping + customs

    def calculate_wholesale_order(self, items):
        subtotal = sum(item.price * item.quantity for item in items)
        tax = subtotal * 0.1  # 세금 10% (또 반복!)
        discount = subtotal * 0.15
        return subtotal + tax - discount
```

### 4.2 매직 넘버

```csharp
// ❌ Bad: 매직 넘버가 코드 전체에 흩어져 있음
public class UserService
{
    public bool ValidatePassword(string password)
    {
        return password.Length >= 8;  // 왜 8인가?
    }

    public string GeneratePassword()
    {
        // ... 비밀번호 생성 로직 ...
        while (result.Length < 8)  // 같은 규칙이 반복됨
        {
            // ...
        }
        return result;
    }

    public string GetPasswordPolicy()
    {
        return "비밀번호는 8자 이상이어야 합니다.";  // 문자열에도 중복!
    }
}
```

### 4.3 반복 로직

```python
# ❌ Bad: 로깅 + 예외처리 패턴이 모든 메서드에 반복됨
class PaymentService:
    def process_credit_card(self, amount):
        try:
            logger.info(f"신용카드 결제 시작: {amount}원")
            start_time = time.time()
            result = self._do_credit_card_payment(amount)
            elapsed = time.time() - start_time
            logger.info(f"신용카드 결제 완료: {elapsed:.2f}초")
            return result
        except Exception as e:
            logger.error(f"신용카드 결제 실패: {e}")
            raise PaymentError(f"결제 실패: {e}")

    def process_bank_transfer(self, amount):
        try:
            logger.info(f"계좌이체 시작: {amount}원")
            start_time = time.time()
            result = self._do_bank_transfer(amount)
            elapsed = time.time() - start_time
            logger.info(f"계좌이체 완료: {elapsed:.2f}초")
            return result
        except Exception as e:
            logger.error(f"계좌이체 실패: {e}")
            raise PaymentError(f"결제 실패: {e}")
```

---

## 5. 좋은 예제 (After)

### 5.1 함수 추출

```python
# ✅ Good: 공통 로직을 함수로 추출
class OrderService:
    def _calculate_subtotal(self, items):
        """소계 계산 - 단일 책임"""
        return sum(item.price * item.quantity for item in items)

    def _calculate_tax(self, amount):
        """세금 계산 - 세율이 변경되면 여기만 수정"""
        return amount * TAX_RATE

    def calculate_domestic_order(self, items):
        subtotal = self._calculate_subtotal(items)
        tax = self._calculate_tax(subtotal)
        shipping = 3000 if subtotal < FREE_SHIPPING_THRESHOLD else 0
        return subtotal + tax + shipping

    def calculate_international_order(self, items):
        subtotal = self._calculate_subtotal(items)
        tax = self._calculate_tax(subtotal)
        shipping = INTERNATIONAL_SHIPPING_FEE
        customs = subtotal * CUSTOMS_RATE
        return subtotal + tax + shipping + customs
```

### 5.2 상수 정의

```csharp
// ✅ Good: 상수를 한 곳에서 관리
public static class PasswordPolicy
{
    public const int MinLength = 8;
    public const int MaxLength = 128;
    public const string PolicyMessage = $"비밀번호는 {MinLength}자 이상 {MaxLength}자 이하여야 합니다.";
}

public class UserService
{
    public bool ValidatePassword(string password)
    {
        return password.Length >= PasswordPolicy.MinLength
            && password.Length <= PasswordPolicy.MaxLength;
    }

    public string GetPasswordPolicy()
    {
        return PasswordPolicy.PolicyMessage;
    }
}
```

### 5.3 데코레이터/AOP로 횡단 관심사 제거

```python
# ✅ Good: 데코레이터로 로깅 + 예외처리 패턴 통합
import functools
import time
import logging

logger = logging.getLogger(__name__)

def payment_operation(payment_type: str):
    """결제 작업의 공통 로깅/예외처리를 담당하는 데코레이터"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            amount = args[1] if len(args) > 1 else kwargs.get("amount")
            try:
                logger.info(f"{payment_type} 시작: {amount}원")
                start_time = time.time()
                result = func(*args, **kwargs)
                elapsed = time.time() - start_time
                logger.info(f"{payment_type} 완료: {elapsed:.2f}초")
                return result
            except Exception as e:
                logger.error(f"{payment_type} 실패: {e}")
                raise PaymentError(f"결제 실패: {e}")
        return wrapper
    return decorator

class PaymentService:
    @payment_operation("신용카드 결제")
    def process_credit_card(self, amount):
        return self._do_credit_card_payment(amount)

    @payment_operation("계좌이체")
    def process_bank_transfer(self, amount):
        return self._do_bank_transfer(amount)
```

---

## 6. Python 코드 예제

### 6.1 설정 관리에서의 DRY

```python
# ❌ Bad: 설정값이 여러 파일에 흩어져 있음
# api_client.py
class ApiClient:
    def __init__(self):
        self.base_url = "https://api.example.com"
        self.timeout = 30
        self.retry_count = 3

# notification_service.py
class NotificationService:
    def __init__(self):
        self.api_url = "https://api.example.com"  # 같은 URL 중복
        self.timeout = 30  # 같은 타임아웃 중복
```

```python
# ✅ Good: 설정을 한 곳에서 관리
# config.py
from dataclasses import dataclass

@dataclass(frozen=True)
class ApiConfig:
    base_url: str = "https://api.example.com"
    timeout: int = 30
    retry_count: int = 3

# 환경별 설정도 깔끔하게 관리
@dataclass(frozen=True)
class DevelopmentConfig(ApiConfig):
    base_url: str = "https://dev-api.example.com"
    timeout: int = 60  # 개발 환경에서는 넉넉하게

# api_client.py
class ApiClient:
    def __init__(self, config: ApiConfig):
        self.config = config

# notification_service.py
class NotificationService:
    def __init__(self, config: ApiConfig):
        self.config = config
```

### 6.2 검증 로직의 DRY

```python
# ❌ Bad: 검증 로직이 분산됨
class UserRegistration:
    def register(self, email, phone):
        # 이메일 검증
        import re
        if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', email):
            raise ValueError("유효하지 않은 이메일입니다.")

        # 전화번호 검증
        if not re.match(r'^010-\d{4}-\d{4}$', phone):
            raise ValueError("유효하지 않은 전화번호입니다.")
        # ...

class UserProfileUpdate:
    def update(self, email, phone):
        # 같은 검증 로직 반복!
        import re
        if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', email):
            raise ValueError("유효하지 않은 이메일입니다.")
        if not re.match(r'^010-\d{4}-\d{4}$', phone):
            raise ValueError("유효하지 않은 전화번호입니다.")
        # ...
```

```python
# ✅ Good: 검증 로직을 한 곳에 모음
import re
from typing import Optional

class Validators:
    """검증 로직의 단일 진실 소스(Single Source of Truth)"""

    EMAIL_PATTERN = re.compile(r'^[\w\.-]+@[\w\.-]+\.\w+$')
    PHONE_PATTERN = re.compile(r'^010-\d{4}-\d{4}$')

    @staticmethod
    def validate_email(email: str) -> Optional[str]:
        """유효하지 않으면 에러 메시지 반환, 유효하면 None"""
        if not Validators.EMAIL_PATTERN.match(email):
            return "유효하지 않은 이메일입니다."
        return None

    @staticmethod
    def validate_phone(phone: str) -> Optional[str]:
        if not Validators.PHONE_PATTERN.match(phone):
            return "유효하지 않은 전화번호입니다."
        return None

    @staticmethod
    def validate_all(**fields) -> list[str]:
        """여러 필드를 한 번에 검증"""
        errors = []
        validator_map = {
            "email": Validators.validate_email,
            "phone": Validators.validate_phone,
        }
        for field_name, value in fields.items():
            validator = validator_map.get(field_name)
            if validator:
                error = validator(value)
                if error:
                    errors.append(error)
        return errors

# 사용 예시
class UserRegistration:
    def register(self, email: str, phone: str):
        errors = Validators.validate_all(email=email, phone=phone)
        if errors:
            raise ValueError("; ".join(errors))
        # 등록 로직...

class UserProfileUpdate:
    def update(self, email: str, phone: str):
        errors = Validators.validate_all(email=email, phone=phone)
        if errors:
            raise ValueError("; ".join(errors))
        # 업데이트 로직...
```

### 6.3 데이터 변환의 DRY

```python
# ❌ Bad: 데이터 변환 로직이 여러 곳에 산재
def get_user_response(user):
    return {
        "id": user.id,
        "name": f"{user.first_name} {user.last_name}",
        "email": user.email.lower(),
        "created": user.created_at.isoformat(),
    }

def get_user_for_admin(user):
    return {
        "id": user.id,
        "name": f"{user.first_name} {user.last_name}",  # 같은 변환
        "email": user.email.lower(),  # 같은 변환
        "created": user.created_at.isoformat(),  # 같은 변환
        "role": user.role,
        "last_login": user.last_login_at.isoformat(),
    }
```

```python
# ✅ Good: 기본 변환은 모델에 위임, 필요 시 확장
from dataclasses import dataclass, asdict
from datetime import datetime

@dataclass
class UserDTO:
    """User의 기본 표현 - 변환 로직의 단일 소스"""
    id: int
    full_name: str
    email: str
    created_at: str

    @classmethod
    def from_user(cls, user) -> "UserDTO":
        return cls(
            id=user.id,
            full_name=f"{user.first_name} {user.last_name}",
            email=user.email.lower(),
            created_at=user.created_at.isoformat(),
        )

    def to_dict(self) -> dict:
        return asdict(self)

@dataclass
class AdminUserDTO(UserDTO):
    """관리자용 확장 표현"""
    role: str = ""
    last_login_at: str = ""

    @classmethod
    def from_user(cls, user) -> "AdminUserDTO":
        base = UserDTO.from_user(user)
        return cls(
            **asdict(base),
            role=user.role,
            last_login_at=user.last_login_at.isoformat(),
        )
```

---

## 7. C# 코드 예제

### 7.1 리포지토리 패턴에서의 DRY

```csharp
// ❌ Bad: 각 엔티티마다 거의 동일한 CRUD 코드 반복
public class UserRepository
{
    private readonly DbContext _context;

    public async Task<User> GetByIdAsync(int id)
    {
        var entity = await _context.Users.FindAsync(id);
        if (entity == null)
            throw new NotFoundException($"User with id {id} not found.");
        return entity;
    }

    public async Task<List<User>> GetAllAsync()
    {
        return await _context.Users.ToListAsync();
    }

    public async Task AddAsync(User entity)
    {
        await _context.Users.AddAsync(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        _context.Users.Remove(entity);
        await _context.SaveChangesAsync();
    }
}

// ProductRepository도 거의 동일한 코드... (복사-붙여넣기)
public class ProductRepository
{
    private readonly DbContext _context;

    public async Task<Product> GetByIdAsync(int id)
    {
        var entity = await _context.Products.FindAsync(id);
        if (entity == null)
            throw new NotFoundException($"Product with id {id} not found.");
        return entity;
    }
    // ... 나머지도 거의 동일 ...
}
```

```csharp
// ✅ Good: 제네릭 리포지토리로 공통 CRUD 통합
public class Repository<T> : IRepository<T> where T : class, IEntity
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<T> GetByIdAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity == null)
            throw new NotFoundException(
                $"{typeof(T).Name} with id {id} not found.");
        return entity;
    }

    public async Task<List<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        _dbSet.Remove(entity);
        await _context.SaveChangesAsync();
    }
}

// 특화된 기능이 필요한 경우에만 확장
public class UserRepository : Repository<User>, IUserRepository
{
    public UserRepository(DbContext context) : base(context) { }

    public async Task<User?> FindByEmailAsync(string email)
    {
        return await _dbSet.FirstOrDefaultAsync(u => u.Email == email);
    }
}
```

### 7.2 확장 메서드를 활용한 DRY

```csharp
// ❌ Bad: 문자열 처리 로직이 곳곳에 반복
public class OrderController
{
    public string FormatOrderNumber(int orderId)
    {
        return $"ORD-{orderId:D8}";  // 주문번호 포맷
    }
}

public class InvoiceController
{
    public string FormatOrderNumber(int orderId)
    {
        return $"ORD-{orderId:D8}";  // 같은 포맷 로직 반복!
    }
}
```

```csharp
// ✅ Good: 확장 메서드로 통합
public static class OrderExtensions
{
    public static string ToOrderNumber(this int orderId)
    {
        return $"ORD-{orderId:D8}";
    }

    public static string ToInvoiceNumber(this int invoiceId)
    {
        return $"INV-{invoiceId:D8}";
    }
}

// 사용: 어디서든 동일하게 사용
var orderNumber = orderId.ToOrderNumber();   // "ORD-00000123"
var invoiceNumber = invoiceId.ToInvoiceNumber(); // "INV-00000456"
```

### 7.3 미들웨어를 이용한 횡단 관심사 처리

```csharp
// ❌ Bad: 모든 컨트롤러 액션에 로깅 + 예외 처리 반복
[ApiController]
public class ProductController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        try
        {
            _logger.LogInformation("GetProduct 시작: {Id}", id);
            var product = await _service.GetByIdAsync(id);
            _logger.LogInformation("GetProduct 완료: {Id}", id);
            return Ok(product);
        }
        catch (NotFoundException ex)
        {
            _logger.LogWarning(ex, "Product not found: {Id}", id);
            return NotFound(ex.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "GetProduct 실패: {Id}", id);
            return StatusCode(500, "서버 오류가 발생했습니다.");
        }
    }
    // 다른 액션에서도 동일 패턴 반복...
}
```

```csharp
// ✅ Good: 미들웨어로 공통 패턴 제거
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(
        RequestDelegate next,
        ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (NotFoundException ex)
        {
            _logger.LogWarning(ex, "리소스를 찾을 수 없습니다.");
            context.Response.StatusCode = 404;
            await context.Response.WriteAsJsonAsync(
                new { error = ex.Message });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "처리되지 않은 예외가 발생했습니다.");
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(
                new { error = "서버 오류가 발생했습니다." });
        }
    }
}

// 컨트롤러는 비즈니스 로직에만 집중
[ApiController]
public class ProductController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _service.GetByIdAsync(id);
        return Ok(product);
    }
}
```

---

## 8. DRY vs WET - 과도한 DRY의 위험성

### WET (Write Everything Twice)

WET는 DRY의 반대 개념으로, "Write Everything Twice" 또는 "We Enjoy Typing"의 약자입니다. 하지만 **과도한 DRY는 WET보다 더 위험할 수 있습니다.**

### 과도한 DRY가 문제가 되는 경우

```python
# ❌ 과도한 DRY: 억지로 합친 코드
# "이메일 보내기"와 "SMS 보내기"가 우연히 비슷하다고 합침
def send_message(recipient, content, message_type):
    if message_type == "email":
        client = EmailClient()
        formatted = f"<html><body>{content}</body></html>"
        client.send(recipient, formatted)
    elif message_type == "sms":
        client = SmsClient()
        if len(content) > 160:
            content = content[:157] + "..."
        client.send(recipient, content)
    elif message_type == "push":
        client = PushClient()
        client.send(recipient, content[:100], badge=1)
    # 타입이 추가될 때마다 이 함수가 비대해짐...
```

```python
# ✅ 적절한 분리: 각각의 책임을 명확히
class EmailSender:
    def send(self, recipient: str, content: str):
        formatted = f"<html><body>{content}</body></html>"
        self.client.send(recipient, formatted)

class SmsSender:
    def send(self, recipient: str, content: str):
        if len(content) > 160:
            content = content[:157] + "..."
        self.client.send(recipient, content)

class PushSender:
    def send(self, recipient: str, content: str):
        self.client.send(recipient, content[:100], badge=1)
```

### "잘못된 추상화보다 중복이 낫다"

> *"Duplication is far cheaper than the wrong abstraction."*
> — Sandi Metz

**잘못된 추상화의 비용:**
- 이해하기 어려운 코드
- 변경할 때마다 의도치 않은 부작용
- 하나를 고치면 다른 곳이 깨짐
- 결합도가 높아져 분리가 어려움

---

## 9. Rule of Three

**"세 번 반복되면 추상화하라"** (Three Strikes and You Refactor)

Martin Fowler가 제안한 이 규칙은 DRY를 실무에 적용할 때 유용한 가이드라인입니다.

```
1번째: 그냥 작성한다.
2번째: 중복이 보이지만 참는다. (WET 허용)
3번째: 이제 패턴이 보인다. 리팩토링한다.
```

### 왜 세 번인가?

| 횟수 | 상태 | 행동 |
|------|------|------|
| 1번 | 아직 패턴인지 알 수 없음 | 그냥 작성 |
| 2번 | 패턴일 수도, 우연의 일치일 수도 | 주의 깊게 관찰 |
| 3번 | 확실한 패턴 발견 | 추상화 진행 |

세 번째에 추상화하면 **충분한 데이터 포인트**가 있으므로 올바른 추상화를 만들 가능성이 높아집니다.

---

## 10. 실무 팁: 언제 중복을 허용해야 하는가?

### 중복을 허용해도 되는 경우

1. **서로 다른 속도로 변하는 코드**
   - 결제 모듈과 알림 모듈이 우연히 비슷한 코드를 갖고 있을 때
   - 각 모듈은 독립적으로 변경되므로 억지로 합치지 않음

2. **서로 다른 바운디드 컨텍스트**
   - 마이크로서비스 간에는 중복을 허용하는 것이 일반적
   - 공유 라이브러리는 결합도를 높이고 배포를 복잡하게 만듦

3. **테스트 코드**
   - 테스트는 독립적이고 읽기 쉬워야 함
   - 과도한 추상화는 테스트의 가독성을 해침

4. **아직 패턴이 명확하지 않을 때**
   - Rule of Three를 따름
   - 성급한 추상화보다 일시적 중복이 나음

### 중복을 반드시 제거해야 하는 경우

1. **비즈니스 규칙의 중복**
   - "할인율은 10%"라는 규칙이 여러 곳에 하드코딩
   - 규칙 변경 시 일부만 수정하면 버그 발생

2. **데이터 스키마의 중복**
   - 동일한 데이터 구조가 여러 곳에 정의
   - 스키마 변경 시 불일치 발생

3. **알고리즘의 중복**
   - 동일한 계산 로직이 여러 곳에 존재
   - 알고리즘 수정 시 모든 곳을 수정해야 함

---

## 11. 정리 및 체크리스트

### 핵심 요약

| 개념 | 설명 |
|------|------|
| DRY | 지식의 단일 표현을 유지하라 |
| 코드 중복 | 동일 코드의 복사-붙여넣기 |
| 로직 중복 | 형태는 다르지만 같은 역할 |
| 지식 중복 | 같은 비즈니스 규칙의 분산 |
| Rule of Three | 세 번 반복되면 추상화 |
| WET | 과도한 DRY보다 나은 선택일 수 있음 |

### DRY 적용 체크리스트

- [ ] 동일한 비즈니스 규칙이 여러 곳에 하드코딩되어 있지 않은가?
- [ ] 매직 넘버가 상수로 정의되어 있는가?
- [ ] 동일한 검증 로직이 한 곳에서 관리되고 있는가?
- [ ] 변경 시 한 곳만 수정하면 되는가? (단일 변경 지점)
- [ ] 추상화가 현재의 중복보다 더 이해하기 쉬운가?
- [ ] 합치려는 코드들이 같은 이유로 변경되는가?
- [ ] Rule of Three를 따르고 있는가?

---

## 12. 관련 원칙

| 원칙 | 관계 |
|------|------|
| [KISS](./02-kiss.md) | DRY를 적용할 때도 단순함을 유지해야 함 |
| [YAGNI](./03-yagni.md) | 미래의 중복을 대비한 과도한 추상화를 경계 |
| [Clean Code](./04-clean-code.md) | 의미 있는 이름과 함수 추출로 DRY 실현 |
| SRP (단일 책임 원칙) | DRY와 SRP는 상호 보완적 |
| OCP (개방-폐쇄 원칙) | DRY를 통해 변경 지점을 최소화 |

---

> **다음 문서:** [KISS 원칙 (Keep It Simple, Stupid)](./02-kiss.md)
