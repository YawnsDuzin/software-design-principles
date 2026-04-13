# OOP 기초 - 객체지향 프로그래밍의 4대 원칙

> **대상 독자**: 2~3년차 개발자
> **목표**: 객체지향 프로그래밍의 4대 원칙을 정확히 이해하고, 실무 코드에서 올바르게 적용할 수 있다.

---

## 목차

1. [개요](#1-개요)
2. [캡슐화 (Encapsulation)](#2-캡슐화-encapsulation)
3. [상속 (Inheritance)](#3-상속-inheritance)
4. [다형성 (Polymorphism)](#4-다형성-polymorphism)
5. [추상화 (Abstraction)](#5-추상화-abstraction)
6. [종합 실습: 동물원 관리 시스템](#6-종합-실습-동물원-관리-시스템)
7. [정리 및 체크리스트](#7-정리-및-체크리스트)
8. [다음 단계](#8-다음-단계)

---

## 1. 개요

객체지향 프로그래밍(OOP)은 현실 세계의 사물을 **객체(Object)** 로 모델링하여 소프트웨어를 설계하는 패러다임입니다. OOP의 4대 원칙은 다음과 같습니다.

| 원칙 | 핵심 키워드 | 비유 |
|------|------------|------|
| 캡슐화 | 정보 은닉, 접근 제어 | 자동차 운전자는 엔진 내부를 몰라도 운전할 수 있다 |
| 상속 | 코드 재사용, 계층 구조 | 자녀가 부모의 특성을 물려받되, 자신만의 개성이 있다 |
| 다형성 | 하나의 인터페이스, 여러 구현 | 같은 "말해!" 명령에 개는 짖고, 고양이는 야옹한다 |
| 추상화 | 복잡성 숨기기, 핵심만 노출 | 리모컨의 버튼만 알면 TV의 회로를 몰라도 된다 |

이 4가지 원칙은 서로 독립적이지 않고, **서로 보완하며 함께 동작**합니다.

---

## 2. 캡슐화 (Encapsulation)

### 핵심 개념

캡슐화란 **데이터(속성)와 그 데이터를 조작하는 메서드를 하나의 단위(클래스)로 묶고**, 외부에서 내부 구현 세부사항에 직접 접근하지 못하도록 제한하는 것입니다.

**비유**: 약국의 캡슐 알약을 생각해 보세요. 약의 성분(데이터)은 캡슐 안에 감싸져 있고, 환자(외부 코드)는 캡슐을 통째로 복용할 뿐 내부 성분을 직접 만지지 않습니다.

### 접근 제어자

| 접근 제어자 | Python | C# | 설명 |
|------------|--------|-----|------|
| Public | `name` | `public` | 어디서든 접근 가능 |
| Protected | `_name` | `protected` | 자신과 하위 클래스만 접근 |
| Private | `__name` | `private` | 자신만 접근 가능 |

### Python 예제

```python
class BankAccount:
    """은행 계좌 - 캡슐화 예제"""

    def __init__(self, owner: str, initial_balance: float = 0):
        self.owner = owner              # public: 누구나 접근 가능
        self._account_type = "일반"      # protected: 하위 클래스에서 접근 가능
        self.__balance = initial_balance  # private: 외부에서 직접 접근 불가

    @property
    def balance(self) -> float:
        """잔액 조회 (getter) - 읽기 전용 속성"""
        return self.__balance

    def deposit(self, amount: float) -> None:
        """입금 - 유효성 검사를 거쳐 안전하게 데이터 변경"""
        if amount <= 0:
            raise ValueError("입금액은 0보다 커야 합니다.")
        self.__balance += amount
        print(f"[입금] {amount:,.0f}원 → 잔액: {self.__balance:,.0f}원")

    def withdraw(self, amount: float) -> None:
        """출금 - 비즈니스 로직(잔액 확인)을 캡슐 내부에서 처리"""
        if amount <= 0:
            raise ValueError("출금액은 0보다 커야 합니다.")
        if amount > self.__balance:
            raise ValueError("잔액이 부족합니다.")
        self.__balance -= amount
        print(f"[출금] {amount:,.0f}원 → 잔액: {self.__balance:,.0f}원")


# 사용 예시
account = BankAccount("홍길동", 100_000)
account.deposit(50_000)       # [입금] 50,000원 → 잔액: 150,000원
account.withdraw(30_000)      # [출금] 30,000원 → 잔액: 120,000원
print(account.balance)        # 120000 (property를 통해 안전하게 조회)

# account.__balance = 999_999  # AttributeError! 직접 접근 불가
# account.__balance             # AttributeError! 직접 읽기도 불가
```

### C# 예제

```csharp
public class BankAccount
{
    // public: 누구나 접근 가능
    public string Owner { get; }

    // protected: 하위 클래스에서 접근 가능
    protected string AccountType { get; set; } = "일반";

    // private: 외부에서 직접 접근 불가
    private decimal _balance;

    // Property를 통한 안전한 접근 (읽기 전용)
    public decimal Balance => _balance;

    public BankAccount(string owner, decimal initialBalance = 0)
    {
        Owner = owner;
        _balance = initialBalance;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("입금액은 0보다 커야 합니다.");

        _balance += amount;
        Console.WriteLine($"[입금] {amount:#,0}원 → 잔액: {_balance:#,0}원");
    }

    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("출금액은 0보다 커야 합니다.");
        if (amount > _balance)
            throw new InvalidOperationException("잔액이 부족합니다.");

        _balance -= amount;
        Console.WriteLine($"[출금] {amount:#,0}원 → 잔액: {_balance:#,0}원");
    }
}

// 사용 예시
var account = new BankAccount("홍길동", 100_000);
account.Deposit(50_000);       // [입금] 50,000원 → 잔액: 150,000원
account.Withdraw(30_000);      // [출금] 30,000원 → 잔액: 120,000원
Console.WriteLine(account.Balance);  // 120000

// account._balance = 999_999;  // 컴파일 에러! private 필드 접근 불가
```

### 캡슐화를 지키지 않으면?

```python
# ❌ 나쁜 예: 모든 필드가 public
class BadBankAccount:
    def __init__(self):
        self.balance = 0  # 누구나 직접 수정 가능!

bad = BadBankAccount()
bad.balance = -1_000_000  # 잔액이 음수?! 비즈니스 로직 완전 무시
```

---

## 3. 상속 (Inheritance)

### 핵심 개념

상속은 **기존 클래스(부모/베이스)의 속성과 메서드를 새로운 클래스(자식/파생)가 물려받는 메커니즘**입니다. 코드 재사용과 "is-a" 관계를 표현합니다.

**비유**: 스마트폰은 전화기를 상속받습니다. 전화기의 기본 기능(전화 걸기/받기)은 그대로 사용하면서, 앱 설치나 인터넷 같은 새 기능을 추가합니다.

### Python 예제

```python
class Vehicle:
    """탈것 - 기본 클래스"""

    def __init__(self, brand: str, model: str, year: int):
        self.brand = brand
        self.model = model
        self.year = year
        self._mileage = 0  # protected: 하위 클래스에서 접근 가능

    def drive(self, km: float) -> None:
        self._mileage += km
        print(f"{self.brand} {self.model}이(가) {km}km 주행했습니다.")

    def get_info(self) -> str:
        return f"{self.year}년식 {self.brand} {self.model} (주행거리: {self._mileage}km)"


class ElectricCar(Vehicle):
    """전기차 - Vehicle을 상속받아 확장"""

    def __init__(self, brand: str, model: str, year: int, battery_capacity: float):
        # super()로 부모 클래스의 __init__ 호출
        super().__init__(brand, model, year)
        self.battery_capacity = battery_capacity  # 새로운 속성 추가
        self.__charge_level = 100.0               # 전기차만의 private 속성

    def drive(self, km: float) -> None:
        """메서드 오버라이딩 - 부모의 drive를 재정의"""
        energy_consumed = km * 0.15  # kWh per km
        if energy_consumed > self.__charge_level:
            print("배터리가 부족합니다. 충전이 필요합니다.")
            return
        self.__charge_level -= energy_consumed
        super().drive(km)  # 부모의 drive도 호출 (주행거리 기록)
        print(f"  배터리 잔량: {self.__charge_level:.1f}%")

    def charge(self) -> None:
        """전기차만의 새로운 메서드"""
        self.__charge_level = 100.0
        print(f"{self.brand} {self.model} 충전 완료!")


# 사용 예시
car = Vehicle("현대", "소나타", 2024)
car.drive(50)
print(car.get_info())

ev = ElectricCar("테슬라", "모델 3", 2024, 75.0)
ev.drive(100)       # 오버라이딩된 drive 실행
ev.charge()         # 전기차만의 메서드
print(ev.get_info())  # 부모로부터 상속받은 메서드
```

### C# 예제

```csharp
public class Vehicle
{
    public string Brand { get; }
    public string Model { get; }
    public int Year { get; }
    protected double Mileage { get; set; }

    public Vehicle(string brand, string model, int year)
    {
        Brand = brand;
        Model = model;
        Year = year;
        Mileage = 0;
    }

    // virtual: 하위 클래스에서 재정의 가능
    public virtual void Drive(double km)
    {
        Mileage += km;
        Console.WriteLine($"{Brand} {Model}이(가) {km}km 주행했습니다.");
    }

    public string GetInfo()
        => $"{Year}년식 {Brand} {Model} (주행거리: {Mileage}km)";
}

public class ElectricCar : Vehicle
{
    public double BatteryCapacity { get; }
    private double _chargeLevel = 100.0;

    public ElectricCar(string brand, string model, int year, double batteryCapacity)
        : base(brand, model, year)  // base로 부모 생성자 호출
    {
        BatteryCapacity = batteryCapacity;
    }

    // override: 부모의 virtual 메서드를 재정의
    public override void Drive(double km)
    {
        double energyConsumed = km * 0.15;
        if (energyConsumed > _chargeLevel)
        {
            Console.WriteLine("배터리가 부족합니다. 충전이 필요합니다.");
            return;
        }
        _chargeLevel -= energyConsumed;
        base.Drive(km);  // 부모의 Drive도 호출
        Console.WriteLine($"  배터리 잔량: {_chargeLevel:F1}%");
    }

    public void Charge()
    {
        _chargeLevel = 100.0;
        Console.WriteLine($"{Brand} {Model} 충전 완료!");
    }
}
```

> **주의**: 상속은 강력하지만, "is-a" 관계가 아닌 곳에 남용하면 오히려 복잡성이 증가합니다. **"상속보다 합성(Composition over Inheritance)"** 을 항상 고려하세요.

---

## 4. 다형성 (Polymorphism)

### 핵심 개념

다형성은 **같은 이름의 메서드가 객체의 타입에 따라 다르게 동작하는 것**입니다. "하나의 인터페이스, 여러 구현"이라고도 합니다.

**비유**: 리모컨의 "재생" 버튼을 누르면, DVD 플레이어에서는 영화가 재생되고, CD 플레이어에서는 음악이 재생됩니다. 같은 동작(재생)이 대상(객체)에 따라 다르게 동작합니다.

다형성에는 크게 두 가지가 있습니다:

- **컴파일 타임 다형성 (정적)**: 메서드 오버로딩 (같은 이름, 다른 매개변수)
- **런타임 다형성 (동적)**: 메서드 오버라이딩 (상속 + 재정의)

### Python 예제

```python
from abc import ABC, abstractmethod


class Shape(ABC):
    """도형 - 다형성의 기반이 되는 추상 클래스"""

    @abstractmethod
    def area(self) -> float:
        """넓이 계산 - 각 도형마다 다르게 구현"""
        pass

    @abstractmethod
    def describe(self) -> str:
        """도형 설명"""
        pass


class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius ** 2

    def describe(self) -> str:
        return f"반지름 {self.radius}인 원"


class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

    def describe(self) -> str:
        return f"{self.width} x {self.height} 직사각형"


class Triangle(Shape):
    def __init__(self, base: float, height: float):
        self.base = base
        self.height = height

    def area(self) -> float:
        return 0.5 * self.base * self.height

    def describe(self) -> str:
        return f"밑변 {self.base}, 높이 {self.height}인 삼각형"


# 다형성의 핵심: 같은 타입(Shape)으로 다루지만, 각각 다르게 동작
def print_shape_info(shape: Shape) -> None:
    """어떤 도형이든 동일한 인터페이스로 처리"""
    print(f"{shape.describe()} → 넓이: {shape.area():.2f}")


# 사용 예시
shapes: list[Shape] = [
    Circle(5),
    Rectangle(4, 6),
    Triangle(3, 8),
]

for shape in shapes:
    print_shape_info(shape)  # 각 도형의 구현이 자동으로 호출됨!

# 출력:
# 반지름 5인 원 → 넓이: 78.54
# 4 x 6 직사각형 → 넓이: 24.00
# 밑변 3, 높이 8인 삼각형 → 넓이: 12.00
```

### C# 예제

```csharp
public abstract class Shape
{
    public abstract double Area();
    public abstract string Describe();
}

public class Circle : Shape
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public override double Area() => Math.PI * Radius * Radius;
    public override string Describe() => $"반지름 {Radius}인 원";
}

public class Rectangle : Shape
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public override double Area() => Width * Height;
    public override string Describe() => $"{Width} x {Height} 직사각형";
}

public class Triangle : Shape
{
    public double Base { get; }
    public double Height { get; }

    public Triangle(double @base, double height)
    {
        Base = @base;
        Height = height;
    }

    public override double Area() => 0.5 * Base * Height;
    public override string Describe() => $"밑변 {Base}, 높이 {Height}인 삼각형";
}

// 다형성 활용
Shape[] shapes = { new Circle(5), new Rectangle(4, 6), new Triangle(3, 8) };

foreach (var shape in shapes)
{
    Console.WriteLine($"{shape.Describe()} → 넓이: {shape.Area():F2}");
}
```

### 메서드 오버로딩 (C#)

C#에서는 같은 이름의 메서드를 매개변수를 달리하여 여러 개 정의할 수 있습니다.

```csharp
public class Calculator
{
    // 같은 이름, 다른 매개변수 → 오버로딩
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}
```

> **참고**: Python은 메서드 오버로딩을 기본적으로 지원하지 않습니다. 대신 기본값 인자, `*args`, `**kwargs`, 또는 `@singledispatch`를 활용합니다.

---

## 5. 추상화 (Abstraction)

### 핵심 개념

추상화는 **복잡한 내부 구현을 숨기고, 사용자에게 필요한 핵심 기능만 노출하는 것**입니다. 추상 클래스와 인터페이스가 대표적인 추상화 도구입니다.

**비유**: 커피 머신의 "에스프레소" 버튼을 누르면 커피가 나옵니다. 물의 온도, 압력, 추출 시간 등 복잡한 과정은 머신 내부에 감춰져 있습니다.

### 추상 클래스 vs 인터페이스

| 구분 | 추상 클래스 | 인터페이스 |
|------|-----------|-----------|
| 구현 코드 | 가질 수 있음 | 가질 수 없음 (C# 8.0 이전) |
| 상속 제한 | 단일 상속 | 다중 구현 가능 |
| 용도 | "is-a" + 공통 로직 공유 | "can-do" 능력 정의 |
| 예시 | `Animal` 추상 클래스 | `ISwimmable` 인터페이스 |

### Python 예제

```python
from abc import ABC, abstractmethod


# 추상 클래스: "is-a" 관계 + 공통 로직 제공
class PaymentProcessor(ABC):
    """결제 처리기 추상 클래스"""

    def process_payment(self, amount: float) -> bool:
        """템플릿 메서드 - 결제 흐름은 고정, 세부 구현은 하위 클래스에 위임"""
        if not self._validate(amount):
            print("결제 유효성 검사 실패")
            return False

        result = self._execute_payment(amount)
        self._log_transaction(amount, result)
        return result

    def _validate(self, amount: float) -> bool:
        """공통 유효성 검사 (하위 클래스에서 재사용)"""
        return amount > 0

    @abstractmethod
    def _execute_payment(self, amount: float) -> bool:
        """실제 결제 실행 - 반드시 하위 클래스에서 구현해야 함"""
        pass

    def _log_transaction(self, amount: float, success: bool) -> None:
        """공통 로깅 로직"""
        status = "성공" if success else "실패"
        print(f"[{self.__class__.__name__}] {amount:,.0f}원 결제 {status}")


class CreditCardProcessor(PaymentProcessor):
    def _execute_payment(self, amount: float) -> bool:
        print(f"신용카드로 {amount:,.0f}원 결제 처리 중...")
        return True  # 실제로는 카드사 API 호출


class KakaoPayProcessor(PaymentProcessor):
    def _execute_payment(self, amount: float) -> bool:
        print(f"카카오페이로 {amount:,.0f}원 결제 처리 중...")
        return True  # 실제로는 카카오페이 API 호출


# 사용 - 어떤 결제 수단이든 동일한 인터페이스로 처리
def checkout(processor: PaymentProcessor, amount: float) -> None:
    processor.process_payment(amount)

checkout(CreditCardProcessor(), 50_000)
checkout(KakaoPayProcessor(), 30_000)
```

### C# 예제

```csharp
// 추상 클래스
public abstract class PaymentProcessor
{
    public bool ProcessPayment(decimal amount)
    {
        if (!Validate(amount))
        {
            Console.WriteLine("결제 유효성 검사 실패");
            return false;
        }

        bool result = ExecutePayment(amount);
        LogTransaction(amount, result);
        return result;
    }

    protected virtual bool Validate(decimal amount) => amount > 0;

    // 하위 클래스에서 반드시 구현
    protected abstract bool ExecutePayment(decimal amount);

    private void LogTransaction(decimal amount, bool success)
    {
        string status = success ? "성공" : "실패";
        Console.WriteLine($"[{GetType().Name}] {amount:#,0}원 결제 {status}");
    }
}

// 인터페이스: 능력(can-do)을 정의
public interface IRefundable
{
    bool Refund(decimal amount);
}

public class CreditCardProcessor : PaymentProcessor, IRefundable
{
    protected override bool ExecutePayment(decimal amount)
    {
        Console.WriteLine($"신용카드로 {amount:#,0}원 결제 처리 중...");
        return true;
    }

    public bool Refund(decimal amount)
    {
        Console.WriteLine($"신용카드 {amount:#,0}원 환불 처리");
        return true;
    }
}

public class KakaoPayProcessor : PaymentProcessor
{
    protected override bool ExecutePayment(decimal amount)
    {
        Console.WriteLine($"카카오페이로 {amount:#,0}원 결제 처리 중...");
        return true;
    }
}
```

---

## 6. 종합 실습: 동물원 관리 시스템

4대 원칙을 모두 적용한 실전 예제입니다.

### Python 종합 예제

```python
from abc import ABC, abstractmethod
from datetime import datetime


# ──────────────────────────────────────────────
# 추상화 (Abstraction): 추상 클래스와 인터페이스 정의
# ──────────────────────────────────────────────
class Animal(ABC):
    """동물 추상 클래스 - 모든 동물의 공통 특성 정의"""

    def __init__(self, name: str, species: str, age: int):
        # 캡슐화 (Encapsulation): private/protected 필드
        self.__name = name
        self.__species = species
        self._age = age
        self._health_status = "양호"
        self.__feeding_log: list[str] = []

    # 캡슐화: Property를 통한 안전한 접근
    @property
    def name(self) -> str:
        return self.__name

    @property
    def species(self) -> str:
        return self.__species

    @property
    def health_status(self) -> str:
        return self._health_status

    # 추상화: 하위 클래스에서 반드시 구현해야 할 메서드
    @abstractmethod
    def make_sound(self) -> str:
        """울음소리 - 동물마다 다름 (다형성의 기반)"""
        pass

    @abstractmethod
    def get_diet(self) -> str:
        """식단 정보"""
        pass

    # 공통 구현: 캡슐화된 먹이 기록
    def feed(self, food: str) -> None:
        log_entry = f"[{datetime.now():%H:%M}] {self.__name}에게 {food} 급여"
        self.__feeding_log.append(log_entry)
        print(log_entry)

    def get_feeding_history(self) -> list[str]:
        return self.__feeding_log.copy()  # 방어적 복사

    def __str__(self) -> str:
        return f"{self.__species} '{self.__name}' (나이: {self._age}살, 상태: {self._health_status})"


# 다형성을 위한 행동 인터페이스 (믹스인)
class Swimmable(ABC):
    @abstractmethod
    def swim(self) -> str:
        pass


class Flyable(ABC):
    @abstractmethod
    def fly(self) -> str:
        pass


class Trainable(ABC):
    @abstractmethod
    def perform_trick(self, trick: str) -> str:
        pass


# ──────────────────────────────────────────────
# 상속 (Inheritance) + 다형성 (Polymorphism)
# ──────────────────────────────────────────────
class Lion(Animal):
    """사자 - Animal을 상속"""

    def __init__(self, name: str, age: int, mane_size: str = "보통"):
        super().__init__(name, "사자", age)
        self.mane_size = mane_size

    def make_sound(self) -> str:
        return "으르렁~! 🦁"

    def get_diet(self) -> str:
        return "육식 (소고기, 닭고기)"


class Penguin(Animal, Swimmable):
    """펭귄 - Animal 상속 + Swimmable 구현 (다중 상속)"""

    def __init__(self, name: str, age: int, species_type: str = "황제펭귄"):
        super().__init__(name, f"펭귄({species_type})", age)

    def make_sound(self) -> str:
        return "꽥꽥~!"

    def get_diet(self) -> str:
        return "생선 (멸치, 청어)"

    def swim(self) -> str:
        return f"{self.name}이(가) 물속에서 날듯이 헤엄칩니다!"


class Parrot(Animal, Flyable, Trainable):
    """앵무새 - Animal 상속 + Flyable, Trainable 구현"""

    def __init__(self, name: str, age: int, vocabulary_size: int = 0):
        super().__init__(name, "앵무새", age)
        self.__vocabulary_size = vocabulary_size

    def make_sound(self) -> str:
        return "안녕하세요~! 🦜"

    def get_diet(self) -> str:
        return "과일, 씨앗, 견과류"

    def fly(self) -> str:
        return f"{self.name}이(가) 날아오릅니다!"

    def perform_trick(self, trick: str) -> str:
        return f"{self.name}이(가) '{trick}'을(를) 수행합니다!"


# ──────────────────────────────────────────────
# 동물원 관리자 - 다형성 활용
# ──────────────────────────────────────────────
class Zoo:
    """동물원 - 다형성을 활용한 동물 관리"""

    def __init__(self, name: str):
        self.name = name
        self.__animals: list[Animal] = []

    def add_animal(self, animal: Animal) -> None:
        self.__animals.append(animal)
        print(f"[{self.name}] {animal.name} 입주 완료!")

    def morning_routine(self) -> None:
        """아침 루틴 - 다형성: 같은 메서드 호출, 다른 결과"""
        print(f"\n{'='*50}")
        print(f"  {self.name} - 아침 루틴 시작")
        print(f"{'='*50}")

        for animal in self.__animals:
            print(f"\n▶ {animal}")
            print(f"  울음소리: {animal.make_sound()}")
            print(f"  식단: {animal.get_diet()}")
            animal.feed(animal.get_diet().split('(')[0].strip())

            # 인터페이스 기반 다형성
            if isinstance(animal, Swimmable):
                print(f"  수영: {animal.swim()}")
            if isinstance(animal, Flyable):
                print(f"  비행: {animal.fly()}")
            if isinstance(animal, Trainable):
                print(f"  훈련: {animal.perform_trick('인사하기')}")

    def get_animal_count(self) -> int:
        return len(self.__animals)


# ──────────────────────────────────────────────
# 실행
# ──────────────────────────────────────────────
if __name__ == "__main__":
    zoo = Zoo("서울대공원")

    zoo.add_animal(Lion("심바", 5, "큰"))
    zoo.add_animal(Penguin("뽀로로", 3))
    zoo.add_animal(Parrot("폴리", 2, vocabulary_size=50))

    zoo.morning_routine()

    print(f"\n총 동물 수: {zoo.get_animal_count()}마리")
```

### C# 종합 예제

```csharp
using System;
using System.Collections.Generic;

// ── 추상화: 인터페이스 정의 ──
public interface ISwimmable
{
    string Swim();
}

public interface IFlyable
{
    string Fly();
}

public interface ITrainable
{
    string PerformTrick(string trick);
}

// ── 추상화 + 캡슐화: 추상 클래스 ──
public abstract class Animal
{
    public string Name { get; }
    public string Species { get; }
    protected int Age { get; }

    private readonly List<string> _feedingLog = new();
    public string HealthStatus { get; protected set; } = "양호";

    protected Animal(string name, string species, int age)
    {
        Name = name;
        Species = species;
        Age = age;
    }

    public abstract string MakeSound();
    public abstract string GetDiet();

    public void Feed(string food)
    {
        string entry = $"[{DateTime.Now:HH:mm}] {Name}에게 {food} 급여";
        _feedingLog.Add(entry);
        Console.WriteLine(entry);
    }

    public IReadOnlyList<string> GetFeedingHistory() => _feedingLog.AsReadOnly();

    public override string ToString()
        => $"{Species} '{Name}' (나이: {Age}살, 상태: {HealthStatus})";
}

// ── 상속 + 다형성: 구체 클래스들 ──
public class Lion : Animal
{
    public string ManeSize { get; }

    public Lion(string name, int age, string maneSize = "보통")
        : base(name, "사자", age)
    {
        ManeSize = maneSize;
    }

    public override string MakeSound() => "으르렁~!";
    public override string GetDiet() => "육식 (소고기, 닭고기)";
}

public class Penguin : Animal, ISwimmable
{
    public Penguin(string name, int age, string speciesType = "황제펭귄")
        : base(name, $"펭귄({speciesType})", age) { }

    public override string MakeSound() => "꽥꽥~!";
    public override string GetDiet() => "생선 (멸치, 청어)";
    public string Swim() => $"{Name}이(가) 물속에서 날듯이 헤엄칩니다!";
}

public class Parrot : Animal, IFlyable, ITrainable
{
    private readonly int _vocabularySize;

    public Parrot(string name, int age, int vocabularySize = 0)
        : base(name, "앵무새", age)
    {
        _vocabularySize = vocabularySize;
    }

    public override string MakeSound() => "안녕하세요~!";
    public override string GetDiet() => "과일, 씨앗, 견과류";
    public string Fly() => $"{Name}이(가) 날아오릅니다!";
    public string PerformTrick(string trick) => $"{Name}이(가) '{trick}'을(를) 수행합니다!";
}

// ── 동물원 관리 클래스 ──
public class Zoo
{
    public string Name { get; }
    private readonly List<Animal> _animals = new();

    public Zoo(string name) => Name = name;

    public void AddAnimal(Animal animal)
    {
        _animals.Add(animal);
        Console.WriteLine($"[{Name}] {animal.Name} 입주 완료!");
    }

    public void MorningRoutine()
    {
        Console.WriteLine($"\n{"".PadRight(50, '=')}");
        Console.WriteLine($"  {Name} - 아침 루틴 시작");
        Console.WriteLine($"{"".PadRight(50, '=')}");

        foreach (var animal in _animals)
        {
            Console.WriteLine($"\n▶ {animal}");
            Console.WriteLine($"  울음소리: {animal.MakeSound()}");
            Console.WriteLine($"  식단: {animal.GetDiet()}");
            animal.Feed(animal.GetDiet().Split('(')[0].Trim());

            if (animal is ISwimmable swimmer)
                Console.WriteLine($"  수영: {swimmer.Swim()}");
            if (animal is IFlyable flyer)
                Console.WriteLine($"  비행: {flyer.Fly()}");
            if (animal is ITrainable trainable)
                Console.WriteLine($"  훈련: {trainable.PerformTrick("인사하기")}");
        }
    }

    public int AnimalCount => _animals.Count;
}

// ── 실행 ──
var zoo = new Zoo("서울대공원");
zoo.AddAnimal(new Lion("심바", 5, "큰"));
zoo.AddAnimal(new Penguin("뽀로로", 3));
zoo.AddAnimal(new Parrot("폴리", 2, vocabularySize: 50));
zoo.MorningRoutine();
Console.WriteLine($"\n총 동물 수: {zoo.AnimalCount}마리");
```

---

## 7. 정리 및 체크리스트

### 4대 원칙 핵심 요약

| 원칙 | 한 줄 정리 | 점검 질문 |
|------|-----------|----------|
| **캡슐화** | 데이터를 보호하고, 메서드를 통해서만 접근하게 한다 | 외부에서 내부 상태를 직접 변경할 수 있는가? |
| **상속** | 공통 로직을 상위 클래스에 두고 재사용한다 | "is-a" 관계가 성립하는가? |
| **다형성** | 같은 인터페이스, 다른 구현으로 유연성을 확보한다 | 새 타입 추가 시 기존 코드를 수정해야 하는가? |
| **추상화** | 복잡성을 숨기고 핵심만 노출한다 | 사용자가 내부 구현을 알아야 사용할 수 있는가? |

### 셀프 체크리스트

- [ ] 클래스의 필드가 적절한 접근 제어자로 보호되고 있는가?
- [ ] 상속이 "is-a" 관계를 올바르게 표현하는가?
- [ ] 새로운 타입을 추가할 때 기존 코드를 수정하지 않아도 되는가?
- [ ] 클래스의 공개 API가 사용자에게 꼭 필요한 것만 노출하는가?
- [ ] 상속보다 합성이 더 적합한 상황은 아닌가?

---

## 8. 다음 단계

OOP의 4대 원칙을 이해했다면, 이제 이 원칙들을 **더 세밀하게 적용하는 방법론**인 SOLID 원칙을 학습할 차례입니다.

**다음 문서**: [02-solid-srp.md - S: 단일 책임 원칙 (Single Responsibility Principle)](./02-solid-srp.md)

> SOLID 원칙은 OOP 4대 원칙을 실무에서 "어떻게 잘 적용할 것인가"에 대한 구체적인 가이드라인입니다. 특히 SRP는 캡슐화와 추상화를 올바르게 적용하는 첫 번째 단계입니다.
