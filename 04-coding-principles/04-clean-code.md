# Clean Code 원칙 (클린 코드)

> "깨끗한 코드는 잘 쓴 산문과 같다. 읽기 쉽고, 의도가 명확하며, 불필요한 것이 없다."
> — Robert C. Martin (Uncle Bob)

> "나중에 읽을 사람을 위해 코드를 작성하라. 그 사람은 6개월 후의 당신이다."
> — 소프트웨어 개발 격언

---

## 목차

1. [Clean Code란 무엇인가?](#1-clean-code란-무엇인가)
2. [의미 있는 이름 짓기 (Meaningful Names)](#2-의미-있는-이름-짓기-meaningful-names)
3. [함수 설계](#3-함수-설계)
4. [주석](#4-주석)
5. [코드 포맷팅](#5-코드-포맷팅)
6. [에러 처리](#6-에러-처리)
7. [Python 코드 예제 (Before/After)](#7-python-코드-예제-beforeafter)
8. [C# 코드 예제 (Before/After)](#8-c-코드-예제-beforeafter)
9. [코드 리뷰 체크리스트](#9-코드-리뷰-체크리스트)
10. [실무 팁](#10-실무-팁)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 원칙](#12-관련-원칙)

---

## 1. Clean Code란 무엇인가?

Robert C. Martin(Uncle Bob)의 저서 *Clean Code: A Handbook of Agile Software Craftsmanship* (2008)은 좋은 코드의 기준을 정립한 고전적 저서입니다.

### 대가들의 정의

| 인물 | 정의 |
|------|------|
| **Bjarne Stroustrup** (C++ 창시자) | "우아하고 효율적인 코드. 논리가 직관적이어서 버그가 숨을 곳이 없는 코드" |
| **Grady Booch** (UML 공동 창시자) | "잘 쓴 산문처럼 읽히는 코드" |
| **Dave Thomas** (Pragmatic Programmer) | "작성자가 아닌 다른 개발자가 읽고 개선할 수 있는 코드" |
| **Michael Feathers** | "누군가 주의를 기울여 작성한 것처럼 보이는 코드" |
| **Ward Cunningham** (Wiki 창시자) | "읽으면서 '당연하지'라고 느끼게 하는 코드" |

### 비유로 이해하기

좋은 코드는 **잘 정리된 주방**과 같습니다:
- 도구(함수)가 용도별로 정리되어 있고
- 재료(변수)에 라벨이 붙어 있고
- 레시피(로직)가 순서대로 명확하며
- 쓰레기(불필요한 코드)가 없습니다

나쁜 코드는 **난장판 주방**: 어디에 뭐가 있는지 모르고, 요리를 하려면 30분을 찾아다녀야 합니다.

---

## 2. 의미 있는 이름 짓기 (Meaningful Names)

> "이름을 짓는 데 시간을 투자하라. 좋은 이름은 좋은 설계로 이어진다."
> — Uncle Bob

### 2.1 변수 네이밍 규칙

```python
# ❌ Bad: 의미 없는 이름
d = 7           # 뭐가 7인가?
lst = []        # 무슨 리스트인가?
temp = get()    # 임시 뭐?
flag = True     # 무슨 플래그?
data = load()   # 무슨 데이터?
```

```python
# ✅ Good: 의도가 드러나는 이름
max_retry_count = 7
active_users = []
current_temperature = sensor.get_temperature()
is_premium_member = True
user_profile = load_user_profile(user_id)
```

### 2.2 함수 네이밍 규칙

```python
# ❌ Bad: 모호한 함수명
def process(data):      # 무엇을 process?
    pass

def handle(request):    # 어떻게 handle?
    pass

def do_it():            # 뭘 do?
    pass

def manage_stuff():     # stuff가 뭔데?
    pass
```

```python
# ✅ Good: 동사 + 명사로 명확한 의도
def validate_email_format(email: str) -> bool:
    pass

def calculate_monthly_subscription_fee(plan: Plan) -> Decimal:
    pass

def send_welcome_email(user: User) -> None:
    pass

def deactivate_expired_accounts() -> int:
    """만료된 계정을 비활성화하고, 처리된 수를 반환합니다."""
    pass
```

### 2.3 클래스 네이밍 규칙

```python
# ❌ Bad
class Data:          # 너무 일반적
    pass

class Manager:       # 뭘 관리?
    pass

class Processor:     # 뭘 처리?
    pass

class Helper:        # 뭘 도와?
    pass
```

```python
# ✅ Good: 명사로 역할이 명확한 이름
class UserAuthentication:
    pass

class OrderRepository:
    pass

class InvoiceGenerator:
    pass

class PasswordHasher:
    pass
```

### 2.4 Python vs C# 네이밍 컨벤션

| 대상 | Python (PEP 8) | C# (.NET Convention) |
|------|----------------|---------------------|
| 변수 | `snake_case` | `camelCase` |
| 함수/메서드 | `snake_case` | `PascalCase` |
| 클래스 | `PascalCase` | `PascalCase` |
| 상수 | `UPPER_SNAKE_CASE` | `PascalCase` |
| private 멤버 | `_leading_underscore` | `_camelCase` |
| 인터페이스 | N/A (Protocol) | `IPascalCase` |
| 프로퍼티 | `@property snake_case` | `PascalCase` |

```python
# Python 컨벤션
class UserService:
    MAX_LOGIN_ATTEMPTS = 5              # 상수: UPPER_SNAKE_CASE

    def __init__(self):
        self._user_repository = None    # private: _snake_case
        self.retry_count = 0            # 변수: snake_case

    def find_active_users(self):        # 메서드: snake_case
        pass

    @property
    def total_user_count(self):         # 프로퍼티: snake_case
        pass
```

```csharp
// C# 컨벤션
public class UserService : IUserService     // 클래스: PascalCase, 인터페이스: I 접두사
{
    private const int MaxLoginAttempts = 5; // 상수: PascalCase
    private readonly IUserRepository _userRepository;  // private: _camelCase

    public int RetryCount { get; set; }     // 프로퍼티: PascalCase

    public List<User> FindActiveUsers()     // 메서드: PascalCase
    {
        var activeUsers = new List<User>(); // 지역변수: camelCase
        return activeUsers;
    }
}
```

### 2.5 좋은 이름의 특성

1. **의도를 드러내라**: `elapsed_time_in_days` > `d`
2. **검색 가능하게**: `MAX_RETRY_COUNT` > `3`
3. **발음 가능하게**: `customer_address` > `cstmr_addr`
4. **일관성 유지**: `get_` / `fetch_` / `retrieve_` 중 하나만 사용
5. **도메인 용어 사용**: `invoice`, `ledger` 같은 비즈니스 용어

---

## 3. 함수 설계

### 3.1 함수는 하나의 일만 해야 한다

```python
# ❌ Bad: 한 함수가 너무 많은 일을 함
def process_order(order_data):
    # 1. 입력값 검증 (30줄)
    if not order_data.get("items"):
        raise ValueError("주문 항목이 없습니다.")
    for item in order_data["items"]:
        product = db.find_product(item["product_id"])
        if not product:
            raise ValueError(f"상품 {item['product_id']}이 존재하지 않습니다.")
        if product.stock < item["quantity"]:
            raise ValueError(f"재고 부족: {product.name}")

    # 2. 가격 계산 (20줄)
    subtotal = 0
    for item in order_data["items"]:
        product = db.find_product(item["product_id"])
        price = product.price * item["quantity"]
        if order_data.get("coupon"):
            price *= 0.9
        subtotal += price
    tax = subtotal * 0.1
    total = subtotal + tax

    # 3. 재고 차감 (15줄)
    for item in order_data["items"]:
        product = db.find_product(item["product_id"])
        product.stock -= item["quantity"]
        db.save(product)

    # 4. 주문 저장 (10줄)
    order = Order(items=order_data["items"], total=total)
    db.save(order)

    # 5. 알림 발송 (10줄)
    send_email(order_data["email"], "주문 확인", f"주문번호: {order.id}")

    return order
```

```python
# ✅ Good: 각 함수가 하나의 일만 수행
class OrderService:
    def __init__(self, repository, inventory, notification, pricing):
        self._repository = repository
        self._inventory = inventory
        self._notification = notification
        self._pricing = pricing

    def create_order(self, order_request: OrderRequest) -> Order:
        """주문을 생성합니다. (조율만 담당)"""
        self._validate_order(order_request)
        total = self._pricing.calculate_total(order_request)
        self._inventory.reserve_items(order_request.items)
        order = self._repository.save_order(order_request, total)
        self._notification.send_order_confirmation(order)
        return order

    def _validate_order(self, request: OrderRequest):
        """주문 요청을 검증합니다."""
        if not request.items:
            raise ValueError("주문 항목이 없습니다.")
        for item in request.items:
            self._inventory.check_availability(item)
```

### 3.2 적절한 함수 크기

**이상적인 함수 크기: 5~20줄**

함수가 길어지는 것은 여러 일을 하고 있다는 신호입니다.

```
함수 크기 가이드:
5줄   ████████████  이상적
10줄  ████████████  좋음
20줄  ████████████  괜찮음 (주의 필요)
30줄  ████████████  길다 (분리 고려)
50줄+ ████████████  반드시 분리 필요
```

### 3.3 매개변수 수 줄이기

```python
# ❌ Bad: 매개변수가 너무 많음
def create_user(
    first_name, last_name, email, phone,
    address_line1, address_line2, city,
    state, zip_code, country,
    birth_date, gender
):
    pass
```

```python
# ✅ Good: 관련 매개변수를 객체로 묶음
@dataclass
class Address:
    line1: str
    line2: str = ""
    city: str = ""
    state: str = ""
    zip_code: str = ""
    country: str = "KR"

@dataclass
class CreateUserRequest:
    first_name: str
    last_name: str
    email: str
    phone: str
    address: Address
    birth_date: date | None = None
    gender: str | None = None

def create_user(request: CreateUserRequest) -> User:
    pass
```

| 매개변수 수 | 평가 |
|------------|------|
| 0개 (niladic) | 최고 |
| 1개 (monadic) | 좋음 |
| 2개 (dyadic) | 괜찮음 |
| 3개 (triadic) | 가능하면 줄이기 |
| 4개+ | 객체로 묶기 |

### 3.4 부수효과 없는 함수

```python
# ❌ Bad: 이름은 "검증"인데 실제로는 상태를 변경함
class UserService:
    def validate_and_save_user(self, user):
        """이름에 'validate'라고 했지만 save도 함 → 부수효과!"""
        if not user.email:
            raise ValueError("이메일 필수")
        user.validated_at = datetime.now()  # 부수효과: 상태 변경
        self.repository.save(user)          # 부수효과: DB 저장
        return True

# 호출자가 모르는 사이에 DB가 변경됨
# validate만 할 의도로 호출하면 의도치 않은 저장 발생
```

```python
# ✅ Good: 함수가 하나의 일만 하고, 이름이 정직함
class UserService:
    def validate_user(self, user) -> list[str]:
        """검증만 수행. 상태를 변경하지 않음."""
        errors = []
        if not user.email:
            errors.append("이메일은 필수입니다.")
        if not user.name:
            errors.append("이름은 필수입니다.")
        return errors

    def save_user(self, user):
        """저장만 수행."""
        user.validated_at = datetime.now()
        self.repository.save(user)

    def register_user(self, user):
        """검증 + 저장 (이름이 의도를 정확히 표현)"""
        errors = self.validate_user(user)
        if errors:
            raise ValidationError(errors)
        self.save_user(user)
```

---

## 4. 주석

### 4.1 좋은 주석 vs 나쁜 주석

```python
# ❌ Bad: 코드를 반복하는 주석 (코드 자체로 충분히 명확)
# 사용자 이름을 가져온다
user_name = user.get_name()

# 리스트를 순회한다
for item in items:
    # 가격을 합산한다
    total += item.price

# i를 1 증가시킨다
i += 1
```

```python
# ✅ Good: "왜"를 설명하는 주석
# 한국 세법에 따라 1000원 미만은 절사 (국세기본법 제47조)
tax = math.floor(subtotal * tax_rate / 1000) * 1000

# 레거시 API가 날짜를 YYYYMMDD 형식으로만 받아서 변환 필요
date_str = target_date.strftime("%Y%m%d")

# 동시 접근 이슈 방지를 위해 비관적 락 사용
# (2024-03 결제 중복 건 재발 방지, JIRA-1234 참고)
with db.pessimistic_lock(order_id):
    process_payment(order_id)
```

### 4.2 좋은 주석의 종류

```python
# 1. 법적 주석
# Copyright (c) 2024 MyCompany. All rights reserved.
# Licensed under the MIT License.

# 2. 의도를 설명하는 주석
# 성능상의 이유로 N+1 쿼리 대신 JOIN을 사용
users = session.query(User).join(User.orders).all()

# 3. 결과를 경고하는 주석
# WARNING: 이 함수는 약 10분이 소요됩니다. 배치 작업에서만 사용하세요.
def rebuild_search_index():
    pass

# 4. TODO 주석 (담당자와 날짜 포함)
# TODO(김철수, 2025-03): 인증 토큰 만료 처리 추가 필요
def get_auth_token():
    pass

# 5. API 문서화 (docstring)
def calculate_shipping_cost(
    weight_kg: float,
    destination: str,
    is_express: bool = False
) -> Decimal:
    """배송비를 계산합니다.

    Args:
        weight_kg: 상품 무게 (kg)
        destination: 배송지 국가 코드 (ISO 3166-1 alpha-2)
        is_express: 익스프레스 배송 여부

    Returns:
        배송비 (원)

    Raises:
        ValueError: 무게가 0 이하이거나 지원하지 않는 국가인 경우
    """
    pass
```

### 4.3 코드로 의도를 표현하라

```python
# ❌ Bad: 주석이 필요한 코드
# 직원이 복지혜택을 받을 자격이 있는지 확인
if (employee.flags & 0x02) and (employee.age > 65 or (
    employee.age > 55 and employee.years_of_service > 20)):
    pass
```

```python
# ✅ Good: 주석이 필요 없는 코드
def is_eligible_for_benefits(employee: Employee) -> bool:
    """직원의 복지혜택 수급 자격을 확인합니다."""
    is_active = employee.is_active_status()
    is_retirement_age = employee.age > 65
    is_early_retirement = (
        employee.age > 55
        and employee.years_of_service > 20
    )
    return is_active and (is_retirement_age or is_early_retirement)
```

---

## 5. 코드 포맷팅

### 5.1 수직 형식 (관련 코드 가까이)

```python
# ❌ Bad: 관련 코드가 떨어져 있음
class OrderService:
    def __init__(self):
        self.db = Database()
        self.mailer = Mailer()
        self.logger = Logger()
        self.cache = Cache()
        self.validator = Validator()

    def validate_order(self, order):
        # ... 검증 로직 (50줄)
        pass

    def send_notification(self, order):
        # ... 알림 로직 (30줄)
        pass

    def create_order(self, order_data):
        # create_order가 validate_order와 send_notification을 호출하지만
        # 수십 줄이나 떨어져 있어 맥락 파악이 어려움
        pass

    def cancel_order(self, order_id):
        pass

    def calculate_total(self, items):
        # create_order에서 사용하지만 멀리 떨어져 있음
        pass
```

```python
# ✅ Good: 신문 기사 원칙 - 높은 수준의 개념이 위에, 세부 사항이 아래에
class OrderService:
    def __init__(self, repository, pricing, notification):
        self._repository = repository
        self._pricing = pricing
        self._notification = notification

    # === 공개 API (높은 수준) ===

    def create_order(self, order_data: dict) -> Order:
        """주문을 생성합니다."""
        self._validate_items(order_data["items"])
        total = self._pricing.calculate_total(order_data["items"])
        order = self._save_order(order_data, total)
        self._notification.send_confirmation(order)
        return order

    def cancel_order(self, order_id: int) -> None:
        """주문을 취소합니다."""
        order = self._repository.find_by_id(order_id)
        order.cancel()
        self._repository.save(order)
        self._notification.send_cancellation(order)

    # === 내부 메서드 (낮은 수준) ===

    def _validate_items(self, items: list[dict]) -> None:
        """주문 항목을 검증합니다."""
        if not items:
            raise ValueError("주문 항목이 비어있습니다.")

    def _save_order(self, order_data: dict, total: Decimal) -> Order:
        """주문을 저장합니다."""
        order = Order(items=order_data["items"], total=total)
        return self._repository.save(order)
```

### 5.2 수평 형식 (적절한 들여쓰기)

```python
# ❌ Bad: 가로로 너무 긴 코드
result = database.query(User).filter(User.age >= 18, User.is_active == True, User.country == "KR").order_by(User.created_at.desc()).limit(100).all()

# ❌ Bad: 불필요한 정렬 (유지보수가 어려움)
name            = "Kim"
age             = 30
email           = "kim@example.com"
phone_number    = "010-1234-5678"
```

```python
# ✅ Good: 적절한 줄 바꿈
result = (
    database.query(User)
    .filter(
        User.age >= 18,
        User.is_active == True,
        User.country == "KR"
    )
    .order_by(User.created_at.desc())
    .limit(100)
    .all()
)

# ✅ Good: 자연스러운 들여쓰기
name = "Kim"
age = 30
email = "kim@example.com"
phone_number = "010-1234-5678"
```

```csharp
// ✅ Good: C#에서의 적절한 포맷팅
var result = await context.Users
    .Where(u => u.Age >= 18)
    .Where(u => u.IsActive)
    .Where(u => u.Country == "KR")
    .OrderByDescending(u => u.CreatedAt)
    .Take(100)
    .ToListAsync();
```

---

## 6. 에러 처리

### 6.1 예외 사용 가이드

```python
# ❌ Bad: 에러 코드 반환 (C 스타일)
def find_user(user_id):
    user = db.query(User).get(user_id)
    if user is None:
        return None, "USER_NOT_FOUND"
    if not user.is_active:
        return None, "USER_INACTIVE"
    return user, "SUCCESS"

# 호출자가 매번 에러 코드를 확인해야 함
user, error = find_user(123)
if error == "USER_NOT_FOUND":
    # ...
elif error == "USER_INACTIVE":
    # ...
```

```python
# ✅ Good: 명확한 예외 사용
class UserNotFoundError(Exception):
    """사용자를 찾을 수 없을 때 발생합니다."""
    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"사용자 {user_id}을(를) 찾을 수 없습니다.")

class UserInactiveError(Exception):
    """비활성 사용자에 접근할 때 발생합니다."""
    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"사용자 {user_id}은(는) 비활성 상태입니다.")

def find_active_user(user_id: int) -> User:
    """활성 사용자를 찾아 반환합니다."""
    user = db.query(User).get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    if not user.is_active:
        raise UserInactiveError(user_id)
    return user
```

```csharp
// ✅ Good: C#에서의 명확한 예외 처리
public class UserNotFoundException : Exception
{
    public int UserId { get; }

    public UserNotFoundException(int userId)
        : base($"사용자 {userId}을(를) 찾을 수 없습니다.")
    {
        UserId = userId;
    }
}

public User FindActiveUser(int userId)
{
    var user = _context.Users.Find(userId)
        ?? throw new UserNotFoundException(userId);

    if (!user.IsActive)
        throw new UserInactiveException(userId);

    return user;
}
```

### 6.2 None/null 반환 피하기

```python
# ❌ Bad: None을 반환하면 호출자가 항상 null 체크해야 함
def find_user(user_id):
    return db.query(User).get(user_id)  # None일 수 있음

# 호출자: NoneType 에러의 원인
user = find_user(123)
print(user.name)  # user가 None이면 AttributeError!

# 방어적으로 작성하면 None 체크가 코드 전체에 퍼짐
user = find_user(123)
if user is not None:
    if user.address is not None:
        if user.address.city is not None:
            print(user.address.city)
```

```python
# ✅ Good: 전략 1 - 예외 발생 (해당 데이터가 반드시 있어야 하는 경우)
def get_user(user_id: int) -> User:
    """사용자를 반환합니다. 없으면 예외를 발생시킵니다."""
    user = db.query(User).get(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user

# ✅ Good: 전략 2 - Optional 타입 명시 (없을 수 있는 경우)
def find_user(user_id: int) -> User | None:
    """사용자를 찾습니다. 없으면 None을 반환합니다."""
    return db.query(User).get(user_id)

# ✅ Good: 전략 3 - 기본값 제공 (Null Object Pattern)
def get_user_display_name(user_id: int) -> str:
    """사용자 표시 이름을 반환합니다. 없으면 '알 수 없음'."""
    user = db.query(User).get(user_id)
    if user is None:
        return "알 수 없음"
    return user.display_name
```

```csharp
// ✅ Good: C#에서 null 안전하게 처리하기

// 전략 1: 없으면 예외 (메서드 이름에 Get 사용)
public User GetUser(int id)
{
    return _context.Users.Find(id)
        ?? throw new UserNotFoundException(id);
}

// 전략 2: nullable 반환 명시 (메서드 이름에 Find/Try 사용)
public User? FindUser(int id)
{
    return _context.Users.Find(id);
}

// 전략 3: TryGet 패턴
public bool TryGetUser(int id, out User user)
{
    user = _context.Users.Find(id);
    return user != null;
}

// 전략 4: null 조건 연산자 활용
var cityName = user?.Address?.City?.Name ?? "알 수 없음";
```

### 6.3 예외 처리 원칙

```python
# ❌ Bad: 광범위한 예외 잡기 (Pokemon Exception Handling)
try:
    result = do_something()
except Exception:
    pass  # 모든 예외를 무시... 문제가 숨겨짐

# ❌ Bad: 제어 흐름에 예외 사용
try:
    value = dictionary[key]
except KeyError:
    value = default_value
```

```python
# ✅ Good: 구체적인 예외만 잡기
try:
    result = api_client.fetch_data(url)
except ConnectionError:
    logger.warning("API 연결 실패. 캐시된 데이터를 사용합니다.")
    result = cache.get_last_known_data()
except TimeoutError:
    logger.warning("API 타임아웃. 재시도합니다.")
    result = api_client.fetch_data(url, timeout=60)

# ✅ Good: 제어 흐름에는 조건문 사용
value = dictionary.get(key, default_value)
```

---

## 7. Python 코드 예제 (Before/After)

### 7.1 사용자 서비스 리팩토링

```python
# ❌ Before: 클린 코드 원칙 위반 종합 세트
class US:  # 모호한 이름
    def __init__(self):
        self.d = {}  # 무슨 데이터?

    def p(self, n, e, a):  # 무슨 매개변수?
        # check email
        if "@" not in e:
            return -1  # 에러 코드 반환
        if len(n) < 2:
            return -2
        # make user
        id = len(self.d) + 1
        self.d[id] = {"n": n, "e": e, "a": a, "s": 1}  # 매직 넘버
        # send email
        import smtplib  # 함수 안에서 import
        try:
            server = smtplib.SMTP("smtp.gmail.com", 587)
            server.sendmail("admin@co.com", e, f"Welcome {n}")
        except:  # bare except
            pass  # 에러 무시
        return id

    def g(self, id):  # get인지 generate인지 모호
        if id in self.d:
            u = self.d[id]
            if u["s"] == 1:  # 매직 넘버
                return u
            else:
                return None
        return None
```

```python
# ✅ After: 클린 코드 원칙 적용

from dataclasses import dataclass, field
from datetime import datetime

# 의미 있는 상수 정의
class UserStatus:
    ACTIVE = "active"
    INACTIVE = "inactive"

MIN_NAME_LENGTH = 2

@dataclass
class User:
    """사용자 도메인 모델"""
    id: int
    name: str
    email: str
    age: int
    status: str = UserStatus.ACTIVE
    created_at: datetime = field(default_factory=datetime.now)

class InvalidEmailError(Exception):
    """유효하지 않은 이메일 형식"""
    pass

class InvalidNameError(Exception):
    """유효하지 않은 사용자 이름"""
    pass

class UserNotFoundError(Exception):
    """사용자를 찾을 수 없음"""
    pass

class UserService:
    """사용자 관련 비즈니스 로직을 처리합니다."""

    def __init__(self, repository, email_service):
        self._repository = repository
        self._email_service = email_service

    def register_user(self, name: str, email: str, age: int) -> User:
        """새 사용자를 등록합니다.

        Args:
            name: 사용자 이름 (2자 이상)
            email: 이메일 주소
            age: 나이

        Returns:
            생성된 User 객체

        Raises:
            InvalidEmailError: 이메일 형식이 유효하지 않은 경우
            InvalidNameError: 이름이 너무 짧은 경우
        """
        self._validate_email(email)
        self._validate_name(name)

        user = self._repository.create(
            User(id=0, name=name, email=email, age=age)
        )

        self._send_welcome_email(user)
        return user

    def find_active_user(self, user_id: int) -> User:
        """활성 사용자를 조회합니다.

        Raises:
            UserNotFoundError: 사용자를 찾을 수 없거나 비활성 상태인 경우
        """
        user = self._repository.find_by_id(user_id)
        if user is None or user.status != UserStatus.ACTIVE:
            raise UserNotFoundError(f"활성 사용자 {user_id}을(를) 찾을 수 없습니다.")
        return user

    def _validate_email(self, email: str) -> None:
        if "@" not in email or "." not in email.split("@")[-1]:
            raise InvalidEmailError(f"유효하지 않은 이메일: {email}")

    def _validate_name(self, name: str) -> None:
        if len(name) < MIN_NAME_LENGTH:
            raise InvalidNameError(
                f"이름은 {MIN_NAME_LENGTH}자 이상이어야 합니다."
            )

    def _send_welcome_email(self, user: User) -> None:
        """환영 이메일 발송을 시도합니다. 실패해도 등록은 유지됩니다."""
        try:
            self._email_service.send(
                to=user.email,
                subject="환영합니다!",
                body=f"{user.name}님, 가입을 환영합니다."
            )
        except Exception as e:
            logger.warning(
                "환영 이메일 발송 실패 (user_id=%d): %s",
                user.id, str(e)
            )
```

### 7.2 데이터 처리 리팩토링

```python
# ❌ Before
def proc(f):
    r = []
    with open(f) as fp:
        for l in fp:
            p = l.strip().split(",")
            if len(p) >= 3:
                if p[2].strip().isdigit():
                    v = int(p[2].strip())
                    if v > 0:
                        r.append({"n": p[0].strip(), "t": p[1].strip(), "v": v})
    t = 0
    for i in r:
        t += i["v"]
    for i in r:
        i["p"] = round(i["v"] / t * 100, 2)
    r.sort(key=lambda x: x["v"], reverse=True)
    return r
```

```python
# ✅ After
import csv
from dataclasses import dataclass
from pathlib import Path

@dataclass
class SalesRecord:
    """매출 레코드"""
    product_name: str
    category: str
    amount: int
    percentage: float = 0.0

def load_sales_report(file_path: str | Path) -> list[SalesRecord]:
    """CSV 파일에서 매출 보고서를 로드합니다.

    CSV 형식: 상품명, 카테고리, 매출액
    매출액이 0 이하인 항목은 제외됩니다.

    Args:
        file_path: CSV 파일 경로

    Returns:
        매출액 비중이 포함된 SalesRecord 리스트 (매출액 내림차순)
    """
    records = _parse_sales_csv(file_path)
    _calculate_percentages(records)
    return sorted(records, key=lambda r: r.amount, reverse=True)

def _parse_sales_csv(file_path: str | Path) -> list[SalesRecord]:
    """CSV 파일을 파싱하여 SalesRecord 리스트를 반환합니다."""
    records = []
    with open(file_path, newline="", encoding="utf-8") as f:
        reader = csv.reader(f)
        for row in reader:
            record = _try_parse_row(row)
            if record is not None:
                records.append(record)
    return records

def _try_parse_row(row: list[str]) -> SalesRecord | None:
    """CSV 행을 SalesRecord로 변환합니다. 유효하지 않으면 None."""
    if len(row) < 3:
        return None

    amount_str = row[2].strip()
    if not amount_str.isdigit():
        return None

    amount = int(amount_str)
    if amount <= 0:
        return None

    return SalesRecord(
        product_name=row[0].strip(),
        category=row[1].strip(),
        amount=amount,
    )

def _calculate_percentages(records: list[SalesRecord]) -> None:
    """각 레코드의 매출 비중(%)을 계산합니다."""
    total = sum(r.amount for r in records)
    if total == 0:
        return

    for record in records:
        record.percentage = round(record.amount / total * 100, 2)
```

---

## 8. C# 코드 예제 (Before/After)

### 8.1 주문 처리 리팩토링

```csharp
// ❌ Before: 클린 코드 원칙 위반
public class OM // 모호한 클래스명
{
    public object Do(Dictionary<string, object> d) // 모호한 메서드명
    {
        // validate
        if (d == null) return null;
        if (!d.ContainsKey("items")) return null;
        var items = d["items"] as List<Dictionary<string, object>>;
        if (items == null || items.Count == 0) return null;

        double t = 0; // 모호한 변수명
        foreach (var i in items)
        {
            var p = Convert.ToDouble(i["price"]);
            var q = Convert.ToInt32(i["qty"]);
            t += p * q;
        }

        // tax
        t = t * 1.1; // 매직 넘버

        // discount
        if (t > 100000) // 매직 넘버
            t = t * 0.95; // 매직 넘버

        // save
        var conn = new SqlConnection("Server=localhost;Database=mydb;...");
        conn.Open();
        var cmd = new SqlCommand(
            $"INSERT INTO orders (total, created) VALUES ({t}, '{DateTime.Now}')",
            conn); // SQL Injection 취약점!
        cmd.ExecuteNonQuery();
        conn.Close(); // Dispose 안 함

        // email
        try
        {
            var smtp = new SmtpClient("smtp.gmail.com");
            smtp.Send("admin@co.com", d["email"].ToString(),
                "Order", $"Total: {t}");
        }
        catch { } // 에러 무시

        return new { total = t };
    }
}
```

```csharp
// ✅ After: 클린 코드 원칙 적용

// 의미 있는 상수
public static class OrderPolicy
{
    public const decimal TaxRate = 0.1m;
    public const decimal BulkDiscountThreshold = 100_000m;
    public const decimal BulkDiscountRate = 0.05m;
}

// 명확한 도메인 모델
public record OrderItem(string ProductName, decimal Price, int Quantity)
{
    public decimal Subtotal => Price * Quantity;
}

public record CreateOrderRequest(
    string CustomerEmail,
    List<OrderItem> Items
);

public record OrderResult(int OrderId, decimal Total);

// 명확한 예외
public class EmptyOrderException : Exception
{
    public EmptyOrderException()
        : base("주문에 최소 1개 이상의 항목이 필요합니다.") { }
}

// 책임이 분리된 서비스
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    private readonly ILogger<OrderService> _logger;

    public OrderService(
        IOrderRepository repository,
        IEmailService emailService,
        ILogger<OrderService> logger)
    {
        _repository = repository;
        _emailService = emailService;
        _logger = logger;
    }

    public async Task<OrderResult> CreateOrderAsync(CreateOrderRequest request)
    {
        ValidateOrder(request);

        var subtotal = CalculateSubtotal(request.Items);
        var tax = CalculateTax(subtotal);
        var discount = CalculateBulkDiscount(subtotal);
        var total = subtotal + tax - discount;

        var orderId = await _repository.SaveAsync(request, total);

        await SendOrderConfirmationAsync(request.CustomerEmail, orderId, total);

        return new OrderResult(orderId, total);
    }

    private static void ValidateOrder(CreateOrderRequest request)
    {
        ArgumentNullException.ThrowIfNull(request);
        if (request.Items is not { Count: > 0 })
            throw new EmptyOrderException();
    }

    private static decimal CalculateSubtotal(List<OrderItem> items)
    {
        return items.Sum(item => item.Subtotal);
    }

    private static decimal CalculateTax(decimal subtotal)
    {
        return subtotal * OrderPolicy.TaxRate;
    }

    private static decimal CalculateBulkDiscount(decimal subtotal)
    {
        if (subtotal <= OrderPolicy.BulkDiscountThreshold)
            return 0;
        return subtotal * OrderPolicy.BulkDiscountRate;
    }

    private async Task SendOrderConfirmationAsync(
        string email, int orderId, decimal total)
    {
        try
        {
            await _emailService.SendAsync(
                to: email,
                subject: "주문이 확인되었습니다",
                body: $"주문번호: {orderId}, 총액: {total:N0}원"
            );
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex,
                "주문 확인 이메일 발송 실패 (OrderId: {OrderId})", orderId);
            // 이메일 실패로 주문 자체가 실패하지 않도록 함
        }
    }
}
```

### 8.2 API 컨트롤러 리팩토링

```csharp
// ❌ Before: 모든 로직이 컨트롤러에
[ApiController]
[Route("api/products")]
public class ProductController : ControllerBase
{
    private readonly MyDbContext _db;

    [HttpGet]
    public IActionResult Get(string? q, string? sort, int page = 1)
    {
        var products = _db.Products.AsQueryable();

        if (!string.IsNullOrEmpty(q))
            products = products.Where(p =>
                p.Name.Contains(q) || p.Description.Contains(q));

        if (sort == "price_asc")
            products = products.OrderBy(p => p.Price);
        else if (sort == "price_desc")
            products = products.OrderByDescending(p => p.Price);
        else if (sort == "name")
            products = products.OrderBy(p => p.Name);
        else
            products = products.OrderByDescending(p => p.CreatedAt);

        var pageSize = 20; // 매직 넘버
        var total = products.Count();
        var items = products
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => new
            {
                p.Id,
                p.Name,
                p.Price,
                p.Description,
                Category = p.Category.Name // N+1 가능성
            })
            .ToList();

        return Ok(new { items, total, page, pageSize });
    }
}
```

```csharp
// ✅ After: 역할 분리 + 의미 있는 네이밍

// 요청/응답 모델
public record ProductSearchRequest(
    string? Query = null,
    ProductSortOption Sort = ProductSortOption.Newest,
    int Page = 1,
    int PageSize = 20
);

public enum ProductSortOption
{
    Newest,
    PriceAscending,
    PriceDescending,
    Name
}

public record ProductListItem(
    int Id,
    string Name,
    decimal Price,
    string Description,
    string CategoryName
);

public record PagedResult<T>(
    List<T> Items,
    int TotalCount,
    int Page,
    int PageSize
)
{
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasNextPage => Page < TotalPages;
}

// 컨트롤러: HTTP 관심사만 처리
[ApiController]
[Route("api/products")]
public class ProductController : ControllerBase
{
    private readonly ProductService _productService;

    public ProductController(ProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<IActionResult> SearchProducts(
        [FromQuery] ProductSearchRequest request)
    {
        var result = await _productService.SearchAsync(request);
        return Ok(result);
    }
}

// 서비스: 비즈니스 로직
public class ProductService
{
    private readonly IProductRepository _repository;

    public ProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<PagedResult<ProductListItem>> SearchAsync(
        ProductSearchRequest request)
    {
        return await _repository.SearchAsync(request);
    }
}
```

---

## 9. 코드 리뷰 체크리스트

코드 리뷰 시 다음 항목을 확인하세요:

### 이름 (Naming)

- [ ] 변수/함수/클래스 이름이 의도를 명확히 드러내는가?
- [ ] 약어(abbreviation) 없이 풀어 썼는가?
- [ ] 일관된 네이밍 컨벤션을 따르는가?
- [ ] 매직 넘버가 상수로 정의되어 있는가?

### 함수 (Functions)

- [ ] 함수가 하나의 일만 하는가?
- [ ] 함수 크기가 적절한가? (20줄 이하 권장)
- [ ] 매개변수 수가 3개 이하인가?
- [ ] 부수효과가 없거나, 있다면 이름에 드러나는가?
- [ ] Early Return을 사용하여 중첩을 줄였는가?

### 에러 처리 (Error Handling)

- [ ] 적절한 예외를 사용하고 있는가?
- [ ] bare except / catch(Exception)을 남용하지 않는가?
- [ ] null/None 반환 대신 예외나 Optional을 사용하는가?
- [ ] 에러 메시지가 디버깅에 충분한 정보를 포함하는가?

### 구조 (Structure)

- [ ] 관련 코드가 가까이 위치하는가?
- [ ] 적절한 줄 바꿈과 들여쓰기를 사용하는가?
- [ ] 코드가 "신문 기사"처럼 위에서 아래로 읽히는가?
- [ ] 불필요한 주석 없이 코드 자체로 의도가 전달되는가?

### 일반 (General)

- [ ] 중복 코드가 없는가? (DRY)
- [ ] 불필요한 복잡성이 없는가? (KISS)
- [ ] 지금 필요하지 않은 기능이 있지는 않은가? (YAGNI)
- [ ] 테스트가 작성되어 있는가?

---

## 10. 실무 팁

### 10.1 점진적 개선 (Boy Scout Rule)

> "캠핑장은 도착했을 때보다 깨끗하게 떠나라."
> — 보이스카우트 규칙

코드도 마찬가지입니다. **코드를 수정할 때 발견한 작은 문제를 함께 개선**하세요.

```python
# 버그를 수정하러 왔다가 발견한 문제들:

# 원래 코드
def calc(d, r):  # 모호한 이름
    # calculate total
    return d * (1 + r)  # 불필요한 주석

# 버그 수정 + 보이스카우트 규칙 적용
def calculate_total_with_tax(amount: Decimal, tax_rate: Decimal) -> Decimal:
    """세금을 포함한 총액을 계산합니다."""
    return amount * (1 + tax_rate)
```

단, 너무 큰 리팩토링은 별도의 PR로 분리하세요.

### 10.2 클린 코드 적용 우선순위

모든 원칙을 한 번에 적용하는 것은 비현실적입니다. 다음 순서로 적용하세요:

```
1단계 (즉시): 의미 있는 이름 짓기
    → 비용 없이 가독성 대폭 향상

2단계 (일주일): 함수 크기 줄이기
    → 한 함수가 한 가지 일만 하도록

3단계 (한 달): 에러 처리 개선
    → 예외 사용, null 안전

4단계 (지속적): 보이스카우트 규칙
    → 수정할 때마다 조금씩 개선
```

### 10.3 자주 하는 실수들

| 실수 | 해결 |
|------|------|
| 완벽한 클린 코드를 한번에 하려 함 | 점진적으로 개선 |
| 클린 코드 = 짧은 코드라고 생각 | 가독성이 핵심 |
| 팀의 합의 없이 혼자 스타일 변경 | 팀 컨벤션 우선 |
| 레거시 코드 전체를 리팩토링 | 수정하는 부분만 개선 |
| 주석을 완전히 배제 | "왜"를 설명하는 주석은 좋음 |

### 10.4 도구 활용

| 언어 | 린터/포맷터 | 용도 |
|------|------------|------|
| Python | `ruff` | 린팅 + 포맷팅 (Black + isort + Flake8 통합) |
| Python | `mypy` | 타입 체크 |
| C# | `.editorconfig` | 코딩 스타일 통일 |
| C# | `dotnet format` | 자동 포맷팅 |
| 공통 | SonarQube / SonarLint | 코드 품질 분석 |
| 공통 | IDE 기본 기능 | 리팩토링 (Rename, Extract Method 등) |

```python
# pyproject.toml 설정 예시
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
]

[tool.mypy]
strict = true
```

---

## 11. 정리 및 체크리스트

### 핵심 요약

| 영역 | 핵심 원칙 |
|------|----------|
| 이름 짓기 | 의도를 드러내는 이름, 컨벤션 준수 |
| 함수 설계 | 하나의 일, 작은 크기, 적은 매개변수 |
| 주석 | "왜"를 설명, 코드로 표현 가능하면 주석 삭제 |
| 포맷팅 | 관련 코드 가까이, 일관된 스타일 |
| 에러 처리 | 명확한 예외, null 반환 최소화 |
| 전반 | 보이스카우트 규칙, 점진적 개선 |

### 클린 코드 종합 체크리스트

- [ ] 코드를 읽었을 때 "당연하다"라는 느낌이 드는가?
- [ ] 새로운 팀원이 30분 안에 이해할 수 있는가?
- [ ] 변경이 필요할 때 한 곳만 수정하면 되는가?
- [ ] 테스트가 코드의 의도를 문서처럼 설명하는가?
- [ ] 도착했을 때보다 깨끗하게 남겼는가? (보이스카우트 규칙)

---

## 12. 관련 원칙

| 원칙 | 관계 |
|------|------|
| [DRY](./01-dry.md) | 중복 제거를 통한 깨끗한 코드 |
| [KISS](./02-kiss.md) | 단순함은 클린 코드의 핵심 가치 |
| [YAGNI](./03-yagni.md) | 불필요한 코드가 없는 것이 깨끗한 코드 |
| SRP (단일 책임 원칙) | 함수와 클래스가 하나의 일만 하도록 |
| SOLID | 클린 코드의 설계 수준 확장 |
| 리팩토링 | 클린 코드를 만드는 지속적 과정 |

---

## 참고 자료

- Robert C. Martin, *Clean Code: A Handbook of Agile Software Craftsmanship* (2008)
- Robert C. Martin, *The Clean Coder: A Code of Conduct for Professional Programmers* (2011)
- Martin Fowler, *Refactoring: Improving the Design of Existing Code* (2018, 2nd Edition)
- PEP 8 - Style Guide for Python Code
- .NET Coding Conventions - Microsoft Docs

---

> **이전 문서:** [YAGNI 원칙 (You Ain't Gonna Need It)](./03-yagni.md)
