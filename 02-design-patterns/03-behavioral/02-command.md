# Command 패턴 (커맨드)

> **핵심 의도 한줄 요약**: 요청(작업)을 독립적인 객체로 캡슐화하여, 매개변수화, 큐잉, 로깅, 실행 취소(Undo)를 가능하게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 텍스트 에디터 Undo/Redo](#7-실무-예제-텍스트-에디터-undoredo)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 개요

Command 패턴은 **요청 자체를 객체로 캡슐화**하는 패턴이다. 이를 통해 다음이 가능해진다:

- **매개변수화**: 요청을 다른 객체에 인자로 전달
- **큐잉**: 요청을 큐에 넣어 나중에 실행
- **로깅**: 요청을 기록하여 재실행 가능
- **Undo/Redo**: 실행한 요청을 취소하고 다시 실행

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **Command** | 실행할 작업의 인터페이스 (execute, undo) |
| **ConcreteCommand** | 실제 작업을 구현하는 명령 객체 |
| **Receiver** | 실제 작업을 수행하는 객체 |
| **Invoker** | 명령을 실행하는 객체 (버튼, 메뉴 등) |
| **Client** | 명령 객체를 생성하고 Invoker에 설정 |

---

## 2. 문제 상황

### 문제: 버튼 클릭 처리

GUI 애플리케이션에서 버튼을 클릭하면 다양한 작업이 실행된다.

```python
# 나쁜 예: 버튼이 직접 비즈니스 로직을 호출
class Button:
    def __init__(self, action_type: str):
        self.action_type = action_type

    def click(self):
        if self.action_type == "save":
            document.save()
        elif self.action_type == "copy":
            clipboard.copy(document.selected_text)
        elif self.action_type == "paste":
            document.insert(clipboard.content)
        elif self.action_type == "undo":
            # Undo를 어떻게 구현하지?
            pass
        # 새 기능 추가마다 elif 추가...
```

이 코드의 문제점:

- **강한 결합**: Button이 Document, Clipboard 등 다양한 객체에 의존
- **Undo 구현 어려움**: 어떤 작업이 실행되었는지 추적이 불가능
- **재사용 불가**: 같은 작업을 메뉴, 단축키, 버튼에서 사용하려면 코드 중복

---

## 3. 일상 비유

**레스토랑 주문 시스템**을 생각해 보자.

```
고객 (Client)
  → 주문서 작성 (Command 생성)

웨이터 (Invoker)
  → 주문서를 주방에 전달 (Command 실행)
  → 주문서를 보관 (히스토리 기록)

주방 (Receiver)
  → 주문서 내용대로 요리 실행

주문 취소가 필요하면?
  → 보관된 주문서를 보고 취소 (Undo)
```

- **주문서**가 바로 Command 객체다
- 고객이 직접 주방에 가지 않아도 된다 (느슨한 결합)
- 주문서를 보관하면 취소(Undo)나 재주문(Redo)이 가능하다
- 같은 주문서를 여러 웨이터가 처리할 수 있다 (재사용)

---

## 4. UML 다이어그램

```
 ┌──────────┐       ┌──────────────────┐
 │  Client  │──────>│   <<interface>>  │
 └──────────┘       │     Command      │
      │             ├──────────────────┤
      │             │ + execute()      │
      │             │ + undo()         │
      │             └────────┬─────────┘
      │                      │
      │         ┌────────────┼────────────┐
      │         │                         │
      │  ┌──────▼────────┐    ┌───────────▼──────┐
      │  │ConcreteCommandA│    │ConcreteCommandB  │
      │  ├────────────────┤    ├──────────────────┤
      │  │- receiver      │    │- receiver        │
      │  │- state (backup)│    │- state (backup)  │
      │  │+ execute()     │    │+ execute()       │
      │  │+ undo()        │    │+ undo()          │
      │  └───────┬────────┘    └────────┬─────────┘
      │          │                      │
      │          ▼                      ▼
      │  ┌──────────────┐      ┌──────────────┐
      │  │  ReceiverA   │      │  ReceiverB   │
      │  └──────────────┘      └──────────────┘
      │
      ▼
 ┌──────────────┐
 │   Invoker    │
 ├──────────────┤
 │ - history[]  │
 │ + execute()  │
 │ + undo()     │
 │ + redo()     │
 └──────────────┘
```

---

## 5. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass, field


class Command(ABC):
    """커맨드 인터페이스"""

    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass


class Light:
    """수신자 (Receiver): 조명"""

    def __init__(self, location: str) -> None:
        self.location = location
        self.is_on = False
        self.brightness = 0

    def turn_on(self) -> None:
        self.is_on = True
        self.brightness = 100
        print(f"  {self.location} 조명 ON (밝기: {self.brightness}%)")

    def turn_off(self) -> None:
        self.is_on = False
        self.brightness = 0
        print(f"  {self.location} 조명 OFF")

    def set_brightness(self, level: int) -> None:
        self.brightness = level
        print(f"  {self.location} 밝기 조절: {self.brightness}%")


class LightOnCommand(Command):
    """조명 켜기 커맨드"""

    def __init__(self, light: Light) -> None:
        self._light = light
        self._prev_brightness = 0

    def execute(self) -> None:
        self._prev_brightness = self._light.brightness
        self._light.turn_on()

    def undo(self) -> None:
        if self._prev_brightness == 0:
            self._light.turn_off()
        else:
            self._light.set_brightness(self._prev_brightness)


class LightOffCommand(Command):
    """조명 끄기 커맨드"""

    def __init__(self, light: Light) -> None:
        self._light = light
        self._prev_brightness = 0

    def execute(self) -> None:
        self._prev_brightness = self._light.brightness
        self._light.turn_off()

    def undo(self) -> None:
        self._light.turn_on()
        self._light.set_brightness(self._prev_brightness)


class DimCommand(Command):
    """밝기 조절 커맨드"""

    def __init__(self, light: Light, level: int) -> None:
        self._light = light
        self._level = level
        self._prev_brightness = 0

    def execute(self) -> None:
        self._prev_brightness = self._light.brightness
        self._light.set_brightness(self._level)

    def undo(self) -> None:
        self._light.set_brightness(self._prev_brightness)


class RemoteControl:
    """호출자 (Invoker): 리모컨"""

    def __init__(self) -> None:
        self._history: list[Command] = []
        self._undo_stack: list[Command] = []

    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)
        self._undo_stack.clear()  # 새 명령 실행 시 redo 스택 초기화

    def undo(self) -> None:
        if not self._history:
            print("  되돌릴 작업이 없습니다.")
            return
        command = self._history.pop()
        command.undo()
        self._undo_stack.append(command)

    def redo(self) -> None:
        if not self._undo_stack:
            print("  다시 실행할 작업이 없습니다.")
            return
        command = self._undo_stack.pop()
        command.execute()
        self._history.append(command)


