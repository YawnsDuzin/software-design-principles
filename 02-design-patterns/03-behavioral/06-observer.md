# Observer 패턴 (옵저버)

> **핵심 의도 한줄 요약**: 객체의 상태 변화를 관찰하여 의존하는 다른 객체들에게 자동으로 알리고 갱신하는 일대다(1:N) 의존 관계를 정의한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [Push vs Pull 방식](#7-push-vs-pull-방식)
8. [실무 예제: 주식 가격 알림 시스템](#8-실무-예제-주식-가격-알림-시스템)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 개요

Observer 패턴은 **발행-구독(Publish-Subscribe)** 메커니즘을 구현하는 패턴이다. 하나의 객체(Subject)의 상태가 변경되면, 그 객체에 등록된 모든 관찰자(Observer)에게 자동으로 알림이 전달된다.

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **Subject (Publisher)** | 상태를 가지고 있으며, Observer를 등록/해제하고 알림 전송 |
| **Observer (Subscriber)** | Subject의 상태 변화를 통보받아 반응 |
| **ConcreteSubject** | 구체적인 상태를 관리하는 주체 |
| **ConcreteObserver** | 알림을 받아 구체적인 행동을 수행 |

---

## 2. 문제 상황

### 문제: 상태 변화 알림

쇼핑몰에서 품절 상품이 재입고되면 관심 있는 고객들에게 알려야 한다.

```python
# 나쁜 예 1: 폴링 (Polling)
# 고객이 주기적으로 확인 → 리소스 낭비
while True:
    if product.is_available():
        send_notification()
        break
    time.sleep(60)  # 1분마다 확인

# 나쁜 예 2: 직접 통보
# Product가 모든 고객 클래스를 알아야 함
class Product:
    def restock(self):
        self.quantity += 100
        # Product가 알림 대상을 직접 관리
        email_service.send(customer1, "재입고!")
        sms_service.send(customer2, "재입고!")
        push_service.send(customer3, "재입고!")
        # 새 알림 채널 추가마다 코드 수정...
```

문제점:

- **폴링**: 불필요한 리소스 소비
- **직접 통보**: Subject가 Observer의 구체적인 타입을 알아야 한다 (강한 결합)
- **OCP 위반**: 새로운 알림 방식 추가 시 기존 코드 수정 필요

---

## 3. 일상 비유

**유튜브 구독**을 떠올려 보자.

```
[유튜브 채널] (Subject)
  ↓ 새 영상 업로드
  ↓
  ├──→ 구독자A: 이메일 알림 수신
  ├──→ 구독자B: 모바일 푸시 알림 수신
  ├──→ 구독자C: 알림 끔 (구독 해제)
  └──→ 구독자D: 앱 내 알림 수신

  구독(subscribe) → 알림 받기
  구독 해제(unsubscribe) → 알림 안 받기
```

- 채널은 구독자가 누구인지 세세하게 **알 필요 없다**
- 구독자는 자유롭게 **구독/해제**할 수 있다
- 새 영상이 올라오면 **모든 구독자에게 자동** 알림

---

## 4. UML 다이어그램

```
 ┌────────────────────────┐        ┌──────────────────────┐
 │      Subject           │        │  <<interface>>       │
 ├────────────────────────┤        │    Observer           │
 │ - observers: List      │───────>├──────────────────────┤
 ├────────────────────────┤        │ + update(data)       │
 │ + subscribe(observer)  │        └──────────┬───────────┘
 │ + unsubscribe(observer)│                   │
 │ + notify()             │        ┌──────────┼───────────┐
 └────────────────────────┘        │          │           │
                              ┌────▼───┐ ┌────▼───┐ ┌────▼───┐
                              │ObsrvrA │ │ObsrvrB │ │ObsrvrC │
                              │+update │ │+update │ │+update │
                              └────────┘ └────────┘ └────────┘

 Subject가 notify()를 호출하면,
 등록된 모든 Observer의 update()가 호출된다.
```

---

## 5. Python 구현

### 클래스 기반 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


class Observer(ABC):
    """옵저버 인터페이스"""

    @abstractmethod
    def update(self, subject: Subject, **kwargs: Any) -> None:
        pass


class Subject:
    """주체 (이벤트 발행자)"""

    def __init__(self) -> None:
        self._observers: list[Observer] = []

    def subscribe(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)
            print(f"  + {observer.__class__.__name__} 구독 등록")

    def unsubscribe(self, observer: Observer) -> None:
        self._observers.remove(observer)
        print(f"  - {observer.__class__.__name__} 구독 해제")

    def notify(self, **kwargs: Any) -> None:
        for observer in self._observers:
            observer.update(self, **kwargs)


class WeatherStation(Subject):
    """기상 관측소 (ConcreteSubject)"""

    def __init__(self) -> None:
        super().__init__()
        self._temperature = 0.0
        self._humidity = 0.0

    @property
    def temperature(self) -> float:
        return self._temperature

    @property
    def humidity(self) -> float:
        return self._humidity

    def set_measurements(self, temperature: float, humidity: float) -> None:
        print(f"\n  [기상관측소] 온도: {temperature}도, 습도: {humidity}%")
        self._temperature = temperature
        self._humidity = humidity
        self.notify(temperature=temperature, humidity=humidity)


class TemperatureDisplay(Observer):
    """온도 표시 장치"""

    def update(self, subject: Subject, **kwargs: Any) -> None:
        temp = kwargs.get("temperature", 0)
        print(f"  [온도 디스플레이] 현재 온도: {temp}도")


class HumidityDisplay(Observer):
    """습도 표시 장치"""

    def update(self, subject: Subject, **kwargs: Any) -> None:
        humidity = kwargs.get("humidity", 0)
        print(f"  [습도 디스플레이] 현재 습도: {humidity}%")


class AlertSystem(Observer):
    """경보 시스템"""

    def update(self, subject: Subject, **kwargs: Any) -> None:
        temp = kwargs.get("temperature", 0)
        if temp > 35:
            print(f"  [경보] 폭염 경보! 온도가 {temp}도입니다!")
        elif temp < -10:
            print(f"  [경보] 한파 경보! 온도가 {temp}도입니다!")


# === 사용 예시 ===
if __name__ == "__main__":
    station = WeatherStation()

    temp_display = TemperatureDisplay()
    humidity_display = HumidityDisplay()
    alert = AlertSystem()

    print("=== 옵저버 등록 ===")
    station.subscribe(temp_display)
    station.subscribe(humidity_display)
    station.subscribe(alert)

    print("\n=== 측정값 업데이트 ===")
    station.set_measurements(25.0, 60.0)
    station.set_measurements(37.0, 80.0)

    print("\n=== 습도 디스플레이 구독 해제 ===")
    station.unsubscribe(humidity_display)
    station.set_measurements(-15.0, 30.0)
```

### 콜백(함수) 기반 구현

```python
from typing import Callable, Any


class EventEmitter:
    """이벤트 기반 Observer (콜백 방식)"""

    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        """이벤트 리스너 등록"""
        if event not in self._listeners:
            self._listeners[event] = []
        self._listeners[event].append(callback)

    def off(self, event: str, callback: Callable) -> None:
        """이벤트 리스너 해제"""
        if event in self._listeners:
            self._listeners[event].remove(callback)

    def emit(self, event: str, *args: Any, **kwargs: Any) -> None:
        """이벤트 발생 → 등록된 리스너들 호출"""
        if event in self._listeners:
            for callback in self._listeners[event]:
                callback(*args, **kwargs)


# 사용 예시
emitter = EventEmitter()

# 람다 또는 함수를 직접 등록
emitter.on("user_login", lambda user: print(f"  로그인: {user}"))
emitter.on("user_login", lambda user: print(f"  로그 기록: {user} 로그인"))
emitter.on("user_logout", lambda user: print(f"  로그아웃: {user}"))

emitter.emit("user_login", "Alice")
emitter.emit("user_logout", "Alice")
```

---

## 6. C# 구현

### event/delegate 기반 구현 (C# 관용적 방식)

```csharp
using System;
using System.Collections.Generic;

// EventArgs 정의
public class WeatherChangedEventArgs : EventArgs
{
    public double Temperature { get; }
    public double Humidity { get; }

    public WeatherChangedEventArgs(double temperature, double humidity)
    {
        Temperature = temperature;
        Humidity = humidity;
    }
}

// Subject: 기상 관측소
public class WeatherStation
{
    // C#의 event/delegate는 Observer 패턴의 언어 수준 지원
    public event EventHandler<WeatherChangedEventArgs>? WeatherChanged;

    private double _temperature;
    private double _humidity;

    public double Temperature => _temperature;
    public double Humidity => _humidity;

    public void SetMeasurements(double temperature, double humidity)
    {
        Console.WriteLine($"\n  [기상관측소] 온도: {temperature}도, 습도: {humidity}%");
        _temperature = temperature;
        _humidity = humidity;

        // 이벤트 발생 (등록된 모든 핸들러에게 알림)
        WeatherChanged?.Invoke(this, new WeatherChangedEventArgs(temperature, humidity));
    }
}

// Observer: 온도 디스플레이
public class TemperatureDisplay
{
    public void OnWeatherChanged(object? sender, WeatherChangedEventArgs e)
    {
        Console.WriteLine($"  [온도 디스플레이] 현재 온도: {e.Temperature}도");
    }
}

// Observer: 경보 시스템
public class AlertSystem
{
    public void OnWeatherChanged(object? sender, WeatherChangedEventArgs e)
    {
        if (e.Temperature > 35)
            Console.WriteLine($"  [경보] 폭염 경보! 온도가 {e.Temperature}도입니다!");
        else if (e.Temperature < -10)
            Console.WriteLine($"  [경보] 한파 경보! 온도가 {e.Temperature}도입니다!");
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var station = new WeatherStation();
        var tempDisplay = new TemperatureDisplay();
        var alert = new AlertSystem();

        // 구독 (+=)
        Console.WriteLine("=== 옵저버 등록 ===");
        station.WeatherChanged += tempDisplay.OnWeatherChanged;
        station.WeatherChanged += alert.OnWeatherChanged;

        // 람다로도 구독 가능
        station.WeatherChanged += (sender, e) =>
            Console.WriteLine($"  [습도 디스플레이] 현재 습도: {e.Humidity}%");

        Console.WriteLine("\n=== 측정값 업데이트 ===");
        station.SetMeasurements(25.0, 60.0);
        station.SetMeasurements(37.0, 80.0);

        // 구독 해제 (-=)
        Console.WriteLine("\n=== 온도 디스플레이 구독 해제 ===");
        station.WeatherChanged -= tempDisplay.OnWeatherChanged;
        station.SetMeasurements(-15.0, 30.0);
    }
}
```

### IObservable<T> / IObserver<T> 기반 구현

```csharp
using System;
using System.Collections.Generic;

// 데이터 모델
public record WeatherData(double Temperature, double Humidity);

// Subject: IObservable<T> 구현
public class WeatherStationObservable : IObservable<WeatherData>
{
    private readonly List<IObserver<WeatherData>> _observers = new();

    public IDisposable Subscribe(IObserver<WeatherData> observer)
    {
        if (!_observers.Contains(observer))
            _observers.Add(observer);
        return new Unsubscriber(_observers, observer);
    }

    public void SetMeasurements(double temp, double humidity)
    {
        var data = new WeatherData(temp, humidity);
        Console.WriteLine($"\n  [기상관측소] 온도: {temp}도, 습도: {humidity}%");
        foreach (var observer in _observers)
        {
            observer.OnNext(data);
        }
    }

    public void Complete()
    {
        foreach (var observer in _observers)
            observer.OnCompleted();
        _observers.Clear();
    }

    // 구독 해제를 위한 Disposable 패턴
    private class Unsubscriber : IDisposable
    {
        private readonly List<IObserver<WeatherData>> _observers;
        private readonly IObserver<WeatherData> _observer;

        public Unsubscriber(
            List<IObserver<WeatherData>> observers,
            IObserver<WeatherData> observer)
        {
            _observers = observers;
            _observer = observer;
        }

        public void Dispose()
        {
            if (_observers.Contains(_observer))
                _observers.Remove(_observer);
        }
    }
}

// Observer: IObserver<T> 구현
public class TemperatureMonitor : IObserver<WeatherData>
{
    public void OnNext(WeatherData value)
    {
        Console.WriteLine($"  [온도 모니터] {value.Temperature}도");
    }

    public void OnError(Exception error)
    {
        Console.WriteLine($"  [온도 모니터] 에러: {error.Message}");
    }

    public void OnCompleted()
    {
        Console.WriteLine("  [온도 모니터] 관측 종료");
    }
}
```

---

## 7. Push vs Pull 방식

### Push 방식 (데이터를 밀어넣기)

Subject가 변경 데이터를 Observer에게 **직접 전달**한다.

```python
# Push: 데이터를 인자로 전달
class Subject:
    def notify(self):
        for observer in self._observers:
            observer.update(
                temperature=self._temp,
                humidity=self._humidity
            )
```

### Pull 방식 (데이터를 끌어오기)

Subject는 "변경됐다"는 사실만 알리고, Observer가 필요한 데이터를 **직접 가져간다**.

```python
# Pull: Subject 참조만 전달, Observer가 필요한 것을 가져감
class Subject:
    def notify(self):
        for observer in self._observers:
            observer.update(self)  # Subject 자체를 전달

class ConcreteObserver:
    def update(self, subject: Subject):
        # 필요한 데이터만 가져온다
        temp = subject.temperature
```

### 비교

| 구분 | Push | Pull |
|------|------|------|
| **데이터 전달** | Subject → Observer (전달) | Observer → Subject (요청) |
| **결합도** | Observer가 데이터 형식에 의존 | Observer가 Subject에 의존 |
| **효율성** | 불필요한 데이터도 전달될 수 있음 | 필요한 데이터만 가져옴 |
| **단순성** | 구현이 간단 | Subject가 getter를 제공해야 함 |
| **적합한 경우** | 모든 Observer가 같은 데이터 필요 | Observer마다 다른 데이터 필요 |

---

## 8. 실무 예제: 주식 가격 알림 시스템

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime
from enum import Enum


class AlertType(Enum):
    PRICE_UP = "상승"
    PRICE_DOWN = "하락"
    THRESHOLD = "목표가"


@dataclass
class StockPrice:
    """주식 가격 데이터"""
    symbol: str
    price: float
    prev_price: float
    timestamp: str

    @property
    def change(self) -> float:
        return self.price - self.prev_price

    @property
    def change_percent(self) -> float:
        if self.prev_price == 0:
            return 0.0
        return (self.change / self.prev_price) * 100


class StockObserver(ABC):
    """주식 관찰자 인터페이스"""

    @abstractmethod
    def on_price_update(self, stock: StockPrice) -> None:
        pass


class StockExchange:
    """주식 거래소 (Subject)"""

    def __init__(self) -> None:
        self._observers: dict[str, list[StockObserver]] = {}
        self._prices: dict[str, float] = {}

    def subscribe(self, symbol: str, observer: StockObserver) -> None:
        if symbol not in self._observers:
            self._observers[symbol] = []
        if observer not in self._observers[symbol]:
            self._observers[symbol].append(observer)

    def unsubscribe(self, symbol: str, observer: StockObserver) -> None:
        if symbol in self._observers:
            self._observers[symbol].remove(observer)

    def update_price(self, symbol: str, new_price: float) -> None:
        prev_price = self._prices.get(symbol, new_price)
        self._prices[symbol] = new_price

        stock = StockPrice(
            symbol=symbol,
            price=new_price,
            prev_price=prev_price,
            timestamp=datetime.now().strftime("%H:%M:%S"),
        )

        print(f"\n  [{stock.timestamp}] {symbol}: "
              f"{new_price:,.0f}원 ({stock.change:+,.0f}, "
              f"{stock.change_percent:+.1f}%)")

        self._notify(symbol, stock)

    def _notify(self, symbol: str, stock: StockPrice) -> None:
        if symbol in self._observers:
            for observer in self._observers[symbol]:
                observer.on_price_update(stock)


class PriceAlertObserver(StockObserver):
    """목표가 알림"""

    def __init__(self, name: str, target_price: float, alert_type: str = "above") -> None:
        self.name = name
        self.target_price = target_price
        self.alert_type = alert_type  # "above" or "below"
        self._alerted = False

    def on_price_update(self, stock: StockPrice) -> None:
        if self._alerted:
            return

        if self.alert_type == "above" and stock.price >= self.target_price:
            print(f"    [알림-{self.name}] {stock.symbol} 목표가 "
                  f"{self.target_price:,.0f}원 도달! (현재: {stock.price:,.0f}원)")
            self._alerted = True
        elif self.alert_type == "below" and stock.price <= self.target_price:
            print(f"    [알림-{self.name}] {stock.symbol} 손절가 "
                  f"{self.target_price:,.0f}원 이하! (현재: {stock.price:,.0f}원)")
            self._alerted = True


class PercentChangeObserver(StockObserver):
    """등락률 알림"""

    def __init__(self, name: str, threshold_percent: float) -> None:
        self.name = name
        self.threshold = threshold_percent

    def on_price_update(self, stock: StockPrice) -> None:
        if abs(stock.change_percent) >= self.threshold:
            direction = "급등" if stock.change_percent > 0 else "급락"
            print(f"    [알림-{self.name}] {stock.symbol} {direction}! "
                  f"({stock.change_percent:+.1f}%)")


class TradingLogger(StockObserver):
    """거래 기록 로거"""

    def __init__(self) -> None:
        self._logs: list[str] = []

    def on_price_update(self, stock: StockPrice) -> None:
        log = (f"{stock.timestamp} | {stock.symbol} | "
               f"{stock.price:,.0f}원 | {stock.change_percent:+.1f}%")
        self._logs.append(log)

    def show_logs(self) -> None:
        print("\n--- 거래 기록 ---")
        for log in self._logs:
            print(f"  {log}")


# === 사용 예시 ===
if __name__ == "__main__":
    exchange = StockExchange()

    # Observer 생성
    target_alert = PriceAlertObserver("투자자A", 75000, "above")
    stop_loss = PriceAlertObserver("투자자B", 65000, "below")
    volatility = PercentChangeObserver("트레이더", 3.0)
    logger = TradingLogger()

    # 특정 종목 구독
    for observer in [target_alert, stop_loss, volatility, logger]:
        exchange.subscribe("삼성전자", observer)

    # 가격 변동 시뮬레이션
    print("=== 주식 가격 변동 ===")
    prices = [70000, 71500, 73000, 68000, 64000, 76000]
    for price in prices:
        exchange.update_price("삼성전자", price)

    logger.show_logs()
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **느슨한 결합** | Subject와 Observer가 추상화를 통해 느슨하게 연결 |
| **개방-폐쇄 원칙** | 새로운 Observer를 기존 코드 수정 없이 추가 가능 |
| **런타임 구독/해제** | 실행 중 동적으로 Observer를 추가/제거 가능 |
| **브로드캐스트** | 하나의 이벤트를 여러 Observer에게 동시에 전달 |

### 단점

| 단점 | 설명 |
|------|------|
| **알림 순서** | Observer 호출 순서가 보장되지 않을 수 있다 |
| **메모리 누수** | 구독 해제를 잊으면 Observer가 GC 되지 않는다 (Lapsed Listener) |
| **연쇄 업데이트** | Observer가 다른 Subject를 변경하여 연쇄 반응이 일어날 수 있다 |
| **성능** | Observer가 많으면 알림 비용이 커진다 |
| **디버깅** | 간접적인 호출로 흐름 추적이 어렵다 |

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Mediator** | 양방향 통신을 중재 vs Observer는 단방향 알림 |
| **Singleton** | Subject가 전역적으로 하나만 필요한 경우 함께 사용 |
| **Strategy** | Observer의 update 로직을 Strategy로 교체 가능 |
| **Command** | 알림을 Command 객체로 캡슐화하여 큐잉/로깅 가능 |

---

## 11. 정리 및 체크리스트

### 핵심 정리

1. Observer는 **일대다(1:N) 의존 관계**를 구현한다.
2. Subject의 상태가 변경되면 **모든 Observer에게 자동 알림**된다.
3. Python에서는 **콜백/이벤트 에미터**로, C#에서는 **event/delegate**로 관용적으로 구현한다.
4. **Push 방식**은 데이터를 직접 전달하고, **Pull 방식**은 Observer가 필요한 것을 가져간다.

### 적용 체크리스트

- [ ] 한 객체의 변화가 **다른 여러 객체에 영향**을 미치는가?
- [ ] **발행-구독** 구조가 필요한가?
- [ ] Observer의 종류나 수가 **런타임에 변경**될 수 있는가?
- [ ] Subject와 Observer를 **독립적으로 변경**하고 싶은가?
- [ ] **이벤트 기반** 아키텍처를 구현하고 있는가?

> **2~3년차 개발자를 위한 팁**: React의 `useState`+리렌더링, Vue의 반응성 시스템, Angular의 RxJS, C#의 `event`/`delegate`, Node.js의 `EventEmitter`, DOM의 `addEventListener` 모두 Observer 패턴의 변형이다. 가장 흔히 사용되는 디자인 패턴 중 하나이므로, 의식적으로 인식하는 연습을 하면 코드 설계가 크게 개선된다. 특히 구독 해제를 잊지 않도록 주의하자 (메모리 누수의 주범!).
