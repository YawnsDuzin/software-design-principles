# Builder 패턴 (빌더)

> **핵심 한줄 요약**: 복잡한 객체의 생성 과정을 단계별로 분리하여, 같은 생성 과정으로 다양한 표현을 만들 수 있게 한다.

---

## 목차

1. [의도 (Intent)](#1-의도-intent)
2. [문제 상황 (Problem)](#2-문제-상황-problem)
3. [Telescoping Constructor 안티패턴](#3-telescoping-constructor-안티패턴)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [Director 역할](#7-director-역할)
8. [실무 예제: HTTP Request Builder](#8-실무-예제-http-request-builder)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 의도 (Intent)

Builder 패턴은 **복잡한 객체를 단계적으로 구성**할 수 있게 한다. 같은 구성 코드를 사용하여 객체의 **다양한 타입과 표현**을 생성할 수 있다.

핵심 아이디어:

- 객체 생성 코드를 **객체 자체에서 분리**한다
- 객체를 한 번에 만드는 대신 **단계별로 조립**한다
- 필요한 단계만 호출하여 **원하는 구성**의 객체를 만든다
- **Fluent Interface**(메서드 체이닝)로 가독성 높은 생성 코드 작성

### 실생활 비유

```
피자 주문을 생각해 보자.

"치즈 피자 라지, 치즈 크러스트, 토핑 올리브와 페퍼로니 추가"

이걸 생성자 하나로 표현하면:
  Pizza("라지", "치즈 크러스트", true, false, true, false, true, ...)
  → 파라미터 순서를 외워야 하고, 어떤 값이 뭔지 알기 어렵다

빌더로 표현하면:
  PizzaBuilder()
    .size("라지")
    .crust("치즈")
    .add_topping("올리브")
    .add_topping("페퍼로니")
    .build()
  → 각 단계가 명확하고 읽기 쉽다
```

---

## 2. 문제 상황 (Problem)

### 왜 필요한가?

| 상황 | 설명 |
|------|------|
| **매개변수 폭발** | 생성자에 10개 이상의 매개변수가 필요한 경우 |
| **선택적 매개변수** | 대부분의 매개변수가 선택적(optional)인 경우 |
| **불변 객체 생성** | 한 번 만들면 변경할 수 없는 객체를 유연하게 생성 |
| **복잡한 조립 순서** | 객체의 부분들을 특정 순서로 조립해야 하는 경우 |
| **동일 생성 과정, 다른 결과** | 같은 단계로 다른 표현의 객체를 만들어야 하는 경우 |

---

## 3. Telescoping Constructor 안티패턴

Builder 패턴이 해결하는 대표적인 안티패턴이다.

### 문제: 생성자가 망원경처럼 늘어남

```python
# 안티패턴: Telescoping Constructor
class User:
    def __init__(self, name, email, age=None, phone=None,
                 address=None, company=None, role=None,
                 is_active=True, is_admin=False,
                 preferred_language="ko", timezone="Asia/Seoul",
                 avatar_url=None, bio=None):
        self.name = name
        self.email = email
        self.age = age
        self.phone = phone
        self.address = address
        self.company = company
        self.role = role
        self.is_active = is_active
        self.is_admin = is_admin
        self.preferred_language = preferred_language
        self.timezone = timezone
        self.avatar_url = avatar_url
        self.bio = bio

# 사용 시 문제점:
user = User("김철수", "kim@example.com", 30, None, "서울시",
            "ABC회사", "개발자", True, False, "ko", "Asia/Seoul",
            None, "안녕하세요")
# → None은 뭐고 True는 뭐고 False는 뭐지? 순서를 외워야 한다!
```

```csharp
// C# 안티패턴: Telescoping Constructor
public class User
{
    public User(string name, string email) { }
    public User(string name, string email, int age) { }
    public User(string name, string email, int age, string phone) { }
    public User(string name, string email, int age, string phone,
                string address) { }
    public User(string name, string email, int age, string phone,
                string address, string company, string role) { }
    // → 생성자 오버로드가 끝없이 늘어남!
}
```

### 해결: Builder 패턴

```python
# Builder 패턴 적용
user = (UserBuilder("김철수", "kim@example.com")
    .age(30)
    .address("서울시")
    .company("ABC회사")
    .role("개발자")
    .bio("안녕하세요")
    .build())
# → 필요한 것만 설정, 각 값의 의미가 명확!
```

---

## 4. UML 다이어그램

```
┌────────────────────────┐       ┌──────────────────────┐
│      Director          │       │   Builder (interface) │
├────────────────────────┤       ├──────────────────────┤
│ - builder: Builder     │──────→│ + buildStepA()       │
├────────────────────────┤       │ + buildStepB()       │
│ + construct()          │       │ + buildStepC()       │
│   : builder.buildA()   │       │ + getResult(): Product│
│     builder.buildB()   │       └──────────┬───────────┘
│     builder.buildC()   │                  │ 구현
└────────────────────────┘            ┌─────┴─────┐
                                      │           │
                              ┌───────┴──┐  ┌─────┴────────┐
                              │BuilderA  │  │ BuilderB     │
                              ├──────────┤  ├──────────────┤
                              │+buildA() │  │ +buildA()    │
                              │+buildB() │  │ +buildB()    │
                              │+buildC() │  │ +buildC()    │
                              │+getResult│  │ +getResult() │
                              │ :ProductA│  │  :ProductB   │
                              └──────────┘  └──────────────┘

[흐름]
1. Director가 Builder에게 단계별로 지시
2. Builder가 각 단계를 구현
3. 최종 결과물(Product)을 getResult()로 반환

[Fluent Builder 변형 (Director 없이)]
  product = ConcreteBuilder()
      .stepA(valueA)
      .stepB(valueB)
      .stepC(valueC)
      .build()
```

---

## 5. Python 구현

### 방식 1: Step Builder (단계별 빌더)

```python
from dataclasses import dataclass, field
from typing import Optional


@dataclass(frozen=True)
class Pizza:
    """피자 - 불변 객체 (frozen=True)"""
    size: str
    crust: str
    sauce: str
    cheese: str
    toppings: tuple[str, ...]  # 불변 시퀀스
    extra_cheese: bool = False
    notes: str = ""


class PizzaBuilder:
    """피자 빌더 - 단계별 생성"""

    def __init__(self):
        self._size: str = "미디엄"
        self._crust: str = "오리지널"
        self._sauce: str = "토마토"
        self._cheese: str = "모짜렐라"
        self._toppings: list[str] = []
        self._extra_cheese: bool = False
        self._notes: str = ""

    def size(self, size: str) -> 'PizzaBuilder':
        valid_sizes = ["스몰", "미디엄", "라지", "엑스라지"]
        if size not in valid_sizes:
            raise ValueError(f"유효하지 않은 크기: {size}. 가능한 값: {valid_sizes}")
        self._size = size
        return self  # 메서드 체이닝을 위해 self 반환

    def crust(self, crust: str) -> 'PizzaBuilder':
        self._crust = crust
        return self

    def sauce(self, sauce: str) -> 'PizzaBuilder':
        self._sauce = sauce
        return self

    def cheese(self, cheese: str) -> 'PizzaBuilder':
        self._cheese = cheese
        return self

    def add_topping(self, topping: str) -> 'PizzaBuilder':
        if len(self._toppings) >= 10:
            raise ValueError("토핑은 최대 10개까지 추가할 수 있습니다.")
        self._toppings.append(topping)
        return self

    def extra_cheese(self, extra: bool = True) -> 'PizzaBuilder':
        self._extra_cheese = extra
        return self

    def notes(self, notes: str) -> 'PizzaBuilder':
        self._notes = notes
        return self

    def build(self) -> Pizza:
        """최종 피자 객체 생성 (유효성 검증 포함)"""
        if not self._size:
            raise ValueError("피자 크기는 필수입니다.")
        if not self._crust:
            raise ValueError("크러스트 타입은 필수입니다.")

        return Pizza(
            size=self._size,
            crust=self._crust,
            sauce=self._sauce,
            cheese=self._cheese,
            toppings=tuple(self._toppings),
            extra_cheese=self._extra_cheese,
            notes=self._notes,
        )


# ─── 사용 예시 ───
if __name__ == "__main__":
    # 마르게리타 피자
    margherita = (PizzaBuilder()
        .size("라지")
        .crust("씬")
        .sauce("토마토")
        .cheese("모짜렐라")
        .add_topping("바질")
        .add_topping("올리브오일")
        .build())

    print(f"마르게리타: {margherita}")

    # 페퍼로니 피자
    pepperoni = (PizzaBuilder()
        .size("미디엄")
        .add_topping("페퍼로니")
        .add_topping("피망")
        .extra_cheese()
        .notes("페퍼로니 많이 주세요")
        .build())

    print(f"페퍼로니: {pepperoni}")

    # 기본 피자 (최소 설정)
    basic = PizzaBuilder().build()
    print(f"기본: {basic}")
```

### 방식 2: Fluent Builder with Validation

```python
from __future__ import annotations
from dataclasses import dataclass
from typing import Optional
from datetime import datetime, date


@dataclass(frozen=True)
class User:
    """사용자 - 불변 객체"""
    name: str
    email: str
    age: Optional[int] = None
    phone: Optional[str] = None
    address: Optional[str] = None
    company: Optional[str] = None
    role: str = "member"
    is_active: bool = True
    preferred_language: str = "ko"
    created_at: datetime = None

    def __post_init__(self):
        # frozen=True이므로 object.__setattr__ 사용
        if self.created_at is None:
            object.__setattr__(self, 'created_at', datetime.now())


class UserBuilder:
    """사용자 빌더 - 유효성 검증 포함"""

    def __init__(self, name: str, email: str):
        """필수 매개변수는 생성자에서 받는다"""
        if not name or not name.strip():
            raise ValueError("이름은 필수입니다.")
        if "@" not in email:
            raise ValueError("유효한 이메일 주소를 입력하세요.")

        self._name = name
        self._email = email
        self._age: Optional[int] = None
        self._phone: Optional[str] = None
        self._address: Optional[str] = None
        self._company: Optional[str] = None
        self._role: str = "member"
        self._is_active: bool = True
        self._preferred_language: str = "ko"

    def age(self, age: int) -> UserBuilder:
        if age < 0 or age > 150:
            raise ValueError(f"유효하지 않은 나이: {age}")
        self._age = age
        return self

    def phone(self, phone: str) -> UserBuilder:
        self._phone = phone
        return self

    def address(self, address: str) -> UserBuilder:
        self._address = address
        return self

    def company(self, company: str) -> UserBuilder:
        self._company = company
        return self

    def role(self, role: str) -> UserBuilder:
        valid_roles = ["member", "admin", "moderator", "editor"]
        if role not in valid_roles:
            raise ValueError(f"유효하지 않은 역할: {role}. 가능: {valid_roles}")
        self._role = role
        return self

    def active(self, is_active: bool = True) -> UserBuilder:
        self._is_active = is_active
        return self

    def language(self, lang: str) -> UserBuilder:
        self._preferred_language = lang
        return self

    def build(self) -> User:
        """빌드 시 최종 유효성 검증"""
        if self._role == "admin" and not self._phone:
            raise ValueError("관리자는 전화번호가 필수입니다.")

        return User(
            name=self._name,
            email=self._email,
            age=self._age,
            phone=self._phone,
            address=self._address,
            company=self._company,
            role=self._role,
            is_active=self._is_active,
            preferred_language=self._preferred_language,
        )


# ─── 사용 예시 ───
# 일반 회원
member = (UserBuilder("김철수", "kim@example.com")
    .age(30)
    .address("서울시 강남구")
    .build())

print(f"회원: {member.name}, {member.role}")

# 관리자
admin = (UserBuilder("이관리", "admin@example.com")
    .phone("01012345678")
    .role("admin")
    .company("ABC Corp")
    .build())

print(f"관리자: {admin.name}, {admin.role}")
```

---

## 6. C# 구현

### Fluent Interface with Method Chaining

```csharp
using System;
using System.Collections.Generic;

/// <summary>
/// 피자 - 불변 객체
/// </summary>
public class Pizza
{
    public string Size { get; }
    public string Crust { get; }
    public string Sauce { get; }
    public string Cheese { get; }
    public IReadOnlyList<string> Toppings { get; }
    public bool ExtraCheese { get; }
    public string Notes { get; }

    // 내부 생성자 - Builder만 호출 가능
    internal Pizza(string size, string crust, string sauce,
                   string cheese, List<string> toppings,
                   bool extraCheese, string notes)
    {
        Size = size;
        Crust = crust;
        Sauce = sauce;
        Cheese = cheese;
        Toppings = toppings.AsReadOnly();
        ExtraCheese = extraCheese;
        Notes = notes;
    }

    public override string ToString()
    {
        var toppingStr = Toppings.Count > 0
            ? string.Join(", ", Toppings)
            : "없음";
        return $"[{Size}] {Crust} 크러스트, {Sauce} 소스, " +
               $"{Cheese} 치즈, 토핑: {toppingStr}" +
               (ExtraCheese ? " (+치즈추가)" : "");
    }
}

/// <summary>
/// 피자 빌더 - Fluent Interface
/// </summary>
public class PizzaBuilder
{
    private string _size = "미디엄";
    private string _crust = "오리지널";
    private string _sauce = "토마토";
    private string _cheese = "모짜렐라";
    private readonly List<string> _toppings = new();
    private bool _extraCheese = false;
    private string _notes = "";

    public PizzaBuilder Size(string size)
    {
        var validSizes = new[] { "스몰", "미디엄", "라지", "엑스라지" };
        if (Array.IndexOf(validSizes, size) < 0)
            throw new ArgumentException($"유효하지 않은 크기: {size}");
        _size = size;
        return this;
    }

    public PizzaBuilder Crust(string crust)
    {
        _crust = crust;
        return this;
    }

    public PizzaBuilder Sauce(string sauce)
    {
        _sauce = sauce;
        return this;
    }

    public PizzaBuilder Cheese(string cheese)
    {
        _cheese = cheese;
        return this;
    }

    public PizzaBuilder AddTopping(string topping)
    {
        if (_toppings.Count >= 10)
            throw new InvalidOperationException("토핑은 최대 10개까지");
        _toppings.Add(topping);
        return this;
    }

    public PizzaBuilder WithExtraCheese(bool extra = true)
    {
        _extraCheese = extra;
        return this;
    }

    public PizzaBuilder Notes(string notes)
    {
        _notes = notes;
        return this;
    }

    public Pizza Build()
    {
        // 유효성 검증
        if (string.IsNullOrWhiteSpace(_size))
            throw new InvalidOperationException("크기는 필수입니다.");

        return new Pizza(_size, _crust, _sauce, _cheese,
                        new List<string>(_toppings), _extraCheese, _notes);
    }
}

// ─── 사용 예시 ───
var margherita = new PizzaBuilder()
    .Size("라지")
    .Crust("씬")
    .Sauce("토마토")
    .AddTopping("바질")
    .AddTopping("올리브오일")
    .Build();

Console.WriteLine($"마르게리타: {margherita}");

var pepperoni = new PizzaBuilder()
    .Size("미디엄")
    .AddTopping("페퍼로니")
    .AddTopping("피망")
    .WithExtraCheese()
    .Notes("페퍼로니 많이 주세요")
    .Build();

Console.WriteLine($"페퍼로니: {pepperoni}");
```

### C# Record와 Builder 패턴 결합

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Immutable;

/// <summary>
/// C# record를 활용한 불변 사용자 객체
/// </summary>
public record User
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public int? Age { get; init; }
    public string? Phone { get; init; }
    public string? Address { get; init; }
    public string? Company { get; init; }
    public string Role { get; init; } = "member";
    public bool IsActive { get; init; } = true;
    public string PreferredLanguage { get; init; } = "ko";
    public DateTime CreatedAt { get; init; } = DateTime.Now;
}

/// <summary>
/// 사용자 빌더 - 유효성 검증 포함
/// </summary>
public class UserBuilder
{
    private readonly string _name;
    private readonly string _email;
    private int? _age;
    private string? _phone;
    private string? _address;
    private string? _company;
    private string _role = "member";
    private bool _isActive = true;
    private string _language = "ko";

    /// <summary>필수 매개변수는 생성자에서 받는다</summary>
    public UserBuilder(string name, string email)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("이름은 필수입니다.");
        if (!email.Contains('@'))
            throw new ArgumentException("유효한 이메일을 입력하세요.");

        _name = name;
        _email = email;
    }

    public UserBuilder Age(int age)
    {
        if (age < 0 || age > 150)
            throw new ArgumentOutOfRangeException(nameof(age));
        _age = age;
        return this;
    }

    public UserBuilder Phone(string phone)
    {
        _phone = phone;
        return this;
    }

    public UserBuilder Address(string address)
    {
        _address = address;
        return this;
    }

    public UserBuilder Company(string company)
    {
        _company = company;
        return this;
    }

    public UserBuilder Role(string role)
    {
        var validRoles = new[] { "member", "admin", "moderator", "editor" };
        if (Array.IndexOf(validRoles, role) < 0)
            throw new ArgumentException($"유효하지 않은 역할: {role}");
        _role = role;
        return this;
    }

    public UserBuilder Active(bool isActive = true)
    {
        _isActive = isActive;
        return this;
    }

    public UserBuilder Language(string language)
    {
        _language = language;
        return this;
    }

    public User Build()
    {
        // 빌드 시 교차 검증
        if (_role == "admin" && string.IsNullOrWhiteSpace(_phone))
            throw new InvalidOperationException("관리자는 전화번호가 필수입니다.");

        return new User
        {
            Name = _name,
            Email = _email,
            Age = _age,
            Phone = _phone,
            Address = _address,
            Company = _company,
            Role = _role,
            IsActive = _isActive,
            PreferredLanguage = _language
        };
    }
}

// ─── 사용 예시 ───
var member = new UserBuilder("김철수", "kim@example.com")
    .Age(30)
    .Address("서울시 강남구")
    .Build();

var admin = new UserBuilder("이관리", "admin@example.com")
    .Phone("01012345678")
    .Role("admin")
    .Company("ABC Corp")
    .Build();
```

---

## 7. Director 역할

Director는 Builder의 메서드 호출 **순서를 관리**하는 클래스이다. 자주 사용되는 조합을 미리 정의해 놓을 수 있다.

### Python Director

```python
class PizzaDirector:
    """피자 Director: 자주 주문되는 피자 레시피를 관리"""

    def __init__(self, builder: PizzaBuilder):
        self._builder = builder

    def make_margherita(self) -> Pizza:
        """마르게리타 피자 레시피"""
        return (self._builder
            .size("미디엄")
            .crust("씬")
            .sauce("토마토")
            .cheese("모짜렐라")
            .add_topping("바질")
            .add_topping("토마토 슬라이스")
            .build())

    def make_pepperoni(self) -> Pizza:
        """페퍼로니 피자 레시피"""
        return (self._builder
            .size("라지")
            .crust("오리지널")
            .sauce("토마토")
            .cheese("모짜렐라")
            .add_topping("페퍼로니")
            .extra_cheese()
            .build())

    def make_hawaiian(self) -> Pizza:
        """하와이안 피자 레시피"""
        return (self._builder
            .size("라지")
            .crust("오리지널")
            .sauce("토마토")
            .cheese("모짜렐라")
            .add_topping("햄")
            .add_topping("파인애플")
            .build())


# 사용
director = PizzaDirector(PizzaBuilder())
margherita = director.make_margherita()
pepperoni = director.make_pepperoni()
```

### C# Director

```csharp
/// <summary>
/// Director: 자주 사용되는 빌드 레시피 관리
/// </summary>
public class PizzaDirector
{
    public Pizza MakeMargherita()
    {
        return new PizzaBuilder()
            .Size("미디엄")
            .Crust("씬")
            .Sauce("토마토")
            .AddTopping("바질")
            .AddTopping("토마토 슬라이스")
            .Build();
    }

    public Pizza MakePepperoni()
    {
        return new PizzaBuilder()
            .Size("라지")
            .AddTopping("페퍼로니")
            .WithExtraCheese()
            .Build();
    }

    public Pizza MakeHawaiian()
    {
        return new PizzaBuilder()
            .Size("라지")
            .AddTopping("햄")
            .AddTopping("파인애플")
            .Build();
    }
}

// 사용
var director = new PizzaDirector();
var margherita = director.MakeMargherita();
var pepperoni = director.MakePepperoni();
```

### Director가 필요한 경우 vs 불필요한 경우

```
Director가 유용한 경우:
  - 동일한 빌드 조합이 반복적으로 사용될 때
  - 빌드 순서가 중요하고 복잡할 때
  - 미리 정의된 "프리셋"이 필요할 때

Director가 불필요한 경우:
  - 매번 다른 조합으로 빌드하는 경우
  - 클라이언트가 직접 빌더를 제어해야 하는 경우
  - Fluent Builder로 충분히 직관적인 경우
```

---

## 8. 실무 예제: HTTP Request Builder

### Python 구현

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Optional, Any
from enum import Enum
import json


class HttpMethod(Enum):
    GET = "GET"
    POST = "POST"
    PUT = "PUT"
    PATCH = "PATCH"
    DELETE = "DELETE"


@dataclass(frozen=True)
class HttpRequest:
    """HTTP 요청 - 불변 객체"""
    method: HttpMethod
    url: str
    headers: dict[str, str]
    query_params: dict[str, str]
    body: Optional[str]
    timeout: int  # 초
    retry_count: int
    follow_redirects: bool

    def __str__(self) -> str:
        lines = [
            f"{self.method.value} {self.url}",
            f"Headers: {json.dumps(self.headers, indent=2, ensure_ascii=False)}",
        ]
        if self.query_params:
            lines.append(f"Query: {self.query_params}")
        if self.body:
            lines.append(f"Body: {self.body[:100]}...")
        lines.append(f"Timeout: {self.timeout}s, Retries: {self.retry_count}")
        return "\n".join(lines)


class HttpRequestBuilder:
    """HTTP 요청 빌더

    실무에서 HTTP 클라이언트 라이브러리에서 자주 볼 수 있는 패턴이다.
    requests, httpx 등의 라이브러리가 이와 유사한 인터페이스를 제공한다.
    """

    def __init__(self, method: HttpMethod, url: str):
        """필수 매개변수: HTTP 메서드와 URL"""
        if not url:
            raise ValueError("URL은 필수입니다.")
        self._method = method
        self._url = url
        self._headers: dict[str, str] = {}
        self._query_params: dict[str, str] = {}
        self._body: Optional[str] = None
        self._timeout: int = 30
        self._retry_count: int = 0
        self._follow_redirects: bool = True

    # ─── 헤더 관련 ───

    def header(self, key: str, value: str) -> HttpRequestBuilder:
        """헤더 추가"""
        self._headers[key] = value
        return self

    def content_type(self, content_type: str) -> HttpRequestBuilder:
        """Content-Type 헤더 설정"""
        self._headers["Content-Type"] = content_type
        return self

    def accept(self, accept: str) -> HttpRequestBuilder:
        """Accept 헤더 설정"""
        self._headers["Accept"] = accept
        return self

    def authorization(self, token: str) -> HttpRequestBuilder:
        """Authorization 헤더 설정"""
        self._headers["Authorization"] = f"Bearer {token}"
        return self

    def user_agent(self, agent: str) -> HttpRequestBuilder:
        """User-Agent 헤더 설정"""
        self._headers["User-Agent"] = agent
        return self

    # ─── 쿼리 파라미터 ───

    def query(self, key: str, value: str) -> HttpRequestBuilder:
        """쿼리 파라미터 추가"""
        self._query_params[key] = value
        return self

    # ─── 바디 ───

    def json_body(self, data: dict) -> HttpRequestBuilder:
        """JSON 바디 설정"""
        self._body = json.dumps(data, ensure_ascii=False)
        self._headers["Content-Type"] = "application/json"
        return self

    def form_body(self, data: dict) -> HttpRequestBuilder:
        """Form 바디 설정"""
        from urllib.parse import urlencode
        self._body = urlencode(data)
        self._headers["Content-Type"] = "application/x-www-form-urlencoded"
        return self

    def raw_body(self, body: str) -> HttpRequestBuilder:
        """Raw 바디 설정"""
        self._body = body
        return self

    # ─── 옵션 ───

    def timeout(self, seconds: int) -> HttpRequestBuilder:
        """타임아웃 설정"""
        if seconds <= 0:
            raise ValueError("타임아웃은 양수여야 합니다.")
        self._timeout = seconds
        return self

    def retry(self, count: int) -> HttpRequestBuilder:
        """재시도 횟수 설정"""
        if count < 0:
            raise ValueError("재시도 횟수는 0 이상이어야 합니다.")
        self._retry_count = count
        return self

    def no_redirect(self) -> HttpRequestBuilder:
        """리다이렉트 비활성화"""
        self._follow_redirects = False
        return self

    # ─── 빌드 ───

    def build(self) -> HttpRequest:
        """최종 HttpRequest 객체 생성"""
        # GET 요청에 body가 있으면 경고
        if self._method == HttpMethod.GET and self._body:
            print("[경고] GET 요청에 body가 포함되어 있습니다.")

        return HttpRequest(
            method=self._method,
            url=self._url,
            headers=dict(self._headers),
            query_params=dict(self._query_params),
            body=self._body,
            timeout=self._timeout,
            retry_count=self._retry_count,
            follow_redirects=self._follow_redirects,
        )


# ─── 편의 함수 ───
def get(url: str) -> HttpRequestBuilder:
    return HttpRequestBuilder(HttpMethod.GET, url)

def post(url: str) -> HttpRequestBuilder:
    return HttpRequestBuilder(HttpMethod.POST, url)

def put(url: str) -> HttpRequestBuilder:
    return HttpRequestBuilder(HttpMethod.PUT, url)

def delete(url: str) -> HttpRequestBuilder:
    return HttpRequestBuilder(HttpMethod.DELETE, url)


# ─── 사용 예시 ───
if __name__ == "__main__":
    # GET 요청
    request1 = (get("https://api.example.com/users")
        .authorization("my-secret-token")
        .query("page", "1")
        .query("limit", "20")
        .timeout(10)
        .build())
    print("=== GET 요청 ===")
    print(request1)

    # POST 요청
    request2 = (post("https://api.example.com/users")
        .authorization("my-secret-token")
        .json_body({
            "name": "김철수",
            "email": "kim@example.com",
            "role": "developer",
        })
        .timeout(30)
        .retry(3)
        .build())
    print("\n=== POST 요청 ===")
    print(request2)

    # DELETE 요청
    request3 = (delete("https://api.example.com/users/123")
        .authorization("admin-token")
        .timeout(5)
        .no_redirect()
        .build())
    print("\n=== DELETE 요청 ===")
    print(request3)
```

### C# 구현

```csharp
using System;
using System.Collections.Generic;
using System.Text.Json;

public enum HttpMethod { GET, POST, PUT, PATCH, DELETE }

/// <summary>
/// HTTP 요청 - 불변 객체
/// </summary>
public class HttpRequest
{
    public HttpMethod Method { get; }
    public string Url { get; }
    public IReadOnlyDictionary<string, string> Headers { get; }
    public IReadOnlyDictionary<string, string> QueryParams { get; }
    public string? Body { get; }
    public int TimeoutSeconds { get; }
    public int RetryCount { get; }
    public bool FollowRedirects { get; }

    internal HttpRequest(HttpMethod method, string url,
        Dictionary<string, string> headers,
        Dictionary<string, string> queryParams,
        string? body, int timeout, int retryCount, bool followRedirects)
    {
        Method = method;
        Url = url;
        Headers = headers;
        QueryParams = queryParams;
        Body = body;
        TimeoutSeconds = timeout;
        RetryCount = retryCount;
        FollowRedirects = followRedirects;
    }

    public override string ToString()
    {
        var lines = new List<string>
        {
            $"{Method} {Url}",
            $"Headers: {JsonSerializer.Serialize(Headers)}",
        };
        if (QueryParams.Count > 0)
            lines.Add($"Query: {JsonSerializer.Serialize(QueryParams)}");
        if (!string.IsNullOrEmpty(Body))
            lines.Add($"Body: {Body[..Math.Min(100, Body.Length)]}");
        lines.Add($"Timeout: {TimeoutSeconds}s, Retries: {RetryCount}");
        return string.Join(Environment.NewLine, lines);
    }
}

/// <summary>
/// HTTP 요청 빌더 - Fluent Interface
/// </summary>
public class HttpRequestBuilder
{
    private readonly HttpMethod _method;
    private readonly string _url;
    private readonly Dictionary<string, string> _headers = new();
    private readonly Dictionary<string, string> _queryParams = new();
    private string? _body;
    private int _timeout = 30;
    private int _retryCount = 0;
    private bool _followRedirects = true;

    public HttpRequestBuilder(HttpMethod method, string url)
    {
        if (string.IsNullOrWhiteSpace(url))
            throw new ArgumentException("URL은 필수입니다.");
        _method = method;
        _url = url;
    }

    public HttpRequestBuilder Header(string key, string value)
    {
        _headers[key] = value;
        return this;
    }

    public HttpRequestBuilder ContentType(string contentType)
        => Header("Content-Type", contentType);

    public HttpRequestBuilder Authorization(string token)
        => Header("Authorization", $"Bearer {token}");

    public HttpRequestBuilder Query(string key, string value)
    {
        _queryParams[key] = value;
        return this;
    }

    public HttpRequestBuilder JsonBody(object data)
    {
        _body = JsonSerializer.Serialize(data);
        _headers["Content-Type"] = "application/json";
        return this;
    }

    public HttpRequestBuilder RawBody(string body)
    {
        _body = body;
        return this;
    }

    public HttpRequestBuilder Timeout(int seconds)
    {
        if (seconds <= 0)
            throw new ArgumentException("타임아웃은 양수여야 합니다.");
        _timeout = seconds;
        return this;
    }

    public HttpRequestBuilder Retry(int count)
    {
        if (count < 0)
            throw new ArgumentException("재시도 횟수는 0 이상이어야 합니다.");
        _retryCount = count;
        return this;
    }

    public HttpRequestBuilder NoRedirect()
    {
        _followRedirects = false;
        return this;
    }

    public HttpRequest Build()
    {
        return new HttpRequest(_method, _url,
            new Dictionary<string, string>(_headers),
            new Dictionary<string, string>(_queryParams),
            _body, _timeout, _retryCount, _followRedirects);
    }

    // ── 정적 팩토리 메서드 ──
    public static HttpRequestBuilder Get(string url)
        => new(HttpMethod.GET, url);

    public static HttpRequestBuilder Post(string url)
        => new(HttpMethod.POST, url);

    public static HttpRequestBuilder Put(string url)
        => new(HttpMethod.PUT, url);

    public static HttpRequestBuilder Delete(string url)
        => new(HttpMethod.DELETE, url);
}

// ─── 사용 예시 ───
var getRequest = HttpRequestBuilder.Get("https://api.example.com/users")
    .Authorization("my-token")
    .Query("page", "1")
    .Query("limit", "20")
    .Timeout(10)
    .Build();

Console.WriteLine("=== GET ===");
Console.WriteLine(getRequest);

var postRequest = HttpRequestBuilder.Post("https://api.example.com/users")
    .Authorization("my-token")
    .JsonBody(new { name = "김철수", email = "kim@example.com" })
    .Timeout(30)
    .Retry(3)
    .Build();

Console.WriteLine("\n=== POST ===");
Console.WriteLine(postRequest);
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **가독성** | 메서드 이름으로 각 매개변수의 의미가 명확 |
| **유연한 생성** | 필요한 단계만 호출하여 다양한 구성 가능 |
| **유효성 검증** | build() 시점에 교차 검증 가능 |
| **불변 객체** | 빌더로 만든 후 변경 불가능한 객체 생성 가능 |
| **자기 문서화** | 코드 자체가 설명이 됨 (self-documenting) |
| **단계적 구성** | 복잡한 객체를 단계별로 조립 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **코드량 증가** | 빌더 클래스를 별도로 작성해야 함 |
| **중복 필드** | Product와 Builder에 같은 필드가 존재 |
| **간단한 객체에 과도** | 필드가 2~3개인 경우에는 불필요 |

### 판단 기준

```
Builder를 적용해야 할 때:
  - 생성자 매개변수가 4개 이상인 경우
  - 매개변수 대부분이 선택적(optional)인 경우
  - 불변 객체를 유연하게 생성해야 하는 경우
  - 생성 시 유효성 검증이 복잡한 경우

Builder가 불필요한 경우:
  - 매개변수가 2~3개이고 모두 필수인 경우
  - Python의 경우 키워드 인자로 충분한 경우
  - 객체가 단순하고 변경이 잦은 경우
```

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Abstract Factory** | 복잡한 제품을 만들 때 Abstract Factory + Builder를 결합 |
| **Factory Method** | Builder의 buildPart() 메서드가 Factory Method일 수 있음 |
| **Prototype** | Builder의 대안. 기존 객체를 복제 후 수정 |
| **Composite** | Composite 구조를 Builder로 단계적으로 생성 가능 |
| **Fluent Interface** | Builder 패턴의 일반적인 구현 방식 (GoF에는 없는 패턴) |

---

## 11. 정리 및 체크리스트

### 핵심 정리

```
Builder 패턴 = 복잡한 객체를 단계별로 조립

[핵심 구조]
- Builder: 빌드 단계를 정의하는 인터페이스
- ConcreteBuilder: 각 단계의 구체 구현
- Director: 빌드 순서를 관리 (선택적)
- Product: 최종 생성되는 복잡한 객체

[Telescoping Constructor 문제 해결]
- 매개변수가 많은 생성자 → 단계별 설정 메서드
- 순서 혼동 → 이름 있는 메서드로 명확화
- 선택적 매개변수 → 필요한 것만 호출

[Fluent Interface]
- 각 설정 메서드가 self/this를 반환
- 메서드 체이닝으로 가독성 향상
- .build()로 최종 객체 생성

[실무 활용]
- HTTP Request/Response Builder
- SQL Query Builder
- Configuration Builder
- UI Component Builder
```

### 학습 체크리스트

- [ ] Builder 패턴의 의도를 한 문장으로 설명할 수 있다
- [ ] Telescoping Constructor 안티패턴의 문제점을 설명할 수 있다
- [ ] Fluent Builder를 Python과 C#으로 구현할 수 있다
- [ ] Director의 역할과 필요성을 설명할 수 있다
- [ ] 빌더에서 유효성 검증을 구현할 수 있다
- [ ] 불변 객체와 Builder 패턴의 관계를 이해한다
- [ ] Builder를 적용해야 할 때와 불필요한 때를 구분할 수 있다
- [ ] 실무에서 Builder 패턴의 활용 사례를 3개 이상 열거할 수 있다

### 다음 학습

Builder가 "복잡한 객체를 처음부터 단계별로 만드는" 패턴이라면, 다음으로 배울 [Prototype 패턴](./05-prototype.md)은 "기존 객체를 복제하여 새 객체를 만드는" 패턴이다.