# === 사용 예시 ===
if __name__ == "__main__":
    # Receiver 생성
    living_room_light = Light("거실")
    bedroom_light = Light("침실")

    # Command 생성
    living_on = LightOnCommand(living_room_light)
    living_off = LightOffCommand(living_room_light)
    bedroom_dim = DimCommand(bedroom_light, 50)

    # Invoker 생성
    remote = RemoteControl()

    print("=== 거실 조명 켜기 ===")
    remote.execute(living_on)

    print("\n=== 침실 밝기 50%로 ===")
    remote.execute(bedroom_dim)

    print("\n=== Undo: 침실 밝기 복원 ===")
    remote.undo()

    print("\n=== Undo: 거실 조명 복원 ===")
    remote.undo()

    print("\n=== Redo: 거실 조명 다시 켜기 ===")
    remote.redo()
```

---

## 6. C# 구현

```csharp
using System;
using System.Collections.Generic;

// 커맨드 인터페이스
public interface ICommand
{
    void Execute();
    void Undo();
}

// 수신자 (Receiver)
public class Light
{
    public string Location { get; }
    public bool IsOn { get; private set; }
    public int Brightness { get; private set; }

    public Light(string location)
    {
        Location = location;
    }

    public void TurnOn()
    {
        IsOn = true;
        Brightness = 100;
        Console.WriteLine($"  {Location} 조명 ON (밝기: {Brightness}%)");
    }

    public void TurnOff()
    {
        IsOn = false;
        Brightness = 0;
        Console.WriteLine($"  {Location} 조명 OFF");
    }

