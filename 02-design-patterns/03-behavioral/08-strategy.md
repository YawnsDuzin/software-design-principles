# Strategy 패턴 (전략)

> **핵심 의도 한줄 요약**: 알고리즘(전략)을 캡슐화하여 교환 가능하게 만들고, 클라이언트와 독립적으로 알고리즘을 변경할 수 있게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [if/else 분기에서 Strategy 패턴으로 리팩토링](#5-ifelse-분기에서-strategy-패턴으로-리팩토링)
6. [Python 구현](#6-python-구현)
7. [C# 구현](#7-c-구현)
8. [실무 예제: 결제 시스템](#8-실무-예제-결제-시스템)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 개요

Strategy 패턴은 **동일한 문제를 해결하는 여러 알고리즘을 정의**하고, 각각을 별도의 클래스로 캡슐화한 뒤, 런타임에 교환할 수 있게 하는 패턴이다.

"같은 일을 다른 방식으로" 처리해야 할 때 사용한다.

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **Strategy** | 알고리즘의 공통 인터페이스 |
| **ConcreteStrategy** | 구체적인 알고리즘 구현 |
| **Context** | Strategy를 사용하는 클래스 (전략을 교체할 수 있다) |

---

## 2. 문제 상황

### 문제: 알고리즘의 조건 분기

네비게이션 앱에서 경로 탐색 방식을 선택해야 한다.

```python
# 나쁜 예: if/else로 알고리즘 분기
class Navigator:
    def find_route(self, start, end, mode):
        if mode == "car":
            # 자동차 경로: 고속도로 우선
            route = self._find_car_route(start, end)
        elif mode == "walk":
            # 도보 경로: 보행자 도로만
            route = self._find_walk_route(start, end)
        elif mode == "bike":
            # 자전거 경로: 자전거 도로 우선
            route = self._find_bike_route(start, end)
        elif mode == "public_transport":
            # 대중교통: 버스/지하철 노선
            route = self._find_transit_route(start, end)
        # 새 이동수단 추가마다 elif 추가...
        return route
```

문제점:

- **OCP 위반**: 새 알고리즘 추가 시 기존 코드 수정 필요
- **SRP 위반**: Navigator가 모든 경로 탐색 알고리즘을 포함
- **테스트 어려움**: 특정 알고리즘만 단독으로 테스트하기 어렵다
- **코드 비대화**: 메서드가 점점 거대해진다

---

## 3. 일상 비유

**네비게이션 앱의 경로 선택**을 떠올려 보자.

```
[네비게이션 앱] (Context)
  │
  ├─ 전략 1: 최단 거리 → 직선 거리가 짧은 경로
  ├─ 전략 2: 최소 시간 → 도로 속도를 고려한 경로
  ├─ 전략 3: 최소 비용 → 유료 도로를 피하는 경로
  └─ 전략 4: 관광 코스 → 명소를 경유하는 경로

  사용자가 전략을 선택하면 → 앱은 해당 전략으로 경로 계산
  같은 출발지/도착지도 전략에 따라 완전히 다른 결과
```

- **같은 입력**(출발지, 도착지)에 대해 **다른 알고리즘**으로 결과를 도출한다
- 사용자가 **런타임에** 전략을 선택/변경할 수 있다
- 새로운 전략(예: 전기차 경로)을 **쉽게 추가**할 수 있다

---

## 4. UML 다이어그램

```
 ┌───────────────────┐         ┌──────────────────────┐
 │     Context       │         │   <<interface>>      │
 ├───────────────────┤ has-a   │     Strategy          │
 │ - strategy        │────────>├──────────────────────┤
 ├───────────────────┤         │ + execute(data)      │
 │ + set_strategy()  │         └──────────┬───────────┘
 │ + do_work()       │                    │
 └───────────────────┘         ┌──────────┼──────────┐
                               │          │          │
                        ┌──────▼──┐ ┌─────▼───┐ ┌───▼────────┐
                        │StratA   │ │StratB   │ │ StratC     │
                        ├─────────┤ ├─────────┤ ├────────────┤
                        │+execute │ │+execute │ │+ execute   │
                        └─────────┘ └─────────┘ └────────────┘

 Context.do_work() → strategy.execute(data)
 클라이언트가 set_strategy()로 전략을 교체
```

---

## 5. if/else 분기에서 Strategy 패턴으로 리팩토링

### Before: if/else 분기

```python
class ShippingCalculator:
    def calculate(self, weight: float, method: str) -> float:
        if method == "standard":
            if weight <= 1:
                return 3000
            elif weight <= 5:
                return 5000
            else:
                return 5000 + (weight - 5) * 500
        elif method == "express":
            if weight <= 1:
                return 5000
            elif weight <= 5:
                return 8000
            else:
                return 8000 + (weight - 5) * 1000
        elif method == "overnight":
            return 15000 + weight * 2000
        else:
            raise ValueError(f"Unknown method: {method}")
```

### After: Strategy 패턴

```python
from abc import ABC, abstractmethod


class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight: float) -> float:
        pass


class StandardShipping(ShippingStrategy):
    def calculate(self, weight: float) -> float:
        if weight <= 1:
            return 3000
        elif weight <= 5:
            return 5000
        return 5000 + (weight - 5) * 500


class ExpressShipping(ShippingStrategy):
    def calculate(self, weight: float) -> float:
        if weight <= 1:
            return 5000
        elif weight <= 5:
            return 8000
        return 8000 + (weight - 5) * 1000


class OvernightShipping(ShippingStrategy):
    def calculate(self, weight: float) -> float:
        return 15000 + weight * 2000


class ShippingCalculator:
    def __init__(self, strategy: ShippingStrategy) -> None:
        self._strategy = strategy

    def set_strategy(self, strategy: ShippingStrategy) -> None:
        self._strategy = strategy

    def calculate(self, weight: float) -> float:
        return self._strategy.calculate(weight)
```

**리팩토링 효과:**
- 각 배송 전략을 **독립적으로 테스트** 가능
- 새 배송 방식 추가 시 **기존 코드 수정 불필요**
- 각 전략이 **자신의 로직에만 집중** (SRP)

---

## 6. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Callable


# === 클래스 기반 Strategy ===
class SortStrategy(ABC):
    """정렬 전략 인터페이스"""

    @abstractmethod
    def sort(self, data: list[int]) -> list[int]:
        pass

    @property
    @abstractmethod
    def name(self) -> str:
        pass


class BubbleSort(SortStrategy):
    """버블 정렬: 작은 데이터에 적합"""

    @property
    def name(self) -> str:
        return "버블 정렬"

    def sort(self, data: list[int]) -> list[int]:
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr


class QuickSort(SortStrategy):
    """퀵 정렬: 대용량 데이터에 적합"""

    @property
    def name(self) -> str:
        return "퀵 정렬"

    def sort(self, data: list[int]) -> list[int]:
        if len(data) <= 1:
            return data.copy()
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)


class Sorter:
    """정렬기 (Context)"""

    def __init__(self, strategy: SortStrategy) -> None:
        self._strategy = strategy

    def set_strategy(self, strategy: SortStrategy) -> None:
        self._strategy = strategy
        print(f"  전략 변경: {strategy.name}")

    def sort(self, data: list[int]) -> list[int]:
        print(f"  [{self._strategy.name}] 정렬 실행 ({len(data)}개 항목)")
        result = self._strategy.sort(data)
        return result


# === 함수 기반 Strategy (Python 관용적 방식) ===
class FunctionalSorter:
    """함수를 전략으로 사용하는 정렬기"""

    def __init__(self, strategy: Callable[[list[int]], list[int]]) -> None:
        self._strategy = strategy

    def set_strategy(self, strategy: Callable[[list[int]], list[int]]) -> None:
        self._strategy = strategy

    def sort(self, data: list[int]) -> list[int]:
        return self._strategy(data)


# 전략 함수들
def ascending_sort(data: list[int]) -> list[int]:
    return sorted(data)

def descending_sort(data: list[int]) -> list[int]:
    return sorted(data, reverse=True)

def absolute_sort(data: list[int]) -> list[int]:
    return sorted(data, key=abs)


# === 사용 예시 ===
if __name__ == "__main__":
    data = [38, 27, 43, 3, 9, 82, 10]

    # 클래스 기반
    print("=== 클래스 기반 Strategy ===")
    sorter = Sorter(BubbleSort())
    print(f"  결과: {sorter.sort(data)}")

    sorter.set_strategy(QuickSort())
    print(f"  결과: {sorter.sort(data)}")

    # 함수 기반
    print("\n=== 함수 기반 Strategy ===")
    func_sorter = FunctionalSorter(ascending_sort)
    print(f"  오름차순: {func_sorter.sort(data)}")

    func_sorter.set_strategy(descending_sort)
    print(f"  내림차순: {func_sorter.sort(data)}")

    # 람다도 가능
    func_sorter.set_strategy(lambda d: sorted(d, key=lambda x: x % 10))
    print(f"  일의 자리 기준: {func_sorter.sort(data)}")
```

---

## 7. C# 구현

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// 전략 인터페이스
public interface ISortStrategy
{
    string Name { get; }
    List<int> Sort(List<int> data);
}

// 버블 정렬
public class BubbleSortStrategy : ISortStrategy
{
    public string Name => "버블 정렬";

    public List<int> Sort(List<int> data)
    {
        var arr = new List<int>(data);
        for (int i = 0; i < arr.Count; i++)
        {
            for (int j = 0; j < arr.Count - i - 1; j++)
            {
                if (arr[j] > arr[j + 1])
                    (arr[j], arr[j + 1]) = (arr[j + 1], arr[j]);
            }
        }
        return arr;
    }
}

// 퀵 정렬
public class QuickSortStrategy : ISortStrategy
{
    public string Name => "퀵 정렬";

    public List<int> Sort(List<int> data)
    {
        if (data.Count <= 1) return new List<int>(data);

        int pivot = data[data.Count / 2];
        var left = data.Where(x => x < pivot).ToList();
        var middle = data.Where(x => x == pivot).ToList();
        var right = data.Where(x => x > pivot).ToList();

        var result = Sort(left);
        result.AddRange(middle);
        result.AddRange(Sort(right));
        return result;
    }
}

// Context
public class Sorter
{
    private ISortStrategy _strategy;

    public Sorter(ISortStrategy strategy)
    {
        _strategy = strategy;
    }

    public void SetStrategy(ISortStrategy strategy)
    {
        _strategy = strategy;
        Console.WriteLine($"  전략 변경: {strategy.Name}");
    }

    public List<int> Sort(List<int> data)
    {
        Console.WriteLine($"  [{_strategy.Name}] 정렬 실행 ({data.Count}개 항목)");
        return _strategy.Sort(data);
    }
}

// 함수 기반 Strategy (C#에서는 Func<> 또는 delegate 활용)
public class FunctionalSorter
{
    private Func<List<int>, List<int>> _strategy;

    public FunctionalSorter(Func<List<int>, List<int>> strategy)
    {
        _strategy = strategy;
    }

    public void SetStrategy(Func<List<int>, List<int>> strategy)
    {
        _strategy = strategy;
    }

    public List<int> Sort(List<int> data) => _strategy(data);
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var data = new List<int> { 38, 27, 43, 3, 9, 82, 10 };

        // 클래스 기반
        Console.WriteLine("=== 클래스 기반 Strategy ===");
        var sorter = new Sorter(new BubbleSortStrategy());
        Console.WriteLine($"  결과: [{string.Join(", ", sorter.Sort(data))}]");

        sorter.SetStrategy(new QuickSortStrategy());
        Console.WriteLine($"  결과: [{string.Join(", ", sorter.Sort(data))}]");

        // 함수 기반
        Console.WriteLine("\n=== 함수 기반 Strategy ===");
        var funcSorter = new FunctionalSorter(d => d.OrderBy(x => x).ToList());
        Console.WriteLine($"  오름차순: [{string.Join(", ", funcSorter.Sort(data))}]");

        funcSorter.SetStrategy(d => d.OrderByDescending(x => x).ToList());
        Console.WriteLine($"  내림차순: [{string.Join(", ", funcSorter.Sort(data))}]");
    }
}
```

---

## 8. 실무 예제: 결제 시스템

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime


@dataclass
class PaymentResult:
    """결제 결과"""
    success: bool
    transaction_id: str
    method: str
    amount: int
    message: str


class PaymentStrategy(ABC):
    """결제 전략 인터페이스"""

    @property
    @abstractmethod
    def name(self) -> str:
        pass

    @abstractmethod
    def validate(self, amount: int) -> bool:
        """결제 가능 여부 검증"""
        pass

    @abstractmethod
    def pay(self, amount: int) -> PaymentResult:
        """결제 실행"""
        pass


class CreditCardPayment(PaymentStrategy):
    """신용카드 결제"""

    def __init__(self, card_number: str, cvv: str, expiry: str) -> None:
        self._card_number = card_number
        self._cvv = cvv
        self._expiry = expiry

    @property
    def name(self) -> str:
        return "신용카드"

    def validate(self, amount: int) -> bool:
        if len(self._card_number) != 16:
            print("    카드 번호가 올바르지 않습니다.")
            return False
        if amount > 5_000_000:
            print("    1회 결제 한도(500만원) 초과입니다.")
            return False
        return True

    def pay(self, amount: int) -> PaymentResult:
        masked = f"**** **** **** {self._card_number[-4:]}"
        return PaymentResult(
            success=True,
            transaction_id=f"CC-{datetime.now().strftime('%Y%m%d%H%M%S')}",
            method=f"신용카드 ({masked})",
            amount=amount,
            message=f"카드 결제 완료: {amount:,}원",
        )


class BankTransferPayment(PaymentStrategy):
    """계좌이체 결제"""

    def __init__(self, bank_name: str, account_number: str) -> None:
        self._bank_name = bank_name
        self._account_number = account_number

    @property
    def name(self) -> str:
        return "계좌이체"

    def validate(self, amount: int) -> bool:
        if amount < 100:
            print("    최소 이체 금액은 100원입니다.")
            return False
        return True

    def pay(self, amount: int) -> PaymentResult:
        return PaymentResult(
            success=True,
            transaction_id=f"BT-{datetime.now().strftime('%Y%m%d%H%M%S')}",
            method=f"계좌이체 ({self._bank_name})",
            amount=amount,
            message=f"이체 완료: {self._bank_name} {amount:,}원",
        )


class MobilePayment(PaymentStrategy):
    """간편결제 (카카오페이, 네이버페이 등)"""

    def __init__(self, provider: str, phone: str) -> None:
        self._provider = provider
        self._phone = phone
        self._balance = 100_000  # 잔액

    @property
    def name(self) -> str:
        return f"간편결제({self._provider})"

    def validate(self, amount: int) -> bool:
        if amount > self._balance:
            print(f"    잔액 부족: {self._balance:,}원 < {amount:,}원")
            return False
        return True

    def pay(self, amount: int) -> PaymentResult:
        self._balance -= amount
        return PaymentResult(
            success=True,
            transaction_id=f"MP-{datetime.now().strftime('%Y%m%d%H%M%S')}",
            method=f"{self._provider} ({self._phone[-4:]})",
            amount=amount,
            message=f"{self._provider} 결제 완료: {amount:,}원 (잔액: {self._balance:,}원)",
        )


class PaymentProcessor:
    """결제 처리기 (Context)"""

    def __init__(self) -> None:
        self._strategy: PaymentStrategy | None = None
        self._history: list[PaymentResult] = []

    def set_payment_method(self, strategy: PaymentStrategy) -> None:
        self._strategy = strategy
        print(f"  결제 수단 설정: {strategy.name}")

    def checkout(self, amount: int) -> PaymentResult | None:
        if self._strategy is None:
            print("  결제 수단을 선택해주세요.")
            return None

        print(f"\n  --- {self._strategy.name} 결제 시작 ({amount:,}원) ---")

        # 검증
        if not self._strategy.validate(amount):
            return PaymentResult(
                success=False,
                transaction_id="",
                method=self._strategy.name,
                amount=amount,
                message="결제 검증 실패",
            )

        # 결제 실행
        result = self._strategy.pay(amount)
        if result.success:
            self._history.append(result)
        print(f"  결과: {result.message}")
        return result

    def show_history(self) -> None:
        print("\n--- 결제 히스토리 ---")
        total = 0
        for r in self._history:
            print(f"  [{r.transaction_id}] {r.method}: {r.amount:,}원")
            total += r.amount
        print(f"  총 결제액: {total:,}원")


# === 사용 예시 ===
if __name__ == "__main__":
    processor = PaymentProcessor()

    # 신용카드 결제
    card = CreditCardPayment("1234567890123456", "123", "12/25")
    processor.set_payment_method(card)
    processor.checkout(50_000)

    # 간편결제로 변경
    kakao = MobilePayment("카카오페이", "010-1234-5678")
    processor.set_payment_method(kakao)
    processor.checkout(30_000)

    # 계좌이체로 변경
    bank = BankTransferPayment("국민은행", "123-456-789")
    processor.set_payment_method(bank)
    processor.checkout(100_000)

    # 잔액 부족 테스트
    processor.set_payment_method(kakao)
    processor.checkout(80_000)  # 잔액 70,000원

    processor.show_history()
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **개방-폐쇄 원칙** | 새 전략 추가 시 기존 코드 수정 불필요 |
| **조건문 제거** | if/else, switch 분기를 다형성으로 대체 |
| **런타임 교체** | 실행 중에 알고리즘을 동적으로 변경 가능 |
| **단일 책임 원칙** | 각 알고리즘이 독립적인 클래스로 분리 |
| **테스트 용이** | 각 전략을 독립적으로 단위 테스트 가능 |
| **재사용** | 전략을 다른 Context에서 재사용 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **클래스 증가** | 전략마다 별도 클래스 필요 (함수 기반으로 완화 가능) |
| **클라이언트 인식** | 클라이언트가 전략 간 차이를 알아야 적절히 선택 가능 |
| **과도한 설계** | 전략이 2~3개뿐이면 if/else가 더 간단할 수 있다 |

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **State** | 구조는 동일하나, State는 자동 전이, Strategy는 명시적 교체 |
| **Template Method** | Strategy는 합성(Composition)으로, Template Method는 상속으로 알고리즘 변경 |
| **Command** | 둘 다 객체를 매개변수화하지만, Command는 요청 캡슐화, Strategy는 알고리즘 캡슐화 |
| **Factory Method** | 적절한 전략 객체를 생성하는 데 Factory 활용 가능 |
| **Decorator** | Strategy는 내부 알고리즘을 교체, Decorator는 외부에 기능을 추가 |

---

## 11. 정리 및 체크리스트

### 핵심 정리

1. Strategy 패턴은 **알고리즘을 캡슐화하여 교환 가능하게** 만든다.
2. **if/else 분기를 다형성으로 대체**하는 가장 기본적인 방법이다.
3. Python에서는 **함수/람다를 전략으로** 활용할 수 있어 더 간결하다.
4. State 패턴과 구조가 같지만, **전이가 없고 클라이언트가 명시적으로 선택**하는 점이 다르다.

### 적용 체크리스트

- [ ] **같은 작업을 다른 방식으로** 처리하는 알고리즘이 여러 개인가?
- [ ] 알고리즘을 **런타임에 교체**할 수 있어야 하는가?
- [ ] 조건 분기(if/else/switch)가 **복잡하고 확장이 예상**되는가?
- [ ] 알고리즘을 **독립적으로 테스트**하고 싶은가?
- [ ] 알고리즘이 **3개 이상**인가? (2개 이하면 if/else로 충분할 수 있다)

> **2~3년차 개발자를 위한 팁**: Python의 `sorted(key=...)`, Java의 `Comparator`, C#의 LINQ `OrderBy()`가 Strategy 패턴의 대표적인 예다. 함수형 프로그래밍에서는 고차 함수(Higher-Order Function)로 자연스럽게 Strategy를 구현한다. "조건 분기가 3개를 넘으면 Strategy를 고려하라"는 경험칙을 기억하자.
