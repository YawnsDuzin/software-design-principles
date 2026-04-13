# State 패턴 (상태)

> **핵심 의도 한줄 요약**: 객체의 내부 상태가 변경될 때 행동도 함께 변경되도록 하여, 마치 객체의 클래스가 바뀐 것처럼 보이게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [State vs Strategy 차이점](#7-state-vs-strategy-차이점)
8. [실무 예제: 주문 상태 관리](#8-실무-예제-주문-상태-관리)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 개요

State 패턴은 **객체의 상태에 따라 행동이 달라지는 것**을 상태를 별도의 클래스로 분리하여 구현하는 패턴이다. 상태 전이(State Transition)를 명시적으로 표현하여, if/else나 switch 문의 복잡한 분기를 제거한다.

### 핵심 아이디어

```
if/else 지옥:                    State 패턴:

if state == "대기":              context.state = WaitingState()
    do_waiting()                 context.handle()
elif state == "처리중":           // 자동으로 WaitingState.handle() 호출
    do_processing()
elif state == "완료":             상태 전이: state = ProcessingState()
    do_completed()               context.handle()
elif ...                         // 자동으로 ProcessingState.handle() 호출
```

---

## 2. 문제 상황

### 문제: 상태에 따른 조건 분기

문서 승인 시스템에서 문서의 상태에 따라 가능한 작업이 달라진다.

```python
# 나쁜 예: 상태마다 조건 분기
class Document:
    def __init__(self):
        self.state = "draft"

    def publish(self):
        if self.state == "draft":
            self.state = "review"
            print("리뷰 요청")
        elif self.state == "review":
            self.state = "published"
            print("게시 완료")
        elif self.state == "published":
            print("이미 게시됨")
        # 새 상태 추가 시 모든 메서드에 elif 추가...

    def reject(self):
        if self.state == "draft":
            print("초안은 거부할 수 없음")
        elif self.state == "review":
            self.state = "draft"
            print("초안으로 되돌림")
        elif self.state == "published":
            print("게시된 문서는 거부할 수 없음")

    def archive(self):
        if self.state == "draft":
            # ...
            pass
        elif self.state == "review":
            # ...
            pass
        # 상태 x 행동 = 모든 조합을 if/elif로 처리...
```

문제점:

- **상태 수 x 행동 수**만큼 조건 분기가 생긴다
- 새 상태 추가 시 **모든 메서드를 수정**해야 한다
- 상태 전이 로직이 **여기저기 흩어져** 있어 파악하기 어렵다

---

## 3. 일상 비유

**자판기**를 떠올려 보자.

```
[동전 없음 상태]
  - 동전 투입 → [동전 있음 상태]로 전환
  - 상품 선택 → "동전을 넣어주세요" (거부)
  - 반환 → 아무 일 없음

[동전 있음 상태]
  - 동전 투입 → 금액 추가
  - 상품 선택 → [상품 배출 상태]로 전환
  - 반환 → 동전 반환 → [동전 없음 상태]로 전환

[상품 배출 상태]
  - 상품 배출 완료 → [동전 없음 상태] 또는 [동전 있음 상태]
```

- 같은 버튼(행동)을 눌러도 **현재 상태에 따라 다른 결과**가 나온다
- 각 상태가 **자신이 처리할 행동과 전이할 다음 상태를 알고 있다**

---

## 4. UML 다이어그램

```
 ┌───────────────────┐         ┌──────────────────────┐
 │     Context       │         │   <<interface>>      │
 ├───────────────────┤ has-a   │       State           │
 │ - state: State    │────────>├──────────────────────┤
 ├───────────────────┤         │ + handle(context)    │
 │ + request()       │         └──────────┬───────────┘
 │ + change_state()  │                    │
 └───────────────────┘         ┌──────────┼──────────┐
                               │          │          │
                        ┌──────▼──┐ ┌─────▼───┐ ┌───▼───────┐
                        │ StateA  │ │ StateB  │ │ StateC    │
                        ├─────────┤ ├─────────┤ ├───────────┤
                        │+handle()│ │+handle()│ │+ handle() │
                        └─────────┘ └─────────┘ └───────────┘

 Context.request() → state.handle(context)
 State 내부에서 context.change_state(new_state)로 전이
```

---

## 5. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class VendingMachineState(ABC):
    """자판기 상태 인터페이스"""

    @abstractmethod
    def insert_coin(self, machine: VendingMachine) -> None:
        pass

    @abstractmethod
    def select_product(self, machine: VendingMachine) -> None:
        pass

    @abstractmethod
    def dispense(self, machine: VendingMachine) -> None:
        pass

    @abstractmethod
    def return_coin(self, machine: VendingMachine) -> None:
        pass


class NoCoinState(VendingMachineState):
    """동전 없음 상태"""

    def insert_coin(self, machine: VendingMachine) -> None:
        print("  동전 투입 완료")
        machine.change_state(HasCoinState())

    def select_product(self, machine: VendingMachine) -> None:
        print("  동전을 먼저 넣어주세요.")

    def dispense(self, machine: VendingMachine) -> None:
        print("  동전을 먼저 넣어주세요.")

    def return_coin(self, machine: VendingMachine) -> None:
        print("  반환할 동전이 없습니다.")

    def __str__(self) -> str:
        return "동전 없음"


class HasCoinState(VendingMachineState):
    """동전 있음 상태"""

    def insert_coin(self, machine: VendingMachine) -> None:
        print("  이미 동전이 있습니다.")

    def select_product(self, machine: VendingMachine) -> None:
        if machine.stock > 0:
            print("  상품 선택 완료. 배출 중...")
            machine.change_state(DispensingState())
        else:
            print("  재고가 없습니다. 동전을 반환합니다.")
            machine.change_state(SoldOutState())

    def dispense(self, machine: VendingMachine) -> None:
        print("  먼저 상품을 선택해주세요.")

    def return_coin(self, machine: VendingMachine) -> None:
        print("  동전 반환 완료")
        machine.change_state(NoCoinState())

    def __str__(self) -> str:
        return "동전 있음"


class DispensingState(VendingMachineState):
    """상품 배출 상태"""

    def insert_coin(self, machine: VendingMachine) -> None:
        print("  잠시 기다려주세요. 상품 배출 중입니다.")

    def select_product(self, machine: VendingMachine) -> None:
        print("  잠시 기다려주세요. 상품 배출 중입니다.")

    def dispense(self, machine: VendingMachine) -> None:
        machine.stock -= 1
        print(f"  상품이 나왔습니다! (남은 재고: {machine.stock})")
        if machine.stock > 0:
            machine.change_state(NoCoinState())
        else:
            print("  재고가 소진되었습니다.")
            machine.change_state(SoldOutState())

    def return_coin(self, machine: VendingMachine) -> None:
        print("  상품 배출 중에는 반환할 수 없습니다.")

    def __str__(self) -> str:
        return "배출 중"


class SoldOutState(VendingMachineState):
    """품절 상태"""

    def insert_coin(self, machine: VendingMachine) -> None:
        print("  품절입니다. 동전을 반환합니다.")

    def select_product(self, machine: VendingMachine) -> None:
        print("  품절입니다.")

    def dispense(self, machine: VendingMachine) -> None:
        print("  품절입니다.")

    def return_coin(self, machine: VendingMachine) -> None:
        print("  반환할 동전이 없습니다.")

    def __str__(self) -> str:
        return "품절"


class VendingMachine:
    """자판기 (Context)"""

    def __init__(self, stock: int) -> None:
        self.stock = stock
        self._state: VendingMachineState = NoCoinState()

    def change_state(self, state: VendingMachineState) -> None:
        print(f"    상태 전이: [{self._state}] → [{state}]")
        self._state = state

    def insert_coin(self) -> None:
        self._state.insert_coin(self)

    def select_product(self) -> None:
        self._state.select_product(self)

    def dispense(self) -> None:
        self._state.dispense(self)

    def return_coin(self) -> None:
        self._state.return_coin(self)

    @property
    def status(self) -> str:
        return f"[자판기] 상태: {self._state}, 재고: {self.stock}"


# === 사용 예시 ===
if __name__ == "__main__":
    machine = VendingMachine(stock=2)
    print(machine.status)

    print("\n=== 정상 구매 ===")
    machine.insert_coin()
    machine.select_product()
    machine.dispense()
    print(machine.status)

    print("\n=== 동전 없이 상품 선택 ===")
    machine.select_product()

    print("\n=== 마지막 재고 구매 ===")
    machine.insert_coin()
    machine.select_product()
    machine.dispense()
    print(machine.status)

    print("\n=== 품절 상태에서 구매 시도 ===")
    machine.insert_coin()
```

---

## 6. C# 구현

```csharp
using System;

// 상태 인터페이스
public interface IVendingMachineState
{
    void InsertCoin(VendingMachine machine);
    void SelectProduct(VendingMachine machine);
    void Dispense(VendingMachine machine);
    void ReturnCoin(VendingMachine machine);
}

// Context
public class VendingMachine
{
    private IVendingMachineState _state;
    public int Stock { get; set; }

    public VendingMachine(int stock)
    {
        Stock = stock;
        _state = new NoCoinState();
    }

    public void ChangeState(IVendingMachineState state)
    {
        Console.WriteLine($"    상태 전이: [{_state}] -> [{state}]");
        _state = state;
    }

    public void InsertCoin() => _state.InsertCoin(this);
    public void SelectProduct() => _state.SelectProduct(this);
    public void Dispense() => _state.Dispense(this);
    public void ReturnCoin() => _state.ReturnCoin(this);

    public string Status => $"[자판기] 상태: {_state}, 재고: {Stock}";
}

// 동전 없음 상태
public class NoCoinState : IVendingMachineState
{
    public void InsertCoin(VendingMachine machine)
    {
        Console.WriteLine("  동전 투입 완료");
        machine.ChangeState(new HasCoinState());
    }

    public void SelectProduct(VendingMachine machine)
        => Console.WriteLine("  동전을 먼저 넣어주세요.");

    public void Dispense(VendingMachine machine)
        => Console.WriteLine("  동전을 먼저 넣어주세요.");

    public void ReturnCoin(VendingMachine machine)
        => Console.WriteLine("  반환할 동전이 없습니다.");

    public override string ToString() => "동전 없음";
}

// 동전 있음 상태
public class HasCoinState : IVendingMachineState
{
    public void InsertCoin(VendingMachine machine)
        => Console.WriteLine("  이미 동전이 있습니다.");

    public void SelectProduct(VendingMachine machine)
    {
        if (machine.Stock > 0)
        {
            Console.WriteLine("  상품 선택 완료. 배출 중...");
            machine.ChangeState(new DispensingState());
        }
        else
        {
            Console.WriteLine("  재고가 없습니다.");
            machine.ChangeState(new SoldOutState());
        }
    }

    public void Dispense(VendingMachine machine)
        => Console.WriteLine("  먼저 상품을 선택해주세요.");

    public void ReturnCoin(VendingMachine machine)
    {
        Console.WriteLine("  동전 반환 완료");
        machine.ChangeState(new NoCoinState());
    }

    public override string ToString() => "동전 있음";
}

// 배출 상태
public class DispensingState : IVendingMachineState
{
    public void InsertCoin(VendingMachine machine)
        => Console.WriteLine("  배출 중입니다. 잠시 기다려주세요.");

    public void SelectProduct(VendingMachine machine)
        => Console.WriteLine("  배출 중입니다. 잠시 기다려주세요.");

    public void Dispense(VendingMachine machine)
    {
        machine.Stock--;
        Console.WriteLine($"  상품이 나왔습니다! (남은 재고: {machine.Stock})");
        machine.ChangeState(
            machine.Stock > 0 ? new NoCoinState() : new SoldOutState());
    }

    public void ReturnCoin(VendingMachine machine)
        => Console.WriteLine("  배출 중에는 반환 불가.");

    public override string ToString() => "배출 중";
}

// 품절 상태
public class SoldOutState : IVendingMachineState
{
    public void InsertCoin(VendingMachine machine)
        => Console.WriteLine("  품절입니다. 동전을 반환합니다.");

    public void SelectProduct(VendingMachine machine)
        => Console.WriteLine("  품절입니다.");

    public void Dispense(VendingMachine machine)
        => Console.WriteLine("  품절입니다.");

    public void ReturnCoin(VendingMachine machine)
        => Console.WriteLine("  반환할 동전이 없습니다.");

    public override string ToString() => "품절";
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var machine = new VendingMachine(2);
        Console.WriteLine(machine.Status);

        Console.WriteLine("\n=== 정상 구매 ===");
        machine.InsertCoin();
        machine.SelectProduct();
        machine.Dispense();
        Console.WriteLine(machine.Status);
    }
}
```

---

## 7. State vs Strategy 차이점

| 구분 | State | Strategy |
|------|-------|----------|
| **목적** | 상태에 따라 행동이 변경 | 알고리즘을 교체 |
| **전이** | 상태가 **자동으로 전이**됨 | 클라이언트가 **명시적으로 교체** |
| **상태 인식** | 상태 객체가 Context와 다른 상태를 안다 | Strategy는 Context를 모를 수 있다 |
| **변경 빈도** | 런타임에 **자주 변경** (상태 머신) | 초기 설정 후 **드물게 변경** |
| **관계** | 상태끼리 서로 전이 | 전략끼리 독립적 |

### 핵심 차이

```
State:
  대기 → 처리중 → 완료 (자동 전이, 상태가 다음 상태를 결정)

Strategy:
  결제(카드) → 결제(계좌이체) (클라이언트가 명시적으로 선택)
```

State는 **유한 상태 머신(FSM)**을 객체지향적으로 구현하는 패턴이다.

---

## 8. 실무 예제: 주문 상태 관리

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime


class OrderState(ABC):
    """주문 상태 인터페이스"""

    @property
    @abstractmethod
    def name(self) -> str:
        pass

    def confirm(self, order: Order) -> None:
        print(f"  [{self.name}] 확인할 수 없는 상태입니다.")

    def ship(self, order: Order) -> None:
        print(f"  [{self.name}] 배송할 수 없는 상태입니다.")

    def deliver(self, order: Order) -> None:
        print(f"  [{self.name}] 배달 완료할 수 없는 상태입니다.")

    def cancel(self, order: Order) -> None:
        print(f"  [{self.name}] 취소할 수 없는 상태입니다.")

    def refund(self, order: Order) -> None:
        print(f"  [{self.name}] 환불할 수 없는 상태입니다.")


class PendingState(OrderState):
    """대기 상태"""

    @property
    def name(self) -> str:
        return "대기"

    def confirm(self, order: Order) -> None:
        print("  주문이 확인되었습니다.")
        order.add_log("주문 확인")
        order.change_state(ConfirmedState())

    def cancel(self, order: Order) -> None:
        print("  주문이 취소되었습니다.")
        order.add_log("주문 취소")
        order.change_state(CancelledState())


class ConfirmedState(OrderState):
    """확인 상태"""

    @property
    def name(self) -> str:
        return "확인됨"

    def ship(self, order: Order) -> None:
        print("  상품이 배송 시작되었습니다.")
        order.add_log("배송 시작")
        order.change_state(ShippedState())

    def cancel(self, order: Order) -> None:
        print("  주문이 취소되었습니다. 결제를 환불합니다.")
        order.add_log("주문 취소 (환불)")
        order.change_state(CancelledState())


class ShippedState(OrderState):
    """배송 중 상태"""

    @property
    def name(self) -> str:
        return "배송중"

    def deliver(self, order: Order) -> None:
        print("  배송이 완료되었습니다.")
        order.add_log("배송 완료")
        order.change_state(DeliveredState())

    def cancel(self, order: Order) -> None:
        print("  배송 중에는 취소할 수 없습니다. 수령 후 환불해주세요.")


class DeliveredState(OrderState):
    """배달 완료 상태"""

    @property
    def name(self) -> str:
        return "배달완료"

    def refund(self, order: Order) -> None:
        print("  환불 요청이 접수되었습니다.")
        order.add_log("환불 요청")
        order.change_state(RefundedState())


class CancelledState(OrderState):
    """취소 상태 (최종)"""

    @property
    def name(self) -> str:
        return "취소됨"


class RefundedState(OrderState):
    """환불 상태 (최종)"""

    @property
    def name(self) -> str:
        return "환불됨"


class Order:
    """주문 (Context)"""

    def __init__(self, order_id: str, product: str, amount: int) -> None:
        self.order_id = order_id
        self.product = product
        self.amount = amount
        self._state: OrderState = PendingState()
        self._logs: list[str] = []
        self.add_log("주문 생성")

    def change_state(self, state: OrderState) -> None:
        old = self._state.name
        self._state = state
        print(f"    상태 전이: [{old}] → [{state.name}]")

    def add_log(self, message: str) -> None:
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self._logs.append(f"[{timestamp}] {message}")

    # 상태에 위임
    def confirm(self) -> None:
        self._state.confirm(self)

    def ship(self) -> None:
        self._state.ship(self)

    def deliver(self) -> None:
        self._state.deliver(self)

    def cancel(self) -> None:
        self._state.cancel(self)

    def refund(self) -> None:
        self._state.refund(self)

    @property
    def status(self) -> str:
        return f"주문 {self.order_id}: {self.product} ({self.amount:,}원) - [{self._state.name}]"

    def show_history(self) -> None:
        print(f"\n--- 주문 {self.order_id} 히스토리 ---")
        for log in self._logs:
            print(f"  {log}")


# === 사용 예시 ===
if __name__ == "__main__":
    order = Order("ORD-001", "무선 키보드", 89000)
    print(order.status)

    print("\n=== 정상 흐름: 대기 → 확인 → 배송 → 완료 ===")
    order.confirm()
    order.ship()
    order.deliver()
    print(order.status)

    print("\n=== 환불 요청 ===")
    order.refund()
    print(order.status)

    order.show_history()

    print("\n" + "=" * 50)

    # 취소 시나리오
    order2 = Order("ORD-002", "마우스", 45000)
    print(f"\n{order2.status}")

    print("\n=== 확인 후 취소 ===")
    order2.confirm()
    order2.cancel()
    print(order2.status)

    print("\n=== 취소된 주문에 배송 시도 ===")
    order2.ship()

    order2.show_history()
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **조건문 제거** | if/else, switch 분기를 상태 클래스로 대체 |
| **단일 책임 원칙** | 각 상태가 자신의 행동에만 집중 |
| **개방-폐쇄 원칙** | 새 상태 추가 시 기존 코드 수정 불필요 |
| **상태 전이 명시화** | 상태 전이 규칙이 코드로 명확히 표현됨 |
| **유한 상태 머신** | 복잡한 상태 머신을 체계적으로 구현 |

### 단점

| 단점 | 설명 |
|------|------|
| **클래스 증가** | 상태마다 별도 클래스가 필요하여 코드량 증가 |
| **과도한 설계** | 상태가 2~3개뿐이면 if/else가 더 간단할 수 있다 |
| **상태 간 결합** | 상태 클래스가 다른 상태 클래스를 참조해야 한다 |

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Strategy** | 구조가 유사하지만, State는 상태 전이가 있고 Strategy는 없다 |
| **Singleton** | 상태 객체가 상태를 갖지 않으면 Singleton으로 공유 가능 |
| **Flyweight** | 상태 객체를 경량 패턴으로 공유하여 메모리 절약 |
| **Command** | 상태 전이를 Command로 캡슐화하여 Undo 가능 |

---

## 11. 정리 및 체크리스트

### 핵심 정리

1. State 패턴은 **상태를 별도 클래스로 분리**하여 상태에 따른 행동 변경을 구현한다.
2. **유한 상태 머신(FSM)**을 객체지향적으로 표현하는 패턴이다.
3. Strategy와 구조는 같지만, **상태 전이(자동 변경)**가 핵심 차이다.
4. 상태가 많고 전이 규칙이 복잡할 때 큰 효과를 발휘한다.

### 적용 체크리스트

- [ ] 객체의 행동이 **상태에 따라 크게 달라지는가**?
- [ ] 상태에 따른 **조건 분기(if/switch)가 복잡**한가?
- [ ] 상태 전이 규칙이 **명시적이고 체계적**이어야 하는가?
- [ ] 새로운 상태를 **쉽게 추가**할 수 있어야 하는가?
- [ ] 상태가 **3개 이상**인가? (2개 이하면 if/else로 충분할 수 있다)

> **2~3년차 개발자를 위한 팁**: 주문 관리, 결제 처리, 문서 워크플로우, 게임 캐릭터 AI, 네트워크 연결 상태 등이 State 패턴의 대표적인 적용 사례다. 상태 전이도(State Diagram)를 먼저 그린 후 코드로 옮기면 설계가 훨씬 명확해진다. UML 상태도를 그리는 습관을 들이자.
