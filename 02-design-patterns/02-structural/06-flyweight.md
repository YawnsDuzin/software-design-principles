# Flyweight 패턴 (플라이웨이트)

> **핵심 의도 한줄 요약:** 다수의 유사 객체에서 공유 가능한 상태를 분리하여, 메모리 사용을 대폭 줄이는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [내재 상태 vs 외재 상태](#5-내재-상태-vs-외재-상태)
6. [패턴 적용 전 (Before)](#6-패턴-적용-전-before)
7. [패턴 적용 후 (After)](#7-패턴-적용-후-after)
8. [Python 구현](#8-python-구현)
9. [C# 구현](#9-c-구현)
10. [실무 예제: 게임 파티클 시스템](#10-실무-예제-게임-파티클-시스템)
11. [장단점](#11-장단점)
12. [관련 패턴](#12-관련-패턴)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)

---

## 1. 개요

Flyweight 패턴은 **GoF 구조 패턴** 중 하나로, 대량의 객체를 효율적으로 관리하기 위해 **공유(sharing)**를 사용합니다.

핵심 아이디어: "같은 데이터를 가진 객체를 매번 새로 만들지 말고, 하나를 공유하자."

실무에서 마주치는 상황:
- 텍스트 에디터: 수만 개의 글자 각각에 폰트/색상 정보를 저장
- 게임: 수천 개의 총알/나무/건물 렌더링
- 지도: 수만 개의 마커 아이콘 표시
- 캐릭터 스프라이트: 같은 애니메이션을 공유

---

## 2. 왜 필요한가? - 문제 상황

### 상황: 텍스트 에디터

문서에 100,000개의 글자가 있다고 가정합니다:

```
각 글자 객체가 가진 데이터:
- 문자: 'A' (1 byte)
- 폰트: "Arial" (20 bytes)
- 크기: 12 (4 bytes)
- 색상: #000000 (4 bytes)
- 굵기: bold (4 bytes)
- 위치: x, y (8 bytes)

객체 하나 = ~41 bytes
100,000개 = ~4.1 MB

하지만 실제로 폰트/크기/색상의 고유한 조합은 몇 개뿐!
- Arial, 12pt, 검정, 일반 → 본문 90%
- Arial, 16pt, 검정, 굵게 → 제목 5%
- Courier, 10pt, 파랑, 일반 → 코드 5%

3가지 스타일만 공유하면?
- 스타일 데이터: 3 x 32 bytes = 96 bytes
- 위치+참조 데이터: 100,000 x 12 bytes = 1.2 MB
→ 약 1.2 MB (4.1 MB에서 70% 절감!)
```

---

## 3. 비유로 이해하기

### 글자 폰트 공유

```
문서: "AABA"

❌ 공유하지 않으면:
[A, Arial, 12pt, 검정] [A, Arial, 12pt, 검정] [B, Arial, 12pt, 검정] [A, Arial, 12pt, 검정]
→ 같은 스타일 정보가 4번 반복!

✅ Flyweight로 공유하면:
스타일 풀: { "style1": [Arial, 12pt, 검정] }  ← 공유 데이터(1개만 존재)

글자 배열:
[A, style1, 위치(0,0)]  [A, style1, 위치(10,0)]  [B, style1, 위치(20,0)]  [A, style1, 위치(30,0)]
→ 스타일 정보는 1개만 유지, 위치만 각각 저장!
```

### 체스 게임

```
체스판에 32개의 말이 있다면:

❌ 각 말마다 이미지 데이터 저장:
"흰색 폰" 이미지 x 8개 = 이미지 8개
"검정 폰" 이미지 x 8개 = 이미지 8개
→ 총 32개의 이미지

✅ Flyweight: 종류별로 이미지 하나만 공유
"흰색 폰" 이미지 x 1 (8개 말이 공유)
"검정 폰" 이미지 x 1 (8개 말이 공유)
→ 총 6종류의 이미지 (폰, 룩, 나이트, 비숍, 퀸, 킹 x 흰/검)
위치 정보만 개별 저장
```

---

## 4. UML 다이어그램

```
┌──────────────┐        ┌──────────────────────────┐
│    Client    │        │   FlyweightFactory        │
│              │───────>│──────────────────────────│
│              │        │ - flyweights: dict        │
└──────────────┘        │──────────────────────────│
                        │ + get_flyweight(key)      │
                        │   if key not in cache:    │
                        │     create & store        │
                        │   return cache[key]       │
                        └──────────────┬───────────┘
                                       │ creates/returns
                                       ▼
                        ┌──────────────────────────┐
                        │      Flyweight            │
                        │──────────────────────────│
                        │ - intrinsicState          │ ← 공유 (불변)
                        │──────────────────────────│
                        │ + operation(              │
                        │     extrinsicState)       │ ← 외부 전달 (가변)
                        └──────────────────────────┘

intrinsicState (내재 상태): 공유되는 불변 데이터 (폰트, 색상, 텍스처)
extrinsicState (외재 상태): 컨텍스트마다 다른 데이터 (위치, 크기)
```

---

## 5. 내재 상태 vs 외재 상태

이것이 Flyweight 패턴의 **가장 핵심적인 개념**입니다.

### 내재 상태 (Intrinsic State)

- Flyweight 객체 **내부**에 저장
- 여러 컨텍스트에서 **공유**됨
- **불변(immutable)**이어야 함
- 예시: 폰트 이름, 나무 텍스처, 총알 모양

### 외재 상태 (Extrinsic State)

- Flyweight 객체 **외부**에서 전달
- 각 컨텍스트마다 **고유**함
- **가변(mutable)** 가능
- 예시: 글자의 위치, 나무의 좌표, 총알의 속도

### 구분 기준

```
질문: "이 데이터가 모든 인스턴스에서 같은가?"

YES → 내재 상태 (Flyweight 내부에 저장, 공유)
NO  → 외재 상태 (Flyweight 외부에서 전달)

예시: 게임 나무
- 나무 종류별 텍스처(이미지): 내재 상태 ✅ (같은 종류면 동일)
- 나무의 위치(x, y):          외재 상태 ✅ (나무마다 다름)
- 나무의 크기:                 외재 상태 ✅ (나무마다 다를 수 있음)
```

| 구분 | 내재 상태 (Intrinsic) | 외재 상태 (Extrinsic) |
|------|----------------------|----------------------|
| **위치** | Flyweight 내부 | 클라이언트/컨텍스트 |
| **공유** | 여러 객체가 공유 | 각 객체 고유 |
| **변경** | 불변 (immutable) | 변경 가능 |
| **메모리** | 공유하므로 절약 | 개별 저장 필요 |

---

## 6. 패턴 적용 전 (Before)

```python
# ❌ Before: 나무 하나마다 텍스처 데이터를 개별 저장

import sys


class Tree:
    """나무 객체: 모든 데이터를 개별 저장"""

    def __init__(self, x: float, y: float,
                 tree_type: str, texture_data: bytes,
                 color: str, height: float):
        self.x = x
        self.y = y
        self.tree_type = tree_type
        self.texture_data = texture_data  # 수 MB의 텍스처!
        self.color = color
        self.height = height

    def render(self):
        print(f"나무 렌더링: {self.tree_type} at ({self.x}, {self.y})")


# 숲에 10,000 그루의 나무
forest = []
oak_texture = b"x" * 1_000_000    # 1MB 텍스처
pine_texture = b"y" * 1_000_000   # 1MB 텍스처
birch_texture = b"z" * 1_000_000  # 1MB 텍스처

import random
for _ in range(10000):
    tree_type = random.choice(["참나무", "소나무", "자작나무"])
    texture = {"참나무": oak_texture, "소나무": pine_texture,
               "자작나무": birch_texture}[tree_type]
    forest.append(Tree(
        x=random.uniform(0, 1000),
        y=random.uniform(0, 1000),
        tree_type=tree_type,
        texture_data=texture,  # 매번 같은 텍스처를 참조하긴 하지만...
        color="green",
        height=random.uniform(5, 20)
    ))

# 각 Tree 객체가 텍스처, 색상 등의 데이터를 모두 가지고 있음
# 실제 게임에서는 객체마다 텍스처 복사본을 만들 수도 있음
print(f"나무 수: {len(forest)}")
print(f"Tree 객체 크기: ~{sys.getsizeof(forest[0])}bytes (속성 제외)")
```

---

## 7. 패턴 적용 후 (After)

```python
# ✅ After: Flyweight로 공유 가능한 데이터를 분리

class TreeType:
    """Flyweight: 나무 종류별 공유 데이터 (내재 상태)"""

    def __init__(self, name: str, texture_data: bytes, color: str):
        self.name = name
        self.texture_data = texture_data  # 공유!
        self.color = color                # 공유!

    def render(self, x: float, y: float, height: float):
        """외재 상태를 매개변수로 받아 렌더링"""
        # 실제로는 OpenGL 등으로 렌더링
        pass


class TreeFactory:
    """Flyweight Factory: TreeType을 캐싱하여 공유"""
    _types: dict[str, TreeType] = {}

    @classmethod
    def get_tree_type(cls, name: str, texture: bytes,
                      color: str) -> TreeType:
        key = f"{name}_{color}"
        if key not in cls._types:
            cls._types[key] = TreeType(name, texture, color)
            print(f"[Factory] 새 TreeType 생성: {key}")
        return cls._types[key]

    @classmethod
    def get_type_count(cls) -> int:
        return len(cls._types)


class Tree:
    """컨텍스트: 외재 상태만 저장"""

    def __init__(self, x: float, y: float, height: float,
                 tree_type: TreeType):
        self.x = x          # 외재 상태
        self.y = y          # 외재 상태
        self.height = height # 외재 상태
        self.type = tree_type  # Flyweight 참조 (공유)

    def render(self):
        self.type.render(self.x, self.y, self.height)


# 10,000 그루의 나무, 텍스처는 3개만 생성!
# 메모리: 3 x 1MB + 10,000 x ~24bytes = ~3.24MB (vs ~10GB)
```

---

## 8. Python 구현

```python
from __future__ import annotations
from dataclasses import dataclass
import sys
import random


# ===== Flyweight: 공유 데이터 =====
class CharacterStyle:
    """글자 스타일 Flyweight (내재 상태)

    같은 스타일의 글자들이 이 객체를 공유합니다.
    불변이어야 합니다!
    """

    __slots__ = ('font', 'size', 'color', 'bold', 'italic')

    def __init__(self, font: str, size: int, color: str,
                 bold: bool = False, italic: bool = False):
        self.font = font
        self.size = size
        self.color = color
        self.bold = bold
        self.italic = italic

    def render(self, char: str, x: int, y: int) -> str:
        """외재 상태(char, x, y)를 받아서 렌더링"""
        style = []
        if self.bold:
            style.append("Bold")
        if self.italic:
            style.append("Italic")
        style_str = "+".join(style) if style else "Regular"
        return (f"'{char}' at ({x},{y}) "
                f"[{self.font} {self.size}pt {self.color} {style_str}]")

    def __repr__(self) -> str:
        return (f"Style({self.font}, {self.size}pt, {self.color})")

    def __eq__(self, other) -> bool:
        if not isinstance(other, CharacterStyle):
            return False
        return (self.font == other.font and
                self.size == other.size and
                self.color == other.color and
                self.bold == other.bold and
                self.italic == other.italic)

    def __hash__(self) -> int:
        return hash((self.font, self.size, self.color,
                     self.bold, self.italic))


# ===== Flyweight Factory =====
class StyleFactory:
    """스타일 팩토리: Flyweight 객체를 캐싱하고 공유

    같은 스타일을 요청하면 기존 객체를 반환합니다.
    """

    _styles: dict[int, CharacterStyle] = {}
    _stats = {"hits": 0, "misses": 0}

    @classmethod
    def get_style(cls, font: str, size: int, color: str,
                  bold: bool = False,
                  italic: bool = False) -> CharacterStyle:
        """스타일 Flyweight를 가져오거나 생성"""
        # 키로 사용할 해시값 계산
        style = CharacterStyle(font, size, color, bold, italic)
        key = hash(style)

        if key in cls._styles:
            cls._stats["hits"] += 1
            return cls._styles[key]

        cls._stats["misses"] += 1
        cls._styles[key] = style
        return style

    @classmethod
    def get_style_count(cls) -> int:
        return len(cls._styles)

    @classmethod
    def get_stats(cls) -> dict:
        return {
            "unique_styles": len(cls._styles),
            "cache_hits": cls._stats["hits"],
            "cache_misses": cls._stats["misses"],
        }

    @classmethod
    def clear(cls):
        cls._styles.clear()
        cls._stats = {"hits": 0, "misses": 0}


# ===== Context: 개별 글자 =====
@dataclass
class Character:
    """글자 컨텍스트 (외재 상태 + Flyweight 참조)

    각 글자의 고유한 데이터만 저장합니다.
    스타일 정보는 Flyweight를 참조합니다.
    """
    char: str      # 외재 상태
    x: int         # 외재 상태
    y: int         # 외재 상태
    style: CharacterStyle  # Flyweight 참조 (공유)

    def render(self) -> str:
        return self.style.render(self.char, self.x, self.y)


# ===== 텍스트 에디터 =====
class TextEditor:
    """텍스트 에디터: Flyweight 패턴을 활용한 문서 관리"""

    def __init__(self):
        self._characters: list[Character] = []
        self._cursor_x = 0
        self._cursor_y = 0
        self._char_width = 10
        self._line_height = 20
        self._max_width = 800

    def type_text(self, text: str,
                  font: str = "Arial",
                  size: int = 12,
                  color: str = "#000000",
                  bold: bool = False,
                  italic: bool = False) -> None:
        """텍스트 입력"""
        # Flyweight Factory에서 스타일 가져오기 (공유!)
        style = StyleFactory.get_style(font, size, color, bold, italic)

        for char in text:
            if char == "\n" or self._cursor_x >= self._max_width:
                self._cursor_x = 0
                self._cursor_y += self._line_height
                if char == "\n":
                    continue

            # 외재 상태(위치)는 각 글자마다 고유
            self._characters.append(
                Character(char, self._cursor_x, self._cursor_y, style)
            )
            self._cursor_x += self._char_width

    def render(self, max_chars: int = 50) -> None:
        """문서 렌더링"""
        display_count = min(len(self._characters), max_chars)
        for i in range(display_count):
            print(self._characters[i].render())
        if len(self._characters) > max_chars:
            print(f"... 외 {len(self._characters) - max_chars}개 글자")

    def get_memory_info(self) -> dict:
        """메모리 사용 정보"""
        char_size = sys.getsizeof(Character("a", 0, 0,
                                             CharacterStyle("", 0, "")))
        style_size = sys.getsizeof(CharacterStyle("", 0, ""))

        total_chars = len(self._characters)
        unique_styles = StyleFactory.get_style_count()

        without_flyweight = total_chars * (char_size + style_size)
        with_flyweight = (total_chars * char_size +
                         unique_styles * style_size)

        return {
            "total_characters": total_chars,
            "unique_styles": unique_styles,
            "memory_without_flyweight": without_flyweight,
            "memory_with_flyweight": with_flyweight,
            "memory_saved": without_flyweight - with_flyweight,
            "saving_percent": (
                (1 - with_flyweight / without_flyweight) * 100
                if without_flyweight > 0 else 0
            ),
        }


# ===== 사용 예시 =====
if __name__ == "__main__":
    StyleFactory.clear()
    editor = TextEditor()

    # 본문 텍스트 (같은 스타일 공유)
    body_text = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. " * 100
    editor.type_text(body_text, font="Arial", size=12, color="#000000")

    # 제목 (다른 스타일)
    editor.type_text("\n제목: Flyweight 패턴\n",
                     font="Arial", size=24, color="#333333", bold=True)

    # 코드 블록 (또 다른 스타일)
    code = "def hello(): print('world')\n" * 50
    editor.type_text(code, font="Courier", size=10, color="#0000FF")

    # 강조 텍스트
    editor.type_text("중요한 내용입니다!",
                     font="Arial", size=14, color="#FF0000", bold=True)

    # 결과 출력
    print("=== 렌더링 (처음 10개) ===")
    editor.render(max_chars=10)

    print(f"\n{'=' * 50}")
    print("메모리 분석")
    print(f"{'=' * 50}")
    info = editor.get_memory_info()
    print(f"총 글자 수: {info['total_characters']:,}")
    print(f"고유 스타일 수: {info['unique_styles']}")
    print(f"Flyweight 없이: {info['memory_without_flyweight']:,} bytes")
    print(f"Flyweight 사용: {info['memory_with_flyweight']:,} bytes")
    print(f"절감량: {info['memory_saved']:,} bytes "
          f"({info['saving_percent']:.1f}%)")

    print(f"\n{'=' * 50}")
    print("팩토리 통계")
    print(f"{'=' * 50}")
    stats = StyleFactory.get_stats()
    print(f"고유 스타일: {stats['unique_styles']}")
    print(f"캐시 히트: {stats['cache_hits']:,}")
    print(f"캐시 미스: {stats['cache_misses']}")
```

---

## 9. C# 구현

```csharp
using System;
using System.Collections.Generic;

// ===== Flyweight: 나무 종류 (공유 데이터) =====
public class TreeType
{
    // 내재 상태 (공유, 불변)
    public string Name { get; }
    public string Color { get; }
    public string TexturePath { get; }
    // 실제로는 byte[] textureData 등의 큰 데이터

    public TreeType(string name, string color, string texturePath)
    {
        Name = name;
        Color = color;
        TexturePath = texturePath;
        Console.WriteLine($"[TreeType] 생성: {name} ({color})");
    }

    public void Render(float x, float y, float height, float width)
    {
        // 내재 상태(텍스처) + 외재 상태(위치, 크기)로 렌더링
        // 실제로는 GPU에 텍스처 전달 후 지정 좌표에 렌더링
    }
}

// ===== Flyweight Factory =====
public class TreeTypeFactory
{
    private static readonly Dictionary<string, TreeType> _types = new();

    public static TreeType GetTreeType(string name, string color,
                                        string texturePath)
    {
        var key = $"{name}_{color}";

        if (!_types.ContainsKey(key))
        {
            _types[key] = new TreeType(name, color, texturePath);
        }

        return _types[key];
    }

    public static int TypeCount => _types.Count;
}

// ===== Context: 개별 나무 =====
public class Tree
{
    // 외재 상태 (개별)
    public float X { get; set; }
    public float Y { get; set; }
    public float Height { get; set; }
    public float Width { get; set; }

    // Flyweight 참조 (공유)
    private readonly TreeType _type;

    public Tree(float x, float y, float height, float width,
                TreeType type)
    {
        X = x;
        Y = y;
        Height = height;
        Width = width;
        _type = type;
    }

    public void Render()
    {
        _type.Render(X, Y, Height, Width);
    }
}

// ===== 숲: 대량의 나무 관리 =====
public class Forest
{
    private readonly List<Tree> _trees = new();
    private readonly Random _random = new();

    public void PlantTree(float x, float y,
                          string name, string color,
                          string texturePath)
    {
        // Factory에서 Flyweight 가져오기 (공유!)
        var type = TreeTypeFactory.GetTreeType(
            name, color, texturePath);

        var height = (float)(_random.NextDouble() * 15 + 5);
        var width = height * 0.3f;

        _trees.Add(new Tree(x, y, height, width, type));
    }

    public void RenderAll()
    {
        foreach (var tree in _trees)
            tree.Render();
    }

    public void PrintStats()
    {
        Console.WriteLine($"총 나무 수: {_trees.Count:N0}");
        Console.WriteLine($"고유 나무 타입: {TreeTypeFactory.TypeCount}");

        // 메모리 추정
        long withoutFlyweight = _trees.Count * (16 + 100);
        // 16: 외재 상태, 100: 내재 상태(텍스처 경로 등)
        long withFlyweight = _trees.Count * (16 + 8)
                             + TreeTypeFactory.TypeCount * 100;
        // 8: 참조 크기

        Console.WriteLine($"Flyweight 없이: ~{withoutFlyweight:N0} bytes");
        Console.WriteLine($"Flyweight 사용: ~{withFlyweight:N0} bytes");
        Console.WriteLine($"절감: ~{withoutFlyweight - withFlyweight:N0} bytes");
    }
}

// ===== 사용 예시 =====
public class Program
{
    public static void Main()
    {
        var forest = new Forest();
        var random = new Random(42);

        // 10,000 그루의 나무 심기
        // 하지만 TreeType은 6개만 생성됨!
        string[] treeNames = { "참나무", "소나무", "자작나무" };
        string[] colors = { "green", "dark_green" };
        string[] textures = {
            "oak.png", "pine.png", "birch.png",
            "oak_dark.png", "pine_dark.png", "birch_dark.png"
        };

        Console.WriteLine("=== 나무 심기 ===");
        for (int i = 0; i < 10000; i++)
        {
            int typeIdx = random.Next(treeNames.Length);
            int colorIdx = random.Next(colors.Length);
            int textureIdx = typeIdx * 2 + colorIdx;

            forest.PlantTree(
                x: (float)(random.NextDouble() * 1000),
                y: (float)(random.NextDouble() * 1000),
                name: treeNames[typeIdx],
                color: colors[colorIdx],
                texturePath: textures[textureIdx]
            );
        }

        Console.WriteLine($"\n=== 통계 ===");
        forest.PrintStats();
    }
}
```

---

## 10. 실무 예제: 게임 파티클 시스템

### Python - 파티클 시스템

```python
from __future__ import annotations
from dataclasses import dataclass, field
from enum import Enum
import random
import time


# ===== 파티클 타입 (Flyweight) =====
class ParticleType(Enum):
    FIRE = "fire"
    SMOKE = "smoke"
    SPARK = "spark"
    EXPLOSION = "explosion"
    RAIN = "rain"
    SNOW = "snow"


@dataclass(frozen=True)  # frozen=True → 불변(immutable)!
class ParticleTemplate:
    """파티클 템플릿 (Flyweight)

    같은 종류의 파티클이 공유하는 불변 데이터.
    frozen=True로 불변성을 보장합니다.
    """
    # 내재 상태 (공유, 불변)
    type_name: str
    texture_path: str
    base_color: tuple[int, int, int]  # RGB
    blend_mode: str
    animation_frames: int
    gravity_affect: float
    base_size: float
    base_lifetime: float  # 초

    # 실제 게임에서는 여기에 텍스처 바이트 데이터,
    # 셰이더 정보, 사운드 등 무거운 데이터가 들어감

    def get_frame(self, progress: float) -> int:
        """진행률에 따른 현재 프레임"""
        return int(progress * (self.animation_frames - 1))


# ===== Flyweight Factory =====
class ParticleTemplateFactory:
    """파티클 템플릿 팩토리 (Flyweight Factory)"""

    _templates: dict[str, ParticleTemplate] = {}

    # 미리 정의된 템플릿
    _PRESETS = {
        ParticleType.FIRE: ParticleTemplate(
            type_name="fire",
            texture_path="assets/particles/fire.png",
            base_color=(255, 100, 0),
            blend_mode="additive",
            animation_frames=16,
            gravity_affect=-0.5,  # 위로 올라감
            base_size=2.0,
            base_lifetime=1.5,
        ),
        ParticleType.SMOKE: ParticleTemplate(
            type_name="smoke",
            texture_path="assets/particles/smoke.png",
            base_color=(128, 128, 128),
            blend_mode="alpha",
            animation_frames=8,
            gravity_affect=-0.2,
            base_size=3.0,
            base_lifetime=3.0,
        ),
        ParticleType.SPARK: ParticleTemplate(
            type_name="spark",
            texture_path="assets/particles/spark.png",
            base_color=(255, 255, 200),
            blend_mode="additive",
            animation_frames=4,
            gravity_affect=1.0,  # 중력 영향
            base_size=0.5,
            base_lifetime=0.5,
        ),
        ParticleType.EXPLOSION: ParticleTemplate(
            type_name="explosion",
            texture_path="assets/particles/explosion.png",
            base_color=(255, 200, 50),
            blend_mode="additive",
            animation_frames=32,
            gravity_affect=0.0,
            base_size=5.0,
            base_lifetime=0.8,
        ),
        ParticleType.RAIN: ParticleTemplate(
            type_name="rain",
            texture_path="assets/particles/rain.png",
            base_color=(100, 150, 255),
            blend_mode="alpha",
            animation_frames=1,
            gravity_affect=5.0,
            base_size=0.3,
            base_lifetime=2.0,
        ),
        ParticleType.SNOW: ParticleTemplate(
            type_name="snow",
            texture_path="assets/particles/snow.png",
            base_color=(255, 255, 255),
            blend_mode="alpha",
            animation_frames=1,
            gravity_affect=0.5,
            base_size=0.4,
            base_lifetime=5.0,
        ),
    }

    @classmethod
    def get_template(cls, particle_type: ParticleType) -> ParticleTemplate:
        """템플릿 Flyweight 가져오기"""
        key = particle_type.value
        if key not in cls._templates:
            if particle_type in cls._PRESETS:
                cls._templates[key] = cls._PRESETS[particle_type]
            else:
                raise ValueError(f"알 수 없는 파티클 타입: {particle_type}")
        return cls._templates[key]

    @classmethod
    def get_stats(cls) -> dict:
        return {
            "loaded_templates": len(cls._templates),
            "available_types": len(cls._PRESETS),
        }


# ===== Context: 개별 파티클 =====
@dataclass
class Particle:
    """개별 파티클 (외재 상태)

    각 파티클의 고유한 상태만 저장합니다.
    """
    # 외재 상태 (각 파티클마다 고유)
    x: float
    y: float
    z: float
    velocity_x: float
    velocity_y: float
    velocity_z: float
    size_scale: float
    opacity: float
    age: float  # 현재 나이 (초)

    # Flyweight 참조 (공유)
    template: ParticleTemplate

    @property
    def is_alive(self) -> bool:
        return self.age < self.template.base_lifetime

    @property
    def progress(self) -> float:
        """생명주기 진행률 (0.0 ~ 1.0)"""
        return min(self.age / self.template.base_lifetime, 1.0)

    def update(self, delta_time: float):
        """프레임 업데이트"""
        self.age += delta_time

        # 위치 업데이트 (외재 상태 변경)
        self.x += self.velocity_x * delta_time
        self.y += self.velocity_y * delta_time
        self.z += self.velocity_z * delta_time

        # 중력 적용 (내재 상태의 gravity_affect 사용)
        self.velocity_y -= self.template.gravity_affect * delta_time

        # 페이드 아웃
        self.opacity = max(0, 1.0 - self.progress)

    def render(self) -> dict:
        """렌더링 정보 반환"""
        frame = self.template.get_frame(self.progress)
        actual_size = self.template.base_size * self.size_scale

        return {
            "texture": self.template.texture_path,
            "frame": frame,
            "position": (self.x, self.y, self.z),
            "size": actual_size,
            "color": self.template.base_color,
            "opacity": self.opacity,
            "blend": self.template.blend_mode,
        }


# ===== 파티클 시스템 =====
class ParticleSystem:
    """파티클 시스템: 대량의 파티클을 효율적으로 관리"""

    def __init__(self, max_particles: int = 100_000):
        self._particles: list[Particle] = []
        self._max_particles = max_particles

    def emit(self, particle_type: ParticleType,
             x: float, y: float, z: float,
             count: int = 1,
             spread: float = 1.0,
             speed: float = 1.0) -> None:
        """파티클 방출"""
        # Flyweight Factory에서 공유 템플릿 가져오기
        template = ParticleTemplateFactory.get_template(particle_type)

        for _ in range(count):
            if len(self._particles) >= self._max_particles:
                break

            particle = Particle(
                x=x + random.uniform(-spread, spread),
                y=y + random.uniform(-spread, spread),
                z=z + random.uniform(-spread, spread),
                velocity_x=random.uniform(-1, 1) * speed,
                velocity_y=random.uniform(0, 2) * speed,
                velocity_z=random.uniform(-1, 1) * speed,
                size_scale=random.uniform(0.5, 1.5),
                opacity=1.0,
                age=0.0,
                template=template,  # Flyweight 공유!
            )
            self._particles.append(particle)

    def update(self, delta_time: float) -> None:
        """모든 파티클 업데이트"""
        for particle in self._particles:
            particle.update(delta_time)

        # 죽은 파티클 제거
        self._particles = [
            p for p in self._particles if p.is_alive
        ]

    def render(self) -> list[dict]:
        """모든 파티클 렌더링"""
        return [p.render() for p in self._particles]

    @property
    def active_count(self) -> int:
        return len(self._particles)

    def get_memory_stats(self) -> dict:
        """메모리 사용 통계"""
        import sys

        # Flyweight 없이: 각 파티클이 템플릿 데이터를 모두 가짐
        template_size = sys.getsizeof(ParticleTemplate(
            "", "", (0, 0, 0), "", 0, 0.0, 0.0, 0.0
        ))
        particle_ext_size = 8 * 9  # 9개의 float (외재 상태)
        ref_size = 8  # 참조 크기

        count = len(self._particles)
        unique_templates = ParticleTemplateFactory.get_stats()[
            "loaded_templates"
        ]

        without = count * (particle_ext_size + template_size)
        with_fw = (count * (particle_ext_size + ref_size)
                   + unique_templates * template_size)

        return {
            "particle_count": count,
            "unique_templates": unique_templates,
            "without_flyweight_bytes": without,
            "with_flyweight_bytes": with_fw,
            "saved_bytes": without - with_fw,
            "saving_percent": (
                (1 - with_fw / without) * 100
                if without > 0 else 0
            ),
        }


# ===== 사용 예시 =====
if __name__ == "__main__":
    system = ParticleSystem(max_particles=100_000)

    # 폭발 효과: 다양한 파티클 조합
    print("=== 폭발 효과 생성 ===")
    system.emit(ParticleType.EXPLOSION, 100, 50, 0,
                count=500, spread=2.0, speed=5.0)
    system.emit(ParticleType.FIRE, 100, 50, 0,
                count=1000, spread=3.0, speed=2.0)
    system.emit(ParticleType.SMOKE, 100, 55, 0,
                count=500, spread=4.0, speed=1.0)
    system.emit(ParticleType.SPARK, 100, 50, 0,
                count=2000, spread=5.0, speed=8.0)

    # 비 효과
    print("=== 비 효과 생성 ===")
    for _ in range(50):
        system.emit(ParticleType.RAIN,
                    random.uniform(0, 500),
                    200,
                    random.uniform(0, 500),
                    count=100, spread=10.0, speed=3.0)

    print(f"\n활성 파티클 수: {system.active_count:,}")

    # 메모리 통계
    stats = system.get_memory_stats()
    print(f"\n{'=' * 50}")
    print("메모리 분석")
    print(f"{'=' * 50}")
    print(f"파티클 수: {stats['particle_count']:,}")
    print(f"고유 템플릿: {stats['unique_templates']}")
    print(f"Flyweight 없이: {stats['without_flyweight_bytes']:,} bytes")
    print(f"Flyweight 사용: {stats['with_flyweight_bytes']:,} bytes")
    print(f"절감: {stats['saved_bytes']:,} bytes "
          f"({stats['saving_percent']:.1f}%)")

    # 시뮬레이션
    print(f"\n{'=' * 50}")
    print("시뮬레이션 (3프레임)")
    print(f"{'=' * 50}")
    for frame in range(3):
        system.update(delta_time=0.016)  # ~60fps
        print(f"프레임 {frame + 1}: "
              f"활성 파티클 {system.active_count:,}개")

    # 팩토리 통계
    factory_stats = ParticleTemplateFactory.get_stats()
    print(f"\n팩토리: {factory_stats}")
```

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **메모리 절약** | 공유로 인해 대폭적인 메모리 절감 |
| **성능 향상** | 객체 생성 비용 감소 |
| **대량 객체 처리** | 수십만 개의 객체도 효율적으로 관리 |

### 단점

| 단점 | 설명 |
|------|------|
| **코드 복잡도** | 내재/외재 상태 분리로 코드가 복잡해짐 |
| **CPU 트레이드오프** | 외재 상태 계산에 CPU 시간 소모 (메모리 vs CPU) |
| **스레드 안전성** | 공유 객체이므로 멀티스레드 환경에서 주의 필요 |
| **불변성 강제** | 내재 상태는 반드시 불변이어야 함 |

### 언제 사용하지 말아야 하는가?

- 객체 수가 적을 때 (수십~수백 개면 불필요)
- 객체의 대부분의 상태가 고유할 때 (공유할 것이 없음)
- 메모리가 충분할 때 (최적화의 필요성이 없음)

---

## 12. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Factory Method** | Flyweight 객체 생성/관리에 Factory 사용 |
| **Composite** | Composite의 Leaf를 Flyweight로 공유 가능 |
| **Singleton** | Flyweight Factory가 Singleton일 수 있음 |
| **State** | State 객체를 Flyweight로 공유 가능 |

---

## 13. 정리 및 체크리스트

### 핵심 정리

```
Flyweight 패턴 = "같은 것을 공유하여 메모리 절약"

When: 대량의 유사 객체가 메모리를 과다 사용할 때
How:  공유 가능한 상태(내재)를 분리하여 공유하고,
      고유한 상태(외재)만 개별 저장
Why:  메모리 사용을 대폭 줄이고 성능 향상
```

### 적용 체크리스트

- [ ] 대량의 유사 객체가 존재하는가? (최소 수천 개)
- [ ] 객체의 상태를 내재/외재로 분리할 수 있는가?
- [ ] 내재 상태가 불변(immutable)인가?
- [ ] 메모리 절감 효과가 코드 복잡도 증가를 정당화하는가?
- [ ] 외재 상태를 계산하는 CPU 비용이 수용 가능한가?
- [ ] Flyweight Factory를 스레드 안전하게 구현했는가?

### Python에서의 자연스러운 Flyweight

Python에서는 일부 Flyweight가 이미 내장되어 있습니다:

```python
# Python의 string interning (문자열 인터닝)
a = "hello"
b = "hello"
print(a is b)  # True - 같은 객체!

# Python의 small integer caching
a = 256
b = 256
print(a is b)  # True - 같은 객체! (-5 ~ 256)

# __slots__을 사용한 메모리 최적화
class Particle:
    __slots__ = ('x', 'y', 'z', 'template')
    # dict 대신 고정 슬롯 사용 → 메모리 절감
```

> **기억하세요:** "Flyweight는 메모리 최적화 패턴입니다. 대량의 객체에서 공유할 수 있는 것을 찾아 분리하세요. 단, 코드 복잡도가 정당화될 만큼 객체가 많을 때만 사용하세요."
