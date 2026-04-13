# Template Method 패턴 (템플릿 메서드)

> **핵심 의도 한줄 요약**: 알고리즘의 골격(뼈대)을 상위 클래스에서 정의하고, 각 단계의 세부 구현을 하위 클래스에서 재정의하게 한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Hook 메서드](#5-hook-메서드)
6. [Python 구현](#6-python-구현)
7. [C# 구현](#7-c-구현)
8. [Template Method vs Strategy 차이점](#8-template-method-vs-strategy-차이점)
9. [실무 예제: 데이터 처리 파이프라인](#9-실무-예제-데이터-처리-파이프라인)
10. [장단점](#10-장단점)
11. [관련 패턴](#11-관련-패턴)
12. [정리 및 체크리스트](#12-정리-및-체크리스트)

---

## 1. 개요

Template Method 패턴은 **상위 클래스에서 알고리즘의 전체 흐름을 정의**하고, 특정 단계를 하위 클래스에서 재정의(override)하게 하는 패턴이다.

### 핵심 아이디어

```
상위 클래스 (AbstractClass):
  def template_method():      ← 전체 흐름 (변경 불가)
      step1()                  ← 공통 구현
      step2()                  ← 추상 메서드 (하위 클래스에서 구현)
      step3()                  ← 추상 메서드 (하위 클래스에서 구현)
      hook()                   ← 선택적 재정의 (Hook)

하위 클래스 (ConcreteClass):
  step2()  → 구체적 구현 A
  step3()  → 구체적 구현 A
```

**"변하지 않는 부분은 상위 클래스에, 변하는 부분은 하위 클래스에"** 두는 원칙을 따른다.

---

## 2. 문제 상황

### 문제: 중복되는 알고리즘 구조

다양한 형식의 문서를 파싱하는 시스템을 만든다.

```python
# 나쁜 예: 비슷한 흐름이 중복됨
class CSVParser:
    def parse(self, file_path):
        file = open(file_path)           # 1. 파일 열기 (공통)
        raw_data = file.read()            # 2. 데이터 읽기 (공통)
        data = raw_data.split(",")        # 3. CSV 파싱 (고유)
        result = self.validate(data)      # 4. 유효성 검증 (공통)
        file.close()                      # 5. 파일 닫기 (공통)
        return result

class JSONParser:
    def parse(self, file_path):
        file = open(file_path)           # 1. 파일 열기 (공통) ← 중복!
        raw_data = file.read()            # 2. 데이터 읽기 (공통) ← 중복!
        data = json.loads(raw_data)       # 3. JSON 파싱 (고유)
        result = self.validate(data)      # 4. 유효성 검증 (공통) ← 중복!
        file.close()                      # 5. 파일 닫기 (공통) ← 중복!
        return result

class XMLParser:
    def parse(self, file_path):
        file = open(file_path)           # 중복!
        raw_data = file.read()            # 중복!
        data = ElementTree.parse(...)     # 3. XML 파싱 (고유)
        result = self.validate(data)      # 중복!
        file.close()                      # 중복!
        return result
```

문제점:

- **코드 중복**: 1, 2, 4, 5단계가 모든 파서에서 반복
- **유지보수 어려움**: 공통 로직 변경 시 모든 파서를 수정해야 함
- **일관성 부재**: 각 파서가 약간씩 다르게 구현될 위험

---

## 3. 일상 비유

**요리 레시피**를 떠올려 보자.

```
[기본 파스타 레시피] (Template Method)
  1. 물 끓이기              ← 공통
  2. 면 삶기                ← 공통
  3. 소스 만들기            ← 변하는 부분!
  4. 면과 소스 섞기          ← 공통
  5. 플레이팅               ← 공통 (+ Hook: 토핑 추가?)

[카르보나라]               [토마토 파스타]
  3. 계란+치즈+베이컨        3. 토마토+바질+올리브유
     소스 만들기               소스 만들기

전체 흐름(레시피)은 같지만, 소스 만드는 방법만 다르다.
```

---

## 4. UML 다이어그램

```
 ┌──────────────────────────────┐
 │      AbstractClass           │
 ├──────────────────────────────┤
 │ + template_method()          │  ← final (재정의 불가)
 │   ├── step1()                │  ← 공통 구현
 │   ├── step2()   <<abstract>> │  ← 하위 클래스에서 구현
 │   ├── step3()   <<abstract>> │  ← 하위 클래스에서 구현
 │   └── hook()                 │  ← 선택적 재정의
 ├──────────────────────────────┤
 │ # step1()                    │
 │ # step2()  {abstract}        │
 │ # step3()  {abstract}        │
 │ # hook()   {default impl}    │
 └──────────────┬───────────────┘
                │
       ┌────────┴─────────┐
       │                  │
 ┌─────▼──────┐    ┌──────▼─────┐
 │ConcreteA   │    │ConcreteB   │
 ├────────────┤    ├────────────┤
 │# step2()   │    │# step2()   │
 │# step3()   │    │# step3()   │
 │# hook()    │    │            │  ← hook 재정의 안 함 (선택)
 └────────────┘    └────────────┘
```

---

## 5. Hook 메서드

Hook은 **기본 구현이 있는 메서드**로, 하위 클래스에서 **선택적으로** 재정의할 수 있다.

```python
class AbstractClass:
    def template_method(self):
        self.required_step()           # 반드시 구현해야 하는 추상 메서드
        if self.should_do_optional():  # Hook: 조건부 실행
            self.optional_step()
        self.hook_after_process()      # Hook: 후처리 (기본은 아무것도 안 함)

    @abstractmethod
    def required_step(self):
        """하위 클래스에서 반드시 구현"""
        pass

    def should_do_optional(self) -> bool:
        """Hook: 기본은 True, 하위 클래스에서 변경 가능"""
        return True

    def optional_step(self):
        """조건부 실행되는 단계"""
        pass

    def hook_after_process(self):
        """Hook: 후처리 (기본은 아무것도 안 함)"""
        pass
```

### Hook vs Abstract Method

| 구분 | 추상 메서드 | Hook 메서드 |
|------|------------|-------------|
| **구현 강제** | 반드시 구현해야 함 | 선택적 (기본 구현 있음) |
| **기본 동작** | 없음 | 빈 구현 또는 기본 로직 |
| **용도** | 알고리즘의 필수 단계 | 알고리즘의 선택적 확장 지점 |

---

## 6. Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class BeverageMaker(ABC):
    """음료 제조기 (Abstract Class)"""

    def make(self) -> None:
        """템플릿 메서드: 전체 흐름 정의"""
        print(f"\n=== {self.name} 제조 시작 ===")
        self.boil_water()
        self.brew()              # 추상: 하위 클래스에서 구현
        self.pour_in_cup()
        if self.wants_condiments():  # Hook: 첨가물 추가 여부
            self.add_condiments()    # 추상: 하위 클래스에서 구현
        self.hook_before_serve()     # Hook: 서빙 전 처리
        print(f"  {self.name} 완성!")

    @property
    @abstractmethod
    def name(self) -> str:
        pass

    @abstractmethod
    def brew(self) -> None:
        """추상: 우리기/추출 (반드시 구현)"""
        pass

    @abstractmethod
    def add_condiments(self) -> None:
        """추상: 첨가물 추가 (반드시 구현)"""
        pass

    def boil_water(self) -> None:
        """공통: 물 끓이기"""
        print("  1. 물을 끓입니다.")

    def pour_in_cup(self) -> None:
        """공통: 컵에 따르기"""
        print("  3. 컵에 따릅니다.")

    def wants_condiments(self) -> bool:
        """Hook: 첨가물 추가 여부 (기본: True)"""
        return True

    def hook_before_serve(self) -> None:
        """Hook: 서빙 전 처리 (기본: 아무것도 안 함)"""
        pass


class CoffeeMaker(BeverageMaker):
    """커피 제조기"""

    @property
    def name(self) -> str:
        return "커피"

    def brew(self) -> None:
        print("  2. 커피를 드립합니다.")

    def add_condiments(self) -> None:
        print("  4. 설탕과 우유를 추가합니다.")


class TeaMaker(BeverageMaker):
    """차 제조기"""

    @property
    def name(self) -> str:
        return "녹차"

    def brew(self) -> None:
        print("  2. 찻잎을 우려냅니다.")

    def add_condiments(self) -> None:
        print("  4. 꿀을 추가합니다.")

    def wants_condiments(self) -> bool:
        """Hook 재정의: 첨가물 안 넣음"""
        return False


class LatteeMaker(BeverageMaker):
    """라떼 제조기"""

    @property
    def name(self) -> str:
        return "카페라떼"

    def brew(self) -> None:
        print("  2. 에스프레소를 추출합니다.")

    def add_condiments(self) -> None:
        print("  4. 스팀 우유를 추가합니다.")

    def hook_before_serve(self) -> None:
        """Hook 재정의: 라떼 아트"""
        print("  5. 라떼 아트를 그립니다.")


# === 사용 예시 ===
if __name__ == "__main__":
    makers = [CoffeeMaker(), TeaMaker(), LatteeMaker()]
    for maker in makers:
        maker.make()
```

**출력:**

```
=== 커피 제조 시작 ===
  1. 물을 끓입니다.
  2. 커피를 드립합니다.
  3. 컵에 따릅니다.
  4. 설탕과 우유를 추가합니다.
  커피 완성!

=== 녹차 제조 시작 ===
  1. 물을 끓입니다.
  2. 찻잎을 우려냅니다.
  3. 컵에 따릅니다.
  녹차 완성!

=== 카페라떼 제조 시작 ===
  1. 물을 끓입니다.
  2. 에스프레소를 추출합니다.
  3. 컵에 따릅니다.
  4. 스팀 우유를 추가합니다.
  5. 라떼 아트를 그립니다.
  카페라떼 완성!
```

---

## 7. C# 구현

```csharp
using System;

// Abstract Class
public abstract class BeverageMaker
{
    public abstract string Name { get; }

    /// <summary>
    /// 템플릿 메서드: 전체 흐름 정의
    /// </summary>
    public void Make()
    {
        Console.WriteLine($"\n=== {Name} 제조 시작 ===");
        BoilWater();
        Brew();
        PourInCup();
        if (WantsCondiments())
            AddCondiments();
        HookBeforeServe();
        Console.WriteLine($"  {Name} 완성!");
    }

    // 추상 메서드 (반드시 구현)
    protected abstract void Brew();
    protected abstract void AddCondiments();

    // 공통 구현
    protected void BoilWater()
    {
        Console.WriteLine("  1. 물을 끓입니다.");
    }

    protected void PourInCup()
    {
        Console.WriteLine("  3. 컵에 따릅니다.");
    }

    // Hook 메서드
    protected virtual bool WantsCondiments() => true;
    protected virtual void HookBeforeServe() { }
}

// Concrete: 커피
public class CoffeeMaker : BeverageMaker
{
    public override string Name => "커피";

    protected override void Brew()
    {
        Console.WriteLine("  2. 커피를 드립합니다.");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("  4. 설탕과 우유를 추가합니다.");
    }
}

// Concrete: 녹차
public class TeaMaker : BeverageMaker
{
    public override string Name => "녹차";

    protected override void Brew()
    {
        Console.WriteLine("  2. 찻잎을 우려냅니다.");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("  4. 꿀을 추가합니다.");
    }

    // Hook 재정의: 첨가물 안 넣음
    protected override bool WantsCondiments() => false;
}

// Concrete: 라떼
public class LatteMaker : BeverageMaker
{
    public override string Name => "카페라떼";

    protected override void Brew()
    {
        Console.WriteLine("  2. 에스프레소를 추출합니다.");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("  4. 스팀 우유를 추가합니다.");
    }

    // Hook 재정의: 라떼 아트
    protected override void HookBeforeServe()
    {
        Console.WriteLine("  5. 라떼 아트를 그립니다.");
    }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        BeverageMaker[] makers = { new CoffeeMaker(), new TeaMaker(), new LatteMaker() };
        foreach (var maker in makers)
        {
            maker.Make();
        }
    }
}
```

---

## 8. Template Method vs Strategy 차이점

| 구분 | Template Method | Strategy |
|------|----------------|----------|
| **메커니즘** | **상속** (Inheritance) | **합성** (Composition) |
| **변경 단위** | 알고리즘의 **일부 단계** | **전체 알고리즘** |
| **변경 시점** | **컴파일 타임** (클래스 정의 시) | **런타임** (객체 교체) |
| **유연성** | 상속 계층에 묶임 | 자유로운 조합 가능 |
| **코드 공유** | 상위 클래스에서 공통 코드 공유 | 공통 코드 공유 어려움 |

### 핵심 차이

```
Template Method:
  "전체 흐름은 같고, 특정 단계만 다르다"
  → 상속으로 단계를 재정의

Strategy:
  "같은 문제를 완전히 다른 알고리즘으로 해결한다"
  → 합성으로 전략 객체를 교체
```

### 선택 기준

- 알고리즘의 **골격이 동일**하고 **세부 단계만 다르면** → Template Method
- 알고리즘 **전체가 교체 가능**해야 하면 → Strategy
- **런타임에 동적으로 변경**해야 하면 → Strategy
- **코드 재사용**(공통 로직)이 중요하면 → Template Method

---

## 9. 실무 예제: 데이터 처리 파이프라인

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any
import json
import csv
import io


class DataPipeline(ABC):
    """데이터 처리 파이프라인 (Template Method)"""

    def process(self, source: str) -> dict:
        """템플릿 메서드: 데이터 처리 흐름"""
        print(f"\n=== {self.name} 파이프라인 시작 ===")

        # 1. 데이터 추출
        raw_data = self.extract(source)
        print(f"  1. 추출 완료: {len(raw_data)} 글자")

        # 2. 데이터 파싱
        parsed = self.parse(raw_data)
        print(f"  2. 파싱 완료: {len(parsed)} 레코드")

        # 3. 유효성 검증
        valid_data = self.validate(parsed)
        print(f"  3. 검증 완료: {len(valid_data)} 유효 레코드")

        # 4. 데이터 변환 (Hook)
        if self.should_transform():
            transformed = self.transform(valid_data)
            print(f"  4. 변환 완료")
        else:
            transformed = valid_data
            print(f"  4. 변환 건너뜀")

        # 5. 결과 생성
        result = self.create_result(transformed)
        self.hook_after_process(result)

        print(f"  완료!")
        return result

    @property
    @abstractmethod
    def name(self) -> str:
        pass

    @abstractmethod
    def extract(self, source: str) -> str:
        """추상: 데이터 추출"""
        pass

    @abstractmethod
    def parse(self, raw_data: str) -> list[dict]:
        """추상: 데이터 파싱"""
        pass

    def validate(self, data: list[dict]) -> list[dict]:
        """공통: 유효성 검증 (기본 구현)"""
        return [item for item in data if item]  # None/빈 값 제거

    def should_transform(self) -> bool:
        """Hook: 변환 수행 여부"""
        return True

    def transform(self, data: list[dict]) -> list[dict]:
        """Hook: 데이터 변환 (기본은 그대로 반환)"""
        return data

    def create_result(self, data: list[dict]) -> dict:
        """공통: 결과 생성"""
        return {
            "pipeline": self.name,
            "count": len(data),
            "data": data,
        }

    def hook_after_process(self, result: dict) -> None:
        """Hook: 후처리"""
        pass


class JSONPipeline(DataPipeline):
    """JSON 데이터 파이프라인"""

    @property
    def name(self) -> str:
        return "JSON"

    def extract(self, source: str) -> str:
        # 실제로는 파일이나 API에서 읽어옴
        return source

    def parse(self, raw_data: str) -> list[dict]:
        data = json.loads(raw_data)
        if isinstance(data, list):
            return data
        return [data]

    def validate(self, data: list[dict]) -> list[dict]:
        # 기본 검증 + 필수 필드 검증
        valid = super().validate(data)
        return [item for item in valid if "name" in item]

    def transform(self, data: list[dict]) -> list[dict]:
        # 이름을 대문자로 변환
        for item in data:
            if "name" in item:
                item["name"] = item["name"].upper()
        return data


class CSVPipeline(DataPipeline):
    """CSV 데이터 파이프라인"""

    @property
    def name(self) -> str:
        return "CSV"

    def extract(self, source: str) -> str:
        return source

    def parse(self, raw_data: str) -> list[dict]:
        reader = csv.DictReader(io.StringIO(raw_data))
        return [row for row in reader]

    def should_transform(self) -> bool:
        """CSV는 변환하지 않음"""
        return False

    def hook_after_process(self, result: dict) -> None:
        """후처리: 처리 건수 로깅"""
        print(f"  [LOG] CSV 처리 완료: {result['count']}건")


class APIResponsePipeline(DataPipeline):
    """API 응답 파이프라인"""

    @property
    def name(self) -> str:
        return "API Response"

    def extract(self, source: str) -> str:
        return source

    def parse(self, raw_data: str) -> list[dict]:
        response = json.loads(raw_data)
        # API 응답에서 data 필드 추출
        return response.get("data", [])

    def transform(self, data: list[dict]) -> list[dict]:
        # 타임스탬프 추가
        from datetime import datetime
        timestamp = datetime.now().isoformat()
        for item in data:
            item["processed_at"] = timestamp
        return data

    def hook_after_process(self, result: dict) -> None:
        print(f"  [LOG] API 데이터 처리: {result['count']}건 동기화")


# === 사용 예시 ===
if __name__ == "__main__":
    # JSON 파이프라인
    json_data = '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}, {}]'
    json_pipeline = JSONPipeline()
    result = json_pipeline.process(json_data)
    print(f"  결과: {result['data']}")

    # CSV 파이프라인
    csv_data = "name,age\nAlice,30\nBob,25\nCharlie,35"
    csv_pipeline = CSVPipeline()
    result = csv_pipeline.process(csv_data)
    print(f"  결과: {result['data']}")

    # API 응답 파이프라인
    api_data = '{"status": "ok", "data": [{"id": 1, "name": "Item1"}, {"id": 2, "name": "Item2"}]}'
    api_pipeline = APIResponsePipeline()
    result = api_pipeline.process(api_data)
    print(f"  결과: {result['data']}")
```

### C# 구현

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.Json;

public abstract class DataPipeline
{
    public abstract string Name { get; }

    /// <summary>
    /// 템플릿 메서드
    /// </summary>
    public Dictionary<string, object> Process(string source)
    {
        Console.WriteLine($"\n=== {Name} 파이프라인 시작 ===");

        var rawData = Extract(source);
        Console.WriteLine($"  1. 추출 완료: {rawData.Length} 글자");

        var parsed = Parse(rawData);
        Console.WriteLine($"  2. 파싱 완료: {parsed.Count} 레코드");

        var valid = Validate(parsed);
        Console.WriteLine($"  3. 검증 완료: {valid.Count} 유효 레코드");

        List<Dictionary<string, object>> transformed;
        if (ShouldTransform())
        {
            transformed = Transform(valid);
            Console.WriteLine("  4. 변환 완료");
        }
        else
        {
            transformed = valid;
            Console.WriteLine("  4. 변환 건너뜀");
        }

        var result = CreateResult(transformed);
        HookAfterProcess(result);
        Console.WriteLine("  완료!");
        return result;
    }

    // 추상 메서드
    protected abstract string Extract(string source);
    protected abstract List<Dictionary<string, object>> Parse(string rawData);

    // 공통 구현
    protected virtual List<Dictionary<string, object>> Validate(
        List<Dictionary<string, object>> data)
    {
        return data.Where(d => d != null && d.Count > 0).ToList();
    }

    protected virtual Dictionary<string, object> CreateResult(
        List<Dictionary<string, object>> data)
    {
        return new Dictionary<string, object>
        {
            ["pipeline"] = Name,
            ["count"] = data.Count,
            ["data"] = data
        };
    }

    // Hook 메서드
    protected virtual bool ShouldTransform() => true;

    protected virtual List<Dictionary<string, object>> Transform(
        List<Dictionary<string, object>> data) => data;

    protected virtual void HookAfterProcess(Dictionary<string, object> result) { }
}

public class JsonPipeline : DataPipeline
{
    public override string Name => "JSON";

    protected override string Extract(string source) => source;

    protected override List<Dictionary<string, object>> Parse(string rawData)
    {
        var items = JsonSerializer.Deserialize<List<Dictionary<string, object>>>(rawData);
        return items ?? new();
    }

    protected override List<Dictionary<string, object>> Transform(
        List<Dictionary<string, object>> data)
    {
        foreach (var item in data)
        {
            if (item.ContainsKey("name"))
                item["name"] = item["name"]?.ToString()?.ToUpper() ?? "";
        }
        return data;
    }
}
```

---

## 10. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **코드 중복 제거** | 공통 로직을 상위 클래스에 집중 |
| **일관된 흐름** | 알고리즘의 구조가 상위 클래스에서 보장됨 |
| **확장 용이** | Hook 메서드로 선택적 확장 지점 제공 |
| **제어 역전** | 상위 클래스가 흐름을 제어하고 하위 클래스는 세부만 구현 |

### 단점

| 단점 | 설명 |
|------|------|
| **상속 기반** | 합성보다 유연성이 떨어진다 (단일 상속 언어에서 제약) |
| **Liskov 치환 원칙 위험** | 하위 클래스가 상위 클래스의 의도를 깨뜨릴 수 있다 |
| **복잡성** | 단계가 많아지면 어떤 메서드가 어디서 오버라이드되는지 파악 어려움 |
| **경직성** | 알고리즘 구조 자체를 바꾸기 어렵다 |

---

## 11. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Strategy** | 알고리즘 전체를 교체(합성) vs 일부 단계만 재정의(상속) |
| **Factory Method** | Template Method의 특수한 경우 (객체 생성 단계를 하위 클래스에 위임) |
| **Hook 메서드** | Template Method 내에서 선택적 확장 지점 제공 |

---

## 12. 정리 및 체크리스트

### 핵심 정리

1. Template Method는 **알고리즘의 골격을 상위 클래스에서 정의**한다.
2. **변하는 부분만** 하위 클래스에서 재정의한다.
3. **Hook 메서드**는 선택적 확장 지점을 제공한다.
4. Strategy와 달리 **상속 기반**이며, 알고리즘의 **일부 단계만** 변경한다.

### 적용 체크리스트

- [ ] 여러 클래스에서 **알고리즘의 전체 흐름이 동일**한가?
- [ ] **공통 로직이 중복**되어 있는가?
- [ ] 알고리즘의 **특정 단계만** 클래스마다 다른가?
- [ ] **선택적 확장 지점**(Hook)이 필요한가?
- [ ] 알고리즘의 **구조가 고정적**이고 변경될 가능성이 낮은가?

> **2~3년차 개발자를 위한 팁**: Django의 Class-Based View, JUnit의 TestCase, React의 Component Lifecycle(과거 클래스 컴포넌트), Spring의 AbstractController가 Template Method 패턴의 실무 사례다. "상속보다 합성을 선호하라"는 원칙에 따라 Strategy로 대체할 수 있는지 먼저 검토하되, 공통 흐름이 복잡하고 코드 재사용이 중요한 경우에는 Template Method가 여전히 유효하다.
