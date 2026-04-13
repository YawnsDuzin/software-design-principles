# O - 개방-폐쇄 원칙 (Open/Closed Principle)

> **SOLID 원칙 2/5**
> **대상 독자**: 2~3년차 개발자
> **핵심 문장**: "소프트웨어 엔티티(클래스, 모듈, 함수)는 확장에는 열려 있고, 수정에는 닫혀 있어야 한다." — Bertrand Meyer

---

## 목차

1. [OCP란 무엇인가?](#1-ocp란-무엇인가)
2. [왜 OCP가 중요한가?](#2-왜-ocp가-중요한가)
3. [OCP 위반 사례 (Before)](#3-ocp-위반-사례-before)
4. [OCP 적용 사례 (After)](#4-ocp-적용-사례-after)
5. [실무 적용 가이드](#5-실무-적용-가이드)
6. [정리 및 체크리스트](#6-정리-및-체크리스트)
7. [다음 단계](#7-다음-단계)

---

## 1. OCP란 무엇인가?

### 정의

> **"확장에는 열려 있고(Open for Extension), 수정에는 닫혀 있어야 한다(Closed for Modification)."**

- **확장에 열려 있다**: 새로운 기능을 추가할 수 있다
- **수정에 닫혀 있다**: 새로운 기능을 추가할 때 기존 코드를 변경하지 않는다

### 비유

USB 포트를 생각해 보세요:

- 컴퓨터에 USB 포트가 있으면, 마우스, 키보드, 외장하드, 프린터 등 **무엇이든 연결(확장)** 할 수 있습니다.
- 새로운 USB 기기를 연결할 때 **컴퓨터의 메인보드를 뜯어 고치지(수정) 않습니다**.
- USB라는 **표준 인터페이스**가 이를 가능하게 합니다.

### 핵심 메커니즘

OCP를 달성하는 대표적인 방법:

```
1. 추상화 (인터페이스/추상 클래스) 정의
2. 기존 코드는 추상화에 의존
3. 새 기능은 추상화를 구현하는 새 클래스로 추가
→ 기존 코드 수정 없이 기능 확장!
```

---

## 2. 왜 OCP가 중요한가?

### OCP를 지키면

- **기존 코드 안정성 보장**: 잘 동작하는 코드를 건드리지 않음
- **회귀 버그 방지**: 기존 기능이 새 기능 추가로 깨지지 않음
- **독립적 배포 가능**: 새 기능만 별도로 배포/테스트 가능
- **팀 협업 용이**: 서로 다른 개발자가 독립적으로 기능 추가 가능

### OCP를 무시하면

- **if/else 지옥**: 새 요구사항마다 조건문이 추가됨
- **산탄총 수술**: 하나의 타입이 추가될 때 여러 곳을 수정해야 함
- **회귀 버그**: 기존 코드 수정 시 예상치 못한 부분이 깨짐
- **테스트 재실행**: 수정할 때마다 모든 테스트를 다시 돌려야 함

---

## 3. OCP 위반 사례 (Before)

### 시나리오: 쇼핑몰 할인 정책

요구사항이 점점 추가되는 상황을 생각해 봅시다:
1. 일반 할인 (10%)
2. VIP 할인 (20%) ← 추가!
3. 블랙프라이데이 할인 (30%) ← 추가!
4. 쿠폰 할인 (고정 금액) ← 또 추가!

### ❌ Python - 나쁜 예: if/else 체인

```python
class DiscountCalculator:
    """
    ❌ OCP 위반!
    새 할인 정책이 추가될 때마다 이 클래스를 수정해야 한다.
    """

    def calculate_discount(self, price: float, discount_type: str, **kwargs) -> float:
        if discount_type == "regular":
            return price * 0.10

        elif discount_type == "vip":
            return price * 0.20

        elif discount_type == "black_friday":
            # 블랙프라이데이 추가 시 기존 메서드 수정!
            return price * 0.30

        elif discount_type == "coupon":
            # 쿠폰 할인 추가 시 또 수정!
            coupon_amount = kwargs.get("coupon_amount", 0)
            return min(coupon_amount, price)

        elif discount_type == "seasonal":
            # 시즌 할인 추가 시 또또 수정!
            season = kwargs.get("season", "")
            if season == "summer":
                return price * 0.15
            elif season == "winter":
                return price * 0.25
            return 0

        else:
            return 0


class OrderService:
    """
    ❌ 이 클래스도 할인 타입을 알아야 하므로 함께 수정되어야 함
    """

    def __init__(self):
        self.calculator = DiscountCalculator()

    def process_order(self, price: float, discount_type: str, **kwargs) -> float:
        discount = self.calculator.calculate_discount(price, discount_type, **kwargs)

        # ❌ 여기서도 타입별 분기가 필요할 수 있음
        if discount_type == "coupon":
            print(f"쿠폰 할인 적용: {discount:,.0f}원")
        elif discount_type == "vip":
            print(f"VIP 할인 적용: {discount:,.0f}원")
        else:
            print(f"할인 적용: {discount:,.0f}원")

        return price - discount
```

**문제점**:
- 새 할인 정책이 추가될 때마다 `calculate_discount` 메서드를 수정해야 함
- `OrderService`도 함께 수정해야 할 가능성이 높음
- 기존 if/else 중 하나라도 실수로 건드리면 회귀 버그 발생
- 할인 정책이 20개가 되면? 읽기 힘든 거대한 if/else 체인

### ❌ C# - 나쁜 예: switch 문

```csharp
public class DiscountCalculator
{
    // ❌ OCP 위반! 새 할인 정책 추가 시 switch 수정 필요
    public decimal CalculateDiscount(decimal price, string discountType, Dictionary<string, object>? options = null)
    {
        switch (discountType)
        {
            case "regular":
                return price * 0.10m;

            case "vip":
                return price * 0.20m;

            case "black_friday":
                return price * 0.30m;

            case "coupon":
                var amount = (decimal)(options?["coupon_amount"] ?? 0m);
                return Math.Min(amount, price);

            case "seasonal":
                var season = options?["season"]?.ToString();
                return season switch
                {
                    "summer" => price * 0.15m,
                    "winter" => price * 0.25m,
                    _ => 0m
                };

            default:
                return 0m;
        }
    }
}
```

---

## 4. OCP 적용 사례 (After)

### 전략 패턴(Strategy Pattern)으로 해결

핵심 아이디어: **할인 정책을 인터페이스로 추상화하고, 각 정책을 별도 클래스로 구현**한다.

### ✅ Python - 좋은 예: Strategy 패턴

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass


# ──────────────────────────────────────────────
# 추상화: 할인 정책 인터페이스
# ──────────────────────────────────────────────
class DiscountPolicy(ABC):
    """할인 정책 추상화 - 모든 정책은 이 인터페이스를 따른다"""

    @abstractmethod
    def calculate(self, price: float) -> float:
        """할인 금액을 계산하여 반환"""
        pass

    @abstractmethod
    def description(self) -> str:
        """할인 정책 설명"""
        pass


# ──────────────────────────────────────────────
# 구현: 각 할인 정책 (기존 코드 수정 없이 추가 가능!)
# ──────────────────────────────────────────────
class RegularDiscount(DiscountPolicy):
    """일반 할인 (10%)"""

    def calculate(self, price: float) -> float:
        return price * 0.10

    def description(self) -> str:
        return "일반 할인 10%"


class VipDiscount(DiscountPolicy):
    """VIP 할인 (20%)"""

    def calculate(self, price: float) -> float:
        return price * 0.20

    def description(self) -> str:
        return "VIP 할인 20%"


class BlackFridayDiscount(DiscountPolicy):
    """블랙프라이데이 할인 (30%)"""

    def calculate(self, price: float) -> float:
        return price * 0.30

    def description(self) -> str:
        return "블랙프라이데이 할인 30%"


class CouponDiscount(DiscountPolicy):
    """쿠폰 할인 (고정 금액)"""

    def __init__(self, coupon_amount: float):
        self._coupon_amount = coupon_amount

    def calculate(self, price: float) -> float:
        return min(self._coupon_amount, price)

    def description(self) -> str:
        return f"쿠폰 할인 {self._coupon_amount:,.0f}원"


class SeasonalDiscount(DiscountPolicy):
    """시즌 할인 - 시즌별 할인율"""

    SEASON_RATES = {
        "spring": 0.10,
        "summer": 0.15,
        "autumn": 0.12,
        "winter": 0.25,
    }

    def __init__(self, season: str):
        self._season = season
        self._rate = self.SEASON_RATES.get(season, 0)

    def calculate(self, price: float) -> float:
        return price * self._rate

    def description(self) -> str:
        return f"시즌({self._season}) 할인 {self._rate:.0%}"


# ──────────────────────────────────────────────
# 복합 할인 정책 (정책 조합도 새 클래스로!)
# ──────────────────────────────────────────────
class CompositeDiscount(DiscountPolicy):
    """여러 할인을 조합 (최대 할인 선택)"""

    def __init__(self, policies: list[DiscountPolicy]):
        self._policies = policies

    def calculate(self, price: float) -> float:
        if not self._policies:
            return 0
        return max(policy.calculate(price) for policy in self._policies)

    def description(self) -> str:
        names = ", ".join(p.description() for p in self._policies)
        return f"복합 할인 (최대값 적용): [{names}]"


# ──────────────────────────────────────────────
# OrderService - 수정에 닫혀 있다! (기존 코드 변경 불필요)
# ──────────────────────────────────────────────
@dataclass
class Order:
    product_name: str
    price: float
    discount_policy: DiscountPolicy

    @property
    def discount_amount(self) -> float:
        return self.discount_policy.calculate(self.price)

    @property
    def final_price(self) -> float:
        return self.price - self.discount_amount


class OrderService:
    """
    ✅ OCP 준수!
    새 할인 정책이 추가되어도 이 클래스는 전혀 수정할 필요 없다.
    DiscountPolicy 인터페이스에만 의존하기 때문이다.
    """

    def process_order(self, order: Order) -> None:
        print(f"상품: {order.product_name}")
        print(f"원가: {order.price:,.0f}원")
        print(f"할인: {order.discount_policy.description()}")
        print(f"할인액: -{order.discount_amount:,.0f}원")
        print(f"최종 가격: {order.final_price:,.0f}원")
        print("-" * 40)


# ──────────────────────────────────────────────
# 사용 예시
# ──────────────────────────────────────────────
if __name__ == "__main__":
    service = OrderService()

    # 다양한 할인 정책 적용 - OrderService 수정 없이!
    orders = [
        Order("노트북", 1_500_000, RegularDiscount()),
        Order("모니터", 800_000, VipDiscount()),
        Order("키보드", 150_000, BlackFridayDiscount()),
        Order("마우스", 80_000, CouponDiscount(15_000)),
        Order("헤드셋", 200_000, SeasonalDiscount("winter")),
        Order("태블릿", 1_000_000, CompositeDiscount([
            VipDiscount(),
            SeasonalDiscount("winter"),
            CouponDiscount(180_000),
        ])),
    ]

    for order in orders:
        service.process_order(order)


# ──────────────────────────────────────────────
# 🎯 나중에 새 할인 정책 추가하기 (기존 코드 수정 0!)
# ──────────────────────────────────────────────
class FirstPurchaseDiscount(DiscountPolicy):
    """첫 구매 할인 - 새 파일에 새 클래스만 추가하면 끝!"""

    def calculate(self, price: float) -> float:
        return price * 0.15

    def description(self) -> str:
        return "첫 구매 할인 15%"

# 기존 OrderService, Order 등은 전혀 수정하지 않음!
# new_order = Order("스피커", 300_000, FirstPurchaseDiscount())
# service.process_order(new_order)  # 바로 동작!
```

### ✅ C# - 좋은 예: Strategy 패턴

```csharp
// ── 추상화: 할인 정책 인터페이스 ──
public interface IDiscountPolicy
{
    decimal Calculate(decimal price);
    string Description { get; }
}

// ── 각 할인 정책 구현 ──
public class RegularDiscount : IDiscountPolicy
{
    public decimal Calculate(decimal price) => price * 0.10m;
    public string Description => "일반 할인 10%";
}

public class VipDiscount : IDiscountPolicy
{
    public decimal Calculate(decimal price) => price * 0.20m;
    public string Description => "VIP 할인 20%";
}

public class BlackFridayDiscount : IDiscountPolicy
{
    public decimal Calculate(decimal price) => price * 0.30m;
    public string Description => "블랙프라이데이 할인 30%";
}

public class CouponDiscount : IDiscountPolicy
{
    private readonly decimal _couponAmount;

    public CouponDiscount(decimal couponAmount) => _couponAmount = couponAmount;

    public decimal Calculate(decimal price) => Math.Min(_couponAmount, price);
    public string Description => $"쿠폰 할인 {_couponAmount:#,0}원";
}

public class SeasonalDiscount : IDiscountPolicy
{
    private static readonly Dictionary<string, decimal> SeasonRates = new()
    {
        ["spring"] = 0.10m,
        ["summer"] = 0.15m,
        ["autumn"] = 0.12m,
        ["winter"] = 0.25m,
    };

    private readonly string _season;
    private readonly decimal _rate;

    public SeasonalDiscount(string season)
    {
        _season = season;
        _rate = SeasonRates.GetValueOrDefault(season, 0m);
    }

    public decimal Calculate(decimal price) => price * _rate;
    public string Description => $"시즌({_season}) 할인 {_rate:P0}";
}

// ── 복합 할인 ──
public class CompositeDiscount : IDiscountPolicy
{
    private readonly IReadOnlyList<IDiscountPolicy> _policies;

    public CompositeDiscount(IEnumerable<IDiscountPolicy> policies)
        => _policies = policies.ToList();

    public decimal Calculate(decimal price)
        => _policies.Any() ? _policies.Max(p => p.Calculate(price)) : 0m;

    public string Description
        => $"복합 할인: [{string.Join(", ", _policies.Select(p => p.Description))}]";
}

// ── 주문 모델 ──
public class Order
{
    public string ProductName { get; }
    public decimal Price { get; }
    public IDiscountPolicy DiscountPolicy { get; }

    public Order(string productName, decimal price, IDiscountPolicy discountPolicy)
    {
        ProductName = productName;
        Price = price;
        DiscountPolicy = discountPolicy;
    }

    public decimal DiscountAmount => DiscountPolicy.Calculate(Price);
    public decimal FinalPrice => Price - DiscountAmount;
}

// ── 주문 서비스: 수정에 닫혀 있다! ──
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        Console.WriteLine($"상품: {order.ProductName}");
        Console.WriteLine($"원가: {order.Price:#,0}원");
        Console.WriteLine($"할인: {order.DiscountPolicy.Description}");
        Console.WriteLine($"할인액: -{order.DiscountAmount:#,0}원");
        Console.WriteLine($"최종 가격: {order.FinalPrice:#,0}원");
        Console.WriteLine(new string('-', 40));
    }
}

// ── 사용 예시 ──
var service = new OrderService();

var orders = new[]
{
    new Order("노트북", 1_500_000m, new RegularDiscount()),
    new Order("모니터", 800_000m, new VipDiscount()),
    new Order("키보드", 150_000m, new BlackFridayDiscount()),
    new Order("마우스", 80_000m, new CouponDiscount(15_000m)),
    new Order("헤드셋", 200_000m, new SeasonalDiscount("winter")),
};

foreach (var order in orders)
    service.ProcessOrder(order);

// 나중에 추가: 기존 코드 수정 없이 새 클래스만 작성!
public class FirstPurchaseDiscount : IDiscountPolicy
{
    public decimal Calculate(decimal price) => price * 0.15m;
    public string Description => "첫 구매 할인 15%";
}
```

---

## 5. 실무 적용 가이드

### OCP 적용이 적합한 상황

| 상황 | 해결 패턴 | 예시 |
|------|----------|------|
| 타입별 분기 처리 | Strategy / State | 할인 정책, 결제 수단, 알림 채널 |
| 데이터 변환/포맷 | Adapter / Factory | JSON/XML/CSV 내보내기 |
| 이벤트 처리 | Observer | 주문 완료 후 처리 (이메일, 포인트, 재고) |
| 필터/검증 규칙 | Chain of Responsibility | 주문 검증, 데이터 필터링 |

### 단계별 적용 방법

```
1단계: if/else 또는 switch 체인에서 "냄새"를 맡는다
2단계: 공통 인터페이스(추상화)를 정의한다
3단계: 각 분기를 별도 클래스로 추출한다
4단계: 기존 코드가 인터페이스에만 의존하도록 변경한다
5단계: 새 요구사항은 새 클래스로만 처리한다
```

### 적용 전후 비교

```python
# ❌ Before: 새 타입 추가 시 기존 코드 수정
def export_data(data, format_type):
    if format_type == "json":
        return json.dumps(data)
    elif format_type == "csv":
        return convert_to_csv(data)
    elif format_type == "xml":     # 추가할 때마다 수정!
        return convert_to_xml(data)

# ✅ After: 새 타입 추가 시 새 클래스만 작성
class DataExporter(ABC):
    @abstractmethod
    def export(self, data: dict) -> str:
        pass

class JsonExporter(DataExporter):
    def export(self, data: dict) -> str:
        return json.dumps(data)

class CsvExporter(DataExporter):
    def export(self, data: dict) -> str:
        return convert_to_csv(data)

# 새로 추가! 기존 코드 수정 없음
class XmlExporter(DataExporter):
    def export(self, data: dict) -> str:
        return convert_to_xml(data)
```

### 주의: OCP를 100% 달성할 필요는 없다

> **"모든 변경에 대해 닫혀 있을 수는 없다."**
>
> OCP는 **가능성이 높은 변경**에 대비하는 것이지, 모든 변경을 예측하라는 뜻이 아닙니다.
>
> 실무 규칙:
> - **처음**에는 간단하게 구현한다 (if/else도 괜찮다)
> - **같은 종류의 분기가 3번 이상** 추가되면 OCP 적용을 고려한다
> - 이를 **"Rule of Three"** (세 번의 법칙)라고 부른다

---

## 6. 정리 및 체크리스트

### 핵심 요약

| 항목 | 설명 |
|------|------|
| **원칙** | 확장에는 열려 있고, 수정에는 닫혀 있어야 한다 |
| **핵심 도구** | 추상화 (인터페이스, 추상 클래스) |
| **대표 패턴** | Strategy, Template Method, Observer |
| **위반 징후** | if/else 또는 switch 체인이 계속 길어짐 |
| **적용 시점** | 같은 종류의 분기가 3회 이상 반복될 때 |

### 셀프 체크리스트

- [ ] 새 기능 추가 시 기존 클래스를 수정하지 않고도 가능한가?
- [ ] if/else 또는 switch 체인이 계속 길어지고 있지 않은가?
- [ ] 타입별 분기 로직이 여러 곳에 중복되어 있지 않은가?
- [ ] 변경 가능성이 높은 부분이 추상화되어 있는가?
- [ ] 과도한 추상화로 오히려 복잡해지지 않았는가? (YAGNI)

---

## 7. 다음 단계

OCP로 확장 가능한 설계를 배웠다면, 이제 **상속을 올바르게 사용하는 방법**을 배울 차례입니다. 잘못된 상속은 OCP를 무너뜨립니다.

**다음 문서**: [04-solid-lsp.md - L: 리스코프 치환 원칙 (Liskov Substitution Principle)](./04-solid-lsp.md)

> "모든 상속이 올바른 것은 아닙니다." 하위 타입이 상위 타입을 정말로 대체할 수 있는지, 대표적인 함정과 해결법을 알아봅니다.
