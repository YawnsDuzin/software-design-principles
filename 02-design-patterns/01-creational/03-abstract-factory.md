# Abstract Factory 패턴 (추상 팩토리)

> **핵심 한줄 요약**: 관련된 객체들의 가족(family)을 생성하는 인터페이스를 제공하되, 구체적인 클래스를 지정하지 않는다.

---

## 목차

1. [의도 (Intent)](#1-의도-intent)
2. [문제 상황 (Problem)](#2-문제-상황-problem)
3. [UML 다이어그램](#3-uml-다이어그램)
4. [Factory Method vs Abstract Factory](#4-factory-method-vs-abstract-factory)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 크로스 플랫폼 UI 컴포넌트](#7-실무-예제-크로스-플랫폼-ui-컴포넌트)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 의도 (Intent)

Abstract Factory 패턴은 **관련 있는 여러 객체를 일관되게 생성**하는 인터페이스를 제공한다.

핵심 아이디어:

- **객체 가족(family)**: 함께 사용되어야 하는 관련 객체들의 묶음
- **일관성 보장**: 같은 가족의 객체들이 섞이지 않도록 보장
- **교체 용이**: 가족 전체를 통째로 교체 가능

### 실생활 비유

```
가구점에서 "거실 세트"를 주문한다고 하자.

모던 스타일 세트:
  → 모던 소파 + 모던 테이블 + 모던 의자

빈티지 스타일 세트:
  → 빈티지 소파 + 빈티지 테이블 + 빈티지 의자

여기서 "모던 소파 + 빈티지 테이블"을 섞으면 어색하다.
Abstract Factory는 스타일별로 가구 세트를 통째로 제공한다.

→ 스타일(가족)을 선택하면, 그 스타일의 모든 가구가 일관되게 생성된다.
```

---

## 2. 문제 상황 (Problem)

### 왜 필요한가?

UI 테마(Light/Dark)를 구현하는 상황을 생각해 보자.

```python
# 문제: 테마별 컴포넌트를 일일이 조건 분기로 생성
class UIRenderer:
    def __init__(self, theme: str):
        self.theme = theme

    def create_button(self):
        if self.theme == "light":
            return LightButton()
        elif self.theme == "dark":
            return DarkButton()

    def create_textbox(self):
        if self.theme == "light":
            return LightTextBox()
        elif self.theme == "dark":
            return DarkTextBox()

    def create_checkbox(self):
        if self.theme == "light":
            return LightCheckBox()
        elif self.theme == "dark":
            return DarkCheckBox()

    # 문제점:
    # 1. 새로운 테마 추가 시 모든 메서드 수정
    # 2. 새로운 컴포넌트 추가 시 또 다른 분기 필요
    # 3. LightButton + DarkTextBox 같은 잘못된 조합 가능
```

### 이 코드의 문제점

| 문제 | 설명 |
|------|------|
| **OCP 위반** | 테마 추가 시 기존 코드 수정 필요 |
| **일관성 미보장** | Light 버튼 + Dark 체크박스 같은 잘못된 조합 가능 |
| **높은 결합도** | 모든 테마의 구체 클래스에 직접 의존 |
| **유지보수 어려움** | 분기가 컴포넌트 수 x 테마 수 만큼 늘어남 |

---

## 3. UML 다이어그램

```
┌─────────────────────────────┐
│  AbstractFactory (interface) │
├─────────────────────────────┤
│ + createButton(): Button     │
│ + createTextBox(): TextBox   │
│ + createCheckBox(): CheckBox │
└──────────┬──────────────────┘
           │ 구현
     ┌─────┴─────────────────┐
     │                       │
┌────┴──────────────┐  ┌─────┴─────────────┐
│  LightFactory     │  │  DarkFactory      │
├───────────────────┤  ├───────────────────┤
│ + createButton()  │  │ + createButton()  │
│   → LightButton   │  │   → DarkButton    │
│ + createTextBox() │  │ + createTextBox() │
│   → LightTextBox  │  │   → DarkTextBox   │
│ + createCheckBox()│  │ + createCheckBox()│
│   → LightCheckBox │  │   → DarkCheckBox  │
└───────────────────┘  └───────────────────┘

제품 계층:
┌──────────┐   ┌──────────┐   ┌───────────┐
│  Button  │   │ TextBox  │   │ CheckBox  │
│(abstract)│   │(abstract)│   │ (abstract)│
└────┬─────┘   └────┬─────┘   └─────┬─────┘
     │              │               │
  ┌──┴──┐        ┌──┴──┐         ┌──┴──┐
  │Light│Dark│   │Light│Dark│    │Light│Dark│
  │Btn  │Btn │   │TBox │TBox│    │CBox │CBox│
  └─────┴────┘   └─────┴────┘    └─────┴────┘

[핵심 포인트]
- 클라이언트는 AbstractFactory와 Abstract Product만 참조
- 구체적인 팩토리를 교체하면 전체 제품 가족이 바뀜
- Light 제품과 Dark 제품이 섞이지 않음을 보장
```

---

## 4. Factory Method vs Abstract Factory

### 구조적 차이

```
Factory Method                    Abstract Factory
─────────────────                 ─────────────────

Creator                          AbstractFactory
  │                                │
  └─ createProduct()               ├─ createProductA()
     → 하나의 제품 생성               ├─ createProductB()
                                   └─ createProductC()
                                      → 관련 제품 가족 생성
```

### 비교표

| 항목 | Factory Method | Abstract Factory |
|------|---------------|-----------------|
| **목적** | 하나의 제품 생성 위임 | 관련 제품 가족 전체 생성 |
| **구현 방식** | 상속 + 오버라이드 | 객체 합성 (composition) |
| **메서드 수** | 팩토리 메서드 1개 | 팩토리 메서드 여러 개 |
| **확장 방향** | 새로운 제품 타입 추가 | 새로운 제품 가족 추가 |
| **복잡도** | 중간 | 높음 |
| **사용 시점** | 단일 제품 생성 유연화 | 관련 제품 일관성 보장 |

### 코드로 비교

```python
# Factory Method: 하나의 팩토리 메서드
class DialogCreator(ABC):
    @abstractmethod
    def create_button(self) -> Button:  # 팩토리 메서드 1개
        pass

# Abstract Factory: 여러 팩토리 메서드 (제품 가족)
class UIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:     # 제품 A
        pass

    @abstractmethod
    def create_textbox(self) -> TextBox:   # 제품 B
        pass

    @abstractmethod
    def create_checkbox(self) -> CheckBox: # 제품 C
        pass
```

---

## 5. Python 구현

### 기본 구현: UI 테마 시스템

```python
from abc import ABC, abstractmethod


# ══════════════════════════════════════════
# Abstract Products (추상 제품)
# ══════════════════════════════════════════

class Button(ABC):
    """버튼 추상 제품"""

    @abstractmethod
    def render(self) -> str:
        pass

    @abstractmethod
    def on_click(self, action: str) -> str:
        pass


class TextBox(ABC):
    """텍스트 박스 추상 제품"""

    @abstractmethod
    def render(self) -> str:
        pass

    @abstractmethod
    def set_text(self, text: str) -> str:
        pass


class CheckBox(ABC):
    """체크박스 추상 제품"""

    @abstractmethod
    def render(self) -> str:
        pass

    @abstractmethod
    def toggle(self) -> str:
        pass


# ══════════════════════════════════════════
# Concrete Products - Light Theme
# ══════════════════════════════════════════

class LightButton(Button):
    def render(self) -> str:
        return "[Light Button] 밝은 배경, 어두운 텍스트"

    def on_click(self, action: str) -> str:
        return f"[Light Button] '{action}' 클릭 - 밝은 리플 효과"


class LightTextBox(TextBox):
    def render(self) -> str:
        return "[Light TextBox] 흰 배경, 검은 테두리"

    def set_text(self, text: str) -> str:
        return f"[Light TextBox] '{text}' 입력 - 검은 글씨"


class LightCheckBox(CheckBox):
    def render(self) -> str:
        return "[Light CheckBox] 밝은 배경 체크박스"

    def toggle(self) -> str:
        return "[Light CheckBox] 토글 - 파란 체크 표시"


# ══════════════════════════════════════════
# Concrete Products - Dark Theme
# ══════════════════════════════════════════

class DarkButton(Button):
    def render(self) -> str:
        return "[Dark Button] 어두운 배경, 밝은 텍스트"

    def on_click(self, action: str) -> str:
        return f"[Dark Button] '{action}' 클릭 - 어두운 리플 효과"


class DarkTextBox(TextBox):
    def render(self) -> str:
        return "[Dark TextBox] 어두운 배경, 밝은 테두리"

    def set_text(self, text: str) -> str:
        return f"[Dark TextBox] '{text}' 입력 - 흰 글씨"


class DarkCheckBox(CheckBox):
    def render(self) -> str:
        return "[Dark CheckBox] 어두운 배경 체크박스"

    def toggle(self) -> str:
        return "[Dark CheckBox] 토글 - 초록 체크 표시"


# ══════════════════════════════════════════
# Abstract Factory
# ══════════════════════════════════════════

class UIFactory(ABC):
    """UI 컴포넌트 팩토리 (Abstract Factory)

    관련된 UI 컴포넌트(Button, TextBox, CheckBox)를
    테마에 맞게 일관되게 생성한다.
    """

    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_textbox(self) -> TextBox:
        pass

    @abstractmethod
    def create_checkbox(self) -> CheckBox:
        pass


# ══════════════════════════════════════════
# Concrete Factories
# ══════════════════════════════════════════

class LightThemeFactory(UIFactory):
    """밝은 테마 팩토리"""

    def create_button(self) -> Button:
        return LightButton()

    def create_textbox(self) -> TextBox:
        return LightTextBox()

    def create_checkbox(self) -> CheckBox:
        return LightCheckBox()


class DarkThemeFactory(UIFactory):
    """어두운 테마 팩토리"""

    def create_button(self) -> Button:
        return DarkButton()

    def create_textbox(self) -> TextBox:
        return DarkTextBox()

    def create_checkbox(self) -> CheckBox:
        return DarkCheckBox()


# ══════════════════════════════════════════
# 클라이언트 코드
# ══════════════════════════════════════════

class Application:
    """애플리케이션 클라이언트

    UIFactory에만 의존하며, 구체적인 테마를 모른다.
    팩토리를 교체하면 전체 UI가 일관되게 변경된다.
    """

    def __init__(self, factory: UIFactory):
        self.factory = factory
        self.button = factory.create_button()
        self.textbox = factory.create_textbox()
        self.checkbox = factory.create_checkbox()

    def render(self):
        print("  " + self.button.render())
        print("  " + self.textbox.render())
        print("  " + self.checkbox.render())

    def interact(self):
        print("  " + self.button.on_click("저장"))
        print("  " + self.textbox.set_text("Hello World"))
        print("  " + self.checkbox.toggle())


# ═════ 실행 ═════
def get_factory(theme: str) -> UIFactory:
    """설정에 따라 적절한 팩토리를 반환"""
    factories = {
        "light": LightThemeFactory,
        "dark": DarkThemeFactory,
    }
    factory_class = factories.get(theme)
    if factory_class is None:
        raise ValueError(f"알 수 없는 테마: {theme}")
    return factory_class()


if __name__ == "__main__":
    print("=== Light 테마 ===")
    app = Application(get_factory("light"))
    app.render()
    print()
    app.interact()

    print("\n=== Dark 테마 ===")
    app = Application(get_factory("dark"))
    app.render()
    print()
    app.interact()
```

**출력:**
```
=== Light 테마 ===
  [Light Button] 밝은 배경, 어두운 텍스트
  [Light TextBox] 흰 배경, 검은 테두리
  [Light CheckBox] 밝은 배경 체크박스

  [Light Button] '저장' 클릭 - 밝은 리플 효과
  [Light TextBox] 'Hello World' 입력 - 검은 글씨
  [Light CheckBox] 토글 - 파란 체크 표시

=== Dark 테마 ===
  [Dark Button] 어두운 배경, 밝은 텍스트
  [Dark TextBox] 어두운 배경, 밝은 테두리
  [Dark CheckBox] 어두운 배경 체크박스

  [Dark Button] '저장' 클릭 - 어두운 리플 효과
  [Dark TextBox] 'Hello World' 입력 - 흰 글씨
  [Dark CheckBox] 토글 - 초록 체크 표시
```

### 새 테마 추가하기 (확장의 용이성)

```python
# 새로운 "High Contrast" 테마를 추가하는 경우
# 기존 코드를 전혀 수정하지 않고, 새로운 클래스만 추가하면 된다.

class HighContrastButton(Button):
    def render(self) -> str:
        return "[HC Button] 검은 배경, 노란 텍스트, 흰 테두리"

    def on_click(self, action: str) -> str:
        return f"[HC Button] '{action}' 클릭 - 강렬한 노란 리플"


class HighContrastTextBox(TextBox):
    def render(self) -> str:
        return "[HC TextBox] 검은 배경, 노란 테두리, 큰 글씨"

    def set_text(self, text: str) -> str:
        return f"[HC TextBox] '{text}' 입력 - 큰 노란 글씨"


class HighContrastCheckBox(CheckBox):
    def render(self) -> str:
        return "[HC CheckBox] 검은 배경, 흰 테두리, 큰 체크 마크"

    def toggle(self) -> str:
        return "[HC CheckBox] 토글 - 큰 노란 체크 표시"


class HighContrastFactory(UIFactory):
    """고대비 테마 팩토리 - 새로 추가"""

    def create_button(self) -> Button:
        return HighContrastButton()

    def create_textbox(self) -> TextBox:
        return HighContrastTextBox()

    def create_checkbox(self) -> CheckBox:
        return HighContrastCheckBox()


# 기존의 Application 클래스는 전혀 수정하지 않는다!
app = Application(HighContrastFactory())
app.render()
```

---

## 6. C# 구현

### 기본 구현

```csharp
using System;

// ══════════════════════════════════════════
// Abstract Products
// ══════════════════════════════════════════

public interface IButton
{
    string Render();
    string OnClick(string action);
}

public interface ITextBox
{
    string Render();
    string SetText(string text);
}

public interface ICheckBox
{
    string Render();
    string Toggle();
}

// ══════════════════════════════════════════
// Concrete Products - Light Theme
// ══════════════════════════════════════════

public class LightButton : IButton
{
    public string Render() => "[Light Button] 밝은 배경, 어두운 텍스트";
    public string OnClick(string action) => $"[Light Button] '{action}' 클릭 - 밝은 리플 효과";
}

public class LightTextBox : ITextBox
{
    public string Render() => "[Light TextBox] 흰 배경, 검은 테두리";
    public string SetText(string text) => $"[Light TextBox] '{text}' 입력 - 검은 글씨";
}

public class LightCheckBox : ICheckBox
{
    public string Render() => "[Light CheckBox] 밝은 배경 체크박스";
    public string Toggle() => "[Light CheckBox] 토글 - 파란 체크 표시";
}

// ══════════════════════════════════════════
// Concrete Products - Dark Theme
// ══════════════════════════════════════════

public class DarkButton : IButton
{
    public string Render() => "[Dark Button] 어두운 배경, 밝은 텍스트";
    public string OnClick(string action) => $"[Dark Button] '{action}' 클릭 - 어두운 리플 효과";
}

public class DarkTextBox : ITextBox
{
    public string Render() => "[Dark TextBox] 어두운 배경, 밝은 테두리";
    public string SetText(string text) => $"[Dark TextBox] '{text}' 입력 - 흰 글씨";
}

public class DarkCheckBox : ICheckBox
{
    public string Render() => "[Dark CheckBox] 어두운 배경 체크박스";
    public string Toggle() => "[Dark CheckBox] 토글 - 초록 체크 표시";
}

// ══════════════════════════════════════════
// Abstract Factory
// ══════════════════════════════════════════

/// <summary>
/// UI 컴포넌트 팩토리 인터페이스
/// 관련 컴포넌트(Button, TextBox, CheckBox)를 일관되게 생성
/// </summary>
public interface IUIFactory
{
    IButton CreateButton();
    ITextBox CreateTextBox();
    ICheckBox CreateCheckBox();
}

// ══════════════════════════════════════════
// Concrete Factories
// ══════════════════════════════════════════

public class LightThemeFactory : IUIFactory
{
    public IButton CreateButton() => new LightButton();
    public ITextBox CreateTextBox() => new LightTextBox();
    public ICheckBox CreateCheckBox() => new LightCheckBox();
}

public class DarkThemeFactory : IUIFactory
{
    public IButton CreateButton() => new DarkButton();
    public ITextBox CreateTextBox() => new DarkTextBox();
    public ICheckBox CreateCheckBox() => new DarkCheckBox();
}

// ══════════════════════════════════════════
// 클라이언트 코드
// ══════════════════════════════════════════

/// <summary>
/// 클라이언트: IUIFactory에만 의존
/// 팩토리 교체로 전체 UI 테마가 일관되게 변경됨
/// </summary>
public class Application
{
    private readonly IButton _button;
    private readonly ITextBox _textBox;
    private readonly ICheckBox _checkBox;

    public Application(IUIFactory factory)
    {
        _button = factory.CreateButton();
        _textBox = factory.CreateTextBox();
        _checkBox = factory.CreateCheckBox();
    }

    public void Render()
    {
        Console.WriteLine($"  {_button.Render()}");
        Console.WriteLine($"  {_textBox.Render()}");
        Console.WriteLine($"  {_checkBox.Render()}");
    }

    public void Interact()
    {
        Console.WriteLine($"  {_button.OnClick("저장")}");
        Console.WriteLine($"  {_textBox.SetText("Hello World")}");
        Console.WriteLine($"  {_checkBox.Toggle()}");
    }
}

// ═════ 실행 ═════
public class Program
{
    /// <summary>설정에 따라 팩토리 선택</summary>
    static IUIFactory GetFactory(string theme) => theme switch
    {
        "light" => new LightThemeFactory(),
        "dark" => new DarkThemeFactory(),
        _ => throw new ArgumentException($"알 수 없는 테마: {theme}")
    };

    static void Main()
    {
        Console.WriteLine("=== Light 테마 ===");
        var lightApp = new Application(GetFactory("light"));
        lightApp.Render();
        Console.WriteLine();
        lightApp.Interact();

        Console.WriteLine("\n=== Dark 테마 ===");
        var darkApp = new Application(GetFactory("dark"));
        darkApp.Render();
        Console.WriteLine();
        darkApp.Interact();
    }
}
```

### DI 컨테이너와 함께 사용 (ASP.NET Core)

```csharp
// ASP.NET Core에서 Abstract Factory를 DI와 결합하는 실무 패턴

// Program.cs
var theme = builder.Configuration["Theme"] ?? "light";

builder.Services.AddSingleton<IUIFactory>(sp =>
{
    return theme switch
    {
        "light" => new LightThemeFactory(),
        "dark" => new DarkThemeFactory(),
        _ => new LightThemeFactory()
    };
});

// 컨트롤러에서 사용
public class HomeController : Controller
{
    private readonly IUIFactory _uiFactory;

    public HomeController(IUIFactory uiFactory)
    {
        _uiFactory = uiFactory;
    }

    public IActionResult Index()
    {
        var button = _uiFactory.CreateButton();
        ViewData["ButtonHtml"] = button.Render();
        return View();
    }
}
```

---

## 7. 실무 예제: 크로스 플랫폼 UI 컴포넌트

### 시나리오

데스크톱 앱이 Windows와 macOS를 모두 지원해야 한다. 각 플랫폼에 맞는 네이티브 UI 컴포넌트를 사용해야 하며, 컴포넌트들의 일관성이 보장되어야 한다.

### Python 구현

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional


# ══════════════════════════════════════════
# Abstract Products: 크로스 플랫폼 UI 컴포넌트
# ══════════════════════════════════════════

class Window(ABC):
    """윈도우(창) 추상 컴포넌트"""

    @abstractmethod
    def create(self, title: str, width: int, height: int) -> str:
        pass

    @abstractmethod
    def close(self) -> str:
        pass


class Menu(ABC):
    """메뉴 추상 컴포넌트"""

    @abstractmethod
    def create(self, items: list[str]) -> str:
        pass

    @abstractmethod
    def add_item(self, item: str) -> str:
        pass


class Dialog(ABC):
    """대화상자 추상 컴포넌트"""

    @abstractmethod
    def show_alert(self, message: str) -> str:
        pass

    @abstractmethod
    def show_confirm(self, message: str) -> str:
        pass


class FileChooser(ABC):
    """파일 선택 대화상자 추상 컴포넌트"""

    @abstractmethod
    def open_file(self, filters: list[str]) -> str:
        pass

    @abstractmethod
    def save_file(self, default_name: str) -> str:
        pass


# ══════════════════════════════════════════
# Concrete Products - Windows
# ══════════════════════════════════════════

class WindowsWindow(Window):
    def create(self, title: str, width: int, height: int) -> str:
        return f"[Win32] CreateWindowEx('{title}', {width}x{height})"

    def close(self) -> str:
        return "[Win32] DestroyWindow() 호출"


class WindowsMenu(Menu):
    def create(self, items: list[str]) -> str:
        return f"[Win32] CreateMenu() - 항목: {', '.join(items)}"

    def add_item(self, item: str) -> str:
        return f"[Win32] AppendMenu('{item}')"


class WindowsDialog(Dialog):
    def show_alert(self, message: str) -> str:
        return f"[Win32] MessageBox(MB_OK, '{message}')"

    def show_confirm(self, message: str) -> str:
        return f"[Win32] MessageBox(MB_YESNO, '{message}')"


class WindowsFileChooser(FileChooser):
    def open_file(self, filters: list[str]) -> str:
        return f"[Win32] GetOpenFileName(filters={filters})"

    def save_file(self, default_name: str) -> str:
        return f"[Win32] GetSaveFileName('{default_name}')"


# ══════════════════════════════════════════
# Concrete Products - macOS
# ══════════════════════════════════════════

class MacWindow(Window):
    def create(self, title: str, width: int, height: int) -> str:
        return f"[Cocoa] NSWindow('{title}', {width}x{height})"

    def close(self) -> str:
        return "[Cocoa] [window close] 호출"


class MacMenu(Menu):
    def create(self, items: list[str]) -> str:
        return f"[Cocoa] NSMenu - 항목: {', '.join(items)}"

    def add_item(self, item: str) -> str:
        return f"[Cocoa] [menu addItem:'{item}']"


class MacDialog(Dialog):
    def show_alert(self, message: str) -> str:
        return f"[Cocoa] NSAlert('{message}') runModal"

    def show_confirm(self, message: str) -> str:
        return f"[Cocoa] NSAlert('{message}') addButtons:['확인','취소']"


class MacFileChooser(FileChooser):
    def open_file(self, filters: list[str]) -> str:
        return f"[Cocoa] NSOpenPanel(types={filters})"

    def save_file(self, default_name: str) -> str:
        return f"[Cocoa] NSSavePanel('{default_name}')"


# ══════════════════════════════════════════
# Abstract Factory
# ══════════════════════════════════════════

class PlatformUIFactory(ABC):
    """플랫폼별 UI 팩토리 인터페이스"""

    @abstractmethod
    def create_window(self) -> Window:
        pass

    @abstractmethod
    def create_menu(self) -> Menu:
        pass

    @abstractmethod
    def create_dialog(self) -> Dialog:
        pass

    @abstractmethod
    def create_file_chooser(self) -> FileChooser:
        pass


# ══════════════════════════════════════════
# Concrete Factories
# ══════════════════════════════════════════

class WindowsUIFactory(PlatformUIFactory):
    """Windows UI 팩토리"""

    def create_window(self) -> Window:
        return WindowsWindow()

    def create_menu(self) -> Menu:
        return WindowsMenu()

    def create_dialog(self) -> Dialog:
        return WindowsDialog()

    def create_file_chooser(self) -> FileChooser:
        return WindowsFileChooser()


class MacUIFactory(PlatformUIFactory):
    """macOS UI 팩토리"""

    def create_window(self) -> Window:
        return MacWindow()

    def create_menu(self) -> Menu:
        return MacMenu()

    def create_dialog(self) -> Dialog:
        return MacDialog()

    def create_file_chooser(self) -> FileChooser:
        return MacFileChooser()


# ══════════════════════════════════════════
# 클라이언트: 텍스트 에디터 앱
# ══════════════════════════════════════════

class TextEditor:
    """텍스트 에디터 애플리케이션

    PlatformUIFactory에만 의존하여,
    플랫폼 독립적인 코드를 작성한다.
    """

    def __init__(self, ui_factory: PlatformUIFactory):
        self.ui = ui_factory
        self.window = None
        self.menu = None

    def initialize(self):
        """앱 초기화"""
        print("  [초기화 시작]")

        # 메인 윈도우 생성
        self.window = self.ui.create_window()
        print(f"    {self.window.create('텍스트 에디터', 1024, 768)}")

        # 메뉴 생성
        self.menu = self.ui.create_menu()
        print(f"    {self.menu.create(['파일', '편집', '보기', '도움말'])}")

        print("  [초기화 완료]\n")

    def open_file(self):
        """파일 열기"""
        chooser = self.ui.create_file_chooser()
        print(f"    {chooser.open_file(['.txt', '.md', '.py'])}")

    def save_file(self):
        """파일 저장"""
        chooser = self.ui.create_file_chooser()
        print(f"    {chooser.save_file('untitled.txt')}")

    def show_about(self):
        """정보 대화상자"""
        dialog = self.ui.create_dialog()
        print(f"    {dialog.show_alert('텍스트 에디터 v1.0')}")

    def close(self):
        """앱 종료"""
        dialog = self.ui.create_dialog()
        print(f"    {dialog.show_confirm('저장하지 않은 변경사항이 있습니다. 종료하시겠습니까?')}")
        print(f"    {self.window.close()}")


# ═════ 실행 ═════
import platform

def get_platform_factory() -> PlatformUIFactory:
    """현재 플랫폼에 맞는 팩토리를 반환"""
    system = platform.system()
    if system == "Windows":
        return WindowsUIFactory()
    elif system == "Darwin":
        return MacUIFactory()
    else:
        # 기본값
        return WindowsUIFactory()


if __name__ == "__main__":
    # Windows에서 실행
    print("=" * 50)
    print("텍스트 에디터 - Windows")
    print("=" * 50)
    editor = TextEditor(WindowsUIFactory())
    editor.initialize()
    editor.open_file()
    editor.save_file()
    editor.show_about()
    editor.close()

    # macOS에서 실행
    print("\n" + "=" * 50)
    print("텍스트 에디터 - macOS")
    print("=" * 50)
    editor = TextEditor(MacUIFactory())
    editor.initialize()
    editor.open_file()
    editor.save_file()
    editor.show_about()
    editor.close()
```

### C# 구현

```csharp
using System;
using System.Collections.Generic;

// ══════════════════════════════════════════
// Abstract Products
// ══════════════════════════════════════════

public interface IWindow
{
    string Create(string title, int width, int height);
    string Close();
}

public interface IMenu
{
    string Create(List<string> items);
    string AddItem(string item);
}

public interface IDialog
{
    string ShowAlert(string message);
    string ShowConfirm(string message);
}

public interface IFileChooser
{
    string OpenFile(List<string> filters);
    string SaveFile(string defaultName);
}

// ══════════════════════════════════════════
// Concrete Products - Windows
// ══════════════════════════════════════════

public class WindowsWindow : IWindow
{
    public string Create(string title, int width, int height)
        => $"[Win32] CreateWindowEx('{title}', {width}x{height})";
    public string Close() => "[Win32] DestroyWindow() 호출";
}

public class WindowsMenu : IMenu
{
    public string Create(List<string> items)
        => $"[Win32] CreateMenu() - 항목: {string.Join(", ", items)}";
    public string AddItem(string item)
        => $"[Win32] AppendMenu('{item}')";
}

public class WindowsDialog : IDialog
{
    public string ShowAlert(string message)
        => $"[Win32] MessageBox(MB_OK, '{message}')";
    public string ShowConfirm(string message)
        => $"[Win32] MessageBox(MB_YESNO, '{message}')";
}

public class WindowsFileChooser : IFileChooser
{
    public string OpenFile(List<string> filters)
        => $"[Win32] GetOpenFileName(filters=[{string.Join(", ", filters)}])";
    public string SaveFile(string defaultName)
        => $"[Win32] GetSaveFileName('{defaultName}')";
}

// ══════════════════════════════════════════
// Concrete Products - macOS
// ══════════════════════════════════════════

public class MacWindow : IWindow
{
    public string Create(string title, int width, int height)
        => $"[Cocoa] NSWindow('{title}', {width}x{height})";
    public string Close() => "[Cocoa] [window close] 호출";
}

public class MacMenu : IMenu
{
    public string Create(List<string> items)
        => $"[Cocoa] NSMenu - 항목: {string.Join(", ", items)}";
    public string AddItem(string item)
        => $"[Cocoa] [menu addItem:'{item}']";
}

public class MacDialog : IDialog
{
    public string ShowAlert(string message)
        => $"[Cocoa] NSAlert('{message}') runModal";
    public string ShowConfirm(string message)
        => $"[Cocoa] NSAlert('{message}') addButtons:['확인','취소']";
}

public class MacFileChooser : IFileChooser
{
    public string OpenFile(List<string> filters)
        => $"[Cocoa] NSOpenPanel(types=[{string.Join(", ", filters)}])";
    public string SaveFile(string defaultName)
        => $"[Cocoa] NSSavePanel('{defaultName}')";
}

// ══════════════════════════════════════════
// Abstract Factory
// ══════════════════════════════════════════

public interface IPlatformUIFactory
{
    IWindow CreateWindow();
    IMenu CreateMenu();
    IDialog CreateDialog();
    IFileChooser CreateFileChooser();
}

// ══════════════════════════════════════════
// Concrete Factories
// ══════════════════════════════════════════

public class WindowsUIFactory : IPlatformUIFactory
{
    public IWindow CreateWindow() => new WindowsWindow();
    public IMenu CreateMenu() => new WindowsMenu();
    public IDialog CreateDialog() => new WindowsDialog();
    public IFileChooser CreateFileChooser() => new WindowsFileChooser();
}

public class MacUIFactory : IPlatformUIFactory
{
    public IWindow CreateWindow() => new MacWindow();
    public IMenu CreateMenu() => new MacMenu();
    public IDialog CreateDialog() => new MacDialog();
    public IFileChooser CreateFileChooser() => new MacFileChooser();
}

// ══════════════════════════════════════════
// 클라이언트: 텍스트 에디터
// ══════════════════════════════════════════

public class TextEditor
{
    private readonly IPlatformUIFactory _ui;
    private IWindow? _window;

    public TextEditor(IPlatformUIFactory uiFactory)
    {
        _ui = uiFactory;
    }

    public void Initialize()
    {
        Console.WriteLine("  [초기화 시작]");

        _window = _ui.CreateWindow();
        Console.WriteLine($"    {_window.Create("텍스트 에디터", 1024, 768)}");

        var menu = _ui.CreateMenu();
        Console.WriteLine($"    {menu.Create(new List<string> { "파일", "편집", "보기" })}");

        Console.WriteLine("  [초기화 완료]\n");
    }

    public void OpenFile()
    {
        var chooser = _ui.CreateFileChooser();
        Console.WriteLine($"    {chooser.OpenFile(new List<string> { ".txt", ".md" })}");
    }

    public void Close()
    {
        var dialog = _ui.CreateDialog();
        Console.WriteLine($"    {dialog.ShowConfirm("종료하시겠습니까?")}");
        Console.WriteLine($"    {_window?.Close()}");
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **제품 일관성** | 같은 가족의 제품만 함께 사용됨을 보장 |
| **OCP 준수** | 새로운 제품 가족 추가 시 기존 코드 수정 불필요 |
| **느슨한 결합** | 클라이언트가 구체 클래스를 직접 참조하지 않음 |
| **가족 교체 용이** | 팩토리 하나만 교체하면 전체 제품이 일관되게 변경 |
| **SRP 준수** | 제품 생성 코드가 한 곳에 집중됨 |

### 단점

| 단점 | 설명 |
|------|------|
| **클래스 폭발** | 제품 가족 x 제품 종류 수만큼 클래스가 필요 |
| **새 제품 종류 추가 어려움** | 새 제품 타입을 추가하면 모든 팩토리를 수정해야 함 |
| **복잡도** | 단순한 경우에는 과도한 설계가 될 수 있음 |

### 주의: 확장 방향에 따른 장단점

```
쉬운 확장 (팩토리만 추가):
  기존: Light, Dark
  추가: HighContrast → HighContrastFactory 하나만 추가

어려운 확장 (모든 팩토리 수정):
  기존: Button, TextBox, CheckBox
  추가: Slider → 모든 팩토리에 createSlider() 추가 필요!

→ Abstract Factory는 "제품 가족 추가"에는 강하지만
   "새로운 제품 종류 추가"에는 약하다
```

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Factory Method** | Abstract Factory의 각 메서드가 Factory Method로 구현됨 |
| **Singleton** | Abstract Factory는 보통 Singleton으로 구현됨 |
| **Prototype** | Prototype으로 Abstract Factory를 대체할 수 있음 |
| **Builder** | Abstract Factory가 "무엇을" 만들지 결정하면, Builder는 "어떻게" 만들지 결정 |
| **Bridge** | Abstract Factory와 Bridge를 함께 사용하여 추상화/구현 분리 가능 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

```
Abstract Factory = 관련 객체 가족을 일관되게 생성

[핵심 구조]
- AbstractFactory: 제품 생성 인터페이스 (여러 create 메서드)
- ConcreteFactory: 특정 가족의 제품들을 생성
- AbstractProduct: 제품의 인터페이스
- ConcreteProduct: 특정 가족에 속하는 제품

[Factory Method와의 차이]
- Factory Method: 하나의 제품 생성 위임
- Abstract Factory: 여러 관련 제품을 가족 단위로 생성

[확장 특성]
- 새 가족 추가: 쉬움 (새 Factory만 추가)
- 새 제품 추가: 어려움 (모든 Factory 수정 필요)
```

### 학습 체크리스트

- [ ] Abstract Factory의 의도를 한 문장으로 설명할 수 있다
- [ ] "제품 가족(family)"의 개념을 실제 예시로 설명할 수 있다
- [ ] Factory Method와 Abstract Factory의 차이를 명확히 설명할 수 있다
- [ ] Python과 C#에서 Abstract Factory를 구현할 수 있다
- [ ] 새로운 제품 가족을 기존 코드 수정 없이 추가할 수 있다
- [ ] Abstract Factory의 확장 방향별 장단점을 이해한다
- [ ] DI 컨테이너와 함께 Abstract Factory를 사용하는 방법을 알고 있다
- [ ] 실무에서 Abstract Factory가 적합한 시나리오를 판단할 수 있다

### 다음 학습

Abstract Factory가 "관련 객체들을 한꺼번에" 생성한다면, 다음으로 배울 [Builder 패턴](./04-builder.md)은 "복잡한 하나의 객체를 단계별로" 생성하는 패턴이다.
