# Decorator 패턴 (데코레이터)

> **핵심 의도 한줄 요약:** 상속 없이, 객체에 동적으로 새로운 기능을 추가하는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Decorator vs 상속 비교](#5-decorator-vs-상속-비교)
6. [패턴 적용 전 (Before)](#6-패턴-적용-전-before)
7. [패턴 적용 후 (After)](#7-패턴-적용-후-after)
8. [Python 구현](#8-python-구현)
9. [C# 구현](#9-c-구현)
10. [실무 예제: 로깅, 캐싱, 인증 데코레이터](#10-실무-예제-로깅-캐싱-인증-데코레이터)
11. [장단점](#11-장단점)
12. [관련 패턴](#12-관련-패턴)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)

---

## 1. 개요

Decorator 패턴은 **GoF 구조 패턴** 중 하나로, 객체에 추가 책임(기능)을 동적으로 부여합니다. 기능 확장을 위해 서브클래싱 대신 유연한 대안을 제공합니다.

핵심 아이디어: "객체를 다른 객체로 감싸서(wrap) 새로운 행동을 추가한다."

실무에서 마주치는 상황:
- HTTP 요청에 로깅, 인증, 캐싱을 단계적으로 추가
- 데이터 스트림에 암호화, 압축, 버퍼링을 조합
- 알림 메시지에 포맷팅, 필터링, 우선순위를 동적으로 적용

---

## 2. 왜 필요한가? - 문제 상황

### 상속으로 기능을 조합하면?

커피 주문 시스템에서 옵션 조합을 상속으로 처리:

```
기본 커피 종류: 에스프레소, 콜드브루
옵션: 우유, 시럽, 휘핑크림

상속으로 하면?
├── 에스프레소
├── 에스프레소 + 우유
├── 에스프레소 + 시럽
├── 에스프레소 + 우유 + 시럽
├── 에스프레소 + 우유 + 휘핑크림
├── 에스프레소 + 시럽 + 휘핑크림
├── 에스프레소 + 우유 + 시럽 + 휘핑크림
├── 콜드브루
├── 콜드브루 + 우유
├── ... (반복)
→ 2 x 2^3 = 16개 클래스!

새 옵션 "카라멜" 추가? → 클래스 2배로 증가!
```

**문제:** 옵션 조합마다 새 클래스가 필요하고, 기능 조합이 기하급수적으로 증가합니다.

---

## 3. 비유로 이해하기

### 커피 옵션 추가

```
[에스프레소] → 기본 가격 3,000원

옵션 추가 과정 (데코레이터를 겹겹이 감싸기):

1. 에스프레소 주문             → 3,000원
2. [우유]로 감싸기             → 3,000 + 500 = 3,500원
3. [바닐라 시럽]으로 감싸기     → 3,500 + 300 = 3,800원
4. [휘핑크림]으로 감싸기       → 3,800 + 500 = 4,300원

구조:
┌─────────────────────────────────────┐
│ 휘핑크림 데코레이터 (+500원)        │
│  ┌─────────────────────────────┐    │
│  │ 바닐라시럽 데코레이터 (+300원)│   │
│  │  ┌───────────────────────┐  │    │
│  │  │ 우유 데코레이터 (+500원)│  │   │
│  │  │  ┌─────────────────┐  │  │    │
│  │  │  │  에스프레소      │  │  │    │
│  │  │  │  (3,000원)      │  │  │    │
│  │  │  └─────────────────┘  │  │    │
│  │  └───────────────────────┘  │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

## 4. UML 다이어그램

```
┌────────────────────────┐
│   <<interface>>         │
│     Component           │
│────────────────────────│
│ + operation()           │
└────────────────────────┘
            △
            │ implements
   ┌────────┴──────────────────────────┐
   │                                    │
┌──────────────┐          ┌─────────────────────────┐
│ Concrete     │          │    Decorator             │
│ Component    │          │─────────────────────────│
│──────────────│          │ - wrapped: Component     │──> Component
│ + operation()│          │─────────────────────────│
│              │          │ + operation()             │
└──────────────┘          │   wrapped.operation()    │
                          │   + 추가 기능            │
                          └─────────────────────────┘
                                     △
                                     │
                          ┌──────────┴──────────┐
                   ┌──────────────┐    ┌──────────────┐
                   │ ConcreteDecA │    │ ConcreteDecB │
                   │──────────────│    │──────────────│
                   │ + operation()│    │ + operation()│
                   └──────────────┘    └──────────────┘
```

**핵심:** Decorator는 Component와 같은 인터페이스를 구현하면서, 내부에 Component를 가지고 있습니다.

---

## 5. Decorator vs 상속 비교

| 구분 | 상속 | Decorator |
|------|------|-----------|
| **시점** | 컴파일 타임 (정적) | 런타임 (동적) |
| **조합** | 모든 조합마다 클래스 필요 | 필요한 만큼 감싸기만 하면 됨 |
| **유연성** | 낮음 | 높음 (런타임에 기능 추가/제거) |
| **클래스 수** | 조합 폭발 | 기본 + 데코레이터 수만큼 |
| **단일 책임** | 클래스마다 여러 기능 | 데코레이터 하나에 기능 하나 |
| **코드 이해** | 직관적 | 래핑 구조 파악 필요 |

### 코드로 보면

```python
# 상속: 조합별 클래스
class EspressoWithMilk(Espresso): ...
class EspressoWithMilkAndSyrup(EspressoWithMilk): ...
class EspressoWithMilkAndSyrupAndWhip(EspressoWithMilkAndSyrup): ...

# Decorator: 동적 조합
coffee = Espresso()
coffee = MilkDecorator(coffee)
coffee = SyrupDecorator(coffee)
coffee = WhipDecorator(coffee)
```

---

## 6. 패턴 적용 전 (Before)

```python
# ❌ Before: 상속으로 모든 조합을 표현

class DataService:
    def get_data(self, key: str) -> str:
        # DB에서 데이터 조회 (느린 작업)
        print(f"[DB] {key} 조회 중...")
        return f"data_{key}"


# 로깅이 필요하면?
class LoggingDataService(DataService):
    def get_data(self, key: str) -> str:
        print(f"[LOG] get_data 호출: key={key}")
        result = super().get_data(key)
        print(f"[LOG] get_data 결과: {result}")
        return result


# 캐싱이 필요하면?
class CachingDataService(DataService):
    def __init__(self):
        self._cache = {}

    def get_data(self, key: str) -> str:
        if key in self._cache:
            print(f"[CACHE] 캐시 히트: {key}")
            return self._cache[key]
        result = super().get_data(key)
        self._cache[key] = result
        return result


# 로깅 + 캐싱이 필요하면? 또 새 클래스!
class LoggingCachingDataService(DataService):
    def __init__(self):
        self._cache = {}

    def get_data(self, key: str) -> str:
        print(f"[LOG] get_data 호출: key={key}")  # 로깅 코드 중복!
        if key in self._cache:
            result = self._cache[key]
        else:
            result = super().get_data(key)
            self._cache[key] = result
        print(f"[LOG] get_data 결과: {result}")  # 로깅 코드 중복!
        return result


# 인증 + 로깅 + 캐싱? → 또 새 클래스...
# 인증 + 캐싱 (로깅 없이)? → 또 새 클래스...
```

**문제점:**
- 기능 조합마다 새로운 클래스 필요
- 코드 중복 (로깅 코드가 여러 클래스에 반복)
- 기능 추가/제거가 유연하지 않음

---

## 7. 패턴 적용 후 (After)

```python
# ✅ After: Decorator 패턴으로 동적 기능 조합

from abc import ABC, abstractmethod


class DataService(ABC):
    @abstractmethod
    def get_data(self, key: str) -> str:
        pass


class DatabaseService(DataService):
    def get_data(self, key: str) -> str:
        print(f"[DB] {key} 조회 중...")
        return f"data_{key}"


# 데코레이터 베이스
class DataServiceDecorator(DataService):
    def __init__(self, wrapped: DataService):
        self._wrapped = wrapped

    def get_data(self, key: str) -> str:
        return self._wrapped.get_data(key)


# 각 기능은 독립적인 데코레이터
class LoggingDecorator(DataServiceDecorator):
    def get_data(self, key: str) -> str:
        print(f"[LOG] get_data 호출: key={key}")
        result = self._wrapped.get_data(key)
        print(f"[LOG] get_data 결과: {result}")
        return result


class CachingDecorator(DataServiceDecorator):
    def __init__(self, wrapped: DataService):
        super().__init__(wrapped)
        self._cache = {}

    def get_data(self, key: str) -> str:
        if key in self._cache:
            print(f"[CACHE] 캐시 히트: {key}")
            return self._cache[key]
        result = self._wrapped.get_data(key)
        self._cache[key] = result
        return result


# 자유로운 조합!
service = DatabaseService()
service = CachingDecorator(service)  # 캐싱 추가
service = LoggingDecorator(service)  # 로깅 추가
service.get_data("user_1")
```

---

## 8. Python 구현

### GoF 스타일 클래스 데코레이터

```python
from abc import ABC, abstractmethod


# ===== Component =====
class Coffee(ABC):
    """커피 인터페이스 (Component)"""

    @abstractmethod
    def get_description(self) -> str:
        pass

    @abstractmethod
    def get_cost(self) -> int:
        pass

    def __str__(self) -> str:
        return f"{self.get_description()} = {self.get_cost():,}원"


# ===== Concrete Components =====
class Espresso(Coffee):
    def get_description(self) -> str:
        return "에스프레소"

    def get_cost(self) -> int:
        return 3000


class ColdBrew(Coffee):
    def get_description(self) -> str:
        return "콜드브루"

    def get_cost(self) -> int:
        return 4000


class Drip(Coffee):
    def get_description(self) -> str:
        return "드립커피"

    def get_cost(self) -> int:
        return 3500


# ===== Decorator Base =====
class CoffeeDecorator(Coffee):
    """데코레이터 베이스 클래스"""

    def __init__(self, coffee: Coffee):
        self._coffee = coffee

    def get_description(self) -> str:
        return self._coffee.get_description()

    def get_cost(self) -> int:
        return self._coffee.get_cost()


# ===== Concrete Decorators =====
class MilkDecorator(CoffeeDecorator):
    def get_description(self) -> str:
        return f"{self._coffee.get_description()} + 우유"

    def get_cost(self) -> int:
        return self._coffee.get_cost() + 500


class SyrupDecorator(CoffeeDecorator):
    def __init__(self, coffee: Coffee, flavor: str = "바닐라"):
        super().__init__(coffee)
        self._flavor = flavor

    def get_description(self) -> str:
        return f"{self._coffee.get_description()} + {self._flavor}시럽"

    def get_cost(self) -> int:
        return self._coffee.get_cost() + 300


class WhipCreamDecorator(CoffeeDecorator):
    def get_description(self) -> str:
        return f"{self._coffee.get_description()} + 휘핑크림"

    def get_cost(self) -> int:
        return self._coffee.get_cost() + 500


class ShotDecorator(CoffeeDecorator):
    def __init__(self, coffee: Coffee, shots: int = 1):
        super().__init__(coffee)
        self._shots = shots

    def get_description(self) -> str:
        return f"{self._coffee.get_description()} + 샷{self._shots}개"

    def get_cost(self) -> int:
        return self._coffee.get_cost() + (500 * self._shots)


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 기본 에스프레소
    coffee1 = Espresso()
    print(f"주문 1: {coffee1}")

    # 카페라떼 (에스프레소 + 우유)
    coffee2 = Espresso()
    coffee2 = MilkDecorator(coffee2)
    print(f"주문 2: {coffee2}")

    # 바닐라 라떼 + 휘핑 (에스프레소 + 우유 + 바닐라시럽 + 휘핑)
    coffee3 = Espresso()
    coffee3 = MilkDecorator(coffee3)
    coffee3 = SyrupDecorator(coffee3, "바닐라")
    coffee3 = WhipCreamDecorator(coffee3)
    print(f"주문 3: {coffee3}")

    # 더블샷 콜드브루
    coffee4 = ColdBrew()
    coffee4 = ShotDecorator(coffee4, shots=2)
    print(f"주문 4: {coffee4}")
```

**실행 결과:**
```
주문 1: 에스프레소 = 3,000원
주문 2: 에스프레소 + 우유 = 3,500원
주문 3: 에스프레소 + 우유 + 바닐라시럽 + 휘핑크림 = 4,300원
주문 4: 콜드브루 + 샷2개 = 5,000원
```

### Python 함수 데코레이터 (언어 고유 기능)

> **주의:** Python의 `@decorator` 문법은 GoF의 Decorator 패턴과는 다른 개념입니다.
> GoF Decorator: 객체를 감싸서 기능 추가 (구조 패턴)
> Python Decorator: 함수/클래스를 감싸서 기능 추가 (언어 기능)
> 원리는 비슷하나, 적용 대상과 방식이 다릅니다.

```python
import functools
import time
import logging
from typing import Callable, Any

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


# ===== 함수 데코레이터: 로깅 =====
def log_calls(func: Callable) -> Callable:
    """함수 호출 전후에 로그를 남기는 데코레이터"""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"[CALL] {func.__name__}({args}, {kwargs})")
        try:
            result = func(*args, **kwargs)
            logger.info(f"[RETURN] {func.__name__} -> {result}")
            return result
        except Exception as e:
            logger.error(f"[ERROR] {func.__name__} -> {e}")
            raise

    return wrapper


# ===== 함수 데코레이터: 실행 시간 측정 =====
def measure_time(func: Callable) -> Callable:
    """함수 실행 시간을 측정하는 데코레이터"""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"[TIMER] {func.__name__}: {elapsed:.4f}초")
        return result

    return wrapper


# ===== 함수 데코레이터: 캐싱 =====
def cache(max_size: int = 128):
    """결과를 캐시하는 데코레이터 (매개변수 있는 데코레이터)"""

    def decorator(func: Callable) -> Callable:
        _cache: dict = {}

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            if key in _cache:
                print(f"[CACHE HIT] {func.__name__}{args}")
                return _cache[key]
            result = func(*args, **kwargs)
            if len(_cache) >= max_size:
                oldest = next(iter(_cache))
                del _cache[oldest]
            _cache[key] = result
            return result

        wrapper.clear_cache = lambda: _cache.clear()
        return wrapper

    return decorator


# ===== 함수 데코레이터: 재시도 =====
def retry(max_attempts: int = 3, delay: float = 1.0):
    """실패 시 재시도하는 데코레이터"""

    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"[RETRY] {func.__name__} 시도 "
                          f"{attempt}/{max_attempts} 실패: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise last_exception

        return wrapper

    return decorator


# ===== 사용 예시: 데코레이터 조합 =====
@log_calls
@measure_time
@cache(max_size=64)
@retry(max_attempts=3, delay=0.5)
def fetch_user_data(user_id: int) -> dict:
    """외부 API에서 사용자 데이터를 가져오는 함수"""
    # 실제로는 API 호출
    time.sleep(0.1)  # 시뮬레이션
    return {"id": user_id, "name": f"User_{user_id}"}


if __name__ == "__main__":
    # 첫 번째 호출: retry -> cache miss -> measure -> log
    result1 = fetch_user_data(42)
    print(f"결과: {result1}\n")

    # 두 번째 호출: cache hit! (나머지 데코레이터는 캐시 위에 있으므로 동작)
    result2 = fetch_user_data(42)
    print(f"결과: {result2}")
```

### Python 클래스 기반 데코레이터와 함수 데코레이터 차이점 정리

```python
# ===== 1. GoF Decorator 패턴 (클래스 기반) =====
# - "객체"를 감싸서 기능 추가
# - 동일한 인터페이스 유지
# - 런타임에 동적으로 조합/해제 가능

class Component(ABC):
    @abstractmethod
    def execute(self): pass

class ConcreteComponent(Component):
    def execute(self): return "기본 동작"

class LoggingDecorator(Component):
    def __init__(self, wrapped: Component):
        self._wrapped = wrapped
    def execute(self):
        print("로그 시작")
        result = self._wrapped.execute()  # 위임
        print("로그 끝")
        return result

# 사용: 객체를 감쌈
obj = ConcreteComponent()
obj = LoggingDecorator(obj)  # 감싸기
obj.execute()


# ===== 2. Python 함수 데코레이터 (언어 기능) =====
# - "함수/메서드"를 감싸서 기능 추가
# - @문법 사용
# - 정의 시점에 적용 (데코레이터 해제 불가)

def logging_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("로그 시작")
        result = func(*args, **kwargs)
        print("로그 끝")
        return result
    return wrapper

# 사용: @문법으로 함수를 감쌈
@logging_decorator
def my_function():
    return "기본 동작"

my_function()
```

---

## 9. C# 구현

### Stream/IO 데코레이터 패턴

> **참고:** C#의 `System.IO.Stream`은 Decorator 패턴의 대표적인 실제 사례입니다.
> `BufferedStream`, `CryptoStream`, `GZipStream` 등이 모두 Stream을 감싸는 데코레이터입니다.

```csharp
using System;
using System.Collections.Generic;
using System.Text;

// ===== Component =====
public interface IDataReader
{
    string ReadData(string source);
}

// ===== Concrete Component =====
public class FileDataReader : IDataReader
{
    public string ReadData(string source)
    {
        Console.WriteLine($"[File] '{source}' 파일에서 데이터 읽기");
        return $"파일 내용: {source}의 원본 데이터";
    }
}

// ===== Decorator Base =====
public abstract class DataReaderDecorator : IDataReader
{
    protected readonly IDataReader _wrapped;

    protected DataReaderDecorator(IDataReader wrapped)
    {
        _wrapped = wrapped;
    }

    public virtual string ReadData(string source)
    {
        return _wrapped.ReadData(source);
    }
}

// ===== Concrete Decorators =====
public class EncryptionDecorator : DataReaderDecorator
{
    public EncryptionDecorator(IDataReader wrapped)
        : base(wrapped) { }

    public override string ReadData(string source)
    {
        var data = base.ReadData(source);
        return Decrypt(data);
    }

    private string Decrypt(string data)
    {
        Console.WriteLine("[Encryption] 데이터 복호화 중...");
        return $"[복호화됨] {data}";
    }
}

public class CompressionDecorator : DataReaderDecorator
{
    public CompressionDecorator(IDataReader wrapped)
        : base(wrapped) { }

    public override string ReadData(string source)
    {
        var data = base.ReadData(source);
        return Decompress(data);
    }

    private string Decompress(string data)
    {
        Console.WriteLine("[Compression] 데이터 압축 해제 중...");
        return $"[압축해제됨] {data}";
    }
}

public class LoggingDecorator : DataReaderDecorator
{
    public LoggingDecorator(IDataReader wrapped)
        : base(wrapped) { }

    public override string ReadData(string source)
    {
        Console.WriteLine($"[LOG] ReadData 호출: source={source}");
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();

        var result = base.ReadData(source);

        stopwatch.Stop();
        Console.WriteLine(
            $"[LOG] ReadData 완료: {stopwatch.ElapsedMilliseconds}ms");
        return result;
    }
}

public class CachingDecorator : DataReaderDecorator
{
    private readonly Dictionary<string, string> _cache = new();

    public CachingDecorator(IDataReader wrapped)
        : base(wrapped) { }

    public override string ReadData(string source)
    {
        if (_cache.TryGetValue(source, out var cached))
        {
            Console.WriteLine($"[CACHE] 캐시 히트: {source}");
            return cached;
        }

        var data = base.ReadData(source);
        _cache[source] = data;
        Console.WriteLine($"[CACHE] 캐시 저장: {source}");
        return data;
    }

    public void ClearCache() => _cache.Clear();
}

// ===== 사용 예시 =====
public class Program
{
    public static void Main()
    {
        // 기본 리더
        IDataReader reader = new FileDataReader();

        // 데코레이터로 기능 추가 (순서 중요!)
        reader = new CachingDecorator(reader);     // 1. 캐싱
        reader = new CompressionDecorator(reader); // 2. 압축 해제
        reader = new EncryptionDecorator(reader);  // 3. 복호화
        reader = new LoggingDecorator(reader);     // 4. 로깅

        // 실행 흐름:
        // 로깅 -> 복호화 -> 압축해제 -> 캐싱 -> 파일읽기
        Console.WriteLine("=== 첫 번째 읽기 ===");
        var data = reader.ReadData("config.dat");
        Console.WriteLine($"결과: {data}\n");

        Console.WriteLine("=== 두 번째 읽기 (캐시) ===");
        data = reader.ReadData("config.dat");
        Console.WriteLine($"결과: {data}");
    }
}
```

### C# 실제 Stream 데코레이터 예시

```csharp
using System;
using System.IO;
using System.IO.Compression;
using System.Security.Cryptography;
using System.Text;

// C#의 System.IO.Stream이 Decorator 패턴의 실제 사례
public class StreamDecoratorExample
{
    public static void Main()
    {
        var data = "Hello, Decorator Pattern!";
        var filePath = "test.dat";

        // 쓰기: 파일스트림 → 압축 → 버퍼 (데코레이터 체인)
        using (Stream fileStream = new FileStream(filePath, FileMode.Create))
        using (Stream gzipStream = new GZipStream(fileStream,
                                                    CompressionMode.Compress))
        using (Stream bufferedStream = new BufferedStream(gzipStream))
        {
            var bytes = Encoding.UTF8.GetBytes(data);
            bufferedStream.Write(bytes, 0, bytes.Length);
        }

        // 읽기: 파일스트림 → 압축해제 → 버퍼 (역순 데코레이터)
        using (Stream fileStream = new FileStream(filePath, FileMode.Open))
        using (Stream gzipStream = new GZipStream(fileStream,
                                                    CompressionMode.Decompress))
        using (var reader = new StreamReader(gzipStream))
        {
            var result = reader.ReadToEnd();
            Console.WriteLine($"읽은 데이터: {result}");
        }
    }
}
```

---

## 10. 실무 예제: 로깅, 캐싱, 인증 데코레이터

### Python - HTTP 핸들러 데코레이터 체인

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional
import time
import hashlib
import json


@dataclass
class HttpRequest:
    method: str
    path: str
    headers: dict = field(default_factory=dict)
    body: Optional[str] = None


@dataclass
class HttpResponse:
    status_code: int
    body: str
    headers: dict = field(default_factory=dict)


# ===== Component: HTTP 핸들러 =====
class HttpHandler(ABC):
    @abstractmethod
    def handle(self, request: HttpRequest) -> HttpResponse:
        pass


# ===== Concrete Component: 실제 API 핸들러 =====
class UserApiHandler(HttpHandler):
    """실제 사용자 API 로직"""

    def handle(self, request: HttpRequest) -> HttpResponse:
        if request.method == "GET" and request.path == "/api/users":
            time.sleep(0.1)  # DB 조회 시뮬레이션
            users = json.dumps([
                {"id": 1, "name": "Alice"},
                {"id": 2, "name": "Bob"}
            ])
            return HttpResponse(200, users)

        return HttpResponse(404, '{"error": "Not Found"}')


# ===== Decorator 1: 인증 =====
class AuthDecorator(HttpHandler):
    """인증 검사 데코레이터"""

    VALID_TOKENS = {"token-admin", "token-user"}

    def __init__(self, handler: HttpHandler):
        self._handler = handler

    def handle(self, request: HttpRequest) -> HttpResponse:
        token = request.headers.get("Authorization", "")
        if token not in self.VALID_TOKENS:
            print(f"[AUTH] 인증 실패: {token}")
            return HttpResponse(
                401, '{"error": "Unauthorized"}'
            )

        print(f"[AUTH] 인증 성공: {token}")
        return self._handler.handle(request)


# ===== Decorator 2: 로깅 =====
class LoggingDecorator(HttpHandler):
    """요청/응답 로깅 데코레이터"""

    def __init__(self, handler: HttpHandler):
        self._handler = handler

    def handle(self, request: HttpRequest) -> HttpResponse:
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[LOG {timestamp}] "
              f"{request.method} {request.path}")

        start = time.perf_counter()
        response = self._handler.handle(request)
        elapsed = time.perf_counter() - start

        print(f"[LOG] 응답: {response.status_code} "
              f"({elapsed:.4f}초)")
        return response


# ===== Decorator 3: 캐싱 =====
class CachingDecorator(HttpHandler):
    """GET 요청 캐싱 데코레이터"""

    def __init__(self, handler: HttpHandler, ttl_seconds: int = 60):
        self._handler = handler
        self._cache: dict[str, tuple[HttpResponse, float]] = {}
        self._ttl = ttl_seconds

    def handle(self, request: HttpRequest) -> HttpResponse:
        # GET 요청만 캐싱
        if request.method != "GET":
            return self._handler.handle(request)

        cache_key = self._make_key(request)

        # 캐시 확인
        if cache_key in self._cache:
            cached_response, cached_time = self._cache[cache_key]
            if time.time() - cached_time < self._ttl:
                print(f"[CACHE] 히트: {request.path}")
                return cached_response
            else:
                print(f"[CACHE] 만료: {request.path}")
                del self._cache[cache_key]

        # 캐시 미스
        print(f"[CACHE] 미스: {request.path}")
        response = self._handler.handle(request)

        if response.status_code == 200:
            self._cache[cache_key] = (response, time.time())

        return response

    def _make_key(self, request: HttpRequest) -> str:
        raw = f"{request.method}:{request.path}"
        return hashlib.md5(raw.encode()).hexdigest()


# ===== Decorator 4: Rate Limiting =====
class RateLimitDecorator(HttpHandler):
    """요청 속도 제한 데코레이터"""

    def __init__(self, handler: HttpHandler,
                 max_requests: int = 10,
                 window_seconds: int = 60):
        self._handler = handler
        self._max_requests = max_requests
        self._window = window_seconds
        self._requests: list[float] = []

    def handle(self, request: HttpRequest) -> HttpResponse:
        now = time.time()
        # 윈도우 밖의 요청 제거
        self._requests = [
            t for t in self._requests
            if now - t < self._window
        ]

        if len(self._requests) >= self._max_requests:
            print(f"[RATE LIMIT] 초과! "
                  f"({len(self._requests)}/{self._max_requests})")
            return HttpResponse(
                429, '{"error": "Too Many Requests"}'
            )

        self._requests.append(now)
        remaining = self._max_requests - len(self._requests)
        print(f"[RATE LIMIT] 통과 (남은 횟수: {remaining})")

        response = self._handler.handle(request)
        response.headers["X-RateLimit-Remaining"] = str(remaining)
        return response


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 데코레이터 체인 구성
    # 실행 순서: RateLimit -> Logging -> Auth -> Caching -> API
    handler = UserApiHandler()
    handler = CachingDecorator(handler, ttl_seconds=300)
    handler = AuthDecorator(handler)
    handler = LoggingDecorator(handler)
    handler = RateLimitDecorator(handler, max_requests=5)

    # 테스트 1: 정상 요청
    print("=" * 50)
    print("테스트 1: 정상 GET 요청")
    print("=" * 50)
    request = HttpRequest(
        method="GET",
        path="/api/users",
        headers={"Authorization": "token-admin"}
    )
    response = handler.handle(request)
    print(f"결과: {response.status_code} {response.body}\n")

    # 테스트 2: 같은 요청 (캐시 히트)
    print("=" * 50)
    print("테스트 2: 같은 요청 (캐시)")
    print("=" * 50)
    response = handler.handle(request)
    print(f"결과: {response.status_code}\n")

    # 테스트 3: 인증 실패
    print("=" * 50)
    print("테스트 3: 인증 실패")
    print("=" * 50)
    bad_request = HttpRequest(
        method="GET",
        path="/api/users",
        headers={"Authorization": "invalid-token"}
    )
    response = handler.handle(bad_request)
    print(f"결과: {response.status_code} {response.body}")
```

### C# - ASP.NET 스타일 미들웨어 데코레이터

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;

public class HttpRequest
{
    public string Method { get; set; }
    public string Path { get; set; }
    public Dictionary<string, string> Headers { get; set; } = new();
    public string Body { get; set; }
}

public class HttpResponse
{
    public int StatusCode { get; set; }
    public string Body { get; set; }
    public Dictionary<string, string> Headers { get; set; } = new();
}

// Component
public interface IHttpHandler
{
    HttpResponse Handle(HttpRequest request);
}

// Concrete Component
public class ApiHandler : IHttpHandler
{
    public HttpResponse Handle(HttpRequest request)
    {
        return new HttpResponse
        {
            StatusCode = 200,
            Body = "{\"users\": [{\"id\": 1}]}"
        };
    }
}

// Decorator: 인증
public class AuthMiddleware : IHttpHandler
{
    private readonly IHttpHandler _next;
    private readonly HashSet<string> _validTokens = new()
        { "token-admin", "token-user" };

    public AuthMiddleware(IHttpHandler next) => _next = next;

    public HttpResponse Handle(HttpRequest request)
    {
        request.Headers.TryGetValue("Authorization", out var token);
        if (!_validTokens.Contains(token ?? ""))
        {
            Console.WriteLine($"[AUTH] 인증 실패");
            return new HttpResponse
            {
                StatusCode = 401,
                Body = "{\"error\": \"Unauthorized\"}"
            };
        }

        Console.WriteLine($"[AUTH] 인증 성공");
        return _next.Handle(request);
    }
}

// Decorator: 로깅
public class LoggingMiddleware : IHttpHandler
{
    private readonly IHttpHandler _next;

    public LoggingMiddleware(IHttpHandler next) => _next = next;

    public HttpResponse Handle(HttpRequest request)
    {
        Console.WriteLine(
            $"[LOG] {request.Method} {request.Path}");
        var sw = Stopwatch.StartNew();

        var response = _next.Handle(request);

        sw.Stop();
        Console.WriteLine(
            $"[LOG] {response.StatusCode} ({sw.ElapsedMilliseconds}ms)");
        return response;
    }
}

// Decorator: 캐싱
public class CachingMiddleware : IHttpHandler
{
    private readonly IHttpHandler _next;
    private readonly Dictionary<string, HttpResponse> _cache = new();

    public CachingMiddleware(IHttpHandler next) => _next = next;

    public HttpResponse Handle(HttpRequest request)
    {
        if (request.Method != "GET")
            return _next.Handle(request);

        var key = $"{request.Method}:{request.Path}";
        if (_cache.TryGetValue(key, out var cached))
        {
            Console.WriteLine($"[CACHE] 히트: {request.Path}");
            return cached;
        }

        var response = _next.Handle(request);
        if (response.StatusCode == 200)
            _cache[key] = response;

        return response;
    }
}

// 사용
public class Program
{
    public static void Main()
    {
        // 파이프라인 구성 (안에서 바깥으로)
        IHttpHandler pipeline = new ApiHandler();
        pipeline = new CachingMiddleware(pipeline);
        pipeline = new AuthMiddleware(pipeline);
        pipeline = new LoggingMiddleware(pipeline);

        var request = new HttpRequest
        {
            Method = "GET",
            Path = "/api/users",
            Headers = new() { ["Authorization"] = "token-admin" }
        };

        Console.WriteLine("=== 첫 번째 요청 ===");
        var response = pipeline.Handle(request);
        Console.WriteLine($"결과: {response.StatusCode}\n");

        Console.WriteLine("=== 두 번째 요청 (캐시) ===");
        response = pipeline.Handle(request);
        Console.WriteLine($"결과: {response.StatusCode}");
    }
}
```

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **동적 기능 추가** | 런타임에 기능을 추가/제거 가능 |
| **단일 책임 원칙** | 각 데코레이터가 하나의 기능만 담당 |
| **조합 폭발 방지** | N개 기능의 모든 조합을 N개 클래스로 해결 |
| **개방/폐쇄 원칙** | 기존 코드 수정 없이 새 데코레이터 추가 |
| **순서 제어** | 데코레이터 적용 순서로 실행 흐름 제어 |

### 단점

| 단점 | 설명 |
|------|------|
| **순서 의존성** | 데코레이터 순서가 잘못되면 예상치 못한 동작 |
| **디버깅 어려움** | 여러 겹의 래핑으로 스택 트레이스가 복잡 |
| **초기 설정 복잡** | 래핑 코드가 길어질 수 있음 |
| **특정 데코레이터 제거** | 중간 데코레이터만 제거하기 어려움 |

---

## 12. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Adapter** | 인터페이스를 변환 vs Decorator는 인터페이스를 유지하면서 기능 추가 |
| **Proxy** | Proxy는 생명주기 관리, Decorator는 기능 추가에 초점 |
| **Composite** | Decorator는 래핑이 하나, Composite는 여러 자식 |
| **Strategy** | Strategy는 내부 알고리즘 교체, Decorator는 외부에서 기능 추가 |
| **Chain of Responsibility** | 데코레이터 체인과 구조적으로 유사 |

---

## 13. 정리 및 체크리스트

### 핵심 정리

```
Decorator 패턴 = "기능을 겹겹이 감싸기"

When: 객체에 동적으로 기능을 추가/제거해야 할 때
How:  같은 인터페이스를 구현하는 래퍼 클래스로 감싸기
Why:  상속의 조합 폭발을 피하고, 단일 책임을 유지
```

### 적용 체크리스트

- [ ] 여러 기능을 조합해야 하는가? (상속으로 하면 클래스 폭발?)
- [ ] 런타임에 기능을 추가/제거해야 하는가?
- [ ] 각 기능이 독립적이고 재사용 가능한가?
- [ ] 데코레이터 순서가 중요한가? (순서를 문서화했는가?)
- [ ] Python의 함수 데코레이터와 GoF 데코레이터를 혼동하지 않았는가?

### 실무 적용 시나리오

1. **HTTP 미들웨어:** 인증, 로깅, 캐싱, 압축, CORS
2. **IO 스트림:** 버퍼링, 암호화, 압축 (C#의 Stream)
3. **로깅:** 다양한 출력 대상과 포맷 조합
4. **데이터 검증:** 입력 데이터에 여러 검증 규칙 적용
5. **메시지 처리:** 직렬화, 압축, 암호화 파이프라인

> **기억하세요:** "상속 대신 구성으로, 정적 대신 동적으로, 기능을 하나씩 겹겹이 추가하세요."
