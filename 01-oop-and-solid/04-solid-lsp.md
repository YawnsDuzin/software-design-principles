# L - 리스코프 치환 원칙 (Liskov Substitution Principle)

> **SOLID 원칙 3/5**
> **대상 독자**: 2~3년차 개발자
> **핵심 문장**: "하위 타입(Subtype)은 그것의 상위 타입(Base type)으로 치환할 수 있어야 한다." — Barbara Liskov, 1987

---

## 목차

1. [LSP란 무엇인가?](#1-lsp란-무엇인가)
2. [왜 LSP가 중요한가?](#2-왜-lsp가-중요한가)
3. [LSP 위반 사례 (Before)](#3-lsp-위반-사례-before)
4. [LSP 적용 사례 (After)](#4-lsp-적용-사례-after)
5. [LSP 위반 징후 및 해결 방법](#5-lsp-위반-징후-및-해결-방법)
6. [정리 및 체크리스트](#6-정리-및-체크리스트)
7. [다음 단계](#7-다음-단계)

---

## 1. LSP란 무엇인가?

### 정의

> **"S가 T의 하위 타입이라면, 프로그램에서 T 타입의 객체를 S 타입의 객체로 치환해도 프로그램의 정확성이 깨지지 않아야 한다."**

쉽게 말하면:

```
부모 클래스가 사용되는 곳에 자식 클래스를 넣어도
아무 문제 없이 동작해야 한다.
```

### 비유

택배 기사에게 "차량"을 제공한다고 생각해 보세요:

- **트럭**을 주면? 짐을 싣고 배달합니다. OK!
- **승합차**를 주면? 짐을 싣고 배달합니다. OK!
- **장난감 자동차**를 주면? 짐도 못 싣고, 주행도 못 합니다. 위반!

장난감 자동차는 "자동차"라는 이름을 갖고 있지만, **실제 자동차가 할 수 있는 일을 할 수 없습니다**. 이것이 LSP 위반입니다.

### 핵심 규칙

LSP를 지키려면 하위 타입이 다음을 만족해야 합니다:

| 규칙 | 설명 | 예시 |
|------|------|------|
| **사전조건 강화 금지** | 부모보다 더 까다로운 입력 조건 불가 | 부모가 0 이상 허용 → 자식이 양수만 허용 (위반!) |
| **사후조건 약화 금지** | 부모가 보장하는 결과를 자식도 보장 | 부모가 양수 반환 → 자식이 음수 반환 (위반!) |
| **불변조건 유지** | 부모의 불변 속성을 자식도 유지 | 부모의 width와 height가 독립 → 자식도 독립 |

---

## 2. 왜 LSP가 중요한가?

### LSP를 지키면

- **다형성이 안전하게 동작**: 부모 타입으로 코딩해도 자식 타입으로 교체 가능
- **OCP와 시너지**: 새 하위 타입을 안전하게 확장 가능
- **코드 신뢰성 향상**: 타입 계층 구조를 믿고 사용할 수 있음

### LSP를 무시하면

- **예상치 못한 런타임 오류**: 자식 클래스가 부모의 계약을 깨뜨림
- **instanceof/타입 체크 남발**: "혹시 이 타입이면..." 분기가 발생
- **다형성 파괴**: 부모 타입으로 코딩하는 의미가 사라짐

---

## 3. LSP 위반 사례 (Before)

### 사례 1: 직사각형-정사각형 문제 (가장 유명한 LSP 위반)

수학적으로 "정사각형은 직사각형이다(is-a)"는 맞지만, **코드에서는** 그렇지 않을 수 있습니다.

### ❌ Python - 나쁜 예: Rectangle-Square

```python
class Rectangle:
    """직사각형"""

    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height

    @property
    def width(self) -> float:
        return self._width

    @width.setter
    def width(self, value: float) -> None:
        self._width = value

    @property
    def height(self) -> float:
        return self._height

    @height.setter
    def height(self, value: float) -> None:
        self._height = value

    def area(self) -> float:
        return self._width * self._height


class Square(Rectangle):
    """
    ❌ LSP 위반!
    정사각형은 width와 height가 항상 같아야 하므로,
    하나를 변경하면 다른 하나도 변경된다.
    → 직사각형의 "width와 height는 독립적이다"라는 불변조건을 깨뜨린다.
    """

    def __init__(self, size: float):
        super().__init__(size, size)

    @Rectangle.width.setter
    def width(self, value: float) -> None:
        self._width = value
        self._height = value  # ❌ height도 함께 변경!

    @Rectangle.height.setter
    def height(self, value: float) -> None:
        self._width = value   # ❌ width도 함께 변경!
        self._height = value


def test_rectangle_area(rect: Rectangle) -> None:
    """
    직사각형의 넓이를 테스트하는 함수.
    Rectangle의 계약: width와 height를 각각 설정하면 area = width * height
    """
    rect.width = 5
    rect.height = 4
    expected_area = 5 * 4  # 20

    actual_area = rect.area()
    assert actual_area == expected_area, \
        f"기대값: {expected_area}, 실제값: {actual_area}"
    print(f"테스트 통과! 넓이 = {actual_area}")


# Rectangle로 테스트 → 통과!
test_rectangle_area(Rectangle(0, 0))

# Square로 테스트 → 실패!
# Square에서 height=4로 설정하면 width도 4가 되어 area=16
try:
    test_rectangle_area(Square(0))  # AssertionError! 기대값: 20, 실제값: 16
except AssertionError as e:
    print(f"❌ LSP 위반! {e}")
```

### ❌ C# - 나쁜 예: Rectangle-Square

```csharp
public class Rectangle
{
    public virtual double Width { get; set; }
    public virtual double Height { get; set; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public double Area() => Width * Height;
}

public class Square : Rectangle
{
    // ❌ LSP 위반! setter를 오버라이드하여 불변조건 변경
    public override double Width
    {
        get => base.Width;
        set
        {
            base.Width = value;
            base.Height = value;  // height도 변경!
        }
    }

    public override double Height
    {
        get => base.Height;
        set
        {
            base.Width = value;   // width도 변경!
            base.Height = value;
        }
    }

    public Square(double size) : base(size, size) { }
}

// 테스트
void TestRectangleArea(Rectangle rect)
{
    rect.Width = 5;
    rect.Height = 4;
    double expected = 20;  // 5 * 4

    double actual = rect.Area();
    Debug.Assert(actual == expected,
        $"기대값: {expected}, 실제값: {actual}");
}

TestRectangleArea(new Rectangle(0, 0));  // 통과!
TestRectangleArea(new Square(0));        // ❌ 실패! actual = 16
```

### 사례 2: 읽기 전용 컬렉션에 쓰기

```python
class FileStorage:
    """파일 저장소"""

    def save(self, filename: str, data: str) -> None:
        print(f"'{filename}' 저장 완료")

    def delete(self, filename: str) -> None:
        print(f"'{filename}' 삭제 완료")


class ReadOnlyFileStorage(FileStorage):
    """
    ❌ LSP 위반!
    부모가 제공하는 delete 기능을 예외로 막는다.
    FileStorage를 기대하는 코드에서 갑자기 예외가 발생한다.
    """

    def delete(self, filename: str) -> None:
        raise PermissionError("읽기 전용 저장소입니다. 삭제할 수 없습니다.")


def cleanup_old_files(storage: FileStorage, filenames: list[str]) -> None:
    """FileStorage를 기대하는 함수"""
    for name in filenames:
        storage.delete(name)  # ReadOnlyFileStorage이면 폭발!
```

---

## 4. LSP 적용 사례 (After)

### ✅ Python - 좋은 예: 올바른 도형 설계

```python
from abc import ABC, abstractmethod
import math


# ──────────────────────────────────────────────
# 해결 1: 공통 추상화로 분리
# ──────────────────────────────────────────────
class Shape(ABC):
    """도형 추상 클래스 - 모든 도형의 공통 인터페이스"""

    @abstractmethod
    def area(self) -> float:
        pass

    @abstractmethod
    def perimeter(self) -> float:
        pass


class Rectangle(Shape):
    """직사각형 - width와 height가 독립적"""

    def __init__(self, width: float, height: float):
        if width <= 0 or height <= 0:
            raise ValueError("너비와 높이는 양수여야 합니다.")
        self._width = width
        self._height = height

    @property
    def width(self) -> float:
        return self._width

    @property
    def height(self) -> float:
        return self._height

    def area(self) -> float:
        return self._width * self._height

    def perimeter(self) -> float:
        return 2 * (self._width + self._height)

    def with_width(self, new_width: float) -> "Rectangle":
        """불변 객체 패턴 - 새 객체 반환"""
        return Rectangle(new_width, self._height)

    def with_height(self, new_height: float) -> "Rectangle":
        return Rectangle(self._width, new_height)


class Square(Shape):
    """
    ✅ LSP 준수!
    Rectangle을 상속하지 않고 Shape를 직접 구현.
    정사각형은 정사각형만의 불변조건(한 변의 길이)을 유지.
    """

    def __init__(self, side: float):
        if side <= 0:
            raise ValueError("변의 길이는 양수여야 합니다.")
        self._side = side

    @property
    def side(self) -> float:
        return self._side

    def area(self) -> float:
        return self._side ** 2

    def perimeter(self) -> float:
        return 4 * self._side

    def with_side(self, new_side: float) -> "Square":
        return Square(new_side)


class Circle(Shape):
    def __init__(self, radius: float):
        if radius <= 0:
            raise ValueError("반지름은 양수여야 합니다.")
        self._radius = radius

    @property
    def radius(self) -> float:
        return self._radius

    def area(self) -> float:
        return math.pi * self._radius ** 2

    def perimeter(self) -> float:
        return 2 * math.pi * self._radius


# ──────────────────────────────────────────────
# 모든 Shape는 안전하게 치환 가능!
# ──────────────────────────────────────────────
def print_shape_info(shape: Shape) -> None:
    """어떤 Shape든 안전하게 사용 가능 - LSP 준수!"""
    print(f"넓이: {shape.area():.2f}")
    print(f"둘레: {shape.perimeter():.2f}")


shapes: list[Shape] = [
    Rectangle(5, 4),
    Square(5),
    Circle(3),
]

for shape in shapes:
    print_shape_info(shape)  # 모든 도형에서 안전하게 동작
    print()
```

### ✅ Python - 좋은 예: 저장소 인터페이스 분리

```python
from abc import ABC, abstractmethod


# 읽기/쓰기 능력을 분리하여 LSP 준수
class Readable(ABC):
    """읽기 전용 인터페이스"""

    @abstractmethod
    def read(self, filename: str) -> str:
        pass

    @abstractmethod
    def list_files(self) -> list[str]:
        pass


class Writable(ABC):
    """쓰기 인터페이스"""

    @abstractmethod
    def save(self, filename: str, data: str) -> None:
        pass


class Deletable(ABC):
    """삭제 인터페이스"""

    @abstractmethod
    def delete(self, filename: str) -> None:
        pass


class FullFileStorage(Readable, Writable, Deletable):
    """읽기/쓰기/삭제 모두 가능한 저장소"""

    def read(self, filename: str) -> str:
        return f"'{filename}' 내용"

    def save(self, filename: str, data: str) -> None:
        print(f"'{filename}' 저장 완료")

    def delete(self, filename: str) -> None:
        print(f"'{filename}' 삭제 완료")

    def list_files(self) -> list[str]:
        return ["file1.txt", "file2.txt"]


class ReadOnlyStorage(Readable):
    """
    ✅ LSP 준수!
    읽기 전용 저장소는 Readable만 구현.
    delete를 제공하지 않으므로 계약을 깨뜨리지 않는다.
    """

    def read(self, filename: str) -> str:
        return f"'{filename}' 내용 (읽기 전용)"

    def list_files(self) -> list[str]:
        return ["readme.txt"]


# 사용: 필요한 능력만 요구
def backup_files(source: Readable, target: Writable) -> None:
    """읽기 가능한 소스에서 쓰기 가능한 대상으로 백업"""
    for filename in source.list_files():
        data = source.read(filename)
        target.save(filename, data)

def cleanup(storage: Deletable, filenames: list[str]) -> None:
    """삭제 가능한 저장소에서만 정리 수행"""
    for name in filenames:
        storage.delete(name)

# ReadOnlyStorage는 cleanup에 전달할 수 없다 → 컴파일/타입 체크 단계에서 오류 감지!
```

### ✅ C# - 좋은 예: 올바른 도형 설계

```csharp
// ── 공통 추상화 ──
public abstract class Shape
{
    public abstract double Area();
    public abstract double Perimeter();
}

public class Rectangle : Shape
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double width, double height)
    {
        if (width <= 0 || height <= 0)
            throw new ArgumentException("너비와 높이는 양수여야 합니다.");
        Width = width;
        Height = height;
    }

    public override double Area() => Width * Height;
    public override double Perimeter() => 2 * (Width + Height);

    // 불변 객체 패턴
    public Rectangle WithWidth(double newWidth) => new(newWidth, Height);
    public Rectangle WithHeight(double newHeight) => new(Width, newHeight);
}

public class Square : Shape
{
    // ✅ Rectangle을 상속하지 않음!
    public double Side { get; }

    public Square(double side)
    {
        if (side <= 0) throw new ArgumentException("변의 길이는 양수여야 합니다.");
        Side = side;
    }

    public override double Area() => Side * Side;
    public override double Perimeter() => 4 * Side;

    public Square WithSide(double newSide) => new(newSide);
}

// ── 저장소 인터페이스 분리 ──
public interface IReadable
{
    string Read(string filename);
    IReadOnlyList<string> ListFiles();
}

public interface IWritable
{
    void Save(string filename, string data);
}

public interface IDeletable
{
    void Delete(string filename);
}

// 전체 기능 저장소
public class FullFileStorage : IReadable, IWritable, IDeletable
{
    public string Read(string filename) => $"'{filename}' 내용";
    public void Save(string filename, string data) => Console.WriteLine($"'{filename}' 저장 완료");
    public void Delete(string filename) => Console.WriteLine($"'{filename}' 삭제 완료");
    public IReadOnlyList<string> ListFiles() => new[] { "file1.txt", "file2.txt" };
}

// ✅ 읽기 전용 저장소 - IDeletable을 구현하지 않음
public class ReadOnlyStorage : IReadable
{
    public string Read(string filename) => $"'{filename}' 내용 (읽기 전용)";
    public IReadOnlyList<string> ListFiles() => new[] { "readme.txt" };
}

// 사용 예
void Cleanup(IDeletable storage, IEnumerable<string> filenames)
{
    foreach (var name in filenames)
        storage.Delete(name);
}

// ReadOnlyStorage는 IDeletable이 아니므로 Cleanup에 전달 불가 → 컴파일 에러
```

---

## 5. LSP 위반 징후 및 해결 방법

### 위반 징후

| 징후 | 예시 | 해결 |
|------|------|------|
| **자식 클래스에서 예외를 던짐** | `raise NotImplementedError` | 인터페이스 분리 (ISP 적용) |
| **자식이 부모 메서드를 빈 구현으로 오버라이드** | `def fly(self): pass` | 해당 능력을 별도 인터페이스로 |
| **isinstance/타입 체크 필요** | `if isinstance(x, Square)` | 상속 구조 재설계 |
| **자식이 부모의 불변조건을 변경** | Square가 width/height 연동 | 상속 대신 별도 클래스 |
| **자식이 부모보다 제한적** | ReadOnly가 delete에서 예외 | 능력 기반 인터페이스 분리 |

### LSP를 지키는 설계 원칙

```
1. "is-a" 관계를 의심하라
   → 수학적 is-a ≠ 행위적 is-a

2. 부모의 모든 계약(사전조건, 사후조건, 불변조건)을 자식이 지킬 수 있는가?
   → 하나라도 못 지키면 상속하지 마라

3. 자식이 부모의 메서드를 "빈 구현"이나 "예외"로 오버라이드하는가?
   → 그렇다면 상속이 잘못된 것

4. 불변 객체를 활용하라
   → setter를 없애면 불변조건 위반 가능성이 줄어듦
```

### Design by Contract (계약에 의한 설계)

```python
class Bird(ABC):
    """새 - 계약 정의"""

    @abstractmethod
    def move(self) -> str:
        """
        계약:
        - 사전조건: 없음
        - 사후조건: 이동 방법을 문자열로 반환
        - 불변조건: 이동 후에도 새의 상태는 유효
        """
        pass


class Eagle(Bird):
    def move(self) -> str:
        return "하늘을 날아 이동합니다"  # ✅ 계약 준수


class Penguin(Bird):
    def move(self) -> str:
        return "뒤뚱뒤뚱 걸어서 이동합니다"  # ✅ 계약 준수 (이동 방법이 다를 뿐)


# move()라는 계약만 지키면 어떤 새든 안전하게 치환 가능
def migrate(birds: list[Bird]) -> None:
    for bird in birds:
        print(bird.move())  # 항상 안전!
```

---

## 6. 정리 및 체크리스트

### 핵심 요약

| 항목 | 설명 |
|------|------|
| **원칙** | 하위 타입은 상위 타입을 대체할 수 있어야 한다 |
| **핵심 질문** | 부모 자리에 자식을 넣어도 프로그램이 올바르게 동작하는가? |
| **유명한 위반** | 정사각형-직사각형 문제 |
| **해결 도구** | 인터페이스 분리, 합성, 불변 객체 |
| **관련 원칙** | OCP (LSP를 지켜야 OCP가 동작), ISP (능력별 인터페이스 분리) |

### 셀프 체크리스트

- [ ] 자식 클래스가 부모의 모든 메서드를 의미 있게 구현하는가?
- [ ] `NotImplementedError`나 빈 메서드로 오버라이드하는 곳이 없는가?
- [ ] `isinstance`나 타입 체크로 분기하는 코드가 없는가?
- [ ] 부모의 사전조건/사후조건/불변조건을 자식이 모두 지키는가?
- [ ] "is-a" 관계가 수학적으로만이 아니라 행위적으로도 성립하는가?

---

## 7. 다음 단계

LSP로 올바른 상속 관계를 설계하는 방법을 배웠습니다. LSP 위반 해결 과정에서 "인터페이스 분리"를 자주 사용했는데, 이를 체계적으로 다루는 것이 바로 ISP입니다.

**다음 문서**: [05-solid-isp.md - I: 인터페이스 분리 원칙 (Interface Segregation Principle)](./05-solid-isp.md)

> "인터페이스가 너무 크면 구현체가 불필요한 메서드를 강제로 구현해야 합니다." 뚱뚱한 인터페이스를 역할별로 분리하는 방법을 알아봅니다.