    public void SetBrightness(int level)
    {
        Brightness = level;
        Console.WriteLine($"  {Location} 밝기 조절: {Brightness}%");
    }
}

// 조명 켜기 커맨드
public class LightOnCommand : ICommand
{
    private readonly Light _light;
    private int _prevBrightness;

    public LightOnCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _prevBrightness = _light.Brightness;
        _light.TurnOn();
    }

    public void Undo()
    {
        if (_prevBrightness == 0)
            _light.TurnOff();
        else
            _light.SetBrightness(_prevBrightness);
    }
}

// 조명 끄기 커맨드
public class LightOffCommand : ICommand
{
    private readonly Light _light;
    private int _prevBrightness;

    public LightOffCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _prevBrightness = _light.Brightness;
        _light.TurnOff();
    }

    public void Undo()
    {
        _light.TurnOn();
        _light.SetBrightness(_prevBrightness);
    }
}

// 호출자 (Invoker)
public class RemoteControl
{
    private readonly Stack<ICommand> _history = new();
    private readonly Stack<ICommand> _undoStack = new();

    public void Execute(ICommand command)
    {
        command.Execute();
        _history.Push(command);
        _undoStack.Clear();
    }

    public void Undo()
    {
        if (_history.Count == 0)
        {
            Console.WriteLine("  되돌릴 작업이 없습니다.");
            return;
        }
        var command = _history.Pop();
        command.Undo();
        _undoStack.Push(command);
    }

    public void Redo()
    {
        if (_undoStack.Count == 0)
        {
            Console.WriteLine("  다시 실행할 작업이 없습니다.");
            return;
        }
        var command = _undoStack.Pop();
        command.Execute();
        _history.Push(command);
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var livingRoom = new Light("거실");
        var remote = new RemoteControl();

        Console.WriteLine("=== 조명 켜기 ===");
        remote.Execute(new LightOnCommand(livingRoom));

        Console.WriteLine("\n=== 조명 끄기 ===");
        remote.Execute(new LightOffCommand(livingRoom));

        Console.WriteLine("\n=== Undo ===");
        remote.Undo();

        Console.WriteLine("\n=== Redo ===");
        remote.Redo();
    }
}
```

---

## 7. 실무 예제: 텍스트 에디터 Undo/Redo

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass


class TextDocument:
    """텍스트 문서 (Receiver)"""

    def __init__(self) -> None:
        self._content: list[str] = []

    @property
    def text(self) -> str:
        return "".join(self._content)

    def insert(self, position: int, text: str) -> None:
        self._content.insert(position, text)

    def delete(self, position: int, length: int) -> str:
        deleted = []
        for _ in range(length):
            if position < len(self._content):
                deleted.append(self._content.pop(position))
        return "".join(deleted)

    def __str__(self) -> str:
        return self.text or "(빈 문서)"


class TextCommand(ABC):
    """텍스트 편집 커맨드 인터페이스"""

    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass

    @abstractmethod
    def description(self) -> str:
        pass


class InsertTextCommand(TextCommand):
    """텍스트 삽입 커맨드"""

    def __init__(self, document: TextDocument, position: int, text: str) -> None:
        self._document = document
        self._position = position
        self._text = text

    def execute(self) -> None:
        for i, char in enumerate(self._text):
            self._document.insert(self._position + i, char)

    def undo(self) -> None:
        self._document.delete(self._position, len(self._text))

    def description(self) -> str:
        preview = self._text[:20] + "..." if len(self._text) > 20 else self._text
        return f"삽입: '{preview}' (위치: {self._position})"


class DeleteTextCommand(TextCommand):
    """텍스트 삭제 커맨드"""

    def __init__(self, document: TextDocument, position: int, length: int) -> None:
        self._document = document
        self._position = position
        self._length = length
        self._deleted_text = ""

    def execute(self) -> None:
        self._deleted_text = self._document.delete(self._position, self._length)

    def undo(self) -> None:
        for i, char in enumerate(self._deleted_text):
            self._document.insert(self._position + i, char)

    def description(self) -> str:
        return f"삭제: '{self._deleted_text}' (위치: {self._position})"


class ReplaceTextCommand(TextCommand):
    """텍스트 교체 커맨드 (매크로 커맨드 예시)"""

    def __init__(
        self, document: TextDocument, position: int, length: int, new_text: str
    ) -> None:
        self._document = document
        self._delete_cmd = DeleteTextCommand(document, position, length)
        self._insert_cmd = InsertTextCommand(document, position, new_text)
        self._position = position
        self._new_text = new_text

    def execute(self) -> None:
        self._delete_cmd.execute()
        self._insert_cmd.execute()

    def undo(self) -> None:
        self._insert_cmd.undo()
        self._delete_cmd.undo()

    def description(self) -> str:
        return f"교체: -> '{self._new_text}' (위치: {self._position})"


class TextEditor:
    """텍스트 에디터 (Invoker)"""

    def __init__(self) -> None:
        self.document = TextDocument()
        self._history: list[TextCommand] = []
        self._redo_stack: list[TextCommand] = []

    def execute(self, command: TextCommand) -> None:
        command.execute()
        self._history.append(command)
        self._redo_stack.clear()
        print(f"  실행: {command.description()}")
        print(f"  문서: {self.document}")

    def undo(self) -> None:
        if not self._history:
            print("  Undo 불가: 히스토리가 비어있습니다.")
            return
        command = self._history.pop()
        command.undo()
        self._redo_stack.append(command)
        print(f"  Undo: {command.description()}")
        print(f"  문서: {self.document}")

    def redo(self) -> None:
        if not self._redo_stack:
            print("  Redo 불가: 스택이 비어있습니다.")
            return
        command = self._redo_stack.pop()
        command.execute()
        self._history.append(command)
        print(f"  Redo: {command.description()}")
        print(f"  문서: {self.document}")

    def show_history(self) -> None:
        print("\n--- 실행 히스토리 ---")
        for i, cmd in enumerate(self._history, 1):
            print(f"  {i}. {cmd.description()}")
        if not self._history:
            print("  (비어있음)")


# === 사용 예시 ===
if __name__ == "__main__":
    editor = TextEditor()

    print("=== 텍스트 입력 ===")
    editor.execute(InsertTextCommand(editor.document, 0, "Hello, "))
    editor.execute(InsertTextCommand(editor.document, 7, "World!"))

    print("\n=== 텍스트 교체 ===")
    editor.execute(ReplaceTextCommand(editor.document, 7, 6, "Python!"))

    print("\n=== Undo 2번 ===")
    editor.undo()
    editor.undo()

    print("\n=== Redo 1번 ===")
    editor.redo()

    editor.show_history()
```

