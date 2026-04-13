# Bridge 패턴 (브릿지)

> **핵심 의도 한줄 요약:** 추상화(Abstraction)와 구현(Implementation)을 분리하여 각각 독립적으로 변경/확장할 수 있게 하는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [패턴 적용 전 (Before)](#5-패턴-적용-전-before)
6. [패턴 적용 후 (After)](#6-패턴-적용-후-after)
7. [Python 구현](#7-python-구현)
8. [C# 구현](#8-c-구현)
9. [Bridge vs Adapter 차이점](#9-bridge-vs-adapter-차이점)
10. [실무 예제: 메시지 발송 시스템](#10-실무-예제-메시지-발송-시스템)
11. [장단점](#11-장단점)
12. [관련 패턴](#12-관련-패턴)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)

---

## 1. 개요

Bridge 패턴은 **GoF 구조 패턴** 중 하나로, 큰 클래스 또는 밀접하게 연관된 클래스들을 **추상화 계층**과 **구현 계층** 두 개의 독립적인 계층으로 분리합니다.

핵심 아이디어: "상속 대신 구성(Composition)을 사용하여 두 차원의 변화를 독립적으로 관리한다."

실무에서 마주치는 상황:
- 플랫폼별 UI 렌더링 (Windows/Mac/Linux x Button/Checkbox/Dialog)
- 알림 채널과 알림 유형의 조합 (Email/SMS/Push x 일반/긴급/예약)
- 데이터 저장소와 형식 (DB/File/Cloud x JSON/XML/CSV)

---

## 2. 왜 필요한가? - 문제 상황

### 조합 폭발 (Combinatorial Explosion) 문제

Shape(도형)과 Color(색상) 두 차원이 있다고 가정합니다:

```
Shape 종류: Circle, Rectangle, Triangle
Color 종류: Red, Blue, Green

상속으로 해결하면? → 3 x 3 = 9개 클래스 필요!

RedCircle, BlueCircle, GreenCircle,
RedRectangle, BlueRectangle, GreenRectangle,
RedTriangle, BlueTriangle, GreenTriangle
```

Shape이 4개가 되면? → 4 x 3 = 12개 클래스
Color가 4개가 되면? → 4 x 4 = 16개 클래스

**차원이 추가될 때마다 클래스가 기하급수적으로 증가합니다!**

---

## 3. 비유로 이해하기

### 리모컨과 TV

```
리모컨 종류: 기본 리모컨, 고급 리모컨(음성 인식)
TV 종류:     삼성 TV, LG TV, 소니 TV

상속으로 하면?
├── 기본리모컨_삼성TV
├── 기본리모컨_LG_TV
├── 기본리모컨_소니TV
├── 고급리모컨_삼성TV
├── 고급리모컨_LG_TV
└── 고급리모컨_소니TV  → 6개 클래스!

Bridge로 하면?
[리모컨] ────bridge────> [TV]
  │                       │
  ├── 기본 리모컨         ├── 삼성 TV
  └── 고급 리모컨         ├── LG TV
                          └── 소니 TV  → 5개 클래스!

리모컨은 TV를 "참조"만 하고, 둘은 독립적으로 확장 가능!
```

---

## 4. UML 다이어그램

```
┌─────────────────────┐          ┌───────────────────────┐
│   Abstraction        │          │  <<interface>>        │
│─────────────────────│          │   Implementation      │
│ - impl: Implementation ───────>│───────────────────────│
│─────────────────────│          │ + operationImpl()     │
│ + operation()        │          └───────────────────────┘
└─────────────────────┘                    △
          △                                │
          │                      ┌─────────┴──────────┐
┌─────────────────────┐  ┌──────────────┐  ┌──────────────┐
│  RefinedAbstraction  │  │ ConcreteImplA│  │ ConcreteImplB│
│─────────────────────│  │──────────────│  │──────────────│
│ + extendedOp()       │  │ + operationImpl()│ + operationImpl()│
└─────────────────────┘  └──────────────┘  └──────────────┘

Abstraction.operation() {
    impl.operationImpl();  // bridge를 통해 구현에 위임
}
```

**핵심 구조:**
- **Abstraction**: 고수준 제어 로직 (리모컨)
- **Implementation**: 저수준 구현 작업 (TV)
- **Bridge**: Abstraction이 Implementation을 참조하는 연결 고리

---

## 5. 패턴 적용 전 (Before)

```python
# ❌ Before: 상속으로 모든 조합을 만드는 방식

class Notification:
    def send(self, message: str):
        raise NotImplementedError


# 채널(Email/SMS) x 유형(일반/긴급) 조합을 모두 상속으로 처리
class EmailNormalNotification(Notification):
    def send(self, message: str):
        print(f"[Email-일반] {message}")


class EmailUrgentNotification(Notification):
    def send(self, message: str):
        subject = f"[긴급] {message}"
        print(f"[Email-긴급] {subject}")
        # 긴급 이메일 추가 로직 (빨간 라벨, 높은 우선순위 등)


class SmsNormalNotification(Notification):
    def send(self, message: str):
        print(f"[SMS-일반] {message}")


class SmsUrgentNotification(Notification):
    def send(self, message: str):
        text = f"[긴급!!!] {message}"
        print(f"[SMS-긴급] {text}")
        # 긴급 SMS 추가 로직 (반복 발송 등)


# Push 채널 추가 시? → PushNormalNotification, PushUrgentNotification 추가
# "예약" 유형 추가 시? → EmailScheduled, SmsScheduled, PushScheduled 추가
# 조합이 기하급수적으로 증가!
```

**문제점:**
- 채널 추가 시 모든 유형별 클래스를 새로 만들어야 함
- 유형 추가 시 모든 채널별 클래스를 새로 만들어야 함
- 코드 중복 발생 (긴급 로직이 Email, SMS, Push에 각각 중복)

---

## 6. 패턴 적용 후 (After)

```python
# ✅ After: Bridge 패턴으로 채널과 유형을 분리

from abc import ABC, abstractmethod


# Implementation 계층: 메시지 채널
class MessageChannel(ABC):
    @abstractmethod
    def deliver(self, title: str, content: str) -> None:
        pass


class EmailChannel(MessageChannel):
    def deliver(self, title: str, content: str) -> None:
        print(f"[Email] 제목: {title}, 내용: {content}")


class SmsChannel(MessageChannel):
    def deliver(self, title: str, content: str) -> None:
        print(f"[SMS] {title}: {content}")


# Abstraction 계층: 알림 유형
class Notification(ABC):
    def __init__(self, channel: MessageChannel):
        self._channel = channel  # bridge!

    @abstractmethod
    def send(self, message: str) -> None:
        pass


class NormalNotification(Notification):
    def send(self, message: str) -> None:
        self._channel.deliver("알림", message)


class UrgentNotification(Notification):
    def send(self, message: str) -> None:
        self._channel.deliver("[긴급] 알림", f"!!! {message} !!!")


# Push 채널 추가? → PushChannel 하나만 추가
# "예약" 유형 추가? → ScheduledNotification 하나만 추가
# 조합은 자동으로 모두 사용 가능!
```

---

## 7. Python 구현

```python
from abc import ABC, abstractmethod


# ===== Implementation 계층: 렌더링 엔진 =====
class Renderer(ABC):
    """구현 인터페이스: 실제 렌더링을 담당"""

    @abstractmethod
    def render_circle(self, x: float, y: float, radius: float) -> str:
        pass

    @abstractmethod
    def render_rectangle(self, x: float, y: float,
                         width: float, height: float) -> str:
        pass


class SVGRenderer(Renderer):
    """SVG 렌더러 - 웹용 벡터 그래픽"""

    def render_circle(self, x: float, y: float, radius: float) -> str:
        svg = f'<circle cx="{x}" cy="{y}" r="{radius}" />'
        print(f"[SVG] {svg}")
        return svg

    def render_rectangle(self, x: float, y: float,
                         width: float, height: float) -> str:
        svg = (f'<rect x="{x}" y="{y}" '
               f'width="{width}" height="{height}" />')
        print(f"[SVG] {svg}")
        return svg


class CanvasRenderer(Renderer):
    """Canvas 렌더러 - 비트맵 그래픽"""

    def render_circle(self, x: float, y: float, radius: float) -> str:
        cmd = f"ctx.arc({x}, {y}, {radius}, 0, 2*Math.PI)"
        print(f"[Canvas] {cmd}")
        return cmd

    def render_rectangle(self, x: float, y: float,
                         width: float, height: float) -> str:
        cmd = f"ctx.fillRect({x}, {y}, {width}, {height})"
        print(f"[Canvas] {cmd}")
        return cmd


class OpenGLRenderer(Renderer):
    """OpenGL 렌더러 - 고성능 3D 그래픽"""

    def render_circle(self, x: float, y: float, radius: float) -> str:
        cmd = f"glDrawCircle({x}, {y}, {radius})"
        print(f"[OpenGL] {cmd}")
        return cmd

    def render_rectangle(self, x: float, y: float,
                         width: float, height: float) -> str:
        cmd = f"glDrawRect({x}, {y}, {width}, {height})"
        print(f"[OpenGL] {cmd}")
        return cmd


# ===== Abstraction 계층: 도형 =====
class Shape(ABC):
    """추상화: 도형의 고수준 로직"""

    def __init__(self, renderer: Renderer):
        self._renderer = renderer  # Bridge!

    @abstractmethod
    def draw(self) -> str:
        pass

    @abstractmethod
    def resize(self, factor: float) -> None:
        pass


class Circle(Shape):
    """원: 추상화의 구체 클래스"""

    def __init__(self, renderer: Renderer, x: float, y: float,
                 radius: float):
        super().__init__(renderer)
        self._x = x
        self._y = y
        self._radius = radius

    def draw(self) -> str:
        return self._renderer.render_circle(
            self._x, self._y, self._radius
        )

    def resize(self, factor: float) -> None:
        self._radius *= factor


class Rectangle(Shape):
    """사각형: 추상화의 구체 클래스"""

    def __init__(self, renderer: Renderer, x: float, y: float,
                 width: float, height: float):
        super().__init__(renderer)
        self._x = x
        self._y = y
        self._width = width
        self._height = height

    def draw(self) -> str:
        return self._renderer.render_rectangle(
            self._x, self._y, self._width, self._height
        )

    def resize(self, factor: float) -> None:
        self._width *= factor
        self._height *= factor


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 렌더러 생성 (Implementation)
    svg = SVGRenderer()
    canvas = CanvasRenderer()
    opengl = OpenGLRenderer()

    # 도형 생성 시 렌더러를 주입 (Bridge 연결)
    print("=== SVG로 렌더링 ===")
    circle_svg = Circle(svg, 10, 20, 5)
    circle_svg.draw()

    rect_svg = Rectangle(svg, 0, 0, 100, 50)
    rect_svg.draw()

    print("\n=== Canvas로 렌더링 ===")
    circle_canvas = Circle(canvas, 10, 20, 5)
    circle_canvas.draw()

    rect_canvas = Rectangle(canvas, 0, 0, 100, 50)
    rect_canvas.draw()

    print("\n=== OpenGL로 렌더링 ===")
    circle_opengl = Circle(opengl, 10, 20, 5)
    circle_opengl.draw()

    # 리사이즈 후 다시 그리기
    print("\n=== 리사이즈 후 ===")
    circle_opengl.resize(2.0)
    circle_opengl.draw()
```

**실행 결과:**
```
=== SVG로 렌더링 ===
[SVG] <circle cx="10" cy="20" r="5" />
[SVG] <rect x="0" y="0" width="100" height="50" />

=== Canvas로 렌더링 ===
[Canvas] ctx.arc(10, 20, 5, 0, 2*Math.PI)
[Canvas] ctx.fillRect(0, 0, 100, 50)

=== OpenGL로 렌더링 ===
[OpenGL] glDrawCircle(10, 20, 5)

=== 리사이즈 후 ===
[OpenGL] glDrawCircle(10, 20, 10.0)
```

---

## 8. C# 구현

```csharp
using System;

// ===== Implementation 계층: 렌더링 엔진 =====
public interface IRenderer
{
    string RenderCircle(float x, float y, float radius);
    string RenderRectangle(float x, float y, float width, float height);
}

public class SVGRenderer : IRenderer
{
    public string RenderCircle(float x, float y, float radius)
    {
        var svg = $"<circle cx=\"{x}\" cy=\"{y}\" r=\"{radius}\" />";
        Console.WriteLine($"[SVG] {svg}");
        return svg;
    }

    public string RenderRectangle(float x, float y,
                                   float width, float height)
    {
        var svg = $"<rect x=\"{x}\" y=\"{y}\" " +
                  $"width=\"{width}\" height=\"{height}\" />";
        Console.WriteLine($"[SVG] {svg}");
        return svg;
    }
}

public class CanvasRenderer : IRenderer
{
    public string RenderCircle(float x, float y, float radius)
    {
        var cmd = $"ctx.arc({x}, {y}, {radius}, 0, 2*Math.PI)";
        Console.WriteLine($"[Canvas] {cmd}");
        return cmd;
    }

    public string RenderRectangle(float x, float y,
                                   float width, float height)
    {
        var cmd = $"ctx.fillRect({x}, {y}, {width}, {height})";
        Console.WriteLine($"[Canvas] {cmd}");
        return cmd;
    }
}

public class OpenGLRenderer : IRenderer
{
    public string RenderCircle(float x, float y, float radius)
    {
        var cmd = $"glDrawCircle({x}, {y}, {radius})";
        Console.WriteLine($"[OpenGL] {cmd}");
        return cmd;
    }

    public string RenderRectangle(float x, float y,
                                   float width, float height)
    {
        var cmd = $"glDrawRect({x}, {y}, {width}, {height})";
        Console.WriteLine($"[OpenGL] {cmd}");
        return cmd;
    }
}

// ===== Abstraction 계층: 도형 =====
public abstract class Shape
{
    protected IRenderer Renderer { get; }  // Bridge!

    protected Shape(IRenderer renderer)
    {
        Renderer = renderer;
    }

    public abstract string Draw();
    public abstract void Resize(float factor);
}

public class Circle : Shape
{
    private float _x, _y, _radius;

    public Circle(IRenderer renderer, float x, float y, float radius)
        : base(renderer)
    {
        _x = x;
        _y = y;
        _radius = radius;
    }

    public override string Draw()
    {
        return Renderer.RenderCircle(_x, _y, _radius);
    }

    public override void Resize(float factor)
    {
        _radius *= factor;
    }
}

public class Rectangle : Shape
{
    private float _x, _y, _width, _height;

    public Rectangle(IRenderer renderer, float x, float y,
                     float width, float height)
        : base(renderer)
    {
        _x = x;
        _y = y;
        _width = width;
        _height = height;
    }

    public override string Draw()
    {
        return Renderer.RenderRectangle(_x, _y, _width, _height);
    }

    public override void Resize(float factor)
    {
        _width *= factor;
        _height *= factor;
    }
}

// ===== 사용 예시 =====
public class Program
{
    public static void Main()
    {
        IRenderer svg = new SVGRenderer();
        IRenderer canvas = new CanvasRenderer();

        // 같은 도형, 다른 렌더러
        var circleSvg = new Circle(svg, 10, 20, 5);
        var circleCanvas = new Circle(canvas, 10, 20, 5);

        Console.WriteLine("=== SVG Circle ===");
        circleSvg.Draw();

        Console.WriteLine("\n=== Canvas Circle ===");
        circleCanvas.Draw();

        // 렌더러 추가해도 도형 코드는 변경 없음
        // 도형 추가해도 렌더러 코드는 변경 없음
    }
}
```

---

## 9. Bridge vs Adapter 차이점

| 구분 | Adapter | Bridge |
|------|---------|--------|
| **목적** | 기존 인터페이스 호환성 확보 | 추상화와 구현의 분리 |
| **적용 시점** | **사후** (이미 존재하는 코드) | **사전** (설계 단계에서) |
| **관계** | 서로 다른 인터페이스 연결 | 하나의 개념을 두 계층으로 분리 |
| **변경** | Adaptee는 수정 불가 | 양쪽 모두 독립 확장 가능 |
| **비유** | 110V → 220V 전원 어댑터 | 리모컨 ↔ TV 분리 |

```
Adapter: "이미 있는 걸 어떻게든 맞추자" (사후 대응)
Bridge:  "처음부터 분리해서 설계하자"  (사전 설계)
```

### 코드로 보는 차이

```python
# Adapter: 인터페이스 변환
class Adapter(TargetInterface):
    def __init__(self, adaptee):  # 기존 코드를 감쌈
        self._adaptee = adaptee

    def target_method(self):
        self._adaptee.different_method()  # 다른 이름의 메서드 호출

# Bridge: 추상화와 구현 분리
class Abstraction:
    def __init__(self, impl):  # 구현을 주입받음
        self._impl = impl

    def operation(self):
        self._impl.operation_impl()  # 구현에 위임
```

---

## 10. 실무 예제: 메시지 발송 시스템

### Email/SMS x 일반/긴급 조합

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime


# ===== Implementation: 메시지 채널 (발송 수단) =====
@dataclass
class MessagePayload:
    sender: str
    recipient: str
    subject: str
    body: str
    metadata: dict


class MessageChannel(ABC):
    """메시지 발송 채널 (Implementation)"""

    @abstractmethod
    def send_message(self, payload: MessagePayload) -> bool:
        pass

    @abstractmethod
    def get_channel_name(self) -> str:
        pass


class EmailChannel(MessageChannel):
    """이메일 채널"""

    def __init__(self, smtp_host: str = "smtp.example.com"):
        self._smtp_host = smtp_host

    def send_message(self, payload: MessagePayload) -> bool:
        print(f"  [EMAIL] {self._smtp_host} 경유")
        print(f"  To: {payload.recipient}")
        print(f"  Subject: {payload.subject}")
        print(f"  Body: {payload.body}")
        return True

    def get_channel_name(self) -> str:
        return "Email"


class SmsChannel(MessageChannel):
    """SMS 채널"""

    def __init__(self, api_key: str = "sms-api-key-xxx"):
        self._api_key = api_key

    def send_message(self, payload: MessagePayload) -> bool:
        # SMS는 제목+본문을 합쳐서 발송
        text = f"[{payload.subject}] {payload.body}"
        if len(text) > 80:
            text = text[:77] + "..."
        print(f"  [SMS] -> {payload.recipient}: {text}")
        return True

    def get_channel_name(self) -> str:
        return "SMS"


class PushChannel(MessageChannel):
    """푸시 알림 채널 - 나중에 추가해도 기존 코드 변경 없음!"""

    def send_message(self, payload: MessagePayload) -> bool:
        print(f"  [PUSH] -> {payload.recipient}")
        print(f"  Title: {payload.subject}")
        print(f"  Body: {payload.body[:50]}")
        return True

    def get_channel_name(self) -> str:
        return "Push"


# ===== Abstraction: 알림 유형 (비즈니스 로직) =====
class Notification(ABC):
    """알림 추상화 (Abstraction)"""

    def __init__(self, channel: MessageChannel):
        self._channel = channel  # Bridge!

    @abstractmethod
    def notify(self, sender: str, recipient: str,
               message: str) -> bool:
        pass

    def _create_payload(self, sender: str, recipient: str,
                        subject: str, body: str,
                        **extra) -> MessagePayload:
        return MessagePayload(
            sender=sender,
            recipient=recipient,
            subject=subject,
            body=body,
            metadata=extra
        )


class NormalNotification(Notification):
    """일반 알림"""

    def notify(self, sender: str, recipient: str,
               message: str) -> bool:
        print(f"\n[일반 알림] via {self._channel.get_channel_name()}")
        payload = self._create_payload(
            sender=sender,
            recipient=recipient,
            subject="알림",
            body=message,
            priority="normal",
            timestamp=datetime.now().isoformat()
        )
        return self._channel.send_message(payload)


class UrgentNotification(Notification):
    """긴급 알림 - 강조 표시 + 반복 발송"""

    MAX_RETRY = 3

    def notify(self, sender: str, recipient: str,
               message: str) -> bool:
        print(f"\n[긴급 알림] via {self._channel.get_channel_name()}")
        payload = self._create_payload(
            sender=sender,
            recipient=recipient,
            subject="[긴급] 즉시 확인 필요",
            body=f"!!! {message} !!!",
            priority="urgent",
            retry_count=self.MAX_RETRY,
            timestamp=datetime.now().isoformat()
        )

        # 긴급 알림은 최대 3번까지 재시도
        for attempt in range(1, self.MAX_RETRY + 1):
            print(f"  시도 {attempt}/{self.MAX_RETRY}")
            success = self._channel.send_message(payload)
            if success:
                return True
        return False


class ScheduledNotification(Notification):
    """예약 알림 - 새 유형 추가해도 채널 코드 변경 없음!"""

    def __init__(self, channel: MessageChannel,
                 scheduled_time: str):
        super().__init__(channel)
        self._scheduled_time = scheduled_time

    def notify(self, sender: str, recipient: str,
               message: str) -> bool:
        print(f"\n[예약 알림] {self._scheduled_time}에 "
              f"via {self._channel.get_channel_name()}")
        payload = self._create_payload(
            sender=sender,
            recipient=recipient,
            subject=f"[예약-{self._scheduled_time}] 알림",
            body=message,
            priority="scheduled",
            scheduled_at=self._scheduled_time
        )
        return self._channel.send_message(payload)


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 채널 생성 (Implementation)
    email = EmailChannel()
    sms = SmsChannel()
    push = PushChannel()

    # 채널 x 유형 자유롭게 조합
    print("=" * 50)
    print("다양한 조합 테스트")
    print("=" * 50)

    # 일반 + 이메일
    NormalNotification(email).notify(
        "system", "user@example.com", "주문이 접수되었습니다."
    )

    # 긴급 + SMS
    UrgentNotification(sms).notify(
        "system", "010-1234-5678", "서버 장애 발생!"
    )

    # 예약 + 푸시
    ScheduledNotification(push, "2024-03-01 09:00").notify(
        "marketing", "user-token-123", "봄맞이 할인 이벤트!"
    )

    # 긴급 + 이메일 (다른 조합도 자유롭게)
    UrgentNotification(email).notify(
        "system", "admin@example.com", "DB 연결 실패!"
    )
```

### C# 실무 버전

```csharp
using System;
using System.Collections.Generic;

// ===== Implementation: 메시지 채널 =====
public interface IMessageChannel
{
    bool SendMessage(string recipient, string subject,
                     string body, Dictionary<string, string> metadata);
    string ChannelName { get; }
}

public class EmailChannel : IMessageChannel
{
    public string ChannelName => "Email";

    public bool SendMessage(string recipient, string subject,
                            string body,
                            Dictionary<string, string> metadata)
    {
        Console.WriteLine($"  [EMAIL] To: {recipient}");
        Console.WriteLine($"  Subject: {subject}");
        Console.WriteLine($"  Body: {body}");
        return true;
    }
}

public class SmsChannel : IMessageChannel
{
    public string ChannelName => "SMS";

    public bool SendMessage(string recipient, string subject,
                            string body,
                            Dictionary<string, string> metadata)
    {
        var text = $"[{subject}] {body}";
        if (text.Length > 80) text = text[..77] + "...";
        Console.WriteLine($"  [SMS] -> {recipient}: {text}");
        return true;
    }
}

// ===== Abstraction: 알림 유형 =====
public abstract class Notification
{
    protected IMessageChannel Channel { get; }  // Bridge

    protected Notification(IMessageChannel channel)
    {
        Channel = channel;
    }

    public abstract bool Notify(string recipient, string message);
}

public class NormalNotification : Notification
{
    public NormalNotification(IMessageChannel channel)
        : base(channel) { }

    public override bool Notify(string recipient, string message)
    {
        Console.WriteLine($"\n[일반 알림] via {Channel.ChannelName}");
        return Channel.SendMessage(
            recipient, "알림", message,
            new Dictionary<string, string> { ["priority"] = "normal" }
        );
    }
}

public class UrgentNotification : Notification
{
    private const int MaxRetry = 3;

    public UrgentNotification(IMessageChannel channel)
        : base(channel) { }

    public override bool Notify(string recipient, string message)
    {
        Console.WriteLine($"\n[긴급 알림] via {Channel.ChannelName}");
        var metadata = new Dictionary<string, string>
        {
            ["priority"] = "urgent",
            ["retry_count"] = MaxRetry.ToString()
        };

        for (int attempt = 1; attempt <= MaxRetry; attempt++)
        {
            Console.WriteLine($"  시도 {attempt}/{MaxRetry}");
            if (Channel.SendMessage(
                    recipient, "[긴급] 즉시 확인",
                    $"!!! {message} !!!", metadata))
                return true;
        }
        return false;
    }
}

// 사용
public class Program
{
    public static void Main()
    {
        var email = new EmailChannel();
        var sms = new SmsChannel();

        // 자유로운 조합
        new NormalNotification(email)
            .Notify("user@test.com", "주문 접수됨");
        new UrgentNotification(sms)
            .Notify("010-1234-5678", "서버 장애!");
        new UrgentNotification(email)
            .Notify("admin@test.com", "DB 연결 실패!");
    }
}
```

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **독립적 확장** | 추상화와 구현을 각각 독립적으로 확장 가능 |
| **조합 폭발 방지** | M x N 클래스 대신 M + N 클래스로 해결 |
| **개방/폐쇄 원칙** | 새 추상화나 구현을 추가해도 기존 코드 불변 |
| **단일 책임 원칙** | 고수준 로직과 저수준 구현을 분리 |
| **런타임 교체** | 구현을 런타임에 동적으로 교체 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **초기 복잡도** | 설계 초기에 분리 지점을 파악해야 함 |
| **과도한 추상화** | 변경 축이 하나뿐이면 오히려 복잡해짐 |
| **이해 난이도** | 팀원들이 패턴을 이해하지 못하면 유지보수 어려움 |

---

## 12. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Adapter** | Bridge는 사전 설계, Adapter는 사후 호환 |
| **Abstract Factory** | Bridge의 Implementation을 생성하는 데 활용 가능 |
| **Strategy** | 유사한 구조이나, Strategy는 알고리즘 교체에 초점 |
| **Template Method** | 상속 기반 행동 변경 vs Bridge는 구성 기반 |

---

## 13. 정리 및 체크리스트

### 핵심 정리

```
Bridge 패턴 = "두 차원의 변화를 독립적으로 관리"

When: 클래스가 두 개 이상의 독립적인 차원으로 확장될 때
How:  추상화 계층과 구현 계층을 분리하고 구성(composition)으로 연결
Why:  조합 폭발을 방지하고, 각 차원을 독립적으로 변경/확장
```

### 적용 체크리스트

- [ ] 두 개 이상의 독립적인 변경 축이 존재하는가?
- [ ] 클래스 조합이 기하급수적으로 증가하는가?
- [ ] 각 차원을 독립적으로 확장해야 하는 요구사항이 있는가?
- [ ] 런타임에 구현을 교체해야 하는가?
- [ ] Bridge와 Strategy를 구분했는가? (구조적 분리 vs 알고리즘 교체)

### 변경 축이 하나뿐이라면?

변경 축이 하나뿐이면 Bridge는 과도합니다. Strategy 패턴이나 단순 상속이 더 적합할 수 있습니다.

> **기억하세요:** "M x N 조합이 보이면 Bridge를 떠올리세요. 상속 대신 구성으로 조합 폭발을 해결합니다."
