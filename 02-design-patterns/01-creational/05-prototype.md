# Prototype 패턴 (프로토타입)

> **핵심 한줄 요약**: 기존 객체를 복제(clone)하여 새 객체를 생성함으로써, 객체 생성 비용을 줄이고 유연성을 확보한다.

---

## 목차

1. [의도 (Intent)](#1-의도-intent)
2. [문제 상황 (Problem)](#2-문제-상황-problem)
3. [얕은 복사 vs 깊은 복사](#3-얕은-복사-vs-깊은-복사)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제](#7-실무-예제)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 의도 (Intent)

Prototype 패턴은 **기존에 존재하는 객체를 복제**하여 새 객체를 만든다. 이를 통해:

- 복잡한 객체를 **처음부터 다시 만들지 않아도** 된다
- 객체의 **구체 클래스에 의존하지 않고** 복제할 수 있다
- 런타임에 동적으로 새 객체를 생성할 수 있다

### 실생활 비유

```
세포 분열을 생각해 보자.

새로운 세포를 "무(無)"에서 만드는 것이 아니라,
기존 세포가 자신을 복제하여 새로운 세포를 만든다.

복제된 세포는 원본과 동일한 구조를 가지지만,
이후 독립적으로 변화할 수 있다.

→ Prototype 패턴은 "세포 분열"과 같은 복제 메커니즘이다.
```

---

## 2. 문제 상황 (Problem)

### 왜 필요한가?

| 상황 | 설명 |
|------|------|
| **비용이 큰 객체 생성** | DB 조회, 네트워크 호출 등을 거쳐 만든 객체를 복제 |
| **복잡한 초기화** | 수십 개의 속성을 설정해야 하는 객체 복제 |
| **게임 캐릭터/아이템** | 기본 캐릭터를 복제 후 약간만 수정 |
| **문서 템플릿** | 기본 문서를 복제 후 내용만 변경 |
| **설정 프리셋** | 기본 설정을 복제 후 일부만 변경 |

### 복제 없이 구현하면?

```python
# 문제: 객체를 "외부에서" 복제하려면 모든 필드를 알아야 한다
class Document:
    def __init__(self, title, content, author, created_at,
                 font, font_size, margin, header, footer,
                 page_orientation, watermark, ...):
        # 수십 개의 속성...
        pass

# 복제를 위해 모든 필드를 일일이 복사해야 한다
def copy_document(original):
    return Document(
        title=original.title,
        content=original.content,
        author=original.author,
        # ... 수십 개의 필드를 빠짐없이 나열해야 한다
        # private 필드에는 접근할 수 없을 수도 있다!
    )
```

### Prototype으로 해결

```python
# 해결: 객체가 스스로를 복제하는 메서드를 제공
class Document:
    def clone(self):
        """자기 자신을 복제 - 모든 필드를 내부적으로 처리"""
        return copy.deepcopy(self)

# 사용: 한 줄로 완전한 복제
new_doc = original_doc.clone()
new_doc.title = "수정된 제목"  # 필요한 부분만 변경
```

---

## 3. 얕은 복사 vs 깊은 복사

### 개념 설명

```
[원본 객체]
  ├── name: "김철수"      (값 타입 / 문자열)
  ├── age: 30             (값 타입 / 정수)
  └── address ──────────→ [Address 객체]
       (참조 타입)            ├── city: "서울"
                             └── zip: "06000"

[얕은 복사 (Shallow Copy)]
  ├── name: "김철수"      ← 값이 복사됨
  ├── age: 30             ← 값이 복사됨
  └── address ──────────→ [같은 Address 객체]  ← 참조만 복사!
                             ├── city: "서울"
                             └── zip: "06000"
  → 원본과 복사본이 같은 Address를 공유한다!
  → 한쪽에서 수정하면 다른 쪽에도 영향이 간다!

[깊은 복사 (Deep Copy)]
  ├── name: "김철수"      ← 값이 복사됨
  ├── age: 30             ← 값이 복사됨
  └── address ──────────→ [새로운 Address 객체]  ← 객체까지 복사!
                             ├── city: "서울"
                             └── zip: "06000"
  → 원본과 복사본이 완전히 독립적이다.
  → 한쪽을 수정해도 다른 쪽에 영향이 없다.
```

### Python에서의 차이

```python
import copy

class Address:
    def __init__(self, city, zipcode):
        self.city = city
        self.zipcode = zipcode

    def __repr__(self):
        return f"Address({self.city}, {self.zipcode})"


class Person:
    def __init__(self, name, age, address):
        self.name = name
        self.age = age
        self.address = address

    def __repr__(self):
        return f"Person({self.name}, {self.age}, {self.address})"


# 원본
original = Person("김철수", 30, Address("서울", "06000"))

# ─── 얕은 복사 ───
shallow = copy.copy(original)
print(f"원본 == 얕은복사: {original is shallow}")           # False (다른 객체)
print(f"주소 공유: {original.address is shallow.address}")   # True  (같은 Address!)

# 얕은 복사본의 주소를 변경하면 원본도 변경된다!
shallow.address.city = "부산"
print(f"원본 주소: {original.address.city}")  # "부산" ← 의도치 않은 변경!

# ─── 깊은 복사 ───
original2 = Person("이영희", 25, Address("대전", "34000"))
deep = copy.deepcopy(original2)
print(f"원본 == 깊은복사: {original2 is deep}")              # False
print(f"주소 공유: {original2.address is deep.address}")      # False (다른 Address!)

# 깊은 복사본의 주소를 변경해도 원본은 그대로다
deep.address.city = "광주"
print(f"원본 주소: {original2.address.city}")  # "대전" ← 영향 없음!
```

### C#에서의 차이

```csharp
public class Address
{
    public string City { get; set; }
    public string ZipCode { get; set; }

    // 깊은 복사를 위한 Clone 메서드
    public Address DeepClone()
    {
        return new Address { City = this.City, ZipCode = this.ZipCode };
    }
}

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    // 얕은 복사: MemberwiseClone()
    public Person ShallowClone()
    {
        return (Person)this.MemberwiseClone();
    }

    // 깊은 복사: 참조 타입도 복제
    public Person DeepClone()
    {
        var clone = (Person)this.MemberwiseClone();
        clone.Address = this.Address.DeepClone();  // Address도 별도 복제
        return clone;
    }
}

// 테스트
var original = new Person
{
    Name = "김철수",
    Age = 30,
    Address = new Address { City = "서울", ZipCode = "06000" }
};

// 얕은 복사
var shallow = original.ShallowClone();
shallow.Address.City = "부산";
Console.WriteLine(original.Address.City);  // "부산" ← 원본도 변경됨!

// 깊은 복사
var deep = original.DeepClone();
deep.Address.City = "대전";
Console.WriteLine(original.Address.City);  // "부산" ← 원본 영향 없음
```

### 판단 기준

```
얕은 복사를 사용해도 되는 경우:
  - 객체가 값 타입(int, string 등)만 포함
  - 참조 타입 필드가 불변(immutable)인 경우
  - 참조 타입 필드를 공유해도 문제없는 경우

깊은 복사를 사용해야 하는 경우:
  - 참조 타입 필드가 있고, 변경 가능(mutable)한 경우
  - 복사본 수정이 원본에 영향을 주면 안 되는 경우
  - 완전히 독립적인 복사본이 필요한 경우
```

---

## 4. UML 다이어그램

```
┌──────────────────────────┐
│   Prototype (interface)   │
├──────────────────────────┤
│ + clone(): Prototype      │
└─────────┬────────────────┘
          │ 구현
    ┌─────┴──────────────┐
    │                    │
┌───┴────────────┐  ┌───┴────────────┐
│ConcreteProtoA  │  │ConcreteProtoB  │
├────────────────┤  ├────────────────┤
│ - fieldA       │  │ - fieldX       │
│ - fieldB       │  │ - fieldY       │
├────────────────┤  ├────────────────┤
│ + clone(): A   │  │ + clone(): B   │
└────────────────┘  └────────────────┘

[흐름]
1. 클라이언트가 prototype.clone() 호출
2. 객체가 자기 자신을 복제하여 새 인스턴스 반환
3. 클라이언트는 필요한 필드만 수정

[Prototype Registry (선택적)]
┌─────────────────────────┐
│   PrototypeRegistry     │
├─────────────────────────┤
│ - prototypes: dict      │
├─────────────────────────┤
│ + register(key, proto)  │
│ + get(key): Prototype   │  → 등록된 프로토타입을 복제하여 반환
└─────────────────────────┘
```

---

## 5. Python 구현

### 기본 구현: `__copy__`와 `__deepcopy__` 커스터마이징

```python
import copy
from datetime import datetime
from typing import Optional


class Prototype:
    """프로토타입 베이스 클래스"""

    def clone(self, deep: bool = True):
        """객체 복제

        Args:
            deep: True이면 깊은 복사, False이면 얕은 복사
        """
        if deep:
            return copy.deepcopy(self)
        return copy.copy(self)


class GameCharacter(Prototype):
    """게임 캐릭터 - Prototype 패턴 적용"""

    def __init__(self, name: str, character_class: str,
                 level: int = 1, hp: int = 100, mp: int = 50):
        self.name = name
        self.character_class = character_class
        self.level = level
        self.hp = hp
        self.mp = mp
        self.skills: list[str] = []
        self.inventory: list[dict] = []
        self.stats: dict = {
            "strength": 10,
            "dexterity": 10,
            "intelligence": 10,
            "vitality": 10,
        }
        self.created_at = datetime.now()

    def add_skill(self, skill: str):
        self.skills.append(skill)

    def add_item(self, item_name: str, quantity: int = 1):
        self.inventory.append({"name": item_name, "quantity": quantity})

    def set_stat(self, stat: str, value: int):
        if stat in self.stats:
            self.stats[stat] = value

    def __copy__(self):
        """얕은 복사 커스터마이징"""
        cls = self.__class__
        new_obj = cls.__new__(cls)
        new_obj.__dict__.update(self.__dict__)
        # 주의: skills, inventory, stats는 같은 객체를 참조!
        return new_obj

    def __deepcopy__(self, memo):
        """깊은 복사 커스터마이징

        memo는 순환 참조를 방지하기 위한 딕셔너리이다.
        """
        cls = self.__class__
        new_obj = cls.__new__(cls)
        memo[id(self)] = new_obj

        for key, value in self.__dict__.items():
            if key == 'created_at':
                # created_at은 복제 시점으로 갱신
                setattr(new_obj, key, datetime.now())
            else:
                setattr(new_obj, key, copy.deepcopy(value, memo))

        return new_obj

    def display(self):
        print(f"  이름: {self.name}")
        print(f"  클래스: {self.character_class}")
        print(f"  레벨: {self.level}")
        print(f"  HP/MP: {self.hp}/{self.mp}")
        print(f"  스탯: {self.stats}")
        print(f"  스킬: {self.skills}")
        print(f"  인벤토리: {self.inventory}")


# ─── 사용 예시 ───
if __name__ == "__main__":
    # 전사 프로토타입 생성 (복잡한 초기화)
    warrior_prototype = GameCharacter("전사 템플릿", "Warrior", level=10, hp=500, mp=100)
    warrior_prototype.set_stat("strength", 25)
    warrior_prototype.set_stat("vitality", 20)
    warrior_prototype.add_skill("파워 스트라이크")
    warrior_prototype.add_skill("배쉬")
    warrior_prototype.add_skill("워 크라이")
    warrior_prototype.add_item("강철 검", 1)
    warrior_prototype.add_item("체력 물약", 10)

    print("=== 전사 프로토타입 ===")
    warrior_prototype.display()

    # 프로토타입을 복제하여 새 캐릭터 생성
    print("\n=== 복제된 전사 1 ===")
    warrior1 = warrior_prototype.clone()
    warrior1.name = "용감한 전사"
    warrior1.add_skill("분노")  # 추가 스킬
    warrior1.display()

    print("\n=== 복제된 전사 2 ===")
    warrior2 = warrior_prototype.clone()
    warrior2.name = "수호자"
    warrior2.set_stat("vitality", 30)  # 스탯 변경
    warrior2.add_item("미스릴 방패", 1)  # 아이템 추가
    warrior2.display()

    # 원본은 영향받지 않음 (깊은 복사)
    print("\n=== 원본 전사 (변경 없음 확인) ===")
    warrior_prototype.display()

    # 독립성 확인
    print(f"\n독립성 확인:")
    print(f"  원본 스킬: {warrior_prototype.skills}")
    print(f"  전사1 스킬: {warrior1.skills}")
    print(f"  전사2 인벤토리: {warrior2.inventory}")
```

### Prototype Registry (프로토타입 레지스트리)

```python
class PrototypeRegistry:
    """프로토타입 레지스트리

    자주 사용되는 프로토타입을 등록해 놓고,
    필요할 때 복제하여 사용한다.
    """

    def __init__(self):
        self._prototypes: dict[str, Prototype] = {}

    def register(self, key: str, prototype: Prototype):
        """프로토타입 등록"""
        self._prototypes[key] = prototype

    def unregister(self, key: str):
        """프로토타입 등록 해제"""
        self._prototypes.pop(key, None)

    def get(self, key: str) -> Prototype:
        """등록된 프로토타입을 복제하여 반환"""
        prototype = self._prototypes.get(key)
        if prototype is None:
            raise KeyError(f"프로토타입 '{key}'를 찾을 수 없습니다.")
        return prototype.clone()

    def list_keys(self) -> list[str]:
        """등록된 프로토타입 키 목록"""
        return list(self._prototypes.keys())


# ─── 사용: 게임 캐릭터 템플릿 시스템 ───
registry = PrototypeRegistry()

# 각 직업별 프로토타입 등록
warrior = GameCharacter("전사", "Warrior", level=1, hp=200, mp=50)
warrior.set_stat("strength", 20)
warrior.set_stat("vitality", 15)
warrior.add_skill("기본 공격")
warrior.add_skill("방어 자세")
registry.register("warrior", warrior)

mage = GameCharacter("마법사", "Mage", level=1, hp=100, mp=200)
mage.set_stat("intelligence", 25)
mage.add_skill("파이어볼")
mage.add_skill("아이스 볼트")
registry.register("mage", mage)

archer = GameCharacter("궁수", "Archer", level=1, hp=150, mp=100)
archer.set_stat("dexterity", 22)
archer.add_skill("더블 샷")
archer.add_skill("회피")
registry.register("archer", archer)

# 레지스트리에서 캐릭터 생성
print("등록된 프로토타입:", registry.list_keys())

my_warrior = registry.get("warrior")
my_warrior.name = "플레이어1_전사"
print(f"\n생성된 캐릭터: {my_warrior.name} ({my_warrior.character_class})")
print(f"스킬: {my_warrior.skills}")

my_mage = registry.get("mage")
my_mage.name = "플레이어2_마법사"
my_mage.add_skill("텔레포트")  # 추가 커스터마이징
print(f"\n생성된 캐릭터: {my_mage.name} ({my_mage.character_class})")
print(f"스킬: {my_mage.skills}")
```

---

## 6. C# 구현

### ICloneable 인터페이스 활용

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

/// <summary>
/// 게임 캐릭터 - Prototype 패턴
/// ICloneable은 .NET 기본 인터페이스이지만,
/// 실무에서는 제네릭 버전을 직접 정의하는 것이 더 좋다.
/// </summary>
public interface IPrototype<T>
{
    T ShallowClone();
    T DeepClone();
}

public class CharacterStats
{
    public int Strength { get; set; }
    public int Dexterity { get; set; }
    public int Intelligence { get; set; }
    public int Vitality { get; set; }

    public CharacterStats DeepClone()
    {
        return new CharacterStats
        {
            Strength = this.Strength,
            Dexterity = this.Dexterity,
            Intelligence = this.Intelligence,
            Vitality = this.Vitality,
        };
    }

    public override string ToString()
        => $"STR:{Strength} DEX:{Dexterity} INT:{Intelligence} VIT:{Vitality}";
}

public class InventoryItem
{
    public string Name { get; set; }
    public int Quantity { get; set; }

    public InventoryItem DeepClone()
        => new InventoryItem { Name = this.Name, Quantity = this.Quantity };

    public override string ToString() => $"{Name}(x{Quantity})";
}

public class GameCharacter : IPrototype<GameCharacter>
{
    public string Name { get; set; }
    public string CharacterClass { get; set; }
    public int Level { get; set; }
    public int Hp { get; set; }
    public int Mp { get; set; }
    public CharacterStats Stats { get; set; }
    public List<string> Skills { get; set; }
    public List<InventoryItem> Inventory { get; set; }
    public DateTime CreatedAt { get; set; }

    public GameCharacter(string name, string characterClass,
                         int level = 1, int hp = 100, int mp = 50)
    {
        Name = name;
        CharacterClass = characterClass;
        Level = level;
        Hp = hp;
        Mp = mp;
        Stats = new CharacterStats
        {
            Strength = 10, Dexterity = 10,
            Intelligence = 10, Vitality = 10
        };
        Skills = new List<string>();
        Inventory = new List<InventoryItem>();
        CreatedAt = DateTime.Now;
    }

    /// <summary>
    /// 얕은 복사: MemberwiseClone 사용
    /// 값 타입은 복사, 참조 타입은 참조 공유
    /// </summary>
    public GameCharacter ShallowClone()
    {
        return (GameCharacter)this.MemberwiseClone();
    }

    /// <summary>
    /// 깊은 복사: 모든 참조 타입도 새로 복제
    /// </summary>
    public GameCharacter DeepClone()
    {
        var clone = (GameCharacter)this.MemberwiseClone();

        // 참조 타입 필드를 별도로 복제
        clone.Stats = this.Stats.DeepClone();
        clone.Skills = new List<string>(this.Skills);
        clone.Inventory = this.Inventory
            .Select(item => item.DeepClone())
            .ToList();
        clone.CreatedAt = DateTime.Now;  // 복제 시점으로 갱신

        return clone;
    }

    public void AddSkill(string skill) => Skills.Add(skill);

    public void AddItem(string name, int quantity = 1)
        => Inventory.Add(new InventoryItem { Name = name, Quantity = quantity });

    public void Display()
    {
        Console.WriteLine($"  이름: {Name}");
        Console.WriteLine($"  클래스: {CharacterClass}");
        Console.WriteLine($"  레벨: {Level}, HP: {Hp}, MP: {Mp}");
        Console.WriteLine($"  스탯: {Stats}");
        Console.WriteLine($"  스킬: [{string.Join(", ", Skills)}]");
        Console.WriteLine($"  인벤토리: [{string.Join(", ", Inventory)}]");
    }
}

// ─── 사용 예시 ───
public class Program
{
    static void Main()
    {
        // 전사 프로토타입 생성
        var warriorProto = new GameCharacter("전사 템플릿", "Warrior", 10, 500, 100);
        warriorProto.Stats.Strength = 25;
        warriorProto.Stats.Vitality = 20;
        warriorProto.AddSkill("파워 스트라이크");
        warriorProto.AddSkill("배쉬");
        warriorProto.AddItem("강철 검");
        warriorProto.AddItem("체력 물약", 10);

        Console.WriteLine("=== 전사 프로토타입 ===");
        warriorProto.Display();

        // 깊은 복사로 새 캐릭터 생성
        Console.WriteLine("\n=== 복제된 전사 ===");
        var warrior1 = warriorProto.DeepClone();
        warrior1.Name = "용감한 전사";
        warrior1.AddSkill("분노");
        warrior1.Display();

        // 원본 확인 (변경 없음)
        Console.WriteLine("\n=== 원본 (변경 없음 확인) ===");
        warriorProto.Display();
    }
}
```

### Prototype Registry (C#)

```csharp
using System;
using System.Collections.Generic;

/// <summary>
/// 프로토타입 레지스트리
/// 자주 사용되는 프로토타입을 등록하고 복제하여 제공
/// </summary>
public class PrototypeRegistry<T> where T : IPrototype<T>
{
    private readonly Dictionary<string, T> _prototypes = new();

    public void Register(string key, T prototype)
    {
        _prototypes[key] = prototype;
    }

    public void Unregister(string key)
    {
        _prototypes.Remove(key);
    }

    public T Get(string key)
    {
        if (!_prototypes.TryGetValue(key, out var prototype))
            throw new KeyNotFoundException($"프로토타입 '{key}'을 찾을 수 없습니다.");

        return prototype.DeepClone();
    }

    public bool Contains(string key) => _prototypes.ContainsKey(key);

    public IEnumerable<string> Keys => _prototypes.Keys;
}

// ─── 사용 예시 ───
var registry = new PrototypeRegistry<GameCharacter>();

// 프로토타입 등록
var warrior = new GameCharacter("전사", "Warrior", 1, 200, 50);
warrior.Stats.Strength = 20;
warrior.AddSkill("기본 공격");
registry.Register("warrior", warrior);

var mage = new GameCharacter("마법사", "Mage", 1, 100, 200);
mage.Stats.Intelligence = 25;
mage.AddSkill("파이어볼");
registry.Register("mage", mage);

// 레지스트리에서 캐릭터 생성
var myWarrior = registry.Get("warrior");
myWarrior.Name = "나의 전사";
myWarrior.AddSkill("추가 스킬");

Console.WriteLine("등록된 키: " + string.Join(", ", registry.Keys));
myWarrior.Display();
```

### JSON 직렬화를 활용한 깊은 복사 (실무 팁)

```csharp
using System.Text.Json;

/// <summary>
/// JSON 직렬화를 이용한 범용 깊은 복사
/// 장점: 참조 타입을 일일이 복제할 필요 없음
/// 단점: 성능 오버헤드, 직렬화 불가능한 필드 처리 불가
/// </summary>
public static class DeepCloneExtension
{
    public static T DeepCloneViaJson<T>(this T source)
    {
        var json = JsonSerializer.Serialize(source);
        return JsonSerializer.Deserialize<T>(json)!;
    }
}

// 사용
var original = new GameCharacter("원본", "Warrior", 10, 500, 100);
// 주의: DateTime, 순환 참조 등은 별도 처리 필요
```

---

## 7. 실무 예제

### 예제 1: 문서 템플릿 시스템 (Python)

```python
import copy
from datetime import datetime
from typing import Optional


class DocumentStyle:
    """문서 스타일 설정"""
    def __init__(self):
        self.font_family = "맑은 고딕"
        self.font_size = 12
        self.line_spacing = 1.5
        self.margin_top = 20
        self.margin_bottom = 20
        self.margin_left = 25
        self.margin_right = 25
        self.header_font_size = 18
        self.page_orientation = "portrait"

    def __repr__(self):
        return (f"Style(font={self.font_family}/{self.font_size}pt, "
                f"spacing={self.line_spacing})")


class DocumentTemplate(Prototype):
    """문서 템플릿 - Prototype으로 복제 가능"""

    def __init__(self, name: str, category: str):
        self.name = name
        self.category = category
        self.style = DocumentStyle()
        self.sections: list[dict] = []
        self.metadata: dict = {}
        self.created_at = datetime.now()

    def add_section(self, title: str, content: str = ""):
        self.sections.append({"title": title, "content": content})

    def set_metadata(self, key: str, value: str):
        self.metadata[key] = value

    def display(self):
        print(f"  문서: {self.name} [{self.category}]")
        print(f"  스타일: {self.style}")
        print(f"  섹션 수: {len(self.sections)}")
        for i, section in enumerate(self.sections, 1):
            content_preview = section['content'][:30] + "..." if section['content'] else "(빈 섹션)"
            print(f"    {i}. {section['title']}: {content_preview}")
        print(f"  메타데이터: {self.metadata}")


# ─── 문서 템플릿 레지스트리 ───
class DocumentTemplateSystem:
    """문서 템플릿 관리 시스템"""

    def __init__(self):
        self.registry = PrototypeRegistry()
        self._register_default_templates()

    def _register_default_templates(self):
        """기본 템플릿 등록"""

        # 비즈니스 보고서 템플릿
        report = DocumentTemplate("비즈니스 보고서", "보고서")
        report.style.font_family = "맑은 고딕"
        report.style.font_size = 11
        report.style.line_spacing = 1.6
        report.add_section("요약", "")
        report.add_section("배경", "")
        report.add_section("분석", "")
        report.add_section("결론 및 제안", "")
        report.add_section("부록", "")
        report.set_metadata("template_version", "1.0")
        report.set_metadata("department", "")
        self.registry.register("business_report", report)

        # 회의록 템플릿
        minutes = DocumentTemplate("회의록", "회의")
        minutes.style.font_size = 10
        minutes.add_section("회의 정보", "일시:\n장소:\n참석자:")
        minutes.add_section("안건", "")
        minutes.add_section("논의 내용", "")
        minutes.add_section("결정 사항", "")
        minutes.add_section("다음 단계", "")
        minutes.set_metadata("template_version", "1.0")
        self.registry.register("meeting_minutes", minutes)

        # 기술 문서 템플릿
        tech_doc = DocumentTemplate("기술 문서", "기술")
        tech_doc.style.font_family = "Consolas"
        tech_doc.style.font_size = 11
        tech_doc.add_section("개요", "")
        tech_doc.add_section("아키텍처", "")
        tech_doc.add_section("API 명세", "")
        tech_doc.add_section("설치 및 설정", "")
        tech_doc.add_section("FAQ", "")
        tech_doc.set_metadata("template_version", "1.0")
        self.registry.register("tech_document", tech_doc)

    def create_from_template(self, template_key: str,
                              document_name: str) -> DocumentTemplate:
        """템플릿을 복제하여 새 문서 생성"""
        doc = self.registry.get(template_key)
        doc.name = document_name
        doc.created_at = datetime.now()
        return doc


# ─── 사용 ───
if __name__ == "__main__":
    system = DocumentTemplateSystem()
    print("사용 가능한 템플릿:", system.registry.list_keys())

    # 보고서 생성
    print("\n=== 분기 보고서 생성 ===")
    q4_report = system.create_from_template("business_report", "2024년 4분기 실적 보고서")
    q4_report.set_metadata("department", "영업팀")
    q4_report.sections[0]["content"] = "4분기 매출 목표 120% 달성..."
    q4_report.display()

    # 같은 템플릿으로 또 다른 보고서 생성
    print("\n=== HR 보고서 생성 ===")
    hr_report = system.create_from_template("business_report", "인사 현황 보고서")
    hr_report.set_metadata("department", "인사팀")
    hr_report.sections[0]["content"] = "현재 인원 150명, 신규 채용 20명..."
    hr_report.display()

    # 두 문서가 독립적인지 확인
    print(f"\n독립성 확인: '{q4_report.metadata['department']}' vs '{hr_report.metadata['department']}'")
```

### 예제 2: 설정 프리셋 시스템 (C#)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

/// <summary>
/// 앱 설정 프리셋 - Prototype 패턴으로 복제 가능
/// </summary>
public class AppSettings : IPrototype<AppSettings>
{
    public string Name { get; set; }
    public string Environment { get; set; }
    public DatabaseSettings Database { get; set; }
    public CacheSettings Cache { get; set; }
    public LoggingSettings Logging { get; set; }
    public Dictionary<string, string> CustomSettings { get; set; }

    public AppSettings(string name)
    {
        Name = name;
        Environment = "development";
        Database = new DatabaseSettings();
        Cache = new CacheSettings();
        Logging = new LoggingSettings();
        CustomSettings = new Dictionary<string, string>();
    }

    public AppSettings ShallowClone()
        => (AppSettings)this.MemberwiseClone();

    public AppSettings DeepClone()
    {
        var clone = (AppSettings)this.MemberwiseClone();
        clone.Database = this.Database.DeepClone();
        clone.Cache = this.Cache.DeepClone();
        clone.Logging = this.Logging.DeepClone();
        clone.CustomSettings = new Dictionary<string, string>(this.CustomSettings);
        return clone;
    }

    public void Display()
    {
        Console.WriteLine($"  설정: {Name} ({Environment})");
        Console.WriteLine($"  DB: {Database}");
        Console.WriteLine($"  Cache: {Cache}");
        Console.WriteLine($"  Logging: {Logging}");
        if (CustomSettings.Any())
            Console.WriteLine($"  Custom: {string.Join(", ", CustomSettings.Select(kv => $"{kv.Key}={kv.Value}"))}");
    }
}

public class DatabaseSettings
{
    public string Host { get; set; } = "localhost";
    public int Port { get; set; } = 5432;
    public string Database { get; set; } = "myapp";
    public int MaxConnections { get; set; } = 10;
    public int TimeoutSeconds { get; set; } = 30;

    public DatabaseSettings DeepClone() => (DatabaseSettings)this.MemberwiseClone();

    public override string ToString()
        => $"{Host}:{Port}/{Database} (max:{MaxConnections}, timeout:{TimeoutSeconds}s)";
}

public class CacheSettings
{
    public bool Enabled { get; set; } = true;
    public string Provider { get; set; } = "memory";
    public int TtlSeconds { get; set; } = 300;
    public int MaxSize { get; set; } = 1000;

    public CacheSettings DeepClone() => (CacheSettings)this.MemberwiseClone();

    public override string ToString()
        => $"{Provider} (ttl:{TtlSeconds}s, max:{MaxSize}, enabled:{Enabled})";
}

public class LoggingSettings
{
    public string Level { get; set; } = "Information";
    public bool ConsoleOutput { get; set; } = true;
    public bool FileOutput { get; set; } = false;
    public string FilePath { get; set; } = "logs/app.log";

    public LoggingSettings DeepClone() => (LoggingSettings)this.MemberwiseClone();

    public override string ToString()
        => $"{Level} (console:{ConsoleOutput}, file:{FileOutput})";
}

// ─── 사용: 환경별 설정 프리셋 ───
public class Program
{
    static void Main()
    {
        // 개발 환경 프리셋 (프로토타입)
        var devSettings = new AppSettings("Development");
        devSettings.Environment = "development";
        devSettings.Database.Host = "localhost";
        devSettings.Database.MaxConnections = 5;
        devSettings.Cache.Enabled = false;
        devSettings.Logging.Level = "Debug";
        devSettings.Logging.ConsoleOutput = true;

        Console.WriteLine("=== 개발 환경 (프로토타입) ===");
        devSettings.Display();

        // 스테이징 환경: 개발 프리셋을 복제하고 일부 수정
        Console.WriteLine("\n=== 스테이징 환경 (복제 + 수정) ===");
        var stagingSettings = devSettings.DeepClone();
        stagingSettings.Name = "Staging";
        stagingSettings.Environment = "staging";
        stagingSettings.Database.Host = "staging-db.example.com";
        stagingSettings.Database.MaxConnections = 20;
        stagingSettings.Cache.Enabled = true;
        stagingSettings.Cache.Provider = "redis";
        stagingSettings.Logging.Level = "Warning";
        stagingSettings.Display();

        // 프로덕션 환경: 스테이징을 복제하고 추가 수정
        Console.WriteLine("\n=== 프로덕션 환경 (복제 + 수정) ===");
        var prodSettings = stagingSettings.DeepClone();
        prodSettings.Name = "Production";
        prodSettings.Environment = "production";
        prodSettings.Database.Host = "prod-db.example.com";
        prodSettings.Database.MaxConnections = 100;
        prodSettings.Database.TimeoutSeconds = 10;
        prodSettings.Cache.TtlSeconds = 600;
        prodSettings.Cache.MaxSize = 10000;
        prodSettings.Logging.Level = "Error";
        prodSettings.Logging.FileOutput = true;
        prodSettings.Logging.ConsoleOutput = false;
        prodSettings.CustomSettings["CDN_URL"] = "https://cdn.example.com";
        prodSettings.Display();

        // 원본 확인
        Console.WriteLine("\n=== 개발 환경 (원본 - 변경 없음) ===");
        devSettings.Display();
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **생성 비용 절감** | 복잡한 객체를 처음부터 만들지 않고 복제 |
| **구체 클래스 비의존** | clone() 인터페이스만으로 복제 가능 |
| **런타임 유연성** | 실행 중에 동적으로 객체 생성 가능 |
| **초기화 보존** | 복잡한 초기화 상태를 그대로 복제 |
| **프리셋 시스템** | 미리 설정된 객체를 템플릿으로 활용 |

### 단점

| 단점 | 설명 |
|------|------|
| **순환 참조** | 객체 간 순환 참조가 있으면 깊은 복사가 복잡 |
| **깊은 복사 비용** | 중첩이 깊은 객체는 깊은 복사 비용이 클 수 있음 |
| **clone 구현 부담** | 모든 클래스에 clone 메서드를 구현해야 함 |
| **얕은/깊은 복사 혼동** | 어떤 수준의 복사가 필요한지 판단이 필요 |

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Factory Method** | Prototype의 대안. 상속 기반 vs 복제 기반 |
| **Abstract Factory** | 프로토타입 집합을 Abstract Factory로 관리 가능 |
| **Builder** | Builder는 처음부터 만들고, Prototype은 복제 후 수정 |
| **Memento** | 객체 상태 저장/복원에 복제를 활용할 수 있음 |
| **Composite** | Composite 구조 전체를 Prototype으로 복제 가능 |
| **Decorator** | Decorator가 적용된 객체를 Prototype으로 복제 가능 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

```
Prototype 패턴 = 기존 객체를 복제하여 새 객체 생성

[핵심 개념]
- 객체가 자기 자신을 복제하는 메서드(clone)를 제공
- 외부에서 객체의 내부 구조를 몰라도 복제 가능
- 프로토타입 레지스트리로 템플릿 관리 가능

[얕은 복사 vs 깊은 복사]
- 얕은 복사: 값 타입만 복사, 참조 타입은 공유
- 깊은 복사: 모든 것을 독립적으로 복사
- 실무에서는 거의 항상 깊은 복사를 사용

[Python]
- copy.copy(): 얕은 복사
- copy.deepcopy(): 깊은 복사
- __copy__, __deepcopy__: 복사 동작 커스터마이징

[C#]
- MemberwiseClone(): 얕은 복사
- 깊은 복사: 수동 구현 또는 JSON 직렬화 활용
- ICloneable 또는 커스텀 IPrototype<T> 인터페이스
```

### 학습 체크리스트

- [ ] Prototype 패턴의 의도를 한 문장으로 설명할 수 있다
- [ ] 얕은 복사와 깊은 복사의 차이를 코드로 설명할 수 있다
- [ ] Python에서 copy.copy()와 copy.deepcopy()를 올바르게 사용할 수 있다
- [ ] Python에서 __copy__와 __deepcopy__를 커스터마이징할 수 있다
- [ ] C#에서 MemberwiseClone()과 수동 깊은 복사를 구현할 수 있다
- [ ] Prototype Registry를 구현하고 활용할 수 있다
- [ ] Prototype 패턴이 유용한 실무 시나리오를 3개 이상 열거할 수 있다
- [ ] 순환 참조가 있는 객체의 깊은 복사 문제를 이해한다

### 생성 패턴 학습 완료

이것으로 5가지 생성 패턴(Singleton, Factory Method, Abstract Factory, Builder, Prototype)을 모두 학습했다. 다음으로는 **구조 패턴(Structural Patterns)** 을 학습하자.

### 생성 패턴 요약 비교

| 패턴 | 핵심 질문 | 사용 시점 |
|------|----------|----------|
| **Singleton** | "인스턴스가 하나만 있어야 하는가?" | 전역 유일 객체가 필요할 때 |
| **Factory Method** | "어떤 타입의 객체를 만들지 서브클래스가 결정해야 하는가?" | 생성할 객체 타입이 다양할 때 |
| **Abstract Factory** | "관련 객체 가족을 일관되게 만들어야 하는가?" | 관련 객체들의 일관성이 중요할 때 |
| **Builder** | "복잡한 객체를 단계별로 조립해야 하는가?" | 매개변수가 많고 선택적일 때 |
| **Prototype** | "기존 객체를 복제해서 약간만 수정하면 되는가?" | 복잡한 객체의 변형이 필요할 때 |
