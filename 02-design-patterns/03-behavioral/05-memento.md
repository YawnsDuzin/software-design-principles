# Memento 패턴 (메멘토)

> **핵심 의도 한줄 요약**: 객체의 내부 상태를 외부에 저장하고 나중에 복원할 수 있게 하되, 캡슐화를 위반하지 않는다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 텍스트 에디터 히스토리](#7-실무-예제-텍스트-에디터-히스토리)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 개요

Memento 패턴은 **객체의 상태 스냅샷**을 만들어 별도의 객체(Memento)에 저장하고, 필요할 때 그 상태로 되돌릴 수 있게 하는 패턴이다.

핵심은 **캡슐화를 유지**한다는 것이다. 외부 객체(Caretaker)는 Memento의 내용을 읽거나 수정할 수 없고, 오직 Originator만이 Memento를 생성하고 복원할 수 있다.

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **Originator** | 자신의 상태를 Memento로 저장하고, Memento로부터 복원 |
| **Memento** | Originator의 상태 스냅샷을 불변(immutable)으로 보관 |
| **Caretaker** | Memento를 보관하는 관리자 (내용은 접근 불가) |

---

## 2. 문제 상황

### 문제: 객체 상태 복원

게임 캐릭터의 상태를 저장하고 복원하고 싶다.

```python
# 나쁜 예: 캡슐화 위반
class GameCharacter:
    def __init__(self):
        self.health = 100
        self.mana = 50
        self.position = (0, 0)
        self._secret_buff = 1.5  # 내부 상태

# 외부에서 직접 상태를 복사
save_data = {
    "health": character.health,
    "mana": character.mana,
    "position": character.position,
    # _secret_buff는 접근 불가 → 불완전한 저장!
}
```

문제점:

- **캡슐화 위반**: 외부에서 내부 상태에 직접 접근해야 한다
- **불완전한 저장**: private 속성은 저장할 수 없다
- **유지보수 어려움**: 필드가 추가/변경되면 저장 로직도 수정해야 한다

---

## 3. 일상 비유

**게임의 세이브/로드 시스템**을 떠올려 보자.

```
[플레이어]                 [세이브 파일]
  레벨: 25                 세이브 슬롯 1: (레벨 25, HP 500...)
  HP: 500     ──저장──→    세이브 슬롯 2: (레벨 20, HP 300...)
  위치: 던전                세이브 슬롯 3: (레벨 15, HP 200...)

  보스전 패배!

  세이브 슬롯 1 ──로드──→  레벨: 25, HP: 500, 위치: 던전
```

- **세이브 파일**이 Memento다
- 플레이어(Originator)가 자신의 상태를 세이브 파일에 저장한다
- 게임 시스템(Caretaker)은 세이브 파일을 보관하지만 내용을 수정하지 않는다
- 로드 시 세이브 파일의 내용으로 **완전히 복원**된다

---

## 4. UML 다이어그램

```
 ┌─────────────┐      creates      ┌──────────────────┐
 │  Originator │──────────────────>│     Memento       │
 ├─────────────┤                   ├──────────────────┤
 │ - state     │                   │ - state (private)│
 ├─────────────┤                   ├──────────────────┤
 │ + save()    │   restores from   │ + get_state()    │
 │ + restore() │<──────────────────│   (Originator    │
 └─────────────┘                   │    only)         │
                                   └──────────────────┘
                                           ▲
                                           │ stores
                                   ┌───────┴──────────┐
                                   │    Caretaker      │
                                   ├──────────────────┤
                                   │ - mementos[]     │
                                   ├──────────────────┤
                                   │ + backup()       │
                                   │ + undo()         │
                                   │ + show_history() │
                                   └──────────────────┘

 Caretaker는 Memento의 내용에 접근할 수 없다 (캡슐화 유지).
```

---

## 5. Python 구현

```python
from __future__ import annotations
from dataclasses import dataclass
from datetime import datetime
from copy import deepcopy


class Memento:
    """메멘토: 상태 스냅샷을 불변으로 보관"""

    def __init__(self, state: dict, description: str = "") -> None:
        self._state = deepcopy(state)  # 깊은 복사로 독립적인 스냅샷 생성
        self._timestamp = datetime.now()
        self._description = description

    @property
    def state(self) -> dict:
        return deepcopy(self._state)

    @property
    def timestamp(self) -> datetime:
        return self._timestamp

    @property
    def description(self) -> str:
        return self._description

    def __str__(self) -> str:
        time_str = self._timestamp.strftime("%H:%M:%S")
        return f"[{time_str}] {self._description}"


class GameCharacter:
    """게임 캐릭터 (Originator)"""

    def __init__(self, name: str) -> None:
        self.name = name
        self._level = 1
        self._health = 100
        self._mana = 50
        self._position = (0, 0)
        self._inventory: list[str] = []

    def level_up(self) -> None:
        self._level += 1
        self._health += 20
        self._mana += 10
        print(f"  레벨 업! Lv.{self._level} (HP:{self._health}, MP:{self._mana})")

    def take_damage(self, damage: int) -> None:
        self._health = max(0, self._health - damage)
        print(f"  {damage} 데미지! (HP: {self._health})")

    def move(self, x: int, y: int) -> None:
        self._position = (x, y)
        print(f"  이동: {self._position}")

    def add_item(self, item: str) -> None:
        self._inventory.append(item)
        print(f"  아이템 획득: {item}")

    def save(self, description: str = "") -> Memento:
        """현재 상태를 Memento로 저장"""
        state = {
            "level": self._level,
            "health": self._health,
            "mana": self._mana,
            "position": self._position,
            "inventory": self._inventory.copy(),
        }
        return Memento(state, description or f"Lv.{self._level} 세이브")

    def restore(self, memento: Memento) -> None:
        """Memento로부터 상태 복원"""
        state = memento.state
        self._level = state["level"]
        self._health = state["health"]
        self._mana = state["mana"]
        self._position = state["position"]
        self._inventory = state["inventory"]
        print(f"  상태 복원 완료! ({memento})")

    def status(self) -> None:
        print(f"  [{self.name}] Lv.{self._level} | "
              f"HP:{self._health} | MP:{self._mana} | "
              f"위치:{self._position} | "
              f"인벤토리:{self._inventory}")


class SaveManager:
    """세이브 관리자 (Caretaker)"""

    def __init__(self, character: GameCharacter) -> None:
        self._character = character
        self._saves: list[Memento] = []
        self._max_saves = 10

    def quick_save(self, description: str = "") -> None:
        memento = self._character.save(description)
        self._saves.append(memento)
        if len(self._saves) > self._max_saves:
            self._saves.pop(0)  # 가장 오래된 세이브 삭제
        print(f"  저장 완료: {memento}")

    def quick_load(self) -> None:
        if not self._saves:
            print("  세이브 파일이 없습니다!")
            return
        memento = self._saves[-1]
        self._character.restore(memento)

    def load(self, index: int) -> None:
        if 0 <= index < len(self._saves):
            self._character.restore(self._saves[index])
        else:
            print(f"  잘못된 세이브 슬롯: {index}")

    def undo(self) -> None:
        """가장 최근 세이브 삭제 후 그 이전으로 복원"""
        if len(self._saves) < 2:
            print("  되돌릴 수 없습니다.")
            return
        self._saves.pop()  # 현재 세이브 삭제
        self._character.restore(self._saves[-1])

    def show_saves(self) -> None:
        print("\n--- 세이브 목록 ---")
        for i, save in enumerate(self._saves):
            print(f"  슬롯 {i}: {save}")
        if not self._saves:
            print("  (비어있음)")


# === 사용 예시 ===
if __name__ == "__main__":
    hero = GameCharacter("용사")
    save_mgr = SaveManager(hero)

    print("=== 모험 시작 ===")
    hero.status()
    save_mgr.quick_save("모험 시작")

    print("\n=== 레벨업 & 아이템 획득 ===")
    hero.level_up()
    hero.add_item("철검")
    hero.move(10, 20)
    save_mgr.quick_save("던전 입구")

    print("\n=== 보스전 ===")
    hero.level_up()
    hero.add_item("마법 방패")
    save_mgr.quick_save("보스전 직전")
    hero.take_damage(999)
    print("  *** 사망! ***")
    hero.status()

    print("\n=== 로드: 보스전 직전으로 복원 ===")
    save_mgr.quick_load()
    hero.status()

    save_mgr.show_saves()
```

---

## 6. C# 구현

```csharp
using System;
using System.Collections.Generic;

// Memento: 상태 스냅샷
public class GameMemento
{
    public int Level { get; }
    public int Health { get; }
    public int Mana { get; }
    public (int X, int Y) Position { get; }
    public List<string> Inventory { get; }
    public DateTime Timestamp { get; }
    public string Description { get; }

    public GameMemento(
        int level, int health, int mana,
        (int, int) position, List<string> inventory,
        string description = "")
    {
        Level = level;
        Health = health;
        Mana = mana;
        Position = position;
        Inventory = new List<string>(inventory); // 방어적 복사
        Timestamp = DateTime.Now;
        Description = description;
    }

    public override string ToString()
        => $"[{Timestamp:HH:mm:ss}] {Description}";
}

// Originator: 게임 캐릭터
public class GameCharacter
{
    public string Name { get; }
    private int _level = 1;
    private int _health = 100;
    private int _mana = 50;
    private (int X, int Y) _position = (0, 0);
    private readonly List<string> _inventory = new();

    public GameCharacter(string name) => Name = name;

    public void LevelUp()
    {
        _level++;
        _health += 20;
        _mana += 10;
        Console.WriteLine($"  레벨 업! Lv.{_level} (HP:{_health}, MP:{_mana})");
    }

    public void TakeDamage(int damage)
    {
        _health = Math.Max(0, _health - damage);
        Console.WriteLine($"  {damage} 데미지! (HP: {_health})");
    }

    public void Move(int x, int y)
    {
        _position = (x, y);
        Console.WriteLine($"  이동: ({x}, {y})");
    }

    public void AddItem(string item)
    {
        _inventory.Add(item);
        Console.WriteLine($"  아이템 획득: {item}");
    }

    // Memento 생성
    public GameMemento Save(string description = "")
    {
        return new GameMemento(
            _level, _health, _mana, _position,
            _inventory,
            string.IsNullOrEmpty(description)
                ? $"Lv.{_level} 세이브"
                : description
        );
    }

    // Memento에서 복원
    public void Restore(GameMemento memento)
    {
        _level = memento.Level;
        _health = memento.Health;
        _mana = memento.Mana;
        _position = memento.Position;
        _inventory.Clear();
        _inventory.AddRange(memento.Inventory);
        Console.WriteLine($"  상태 복원 완료! ({memento})");
    }

    public void ShowStatus()
    {
        Console.WriteLine(
            $"  [{Name}] Lv.{_level} | HP:{_health} | MP:{_mana} | " +
            $"위치:({_position.X},{_position.Y}) | " +
            $"인벤토리:[{string.Join(", ", _inventory)}]");
    }
}

// Caretaker: 세이브 관리자
public class SaveManager
{
    private readonly GameCharacter _character;
    private readonly List<GameMemento> _saves = new();
    private readonly int _maxSaves;

    public SaveManager(GameCharacter character, int maxSaves = 10)
    {
        _character = character;
        _maxSaves = maxSaves;
    }

    public void QuickSave(string description = "")
    {
        var memento = _character.Save(description);
        _saves.Add(memento);
        if (_saves.Count > _maxSaves)
            _saves.RemoveAt(0);
        Console.WriteLine($"  저장 완료: {memento}");
    }

    public void QuickLoad()
    {
        if (_saves.Count == 0)
        {
            Console.WriteLine("  세이브 파일이 없습니다!");
            return;
        }
        _character.Restore(_saves[^1]);
    }

    public void Load(int index)
    {
        if (index >= 0 && index < _saves.Count)
            _character.Restore(_saves[index]);
        else
            Console.WriteLine($"  잘못된 세이브 슬롯: {index}");
    }

    public void ShowSaves()
    {
        Console.WriteLine("\n--- 세이브 목록 ---");
        for (int i = 0; i < _saves.Count; i++)
            Console.WriteLine($"  슬롯 {i}: {_saves[i]}");
        if (_saves.Count == 0)
            Console.WriteLine("  (비어있음)");
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var hero = new GameCharacter("용사");
        var saveMgr = new SaveManager(hero);

        Console.WriteLine("=== 모험 시작 ===");
        hero.ShowStatus();
        saveMgr.QuickSave("모험 시작");

        Console.WriteLine("\n=== 레벨업 & 아이템 획득 ===");
        hero.LevelUp();
        hero.AddItem("철검");
        hero.Move(10, 20);
        saveMgr.QuickSave("던전 입구");

        Console.WriteLine("\n=== 보스전 ===");
        hero.LevelUp();
        saveMgr.QuickSave("보스전 직전");
        hero.TakeDamage(999);
        Console.WriteLine("  *** 사망! ***");
        hero.ShowStatus();

        Console.WriteLine("\n=== 로드: 보스전 직전으로 복원 ===");
        saveMgr.QuickLoad();
        hero.ShowStatus();

        saveMgr.ShowSaves();
    }
}
```

---

## 7. 실무 예제: 텍스트 에디터 히스토리

### Python 구현

```python
from __future__ import annotations
from dataclasses import dataclass, field
from datetime import datetime
from copy import deepcopy


@dataclass(frozen=True)
class EditorMemento:
    """에디터 상태 스냅샷 (불변)"""
    content: str
    cursor_position: int
    selection_start: int | None
    selection_end: int | None
    timestamp: str = field(default_factory=lambda: datetime.now().strftime("%H:%M:%S"))

    def __str__(self) -> str:
        preview = self.content[:30] + "..." if len(self.content) > 30 else self.content
        return f"[{self.timestamp}] \"{preview}\" (커서: {self.cursor_position})"


class TextEditor:
    """텍스트 에디터 (Originator)"""

    def __init__(self) -> None:
        self._content = ""
        self._cursor_position = 0
        self._selection_start: int | None = None
        self._selection_end: int | None = None

    @property
    def content(self) -> str:
        return self._content

    def type_text(self, text: str) -> None:
        """커서 위치에 텍스트 입력"""
        before = self._content[:self._cursor_position]
        after = self._content[self._cursor_position:]
        self._content = before + text + after
        self._cursor_position += len(text)
        self._clear_selection()

    def delete(self, count: int = 1) -> None:
        """커서 앞의 텍스트 삭제 (Backspace)"""
        if self._cursor_position == 0:
            return
        start = max(0, self._cursor_position - count)
        before = self._content[:start]
        after = self._content[self._cursor_position:]
        self._content = before + after
        self._cursor_position = start

    def move_cursor(self, position: int) -> None:
        """커서 이동"""
        self._cursor_position = max(0, min(position, len(self._content)))

    def select(self, start: int, end: int) -> None:
        """텍스트 선택"""
        self._selection_start = max(0, start)
        self._selection_end = min(end, len(self._content))

    def _clear_selection(self) -> None:
        self._selection_start = None
        self._selection_end = None

    # --- Memento 관련 ---
    def save(self) -> EditorMemento:
        """현재 상태를 Memento로 저장"""
        return EditorMemento(
            content=self._content,
            cursor_position=self._cursor_position,
            selection_start=self._selection_start,
            selection_end=self._selection_end,
        )

    def restore(self, memento: EditorMemento) -> None:
        """Memento로부터 상태 복원"""
        self._content = memento.content
        self._cursor_position = memento.cursor_position
        self._selection_start = memento.selection_start
        self._selection_end = memento.selection_end

    def __str__(self) -> str:
        return f'"{self._content}" (커서: {self._cursor_position})'


class EditorHistory:
    """에디터 히스토리 (Caretaker)"""

    def __init__(self, editor: TextEditor) -> None:
        self._editor = editor
        self._undo_stack: list[EditorMemento] = []
        self._redo_stack: list[EditorMemento] = []
        # 초기 상태 저장
        self._undo_stack.append(editor.save())

    def save_state(self) -> None:
        """현재 상태를 히스토리에 저장"""
        self._undo_stack.append(self._editor.save())
        self._redo_stack.clear()

    def undo(self) -> bool:
        """Undo"""
        if len(self._undo_stack) <= 1:
            print("  Undo 불가: 더 이상 되돌릴 수 없습니다.")
            return False
        current = self._undo_stack.pop()
        self._redo_stack.append(current)
        self._editor.restore(self._undo_stack[-1])
        return True

    def redo(self) -> bool:
        """Redo"""
        if not self._redo_stack:
            print("  Redo 불가: 다시 실행할 수 없습니다.")
            return False
        memento = self._redo_stack.pop()
        self._undo_stack.append(memento)
        self._editor.restore(memento)
        return True

    def show_history(self) -> None:
        print("\n--- Undo 히스토리 ---")
        for i, m in enumerate(self._undo_stack):
            marker = " ← 현재" if i == len(self._undo_stack) - 1 else ""
            print(f"  {i}. {m}{marker}")


# === 사용 예시 ===
if __name__ == "__main__":
    editor = TextEditor()
    history = EditorHistory(editor)

    # 텍스트 입력
    print("=== 텍스트 입력 ===")
    editor.type_text("Hello")
    history.save_state()
    print(f"  {editor}")

    editor.type_text(", ")
    history.save_state()
    print(f"  {editor}")

    editor.type_text("World!")
    history.save_state()
    print(f"  {editor}")

    # Undo
    print("\n=== Undo 3번 ===")
    for _ in range(3):
        history.undo()
        print(f"  {editor}")

    # Redo
    print("\n=== Redo 2번 ===")
    for _ in range(2):
        history.redo()
        print(f"  {editor}")

    # 새 입력 (Redo 스택 초기화됨)
    print("\n=== 새 텍스트 입력 (Redo 불가 됨) ===")
    editor.type_text("Python!")
    history.save_state()
    print(f"  {editor}")
    history.redo()  # Redo 불가

    history.show_history()
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **캡슐화 유지** | 객체의 내부 상태를 외부에 노출하지 않고 저장/복원 |
| **Undo/Redo** | 상태 히스토리를 관리하여 실행 취소/재실행 지원 |
| **스냅샷** | 특정 시점의 상태를 독립적으로 보관 |
| **단순한 Originator** | 상태 관리 로직을 Caretaker에 위임 |

### 단점

| 단점 | 설명 |
|------|------|
| **메모리 사용** | 상태 스냅샷이 클수록, 히스토리가 길수록 메모리 소비 증가 |
| **성능** | 깊은 복사(Deep Copy)는 비용이 클 수 있다 |
| **직렬화** | 복잡한 객체의 상태를 완전히 직렬화하기 어려울 수 있다 |
| **관리 부담** | Caretaker가 오래된 Memento를 적절히 삭제해야 한다 |

### 메모리 최적화 전략

```python
# 전략 1: 최대 개수 제한
if len(self._saves) > MAX_SAVES:
    self._saves.pop(0)

# 전략 2: 차이점만 저장 (Delta/Diff 방식)
# 전체 상태 대신 변경된 부분만 저장

# 전략 3: 압축
import zlib
compressed = zlib.compress(state_bytes)
```

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Command** | Command의 Undo 구현에 Memento를 사용하여 이전 상태 저장 |
| **Iterator** | Memento로 Iterator의 현재 위치를 저장/복원 |
| **Prototype** | 객체 복제와 유사하나, Memento는 캡슐화를 더 강조 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

1. Memento는 **캡슐화를 유지하면서** 객체 상태를 저장/복원한다.
2. Originator만 Memento를 **생성하고 복원**할 수 있다.
3. Caretaker는 Memento를 **보관만** 하고 내용에는 접근하지 않는다.
4. Command 패턴의 Undo와 함께 사용하면 **강력한 실행 취소 시스템**을 구축할 수 있다.

### 적용 체크리스트

- [ ] 객체의 **상태를 저장하고 복원**해야 하는가?
- [ ] 상태 저장 시 **캡슐화를 유지**해야 하는가?
- [ ] **Undo/Redo** 기능이 필요한가?
- [ ] 특정 시점의 **스냅샷/체크포인트**가 필요한가?
- [ ] 상태 변경 전 **롤백 지점**을 만들어야 하는가?

> **2~3년차 개발자를 위한 팁**: Git의 커밋이 일종의 Memento다. 각 커밋은 프로젝트의 특정 시점 상태를 저장하며, `git checkout`으로 이전 상태를 복원할 수 있다. 데이터베이스의 트랜잭션 로그, 브라우저의 뒤로 가기 기능도 Memento 패턴의 응용이다.