### C# 구현

```csharp
using System;
using System.Collections.Generic;
using System.Text;

public class TextDocument
{
    private readonly StringBuilder _content = new();

    public string Text => _content.ToString();

    public void Insert(int position, string text)
    {
        _content.Insert(position, text);
    }

    public string Delete(int position, int length)
    {
        int actualLength = Math.Min(length, _content.Length - position);
        string deleted = _content.ToString(position, actualLength);
        _content.Remove(position, actualLength);
        return deleted;
    }

    public override string ToString() => _content.Length > 0 ? Text : "(빈 문서)";
}

public interface ITextCommand
{
    void Execute();
    void Undo();
    string Description { get; }
}

public class InsertTextCommand : ITextCommand
{
    private readonly TextDocument _document;
    private readonly int _position;
    private readonly string _text;

    public InsertTextCommand(TextDocument document, int position, string text)
    {
        _document = document;
        _position = position;
        _text = text;
    }

    public string Description =>
        $"삽입: '{(_text.Length > 20 ? _text[..20] + "..." : _text)}' (위치: {_position})";

    public void Execute() => _document.Insert(_position, _text);
    public void Undo() => _document.Delete(_position, _text.Length);
}

public class DeleteTextCommand : ITextCommand
{
    private readonly TextDocument _document;
    private readonly int _position;
    private readonly int _length;
    private string _deletedText = "";

    public DeleteTextCommand(TextDocument document, int position, int length)
    {
        _document = document;
        _position = position;
        _length = length;
    }

    public string Description => $"삭제: '{_deletedText}' (위치: {_position})";

    public void Execute()
    {
        _deletedText = _document.Delete(_position, _length);
    }

    public void Undo()
    {
        _document.Insert(_position, _deletedText);
    }
}

public class TextEditor
{
    public TextDocument Document { get; } = new();
    private readonly Stack<ITextCommand> _history = new();
    private readonly Stack<ITextCommand> _redoStack = new();

    public void Execute(ITextCommand command)
    {
        command.Execute();
        _history.Push(command);
        _redoStack.Clear();
        Console.WriteLine($"  실행: {command.Description}");
        Console.WriteLine($"  문서: {Document}");
    }

    public void Undo()
    {
        if (_history.Count == 0)
        {
            Console.WriteLine("  Undo 불가: 히스토리가 비어있습니다.");
            return;
        }
        var command = _history.Pop();
        command.Undo();
        _redoStack.Push(command);
        Console.WriteLine($"  Undo: {command.Description}");
        Console.WriteLine($"  문서: {Document}");
    }

    public void Redo()
    {
        if (_redoStack.Count == 0)
        {
            Console.WriteLine("  Redo 불가: 스택이 비어있습니다.");
            return;
        }
        var command = _redoStack.Pop();
        command.Execute();
        _history.Push(command);
        Console.WriteLine($"  Redo: {command.Description}");
        Console.WriteLine($"  문서: {Document}");
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **Undo/Redo** | 실행된 명령을 쉽게 취소하고 재실행할 수 있다 |
| **느슨한 결합** | Invoker와 Receiver 사이의 결합도를 낮춘다 |
| **단일 책임 원칙** | 작업을 호출하는 코드와 수행하는 코드를 분리한다 |
| **개방-폐쇄 원칙** | 기존 코드 수정 없이 새로운 명령을 추가할 수 있다 |
| **매크로 명령** | 여러 명령을 하나의 복합 명령으로 조합할 수 있다 |
| **지연 실행** | 명령을 큐에 넣어 나중에 실행할 수 있다 |

### 단점

| 단점 | 설명 |
|------|------|
| **클래스 증가** | 명령마다 별도의 클래스가 필요하여 코드량이 늘어난다 |
| **복잡성** | 간단한 작업에도 Command 객체를 만들어야 할 수 있다 |
| **상태 관리** | Undo를 위해 이전 상태를 저장해야 하므로 메모리 사용량 증가 |

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Memento** | Undo 구현 시 Command의 상태 저장에 Memento를 활용 |
| **Strategy** | 둘 다 객체를 매개변수화하지만, Strategy는 같은 일을 다른 방식으로, Command는 다른 일을 요청으로 캡슐화 |
| **Chain of Responsibility** | Command를 체인을 따라 전달할 수 있다 |
| **Composite** | 여러 Command를 묶어 매크로 명령(MacroCommand)을 만들 수 있다 |
| **Prototype** | Command 객체를 복제하여 히스토리에 저장할 수 있다 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

1. Command 패턴은 **요청을 독립적인 객체로 캡슐화**한다.
2. **Undo/Redo**를 구현하는 가장 자연스러운 패턴이다.
3. Invoker는 Command의 **구체적인 내용을 알 필요 없다**.
4. 실무에서 **트랜잭션, 작업 큐, 매크로, 이벤트 시스템** 등에 사용된다.

### 적용 체크리스트

- [ ] **Undo/Redo** 기능이 필요한가?
- [ ] 작업을 **큐에 넣어 나중에 실행**해야 하는가?
- [ ] 작업을 **로그에 기록**하여 장애 시 복구해야 하는가?
- [ ] 작업 호출부와 실행부를 **분리**하고 싶은가?
- [ ] 여러 작업을 하나의 **매크로 명령**으로 묶어야 하는가?
- [ ] 트랜잭션처럼 여러 작업을 **원자적으로 실행/롤백**해야 하는가?

> **2~3년차 개발자를 위한 팁**: Redux의 Action이 Command 패턴의 변형이다. `dispatch(action)` 호출 시 action 객체가 reducer에 전달되어 상태를 변경하고, 이를 통해 시간 여행 디버깅(Time Travel Debugging)이 가능해진다. 일상적으로 사용하는 Ctrl+Z도 이 패턴으로 구현된다.
