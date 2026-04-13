# Visitor 패턴 (방문자)

> **핵심 의도 한줄 요약**: 객체 구조를 변경하지 않고 새로운 연산(기능)을 추가할 수 있게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Double Dispatch 설명](#5-double-dispatch-설명)
6. [Python 구현](#6-python-구현)
7. [C# 구현](#7-c-구현)
8. [장단점 및 트레이드오프](#8-장단점-및-트레이드오프)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 개요

Visitor 패턴은 **데이터 구조(Element)와 연산(Visitor)을 분리**하는 패턴이다. 새로운 연산을 추가할 때 기존 Element 클래스를 수정하지 않고, 새로운 Visitor 클래스만 추가하면 된다.

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **Visitor** | 각 Element 타입에 대한 연산(visit 메서드)을 정의 |
| **ConcreteVisitor** | 구체적인 연산을 구현 |
| **Element** | Visitor를 받아들이는 accept 메서드 정의 |
| **ConcreteElement** | accept에서 해당 Visitor의 visit 메서드 호출 |
| **ObjectStructure** | Element들의 컬렉션 |

---

## 2. 문제 상황

### 문제: 기존 클래스에 새 연산 추가

도형 시스템에서 다양한 연산을 추가해야 한다.

```python
# 나쁜 예: 연산 추가마다 모든 도형 클래스를 수정
class Circle:
    def __init__(self, radius):
        self.radius = radius

    def area(self):           # 처음 구현
        return 3.14 * self.radius ** 2

    def perimeter(self):      # 나중에 추가 → 클래스 수정
        return 2 * 3.14 * self.radius

    def to_json(self):        # 또 추가 → 클래스 수정
        return {"type": "circle", "radius": self.radius}

    def to_svg(self):         # 또 추가 → 클래스 수정
        return f'<circle r="{self.radius}"/>'

    # 새 연산 추가마다 Circle, Rectangle, Triangle 등
    # 모든 도형 클래스를 수정해야 한다!
```

문제점:

- **OCP 위반**: 새 연산 추가 시 기존 클래스를 수정해야 한다
- **SRP 위반**: 도형 클래스가 면적 계산, 직렬화, 렌더링 등 다양한 책임을 갖는다
- **관련 코드 분산**: 같은 연산(예: to_json)의 로직이 여러 클래스에 흩어져 있다

---

## 3. 일상 비유

**세무사 방문**을 떠올려 보자.

```
[회사 구조] (Element - 변경 없음)
  ├── 영업부
  ├── 개발부
  └── 인사부

[세무사 방문] (Visitor 1 - 새 연산)
  → 영업부 방문: 매출 세금 계산
  → 개발부 방문: R&D 세액 공제
  → 인사부 방문: 급여 원천징수

[감사관 방문] (Visitor 2 - 또 다른 새 연산)
  → 영업부 방문: 매출 장부 감사
  → 개발부 방문: 프로젝트 비용 감사
  → 인사부 방문: 인건비 감사
```

- 회사 구조(Element)는 **변경되지 않는다**
- 새로운 방문자(Visitor)를 추가하면 **새로운 기능이 추가**된다
- 각 부서는 방문자를 **받아들이기(accept)**만 하면 된다

---

## 4. UML 다이어그램

```
 ┌────────────────────┐         ┌──────────────────────┐
 │  <<interface>>     │         │   <<interface>>      │
 │    Visitor         │         │     Element           │
 ├────────────────────┤         ├──────────────────────┤
 │+visit(ElementA)    │<────────│+accept(visitor)      │
 │+visit(ElementB)    │         └──────────┬───────────┘
 │+visit(ElementC)    │                    │
 └────────┬───────────┘         ┌──────────┼──────────┐
          │                     │          │          │
 ┌────────┼──────────┐  ┌──────▼──┐ ┌─────▼───┐ ┌───▼───────┐
 │        │          │  │ElementA │ │ElementB │ │ElementC   │
 │        │          │  ├─────────┤ ├─────────┤ ├───────────┤
 ┌────────▼──┐ ┌─────▼───┐ │+accept()│ │+accept()│ │+accept()  │
 │VisitorX   │ │VisitorY │ └─────────┘ └─────────┘ └───────────┘
 ├───────────┤ ├─────────┤
 │+visit(A)  │ │+visit(A)│  accept(visitor):
 │+visit(B)  │ │+visit(B)│    visitor.visit(this)  ← Double Dispatch
 │+visit(C)  │ │+visit(C)│
 └───────────┘ └─────────┘
```

---

## 5. Double Dispatch 설명

Visitor 패턴의 핵심 기법은 **Double Dispatch(이중 디스패치)**다.

### 일반 메서드 호출 (Single Dispatch)

```python
shape.draw()  # shape의 실제 타입에 따라 메서드가 결정됨 (1번 디스패치)
```

### Double Dispatch

```python
shape.accept(visitor)        # 1차 디스패치: shape의 타입으로 accept 결정
# accept 내부에서:
visitor.visit(self)          # 2차 디스패치: visitor의 타입으로 visit 결정
```

**두 번의 동적 바인딩**을 통해, Element의 타입과 Visitor의 타입 **둘 다**에 따라 실행될 메서드가 결정된다.

```
                  Visitor 타입
                  ┌──────────┬───────────┐
                  │AreaVisit │ JsonVisit  │
 Element   Circle │ 원 면적  │ 원 JSON   │
 타입   Rectangle │ 사각 면적│ 사각 JSON │
        Triangle  │ 삼각 면적│ 삼각 JSON │
                  └──────────┴───────────┘
```

---

## 6. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from math import pi, sqrt


# === Visitor 인터페이스 ===
class ShapeVisitor(ABC):
    """도형 방문자 인터페이스"""

    @abstractmethod
    def visit_circle(self, circle: Circle) -> str:
        pass

    @abstractmethod
    def visit_rectangle(self, rectangle: Rectangle) -> str:
        pass

    @abstractmethod
    def visit_triangle(self, triangle: Triangle) -> str:
        pass


# === Element 인터페이스 및 구현 ===
class Shape(ABC):
    """도형 인터페이스"""

    @abstractmethod
    def accept(self, visitor: ShapeVisitor) -> str:
        pass


class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def accept(self, visitor: ShapeVisitor) -> str:
        return visitor.visit_circle(self)  # Double Dispatch


class Rectangle(Shape):
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def accept(self, visitor: ShapeVisitor) -> str:
        return visitor.visit_rectangle(self)


class Triangle(Shape):
    def __init__(self, a: float, b: float, c: float) -> None:
        self.a = a
        self.b = b
        self.c = c

    def accept(self, visitor: ShapeVisitor) -> str:
        return visitor.visit_triangle(self)


# === Concrete Visitors ===
class AreaCalculator(ShapeVisitor):
    """면적 계산 방문자"""

    def visit_circle(self, circle: Circle) -> str:
        area = pi * circle.radius ** 2
        return f"원(r={circle.radius}) 면적: {area:.2f}"

    def visit_rectangle(self, rectangle: Rectangle) -> str:
        area = rectangle.width * rectangle.height
        return f"사각형({rectangle.width}x{rectangle.height}) 면적: {area:.2f}"

    def visit_triangle(self, triangle: Triangle) -> str:
        # 헤론의 공식
        s = (triangle.a + triangle.b + triangle.c) / 2
        area = sqrt(s * (s - triangle.a) * (s - triangle.b) * (s - triangle.c))
        return f"삼각형({triangle.a},{triangle.b},{triangle.c}) 면적: {area:.2f}"


class PerimeterCalculator(ShapeVisitor):
    """둘레 계산 방문자"""

    def visit_circle(self, circle: Circle) -> str:
        perimeter = 2 * pi * circle.radius
        return f"원(r={circle.radius}) 둘레: {perimeter:.2f}"

    def visit_rectangle(self, rectangle: Rectangle) -> str:
        perimeter = 2 * (rectangle.width + rectangle.height)
        return f"사각형({rectangle.width}x{rectangle.height}) 둘레: {perimeter:.2f}"

    def visit_triangle(self, triangle: Triangle) -> str:
        perimeter = triangle.a + triangle.b + triangle.c
        return f"삼각형({triangle.a},{triangle.b},{triangle.c}) 둘레: {perimeter:.2f}"


class JsonExporter(ShapeVisitor):
    """JSON 변환 방문자"""

    def visit_circle(self, circle: Circle) -> str:
        return f'{{"type":"circle","radius":{circle.radius}}}'

    def visit_rectangle(self, rectangle: Rectangle) -> str:
        return f'{{"type":"rectangle","width":{rectangle.width},"height":{rectangle.height}}}'

    def visit_triangle(self, triangle: Triangle) -> str:
        return f'{{"type":"triangle","sides":[{triangle.a},{triangle.b},{triangle.c}]}}'


class SvgRenderer(ShapeVisitor):
    """SVG 렌더링 방문자"""

    def visit_circle(self, circle: Circle) -> str:
        return f'<circle cx="50" cy="50" r="{circle.radius}" fill="blue"/>'

    def visit_rectangle(self, rectangle: Rectangle) -> str:
        return (f'<rect width="{rectangle.width}" height="{rectangle.height}" '
                f'fill="green"/>')

    def visit_triangle(self, triangle: Triangle) -> str:
        return f'<polygon points="0,{triangle.a} {triangle.b},0 {triangle.c},{triangle.a}" fill="red"/>'


# === 사용 예시 ===
if __name__ == "__main__":
    # 도형 컬렉션 (Object Structure)
    shapes: list[Shape] = [
        Circle(5),
        Rectangle(4, 6),
        Triangle(3, 4, 5),
    ]

    # 새 연산 = 새 Visitor (기존 도형 클래스 수정 없음!)
    visitors = [
        ("면적 계산", AreaCalculator()),
        ("둘레 계산", PerimeterCalculator()),
        ("JSON 변환", JsonExporter()),
        ("SVG 렌더링", SvgRenderer()),
    ]

    for name, visitor in visitors:
        print(f"\n=== {name} ===")
        for shape in shapes:
            result = shape.accept(visitor)
            print(f"  {result}")
```

**출력:**

```
=== 면적 계산 ===
  원(r=5) 면적: 78.54
  사각형(4x6) 면적: 24.00
  삼각형(3,4,5) 면적: 6.00

=== 둘레 계산 ===
  원(r=5) 둘레: 31.42
  사각형(4x6) 둘레: 20.00
  삼각형(3,4,5) 둘레: 12.00

=== JSON 변환 ===
  {"type":"circle","radius":5}
  {"type":"rectangle","width":4,"height":6}
  {"type":"triangle","sides":[3,4,5]}

=== SVG 렌더링 ===
  <circle cx="50" cy="50" r="5" fill="blue"/>
  <rect width="4" height="6" fill="green"/>
  <polygon points="0,3 4,0 5,3" fill="red"/>
```

---

## 7. C# 구현

```csharp
using System;
using System.Collections.Generic;

// Visitor 인터페이스
public interface IShapeVisitor
{
    string VisitCircle(Circle circle);
    string VisitRectangle(Rectangle rectangle);
    string VisitTriangle(Triangle triangle);
}

// Element 인터페이스
public interface IShape
{
    string Accept(IShapeVisitor visitor);
}

// Concrete Elements
public class Circle : IShape
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public string Accept(IShapeVisitor visitor) => visitor.VisitCircle(this);
}

public class Rectangle : IShape
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public string Accept(IShapeVisitor visitor) => visitor.VisitRectangle(this);
}

public class Triangle : IShape
{
    public double A { get; }
    public double B { get; }
    public double C { get; }

    public Triangle(double a, double b, double c)
    {
        A = a; B = b; C = c;
    }

    public string Accept(IShapeVisitor visitor) => visitor.VisitTriangle(this);
}

// Concrete Visitors
public class AreaCalculator : IShapeVisitor
{
    public string VisitCircle(Circle c)
    {
        double area = Math.PI * c.Radius * c.Radius;
        return $"원(r={c.Radius}) 면적: {area:F2}";
    }

    public string VisitRectangle(Rectangle r)
    {
        double area = r.Width * r.Height;
        return $"사각형({r.Width}x{r.Height}) 면적: {area:F2}";
    }

    public string VisitTriangle(Triangle t)
    {
        double s = (t.A + t.B + t.C) / 2;
        double area = Math.Sqrt(s * (s - t.A) * (s - t.B) * (s - t.C));
        return $"삼각형({t.A},{t.B},{t.C}) 면적: {area:F2}";
    }
}

public class JsonExporter : IShapeVisitor
{
    public string VisitCircle(Circle c)
        => $"{{\"type\":\"circle\",\"radius\":{c.Radius}}}";

    public string VisitRectangle(Rectangle r)
        => $"{{\"type\":\"rectangle\",\"width\":{r.Width},\"height\":{r.Height}}}";

    public string VisitTriangle(Triangle t)
        => $"{{\"type\":\"triangle\",\"sides\":[{t.A},{t.B},{t.C}]}}";
}

public class PerimeterCalculator : IShapeVisitor
{
    public string VisitCircle(Circle c)
    {
        double p = 2 * Math.PI * c.Radius;
        return $"원(r={c.Radius}) 둘레: {p:F2}";
    }

    public string VisitRectangle(Rectangle r)
    {
        double p = 2 * (r.Width + r.Height);
        return $"사각형({r.Width}x{r.Height}) 둘레: {p:F2}";
    }

    public string VisitTriangle(Triangle t)
    {
        double p = t.A + t.B + t.C;
        return $"삼각형({t.A},{t.B},{t.C}) 둘레: {p:F2}";
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var shapes = new List<IShape>
        {
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4, 5)
        };

        var visitors = new (string Name, IShapeVisitor Visitor)[]
        {
            ("면적 계산", new AreaCalculator()),
            ("둘레 계산", new PerimeterCalculator()),
            ("JSON 변환", new JsonExporter()),
        };

        foreach (var (name, visitor) in visitors)
        {
            Console.WriteLine($"\n=== {name} ===");
            foreach (var shape in shapes)
            {
                Console.WriteLine($"  {shape.Accept(visitor)}");
            }
        }
    }
}
```

---

## 8. 장단점 및 트레이드오프

### 장점

| 장점 | 설명 |
|------|------|
| **새 연산 추가 쉬움** | Visitor만 추가하면 됨, 기존 Element 수정 불필요 |
| **관련 코드 집중** | 같은 연산의 모든 로직이 하나의 Visitor에 모임 |
| **단일 책임 원칙** | 연산과 데이터 구조를 분리 |
| **상태 축적** | Visitor가 순회하면서 정보를 축적할 수 있음 |

### 단점

| 단점 | 설명 |
|------|------|
| **새 Element 추가 어려움** | 새 Element 추가 시 모든 Visitor에 visit 메서드 추가 필요 |
| **캡슐화 약화** | Visitor가 Element의 내부 상태에 접근해야 할 수 있다 |
| **복잡성** | Double Dispatch가 직관적이지 않아 이해하기 어렵다 |
| **결합도** | Visitor가 모든 Element 타입을 알아야 한다 |

### 트레이드오프: 연산 추가 vs 타입 추가

```
              새 연산 추가 쉬움    새 타입 추가 쉬움
 Visitor           O                   X
 다형성            X                   O
```

- **Element 타입이 안정적**이고 **연산이 자주 추가**되면 → Visitor
- **연산이 안정적**이고 **타입이 자주 추가**되면 → 일반 다형성

이를 **Expression Problem**이라고 부른다.

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Composite** | 트리 구조를 Visitor로 순회하면서 연산 수행 |
| **Iterator** | 컬렉션을 순회하면서 각 요소에 Visitor 적용 |
| **Strategy** | 단일 알고리즘 교체 vs 여러 타입에 대한 연산 분리 |
| **Interpreter** | AST를 Visitor로 처리하는 것이 일반적 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

1. Visitor는 **데이터 구조와 연산을 분리**한다.
2. **Double Dispatch**를 통해 Element 타입과 Visitor 타입 모두에 따라 동작을 결정한다.
3. **새 연산 추가가 쉽지만, 새 타입 추가는 어렵다** (Expression Problem).
4. 실무에서 **AST 처리, 문서 변환, 직렬화, 보고서 생성** 등에 사용된다.

### 적용 체크리스트

- [ ] 데이터 구조(Element)가 **안정적**이고 자주 변경되지 않는가?
- [ ] **새로운 연산**이 자주 추가될 예정인가?
- [ ] 관련 연산의 코드를 **한 곳에 모아** 관리하고 싶은가?
- [ ] **여러 타입의 객체**에 대해 타입별로 다른 처리가 필요한가?
- [ ] `isinstance` 체크나 타입 캐스팅이 빈번한가?

> **2~3년차 개발자를 위한 팁**: 컴파일러/인터프리터의 AST(Abstract Syntax Tree) 처리가 Visitor 패턴의 가장 대표적인 사용처다. Python의 `ast.NodeVisitor`, Java의 `ElementVisitor`, Roslyn(C# 컴파일러)의 `CSharpSyntaxVisitor`가 실무 사례다. 실무에서 자주 만나는 패턴은 아니지만, 적용 가능한 상황에서는 코드 구조를 극적으로 개선한다.
