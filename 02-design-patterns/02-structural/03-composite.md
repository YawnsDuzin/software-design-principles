# Composite 패턴 (컴포지트)

> **핵심 의도 한줄 요약:** 개별 객체(Leaf)와 복합 객체(Composite)를 동일한 인터페이스로 다루어, 트리 구조를 일관되게 처리하는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [투명성 vs 안전성](#5-투명성-vs-안전성)
6. [패턴 적용 전 (Before)](#6-패턴-적용-전-before)
7. [패턴 적용 후 (After)](#7-패턴-적용-후-after)
8. [Python 구현](#8-python-구현)
9. [C# 구현](#9-c-구현)
10. [실무 예제: 파일 시스템](#10-실무-예제-파일-시스템)
11. [장단점](#11-장단점)
12. [관련 패턴](#12-관련-패턴)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)

---

## 1. 개요

Composite 패턴은 **GoF 구조 패턴** 중 하나로, 객체들을 트리 구조로 구성하여 **부분-전체 계층 구조(Part-Whole Hierarchy)**를 표현합니다.

핵심 아이디어: "클라이언트가 단일 객체와 복합 객체를 구분하지 않고 동일하게 다룰 수 있게 한다."

실무에서 마주치는 상황:
- 파일 시스템: 파일과 폴더를 동일하게 다루기
- UI 컴포넌트: 버튼(Leaf)과 패널(Composite)을 동일하게 렌더링
- 메뉴 시스템: 메뉴 항목과 서브 메뉴
- 조직도: 직원과 부서

---

## 2. 왜 필요한가? - 문제 상황

### 상황: 가격 계산 시스템

온라인 쇼핑몰에서 상품 묶음(패키지)을 판매합니다:

```
선물 세트 A (박스)
├── 초콜릿 (10,000원)
├── 과자 세트 (서브 박스)
│   ├── 감자칩 (3,000원)
│   └── 쿠키 (2,000원)
└── 와인 (25,000원)

총 가격 = ?
```

Composite 패턴 없이 이를 처리하려면:

```python
# 매번 타입을 확인해야 하는 번거로운 코드
def calculate_price(item):
    if isinstance(item, Product):
        return item.price
    elif isinstance(item, Box):
        total = 0
        for child in item.children:
            if isinstance(child, Product):
                total += child.price
            elif isinstance(child, Box):
                total += calculate_price(child)  # 재귀적 타입 체크
        return total
```

**문제점:**
- 타입 검사가 코드 전반에 산재
- 새로운 타입 추가 시 모든 곳의 조건문 수정 필요
- 깊이가 깊어질수록 코드 복잡도 증가

---

## 3. 비유로 이해하기

### 파일 시스템

```
[내 문서] (폴더 = Composite)
├── report.pdf (파일 = Leaf)
├── photo.jpg (파일 = Leaf)
└── [프로젝트] (폴더 = Composite)
    ├── main.py (파일 = Leaf)
    └── [tests] (폴더 = Composite)
        └── test_main.py (파일 = Leaf)

"크기 구하기" 연산:
- 파일: 자기 자신의 크기 반환
- 폴더: 하위 모든 항목의 크기를 합산하여 반환

클라이언트는 파일이든 폴더든 상관없이 .get_size()를 호출하면 됨!
```

---

## 4. UML 다이어그램

```
                      ┌───────────────────────┐
                      │   <<interface>>        │
                      │     Component          │
                      │───────────────────────│
                      │ + operation()          │
                      │ + get_price()          │
                      └───────────────────────┘
                                 △
                                 │
                  ┌──────────────┴──────────────┐
                  │                              │
        ┌─────────────────┐           ┌─────────────────────┐
        │      Leaf        │           │     Composite        │
        │─────────────────│           │─────────────────────│
        │ + operation()    │           │ - children: List     │
        │   (자신만 처리)  │           │─────────────────────│
        │                  │           │ + operation()         │
        │                  │           │   (children에 위임)   │
        └─────────────────┘           │ + add(component)      │
                                      │ + remove(component)   │
                                      │ + get_child(index)    │
                                      └─────────────────────┘
                                              │
                                              │ children
                                              ▼
                                      ┌───────────────┐
                                      │  Component[]   │
                                      │ (Leaf 또는     │
                                      │  Composite)    │
                                      └───────────────┘
```

---

## 5. 투명성 vs 안전성

Composite 패턴 설계에는 두 가지 접근법이 있습니다:

### 투명성 방식 (Transparency)

```
Component에 모든 메서드 선언 (add, remove, get_child 포함)

장점: 클라이언트가 Leaf/Composite를 완전히 동일하게 취급
단점: Leaf에서 add()를 호출하면 의미 없음 (예외 발생 필요)
```

```python
class Component(ABC):
    @abstractmethod
    def operation(self): pass

    def add(self, component):
        raise NotImplementedError("Leaf에는 추가할 수 없습니다")

    def remove(self, component):
        raise NotImplementedError("Leaf에서는 제거할 수 없습니다")
```

### 안전성 방식 (Safety)

```
Composite에만 add, remove, get_child 선언

장점: Leaf에서 의미 없는 메서드가 호출될 수 없음 (컴파일 타임 안전)
단점: 클라이언트가 타입을 확인해야 할 수 있음
```

```python
class Component(ABC):
    @abstractmethod
    def operation(self): pass
    # add, remove 없음!

class Composite(Component):
    def add(self, component): ...      # Composite에만 존재
    def remove(self, component): ...   # Composite에만 존재
```

### 비교

| 구분 | 투명성 방식 | 안전성 방식 |
|------|-----------|-----------|
| **Component** | add/remove 포함 | operation만 포함 |
| **Leaf** | add/remove 호출 시 예외 | add/remove 메서드 없음 |
| **장점** | 클라이언트가 단순 | 타입 안전성 보장 |
| **단점** | 런타임 에러 가능 | 타입 캐스팅 필요할 수 있음 |
| **권장** | Python (덕 타이핑) | C#/Java (정적 타이핑) |

---

## 6. 패턴 적용 전 (Before)

```python
# ❌ Before: 타입 체크가 코드 전반에 산재

class Product:
    def __init__(self, name: str, price: int):
        self.name = name
        self.price = price


class Box:
    def __init__(self, name: str):
        self.name = name
        self.items = []  # Product 또는 Box

    def add(self, item):
        self.items.append(item)


# 가격 계산: 매번 타입 체크 필요
def calculate_total(item) -> int:
    if isinstance(item, Product):
        return item.price
    elif isinstance(item, Box):
        return sum(calculate_total(child) for child in item.items)
    else:
        raise TypeError(f"알 수 없는 타입: {type(item)}")


# 출력: 또 타입 체크
def print_structure(item, indent=0):
    prefix = "  " * indent
    if isinstance(item, Product):
        print(f"{prefix}- {item.name}: {item.price:,}원")
    elif isinstance(item, Box):
        print(f"{prefix}[{item.name}]")
        for child in item.items:
            print_structure(child, indent + 1)


# 할인 적용: 또또 타입 체크
def apply_discount(item, rate: float):
    if isinstance(item, Product):
        item.price = int(item.price * (1 - rate))
    elif isinstance(item, Box):
        for child in item.items:
            apply_discount(child, rate)

# 새로운 기능을 추가할 때마다 isinstance 체크가 계속 필요!
```

---

## 7. 패턴 적용 후 (After)

```python
# ✅ After: Composite 패턴으로 동일한 인터페이스

from abc import ABC, abstractmethod


class OrderItem(ABC):
    """Component: 주문 항목의 공통 인터페이스"""

    @abstractmethod
    def get_price(self) -> int:
        pass

    @abstractmethod
    def display(self, indent: int = 0) -> str:
        pass


class Product(OrderItem):
    """Leaf: 개별 상품"""

    def __init__(self, name: str, price: int):
        self.name = name
        self.price = price

    def get_price(self) -> int:
        return self.price

    def display(self, indent: int = 0) -> str:
        return f"{'  ' * indent}- {self.name}: {self.price:,}원"


class Box(OrderItem):
    """Composite: 상품 묶음"""

    def __init__(self, name: str):
        self.name = name
        self._items: list[OrderItem] = []

    def add(self, item: OrderItem):
        self._items.append(item)

    def get_price(self) -> int:
        return sum(item.get_price() for item in self._items)

    def display(self, indent: int = 0) -> str:
        lines = [f"{'  ' * indent}[{self.name}] ({self.get_price():,}원)"]
        for item in self._items:
            lines.append(item.display(indent + 1))
        return "\n".join(lines)


# 클라이언트: 타입 체크 없이 동일하게 사용
def process_order(item: OrderItem):
    print(item.display())
    print(f"총 가격: {item.get_price():,}원")
```

---

## 8. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Iterator


# ===== Component =====
class MenuComponent(ABC):
    """메뉴 컴포넌트: 메뉴 항목과 서브 메뉴의 공통 인터페이스"""

    def __init__(self, name: str, description: str = ""):
        self._name = name
        self._description = description

    @property
    def name(self) -> str:
        return self._name

    @property
    def description(self) -> str:
        return self._description

    @abstractmethod
    def display(self, indent: int = 0) -> None:
        """메뉴를 출력"""
        pass

    @abstractmethod
    def get_price(self) -> int:
        """총 가격 계산"""
        pass

    @abstractmethod
    def search(self, keyword: str) -> list[MenuComponent]:
        """키워드로 메뉴 검색"""
        pass

    def __iter__(self) -> Iterator[MenuComponent]:
        """반복 가능하게 만들기 (Composite에서 오버라이드)"""
        yield self


# ===== Leaf =====
class MenuItem(MenuComponent):
    """메뉴 항목 (Leaf): 하위 항목을 가질 수 없음"""

    def __init__(self, name: str, description: str, price: int,
                 is_vegetarian: bool = False):
        super().__init__(name, description)
        self._price = price
        self._is_vegetarian = is_vegetarian

    @property
    def is_vegetarian(self) -> bool:
        return self._is_vegetarian

    def display(self, indent: int = 0) -> None:
        prefix = "  " * indent
        veg = " (V)" if self._is_vegetarian else ""
        print(f"{prefix}- {self._name}{veg}: {self._price:,}원")
        if self._description:
            print(f"{prefix}  {self._description}")

    def get_price(self) -> int:
        return self._price

    def search(self, keyword: str) -> list[MenuComponent]:
        if keyword.lower() in self._name.lower():
            return [self]
        return []


# ===== Composite =====
class Menu(MenuComponent):
    """메뉴 (Composite): 하위 메뉴 항목이나 서브 메뉴를 포함"""

    def __init__(self, name: str, description: str = ""):
        super().__init__(name, description)
        self._children: list[MenuComponent] = []

    def add(self, component: MenuComponent) -> Menu:
        """메서드 체이닝을 위해 self 반환"""
        self._children.append(component)
        return self

    def remove(self, component: MenuComponent) -> None:
        self._children.remove(component)

    def display(self, indent: int = 0) -> None:
        prefix = "  " * indent
        print(f"{prefix}[{self._name}] ({self.get_price():,}원)")
        if self._description:
            print(f"{prefix}  {self._description}")
        for child in self._children:
            child.display(indent + 1)

    def get_price(self) -> int:
        """하위 항목의 가격 합산 (재귀)"""
        return sum(child.get_price() for child in self._children)

    def search(self, keyword: str) -> list[MenuComponent]:
        """하위 항목 전체에서 검색 (재귀)"""
        results = []
        if keyword.lower() in self._name.lower():
            results.append(self)
        for child in self._children:
            results.extend(child.search(keyword))
        return results

    def __iter__(self) -> Iterator[MenuComponent]:
        """트리 전체를 순회하는 이터레이터"""
        yield self
        for child in self._children:
            yield from child

    def __len__(self) -> int:
        return len(self._children)


# ===== 사용 예시 =====
if __name__ == "__main__":
    # Leaf 생성
    espresso = MenuItem("에스프레소", "진한 에스프레소", 3000)
    americano = MenuItem("아메리카노", "시원한 아메리카노", 3500)
    latte = MenuItem("카페라떼", "부드러운 라떼", 4000)
    green_tea = MenuItem("녹차", "제주 녹차", 3500, is_vegetarian=True)

    caesar = MenuItem("시저 샐러드", "신선한 로메인", 8000, is_vegetarian=True)
    steak = MenuItem("안심 스테이크", "호주산 안심", 25000)
    pasta = MenuItem("크림 파스타", "수제 크림 소스", 13000)

    cake = MenuItem("치즈 케이크", "뉴욕 스타일", 6000, is_vegetarian=True)
    tiramisu = MenuItem("티라미수", "이탈리안 정통", 7000)

    # Composite 구성 (트리 구조)
    coffee_menu = Menu("커피", "다양한 커피 메뉴")
    coffee_menu.add(espresso).add(americano).add(latte)

    tea_menu = Menu("차", "건강한 차 메뉴")
    tea_menu.add(green_tea)

    drink_menu = Menu("음료", "모든 음료")
    drink_menu.add(coffee_menu).add(tea_menu)

    food_menu = Menu("식사", "점심/저녁 메뉴")
    food_menu.add(caesar).add(steak).add(pasta)

    dessert_menu = Menu("디저트", "달콤한 디저트")
    dessert_menu.add(cake).add(tiramisu)

    # 최상위 메뉴
    all_menu = Menu("전체 메뉴", "레스토랑 전체 메뉴")
    all_menu.add(drink_menu).add(food_menu).add(dessert_menu)

    # 전체 메뉴 출력 (동일한 인터페이스!)
    print("=" * 50)
    print("레스토랑 메뉴")
    print("=" * 50)
    all_menu.display()

    # 검색 (동일한 인터페이스!)
    print(f"\n{'=' * 50}")
    print("'커피' 검색 결과:")
    results = all_menu.search("커피")
    for item in results:
        item.display(indent=1)

    # 이터레이터로 전체 순회
    print(f"\n{'=' * 50}")
    print("채식 메뉴:")
    for item in all_menu:
        if isinstance(item, MenuItem) and item.is_vegetarian:
            item.display(indent=1)

    # 가격 계산 (동일한 인터페이스!)
    print(f"\n{'=' * 50}")
    print(f"전체 메뉴 합계: {all_menu.get_price():,}원")
    print(f"음료 합계: {drink_menu.get_price():,}원")
    print(f"에스프레소 가격: {espresso.get_price():,}원")
```

**실행 결과:**
```
==================================================
레스토랑 메뉴
==================================================
[전체 메뉴] (73,000원)
  레스토랑 전체 메뉴
  [음료] (14,000원)
    모든 음료
    [커피] (10,500원)
      다양한 커피 메뉴
      - 에스프레소: 3,000원
        진한 에스프레소
      - 아메리카노: 3,500원
        시원한 아메리카노
      - 카페라떼: 4,000원
        부드러운 라떼
    [차] (3,500원)
      건강한 차 메뉴
      - 녹차 (V): 3,500원
        제주 녹차
  [식사] (46,000원)
    점심/저녁 메뉴
    - 시저 샐러드 (V): 8,000원
      신선한 로메인
    - 안심 스테이크: 25,000원
      호주산 안심
    - 크림 파스타: 13,000원
      수제 크림 소스
  [디저트] (13,000원)
    달콤한 디저트
    - 치즈 케이크 (V): 6,000원
      뉴욕 스타일
    - 티라미수: 7,000원
      이탈리안 정통
```

---

## 9. C# 구현

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

// ===== Component =====
public abstract class MenuComponent
{
    public string Name { get; }
    public string Description { get; }

    protected MenuComponent(string name, string description = "")
    {
        Name = name;
        Description = description;
    }

    public abstract void Display(int indent = 0);
    public abstract int GetPrice();
    public abstract List<MenuComponent> Search(string keyword);
}

// ===== Leaf =====
public class MenuItem : MenuComponent
{
    public int Price { get; private set; }
    public bool IsVegetarian { get; }

    public MenuItem(string name, string description, int price,
                    bool isVegetarian = false)
        : base(name, description)
    {
        Price = price;
        IsVegetarian = isVegetarian;
    }

    public override void Display(int indent = 0)
    {
        var prefix = new string(' ', indent * 2);
        var veg = IsVegetarian ? " (V)" : "";
        Console.WriteLine(
            $"{prefix}- {Name}{veg}: {Price:N0}원");
        if (!string.IsNullOrEmpty(Description))
            Console.WriteLine($"{prefix}  {Description}");
    }

    public override int GetPrice() => Price;

    public override List<MenuComponent> Search(string keyword)
    {
        var results = new List<MenuComponent>();
        if (Name.Contains(keyword, StringComparison.OrdinalIgnoreCase))
            results.Add(this);
        return results;
    }
}

// ===== Composite =====
public class Menu : MenuComponent, IEnumerable<MenuComponent>
{
    private readonly List<MenuComponent> _children = new();

    public Menu(string name, string description = "")
        : base(name, description) { }

    public Menu Add(MenuComponent component)
    {
        _children.Add(component);
        return this;  // 메서드 체이닝
    }

    public void Remove(MenuComponent component)
    {
        _children.Remove(component);
    }

    public override void Display(int indent = 0)
    {
        var prefix = new string(' ', indent * 2);
        Console.WriteLine(
            $"{prefix}[{Name}] ({GetPrice():N0}원)");
        if (!string.IsNullOrEmpty(Description))
            Console.WriteLine($"{prefix}  {Description}");

        foreach (var child in _children)
            child.Display(indent + 1);
    }

    public override int GetPrice()
    {
        return _children.Sum(child => child.GetPrice());
    }

    public override List<MenuComponent> Search(string keyword)
    {
        var results = new List<MenuComponent>();
        if (Name.Contains(keyword, StringComparison.OrdinalIgnoreCase))
            results.Add(this);

        foreach (var child in _children)
            results.AddRange(child.Search(keyword));

        return results;
    }

    // IEnumerable 구현: 트리 전체를 순회
    public IEnumerator<MenuComponent> GetEnumerator()
    {
        yield return this;
        foreach (var child in _children)
        {
            if (child is Menu menu)
            {
                foreach (var item in menu)
                    yield return item;
            }
            else
            {
                yield return child;
            }
        }
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

// ===== 사용 예시 =====
public class Program
{
    public static void Main()
    {
        // Leaf 생성
        var espresso = new MenuItem("에스프레소", "진한 에스프레소", 3000);
        var americano = new MenuItem("아메리카노", "", 3500);
        var latte = new MenuItem("카페라떼", "부드러운 라떼", 4000);
        var greenTea = new MenuItem("녹차", "제주 녹차", 3500,
                                     isVegetarian: true);

        var steak = new MenuItem("안심 스테이크", "호주산", 25000);
        var pasta = new MenuItem("크림 파스타", "", 13000);

        // Composite 구성
        var coffeeMenu = new Menu("커피", "다양한 커피")
            .Add(espresso).Add(americano).Add(latte);

        var teaMenu = new Menu("차")
            .Add(greenTea);

        var drinkMenu = new Menu("음료", "모든 음료")
            .Add(coffeeMenu).Add(teaMenu);

        var foodMenu = new Menu("식사")
            .Add(steak).Add(pasta);

        var allMenu = new Menu("전체 메뉴")
            .Add(drinkMenu).Add(foodMenu);

        // 동일한 인터페이스로 사용
        Console.WriteLine("=== 전체 메뉴 ===");
        allMenu.Display();

        Console.WriteLine($"\n전체 합계: {allMenu.GetPrice():N0}원");
        Console.WriteLine($"음료 합계: {drinkMenu.GetPrice():N0}원");
        Console.WriteLine($"에스프레소: {espresso.GetPrice():N0}원");

        // 검색
        Console.WriteLine("\n=== '커피' 검색 ===");
        var results = allMenu.Search("커피");
        foreach (var result in results)
            result.Display(indent: 1);

        // 이터레이터로 채식 메뉴 필터
        Console.WriteLine("\n=== 채식 메뉴 ===");
        foreach (var item in allMenu)
        {
            if (item is MenuItem menuItem && menuItem.IsVegetarian)
                menuItem.Display(indent: 1);
        }
    }
}
```

---

## 10. 실무 예제: 파일 시스템

### Python - 파일 시스템 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime
from dataclasses import dataclass, field


@dataclass
class FileSystemInfo:
    """파일 시스템 항목 메타데이터"""
    created_at: datetime = field(default_factory=datetime.now)
    modified_at: datetime = field(default_factory=datetime.now)
    owner: str = "user"
    permissions: str = "rw-r--r--"


class FileSystemComponent(ABC):
    """Component: 파일 시스템 항목 공통 인터페이스"""

    def __init__(self, name: str):
        self._name = name
        self._info = FileSystemInfo()

    @property
    def name(self) -> str:
        return self._name

    @property
    def info(self) -> FileSystemInfo:
        return self._info

    @abstractmethod
    def get_size(self) -> int:
        """크기(바이트) 반환"""
        pass

    @abstractmethod
    def display(self, indent: int = 0) -> None:
        """트리 구조로 출력"""
        pass

    @abstractmethod
    def find(self, name: str) -> list[FileSystemComponent]:
        """이름으로 검색"""
        pass

    def get_path(self, parent_path: str = "") -> str:
        return f"{parent_path}/{self._name}"


class File(FileSystemComponent):
    """Leaf: 파일"""

    def __init__(self, name: str, size: int, content: str = ""):
        super().__init__(name)
        self._size = size
        self._content = content

    def get_size(self) -> int:
        return self._size

    def display(self, indent: int = 0) -> None:
        prefix = "│   " * (indent - 1) + "├── " if indent > 0 else ""
        size_str = self._format_size(self._size)
        print(f"{prefix}{self._name} ({size_str})")

    def find(self, name: str) -> list[FileSystemComponent]:
        if name.lower() in self._name.lower():
            return [self]
        return []

    @staticmethod
    def _format_size(size_bytes: int) -> str:
        for unit in ['B', 'KB', 'MB', 'GB']:
            if size_bytes < 1024:
                return f"{size_bytes:.1f}{unit}"
            size_bytes /= 1024
        return f"{size_bytes:.1f}TB"


class Directory(FileSystemComponent):
    """Composite: 디렉토리"""

    def __init__(self, name: str):
        super().__init__(name)
        self._children: list[FileSystemComponent] = []

    def add(self, component: FileSystemComponent) -> Directory:
        self._children.append(component)
        return self

    def remove(self, component: FileSystemComponent) -> None:
        self._children.remove(component)

    def get_size(self) -> int:
        """하위 모든 항목의 크기 합산"""
        return sum(child.get_size() for child in self._children)

    def display(self, indent: int = 0) -> None:
        prefix = "│   " * (indent - 1) + "├── " if indent > 0 else ""
        size_str = File._format_size(self.get_size())
        count = self._count_files()
        print(f"{prefix}{self._name}/ ({size_str}, {count}개 파일)")
        for child in self._children:
            child.display(indent + 1)

    def find(self, name: str) -> list[FileSystemComponent]:
        results = []
        if name.lower() in self._name.lower():
            results.append(self)
        for child in self._children:
            results.extend(child.find(name))
        return results

    def _count_files(self) -> int:
        """하위 파일 개수 (재귀)"""
        count = 0
        for child in self._children:
            if isinstance(child, File):
                count += 1
            elif isinstance(child, Directory):
                count += child._count_files()
        return count

    def list_files(self) -> list[File]:
        """하위 모든 파일 목록 (재귀)"""
        files = []
        for child in self._children:
            if isinstance(child, File):
                files.append(child)
            elif isinstance(child, Directory):
                files.extend(child.list_files())
        return files


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 파일 시스템 구성
    root = Directory("project")

    # src 디렉토리
    src = Directory("src")
    src.add(File("main.py", 2048, "# main"))
    src.add(File("utils.py", 1024, "# utils"))
    src.add(File("config.py", 512, "# config"))

    # src/models 디렉토리
    models = Directory("models")
    models.add(File("user.py", 1536))
    models.add(File("order.py", 2560))
    src.add(models)

    # tests 디렉토리
    tests = Directory("tests")
    tests.add(File("test_main.py", 3072))
    tests.add(File("test_utils.py", 1024))

    # docs 디렉토리
    docs = Directory("docs")
    docs.add(File("README.md", 4096))
    docs.add(File("API.md", 8192))

    # 루트 구성
    root.add(src).add(tests).add(docs)
    root.add(File(".gitignore", 256))
    root.add(File("requirements.txt", 128))

    # 트리 출력 (동일한 인터페이스)
    print("=== 프로젝트 구조 ===")
    root.display()

    # 크기 확인 (동일한 인터페이스)
    print(f"\n총 크기: {File._format_size(root.get_size())}")
    print(f"src 크기: {File._format_size(src.get_size())}")

    # 검색 (동일한 인터페이스)
    print("\n=== 'test' 검색 ===")
    results = root.find("test")
    for item in results:
        item.display(indent=1)
```

### C# - 파일 시스템

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public abstract class FileSystemComponent
{
    public string Name { get; }
    public DateTime CreatedAt { get; } = DateTime.Now;

    protected FileSystemComponent(string name) => Name = name;

    public abstract long GetSize();
    public abstract void Display(int indent = 0);
    public abstract List<FileSystemComponent> Find(string name);

    protected string FormatSize(long bytes)
    {
        string[] units = { "B", "KB", "MB", "GB", "TB" };
        double size = bytes;
        int unitIndex = 0;
        while (size >= 1024 && unitIndex < units.Length - 1)
        {
            size /= 1024;
            unitIndex++;
        }
        return $"{size:F1}{units[unitIndex]}";
    }
}

public class FileNode : FileSystemComponent
{
    private readonly long _size;

    public FileNode(string name, long size) : base(name)
    {
        _size = size;
    }

    public override long GetSize() => _size;

    public override void Display(int indent = 0)
    {
        var prefix = indent > 0
            ? new string(' ', (indent - 1) * 4) + "+-- "
            : "";
        Console.WriteLine(
            $"{prefix}{Name} ({FormatSize(_size)})");
    }

    public override List<FileSystemComponent> Find(string name)
    {
        return Name.Contains(name, StringComparison.OrdinalIgnoreCase)
            ? new List<FileSystemComponent> { this }
            : new List<FileSystemComponent>();
    }
}

public class DirectoryNode : FileSystemComponent
{
    private readonly List<FileSystemComponent> _children = new();

    public DirectoryNode(string name) : base(name) { }

    public DirectoryNode Add(FileSystemComponent component)
    {
        _children.Add(component);
        return this;
    }

    public void Remove(FileSystemComponent component)
    {
        _children.Remove(component);
    }

    public override long GetSize()
    {
        return _children.Sum(c => c.GetSize());
    }

    public override void Display(int indent = 0)
    {
        var prefix = indent > 0
            ? new string(' ', (indent - 1) * 4) + "+-- "
            : "";
        Console.WriteLine(
            $"{prefix}{Name}/ ({FormatSize(GetSize())})");

        foreach (var child in _children)
            child.Display(indent + 1);
    }

    public override List<FileSystemComponent> Find(string name)
    {
        var results = new List<FileSystemComponent>();
        if (Name.Contains(name, StringComparison.OrdinalIgnoreCase))
            results.Add(this);

        foreach (var child in _children)
            results.AddRange(child.Find(name));

        return results;
    }
}

public class Program
{
    public static void Main()
    {
        var root = new DirectoryNode("project");

        var src = new DirectoryNode("src")
            .Add(new FileNode("main.py", 2048))
            .Add(new FileNode("utils.py", 1024));

        var tests = new DirectoryNode("tests")
            .Add(new FileNode("test_main.py", 3072));

        root.Add(src).Add(tests)
            .Add(new FileNode(".gitignore", 256));

        Console.WriteLine("=== 프로젝트 구조 ===");
        root.Display();

        Console.WriteLine($"\n총 크기: {root.GetSize():N0} bytes");
    }
}
```

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **일관된 인터페이스** | 개별/복합 객체를 동일하게 다룸 |
| **쉬운 확장** | 새로운 Leaf나 Composite 추가 용이 |
| **재귀적 처리** | 트리 구조를 자연스럽게 순회 |
| **클라이언트 단순화** | 타입 체크 없이 깔끔한 코드 |

### 단점

| 단점 | 설명 |
|------|------|
| **과도한 일반화** | 공통 인터페이스가 너무 넓어질 수 있음 |
| **타입 제약 어려움** | 특정 타입만 추가 가능하게 제한하기 어려움 |
| **디버깅 복잡** | 깊은 트리에서 재귀 호출 디버깅이 어려움 |

---

## 12. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Iterator** | Composite의 트리를 순회하는 데 사용 |
| **Visitor** | Composite 트리에 새로운 연산을 추가할 때 사용 |
| **Decorator** | Decorator가 Composite의 Component를 감쌀 수 있음 |
| **Chain of Responsibility** | Leaf가 요청을 부모 Composite로 위임 |

---

## 13. 정리 및 체크리스트

### 핵심 정리

```
Composite 패턴 = "부분과 전체를 동일하게"

When: 트리 구조가 있고, 개별/복합 객체를 동일하게 다루어야 할 때
How:  공통 인터페이스(Component)를 정의하고,
      Leaf는 실제 작업을, Composite는 하위 항목에 위임
Why:  클라이언트가 타입을 신경 쓰지 않고 일관된 코드 작성
```

### 적용 체크리스트

- [ ] 트리(부분-전체) 구조가 존재하는가?
- [ ] 개별 객체와 복합 객체를 동일하게 다루어야 하는가?
- [ ] 재귀적 구조를 가지고 있는가? (Composite가 Component를 포함)
- [ ] 투명성 vs 안전성 중 어떤 방식을 사용할지 결정했는가?
- [ ] 순환 참조 방지 처리를 했는가? (A가 B를 포함하고 B가 A를 포함하는 경우)

### 실무 적용 시나리오

1. **파일 시스템:** 파일과 디렉토리
2. **UI 컴포넌트:** 버튼, 텍스트(Leaf)와 패널, 윈도우(Composite)
3. **조직도:** 직원(Leaf)과 부서(Composite)
4. **메뉴 시스템:** 메뉴 항목과 서브 메뉴
5. **수학 표현식:** 숫자(Leaf)와 연산(Composite)

> **기억하세요:** "트리 구조에서 '이것이 단일인지 복합인지' 묻지 않아도 되게 만드는 것이 Composite의 핵심입니다."
