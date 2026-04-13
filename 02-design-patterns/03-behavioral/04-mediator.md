# Mediator 패턴 (중재자)

> **핵심 의도 한줄 요약**: 객체들 사이의 복잡한 상호작용을 하나의 중재자 객체로 캡슐화하여, 객체들이 직접 참조하지 않고도 협력할 수 있게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [Mediator vs Observer 차이점](#7-mediator-vs-observer-차이점)
8. [실무 예제: 채팅방 시스템](#8-실무-예제-채팅방-시스템)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 개요

Mediator 패턴은 **객체들 간의 직접적인 통신을 금지**하고, 모든 상호작용을 중재자(Mediator)를 통해서만 하도록 강제하는 패턴이다.

N개의 객체가 서로 직접 통신하면 N*(N-1)/2 개의 연결이 필요하다. Mediator를 도입하면 각 객체는 Mediator와만 연결하면 되므로 연결 수가 N개로 줄어든다.

```
직접 통신 (4개 객체)          Mediator 사용 (4개 객체)

  A ←──→ B                    A ──→ ┌──────────┐ ←── B
  ↕ ╲  ╱ ↕                         │ Mediator │
  ↕  ╳   ↕                         └──────────┘
  ↕ ╱  ╲ ↕                    C ──→     ↑       ←── D
  C ←──→ D
                              연결: 4개
  연결: 6개
```

---

## 2. 문제 상황

### 문제: UI 컴포넌트 간 상호작용

로그인 폼을 생각해 보자. 아이디 입력, 비밀번호 입력, "로그인 기억" 체크박스, 로그인 버튼이 있다.

```python
# 나쁜 예: 컴포넌트들이 서로 직접 참조
class LoginButton:
    def __init__(self, username_input, password_input, remember_checkbox):
        self.username_input = username_input
        self.password_input = password_input
        self.remember_checkbox = remember_checkbox

    def click(self):
        if self.username_input.text and self.password_input.text:
            # 로그인 처리
            if self.remember_checkbox.is_checked:
                save_credentials(...)
        # 버튼이 다른 모든 컴포넌트를 알고 있어야 한다!
```

문제점:

- 컴포넌트끼리 **강하게 결합**되어 있다
- 새 컴포넌트 추가 시 **기존 컴포넌트도 수정**해야 한다
- 컴포넌트를 **독립적으로 재사용**할 수 없다

---

## 3. 일상 비유

**채팅방**을 떠올려 보자.

```
[채팅방] ← Mediator

 Alice: "안녕하세요!"
   ↓
 채팅방이 Bob, Charlie에게 전달
   ↓
 Bob: "반갑습니다!"
   ↓
 채팅방이 Alice, Charlie에게 전달
```

- 사용자들은 **서로의 연락처를 몰라도** 채팅방을 통해 대화한다
- 채팅방이 **메시지 라우팅, 필터링, 로깅** 등을 중앙에서 처리한다
- 새로운 사용자가 입장해도 **기존 사용자의 코드는 변경 없다**

비행기 관제탑도 좋은 비유다:

```
비행기들이 서로 직접 통신하지 않고,
관제탑(Mediator)을 통해서만 이착륙 순서를 조율한다.
```

---

## 4. UML 다이어그램

```
                  ┌──────────────────────┐
                  │   <<interface>>      │
                  │     Mediator         │
                  ├──────────────────────┤
                  │ + notify(sender,     │
                  │         event)       │
                  └──────────┬───────────┘
                             │
                  ┌──────────▼───────────┐
                  │  ConcreteMediator    │
                  ├─────────────────────┤
                  │ - componentA         │
                  │ - componentB         │
                  │ - componentC         │
                  │ + notify(sender,     │
                  │         event)       │
                  └──────────────────────┘
                     ▲    ▲    ▲
                     │    │    │
          ┌──────────┘    │    └──────────┐
          │               │               │
 ┌────────┴───┐  ┌───────┴────┐  ┌───────┴────┐
 │ ComponentA │  │ ComponentB │  │ ComponentC │
 ├────────────┤  ├────────────┤  ├────────────┤
 │ - mediator │  │ - mediator │  │ - mediator │
 │ + action() │  │ + action() │  │ + action() │
 └────────────┘  └────────────┘  └────────────┘

 각 Component는 Mediator만 알고, 다른 Component는 모른다.
```

---

## 5. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class Mediator(ABC):
    """중재자 인터페이스"""

    @abstractmethod
    def notify(self, sender: Component, event: str) -> None:
        pass


class Component:
    """기본 컴포넌트 (Mediator 참조 보유)"""

    def __init__(self, name: str) -> None:
        self.name = name
        self._mediator: Mediator | None = None

    @property
    def mediator(self) -> Mediator | None:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -> None:
        self._mediator = mediator


class TextInput(Component):
    """텍스트 입력 컴포넌트"""

    def __init__(self, name: str) -> None:
        super().__init__(name)
        self.text = ""

    def set_text(self, text: str) -> None:
        self.text = text
        print(f"  [{self.name}] 텍스트 입력: '{text}'")
        if self.mediator:
            self.mediator.notify(self, "text_changed")


class CheckBox(Component):
    """체크박스 컴포넌트"""

    def __init__(self, name: str) -> None:
        super().__init__(name)
        self.is_checked = False

    def toggle(self) -> None:
        self.is_checked = not self.is_checked
        state = "체크됨" if self.is_checked else "해제됨"
        print(f"  [{self.name}] {state}")
        if self.mediator:
            self.mediator.notify(self, "checkbox_toggled")


class Button(Component):
    """버튼 컴포넌트"""

    def __init__(self, name: str) -> None:
        super().__init__(name)
        self.enabled = False

    def click(self) -> None:
        if not self.enabled:
            print(f"  [{self.name}] 비활성화 상태 - 클릭 불가")
            return
        print(f"  [{self.name}] 클릭!")
        if self.mediator:
            self.mediator.notify(self, "button_clicked")

    def set_enabled(self, enabled: bool) -> None:
        self.enabled = enabled
        state = "활성화" if enabled else "비활성화"
        print(f"  [{self.name}] {state}")


class Label(Component):
    """라벨 컴포넌트"""

    def __init__(self, name: str) -> None:
        super().__init__(name)
        self.text = ""

    def set_text(self, text: str) -> None:
        self.text = text
        print(f"  [{self.name}] '{text}'")


class LoginFormMediator(Mediator):
    """로그인 폼 중재자"""

    def __init__(
        self,
        username: TextInput,
        password: TextInput,
        remember: CheckBox,
        login_btn: Button,
        status_label: Label,
    ) -> None:
        self._username = username
        self._password = password
        self._remember = remember
        self._login_btn = login_btn
        self._status_label = status_label

        # 각 컴포넌트에 중재자 등록
        username.mediator = self
        password.mediator = self
        remember.mediator = self
        login_btn.mediator = self
        status_label.mediator = self

    def notify(self, sender: Component, event: str) -> None:
        if event == "text_changed":
            self._validate_form()
        elif event == "checkbox_toggled":
            if self._remember.is_checked:
                self._status_label.set_text("로그인 정보가 저장됩니다.")
            else:
                self._status_label.set_text("")
        elif event == "button_clicked":
            self._do_login()

    def _validate_form(self) -> None:
        """폼 유효성 검사"""
        is_valid = bool(self._username.text and self._password.text)
        self._login_btn.set_enabled(is_valid)

    def _do_login(self) -> None:
        """로그인 처리"""
        user = self._username.text
        remember = "저장" if self._remember.is_checked else "미저장"
        self._status_label.set_text(f"'{user}' 로그인 성공! (정보 {remember})")


# === 사용 예시 ===
if __name__ == "__main__":
    # 컴포넌트 생성
    username = TextInput("아이디")
    password = TextInput("비밀번호")
    remember = CheckBox("로그인 기억")
    login_btn = Button("로그인")
    status = Label("상태")

    # 중재자로 연결
    mediator = LoginFormMediator(username, password, remember, login_btn, status)

    print("=== 아이디만 입력 ===")
    username.set_text("hong")

    print("\n=== 로그인 버튼 클릭 시도 (비밀번호 없음) ===")
    login_btn.click()

    print("\n=== 비밀번호 입력 ===")
    password.set_text("1234")

    print("\n=== 로그인 기억 체크 ===")
    remember.toggle()

    print("\n=== 로그인 버튼 클릭 ===")
    login_btn.click()
```

---

## 6. C# 구현

```csharp
using System;

// 중재자 인터페이스
public interface IMediator
{
    void Notify(Component sender, string eventName);
}

// 기본 컴포넌트
public abstract class Component
{
    public string Name { get; }
    public IMediator? Mediator { get; set; }

    protected Component(string name)
    {
        Name = name;
    }
}

// 텍스트 입력
public class TextInput : Component
{
    public string Text { get; private set; } = "";

    public TextInput(string name) : base(name) { }

    public void SetText(string text)
    {
        Text = text;
        Console.WriteLine($"  [{Name}] 텍스트 입력: '{text}'");
        Mediator?.Notify(this, "text_changed");
    }
}

// 체크박스
public class CheckBox : Component
{
    public bool IsChecked { get; private set; }

    public CheckBox(string name) : base(name) { }

    public void Toggle()
    {
        IsChecked = !IsChecked;
        var state = IsChecked ? "체크됨" : "해제됨";
        Console.WriteLine($"  [{Name}] {state}");
        Mediator?.Notify(this, "checkbox_toggled");
    }
}

// 버튼
public class Button : Component
{
    public bool Enabled { get; private set; }

    public Button(string name) : base(name) { }

    public void Click()
    {
        if (!Enabled)
        {
            Console.WriteLine($"  [{Name}] 비활성화 상태 - 클릭 불가");
            return;
        }
        Console.WriteLine($"  [{Name}] 클릭!");
        Mediator?.Notify(this, "button_clicked");
    }

    public void SetEnabled(bool enabled)
    {
        Enabled = enabled;
        var state = enabled ? "활성화" : "비활성화";
        Console.WriteLine($"  [{Name}] {state}");
    }
}

// 라벨
public class Label : Component
{
    public string Text { get; private set; } = "";

    public Label(string name) : base(name) { }

    public void SetText(string text)
    {
        Text = text;
        Console.WriteLine($"  [{Name}] '{text}'");
    }
}

// 로그인 폼 중재자
public class LoginFormMediator : IMediator
{
    private readonly TextInput _username;
    private readonly TextInput _password;
    private readonly CheckBox _remember;
    private readonly Button _loginBtn;
    private readonly Label _statusLabel;

    public LoginFormMediator(
        TextInput username,
        TextInput password,
        CheckBox remember,
        Button loginBtn,
        Label statusLabel)
    {
        _username = username;
        _password = password;
        _remember = remember;
        _loginBtn = loginBtn;
        _statusLabel = statusLabel;

        // 컴포넌트에 중재자 등록
        _username.Mediator = this;
        _password.Mediator = this;
        _remember.Mediator = this;
        _loginBtn.Mediator = this;
        _statusLabel.Mediator = this;
    }

    public void Notify(Component sender, string eventName)
    {
        switch (eventName)
        {
            case "text_changed":
                ValidateForm();
                break;
            case "checkbox_toggled":
                if (_remember.IsChecked)
                    _statusLabel.SetText("로그인 정보가 저장됩니다.");
                else
                    _statusLabel.SetText("");
                break;
            case "button_clicked":
                DoLogin();
                break;
        }
    }

    private void ValidateForm()
    {
        bool isValid = !string.IsNullOrEmpty(_username.Text)
                    && !string.IsNullOrEmpty(_password.Text);
        _loginBtn.SetEnabled(isValid);
    }

    private void DoLogin()
    {
        var remember = _remember.IsChecked ? "저장" : "미저장";
        _statusLabel.SetText($"'{_username.Text}' 로그인 성공! (정보 {remember})");
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var username = new TextInput("아이디");
        var password = new TextInput("비밀번호");
        var remember = new CheckBox("로그인 기억");
        var loginBtn = new Button("로그인");
        var status = new Label("상태");

        var mediator = new LoginFormMediator(
            username, password, remember, loginBtn, status);

        Console.WriteLine("=== 아이디만 입력 ===");
        username.SetText("hong");

        Console.WriteLine("\n=== 비밀번호 입력 ===");
        password.SetText("1234");

        Console.WriteLine("\n=== 로그인 기억 체크 ===");
        remember.Toggle();

        Console.WriteLine("\n=== 로그인 ===");
        loginBtn.Click();
    }
}
```

---

## 7. Mediator vs Observer 차이점

| 구분 | Mediator | Observer |
|------|----------|----------|
| **통신 방향** | 양방향 (컴포넌트 ↔ 중재자) | 단방향 (Subject → Observer) |
| **목적** | 복잡한 상호작용을 중앙 집중화 | 상태 변화를 관찰자에게 알림 |
| **결합도** | 컴포넌트는 중재자만 안다 | Observer는 Subject만 안다 |
| **지식** | 중재자가 모든 컴포넌트를 안다 | Subject는 Observer 인터페이스만 안다 |
| **복잡성 위치** | 중재자에 집중 | 분산되어 있음 |
| **예시** | 채팅방, 관제탑, UI 폼 | 이벤트 시스템, 구독 알림 |

### 핵심 차이

```
Mediator:
  컴포넌트A ──→ [Mediator] ──→ 컴포넌트B
  컴포넌트B ──→ [Mediator] ──→ 컴포넌트A
  (중재자가 "누구에게 어떻게" 전달할지 결정)

Observer:
  Subject ──→ ObserverA
          ──→ ObserverB
          ──→ ObserverC
  (모든 구독자에게 동일하게 알림)
```

실무에서 Mediator는 종종 Observer 패턴을 내부적으로 사용하여 구현된다.

---

## 8. 실무 예제: 채팅방 시스템

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime


class ChatMediator(ABC):
    """채팅 중재자 인터페이스"""

    @abstractmethod
    def send_message(self, message: str, sender: User) -> None:
        pass

    @abstractmethod
    def add_user(self, user: User) -> None:
        pass

    @abstractmethod
    def remove_user(self, user: User) -> None:
        pass


class User:
    """채팅 사용자"""

    def __init__(self, name: str) -> None:
        self.name = name
        self._chatroom: ChatMediator | None = None
        self._messages: list[str] = []

    def join(self, chatroom: ChatMediator) -> None:
        self._chatroom = chatroom
        chatroom.add_user(self)

    def leave(self) -> None:
        if self._chatroom:
            self._chatroom.remove_user(self)
            self._chatroom = None

    def send(self, message: str) -> None:
        if self._chatroom:
            print(f"  {self.name}: {message}")
            self._chatroom.send_message(message, self)
        else:
            print(f"  {self.name}: (채팅방에 참여하지 않음)")

    def receive(self, message: str, sender_name: str) -> None:
        formatted = f"  [{sender_name} → {self.name}] {message}"
        self._messages.append(formatted)
        print(formatted)

    def show_history(self) -> None:
        print(f"\n--- {self.name}의 메시지 히스토리 ---")
        for msg in self._messages:
            print(msg)


class ChatRoom(ChatMediator):
    """채팅방 (ConcreteMediator)"""

    def __init__(self, name: str) -> None:
        self.name = name
        self._users: list[User] = []
        self._message_log: list[dict] = []

    def add_user(self, user: User) -> None:
        self._users.append(user)
        self._broadcast_system(f"'{user.name}'님이 입장했습니다.")

    def remove_user(self, user: User) -> None:
        self._users.remove(user)
        self._broadcast_system(f"'{user.name}'님이 퇴장했습니다.")

    def send_message(self, message: str, sender: User) -> None:
        self._message_log.append({
            "time": datetime.now().strftime("%H:%M:%S"),
            "sender": sender.name,
            "message": message,
        })
        # 발신자를 제외한 모든 사용자에게 전달
        for user in self._users:
            if user != sender:
                user.receive(message, sender.name)

    def _broadcast_system(self, message: str) -> None:
        print(f"  [시스템] {message}")

    def show_log(self) -> None:
        print(f"\n--- {self.name} 채팅 로그 ---")
        for entry in self._message_log:
            print(f"  [{entry['time']}] {entry['sender']}: {entry['message']}")


class PrivateChatRoom(ChatMediator):
    """1:1 채팅방 (최대 2명)"""

    def __init__(self) -> None:
        self._users: list[User] = []

    def add_user(self, user: User) -> None:
        if len(self._users) >= 2:
            print(f"  [시스템] 1:1 채팅방이 꽉 찼습니다.")
            return
        self._users.append(user)

    def remove_user(self, user: User) -> None:
        self._users.remove(user)

    def send_message(self, message: str, sender: User) -> None:
        for user in self._users:
            if user != sender:
                user.receive(message, sender.name)


# === 사용 예시 ===
if __name__ == "__main__":
    # 채팅방 생성
    room = ChatRoom("개발자 라운지")

    # 사용자 생성 및 입장
    alice = User("Alice")
    bob = User("Bob")
    charlie = User("Charlie")

    print("=== 채팅방 입장 ===")
    alice.join(room)
    bob.join(room)
    charlie.join(room)

    print("\n=== 대화 ===")
    alice.send("안녕하세요!")
    bob.send("반갑습니다!")
    charlie.send("저도요!")

    print("\n=== Bob 퇴장 후 대화 ===")
    bob.leave()
    alice.send("Bob이 나갔네요.")

    room.show_log()
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **느슨한 결합** | 컴포넌트들이 서로를 직접 참조하지 않는다 |
| **단일 책임 원칙** | 통신 로직이 중재자에 집중된다 |
| **개방-폐쇄 원칙** | 새 컴포넌트 추가 시 기존 컴포넌트 수정 불필요 |
| **재사용성** | 컴포넌트를 다른 중재자와 함께 재사용 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **God Object 위험** | 중재자가 너무 많은 로직을 담당하게 될 수 있다 |
| **복잡성 이동** | 분산된 복잡성이 중재자로 집중된다 |
| **디버깅** | 통신 흐름이 간접적이므로 추적이 어려울 수 있다 |

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Observer** | Mediator 내부에서 Observer를 사용하여 컴포넌트와 통신 가능 |
| **Facade** | Facade는 서브시스템을 단순화하고, Mediator는 양방향 통신을 중재 |
| **Chain of Responsibility** | 요청을 순차적으로 전달 vs 중앙에서 제어 |
| **Command** | Mediator가 Command 객체를 통해 컴포넌트에게 요청 가능 |

---

## 11. 정리 및 체크리스트

### 핵심 정리

1. Mediator는 **객체 간 복잡한 상호작용을 중앙 집중화**한다.
2. 컴포넌트들은 **서로를 모르고 중재자만 안다**.
3. Observer와 달리 **양방향 통신**을 조율하고, 중재자가 **라우팅 로직을 결정**한다.
4. 실무에서 **채팅 시스템, UI 폼 검증, 이벤트 버스** 등에 사용된다.

### 적용 체크리스트

- [ ] 여러 객체가 **복잡하게 상호 참조**하고 있는가?
- [ ] 객체 간 통신 로직을 **중앙에서 관리**하고 싶은가?
- [ ] 컴포넌트를 **독립적으로 재사용**하고 싶은가?
- [ ] 새 컴포넌트 추가 시 **기존 컴포넌트를 수정**하고 싶지 않은가?
- [ ] 통신 규칙(누가 누구에게 어떤 메시지를)을 **한 곳에서 관리**하고 싶은가?

> **2~3년차 개발자를 위한 팁**: MediatR 라이브러리(C#), Redux의 Store(JavaScript), Android의 ViewModel이 Mediator 패턴의 실무 적용 사례다. 특히 MediatR는 CQRS 패턴과 함께 사용되어 요청 핸들러 간의 결합도를 낮추는 데 널리 활용된다.
