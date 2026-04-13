# Iterator 패턴 (반복자)

> **핵심 의도 한줄 요약**: 컬렉션의 내부 표현 방식을 노출하지 않으면서 모든 요소에 순차적으로 접근할 수 있는 방법을 제공한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 커스텀 컬렉션 순회](#7-실무-예제-커스텀-컬렉션-순회)
8. [장단점](#8-장단점)
9. [관련 패턴](#9-관련-패턴)
10. [정리 및 체크리스트](#10-정리-및-체크리스트)

---

## 1. 개요

Iterator 패턴은 **컬렉션의 내부 구조(배열, 링크드 리스트, 트리 등)를 숨기면서** 요소를 하나씩 순회할 수 있는 통일된 인터페이스를 제공하는 패턴이다.

현대 프로그래밍 언어에서는 이미 언어 차원에서 지원하는 경우가 많다:

| 언어 | Iterator 지원 방식 |
|------|-------------------|
| **Python** | `__iter__()`, `__next__()`, `yield` (Generator) |
| **C#** | `IEnumerable<T>`, `IEnumerator<T>`, `yield return` |
| **Java** | `Iterator<T>`, `Iterable<T>` |
| **JavaScript** | `Symbol.iterator`, `function*` (Generator) |

---

## 2. 문제 상황

### 문제: 다양한 내부 구조의 컬렉션 순회

조직도 시스템에서 부서별 직원 목록을 관리한다. 어떤 부서는 배열로, 어떤 부서는 트리 구조로 관리한다.

```python
# 나쁜 예: 내부 구조에 따라 순회 방식이 달라짐
class DepartmentA:
    def __init__(self):
        self.employees = ["Alice", "Bob", "Charlie"]  # 리스트

class DepartmentB:
    def __init__(self):
        self.employees = {  # 딕셔너리 (트리 구조)
            "팀장": {"팀원1": {}, "팀원2": {}}
        }

# 클라이언트 코드가 내부 구조를 알아야 한다
def print_all_employees(dept):
    if isinstance(dept, DepartmentA):
        for emp in dept.employees:
            print(emp)
    elif isinstance(dept, DepartmentB):
        # 트리를 재귀적으로 순회해야 함
        def traverse(tree):
            for key, value in tree.items():
                print(key)
                traverse(value)
        traverse(dept.employees)
```

문제점:

- **내부 구조 노출**: 클라이언트가 컬렉션의 내부 구조를 알아야 한다
- **다형성 부재**: 구조가 다르면 순회 방식도 달라진다
- **OCP 위반**: 새로운 구조 추가 시 클라이언트 코드 수정 필요

---

## 3. 일상 비유

**음악 플레이리스트**를 생각해 보자.

```
[플레이리스트]
  곡1 → 곡2 → 곡3 → 곡4 → 곡5

재생 방식 (Iterator 종류):
  - 순차 재생: 곡1 → 곡2 → 곡3 → 곡4 → 곡5
  - 셔플 재생: 곡3 → 곡1 → 곡5 → 곡2 → 곡4
  - 반복 재생: 곡1 → 곡2 → ... → 곡5 → 곡1 → ...
```

- 어떤 재생 방식을 선택하든 **"다음 곡"** 버튼을 누르면 된다
- 플레이리스트의 **내부 저장 방식**(배열, 링크드 리스트)을 알 필요 없다
- 같은 플레이리스트에 **다른 순회 방식**을 적용할 수 있다

---

## 4. UML 다이어그램

```
 ┌───────────────────┐         ┌───────────────────────┐
 │  <<interface>>    │         │    <<interface>>       │
 │    Iterable       │────────>│      Iterator          │
 ├───────────────────┤ creates ├───────────────────────┤
 │ + iterator()      │         │ + has_next(): bool     │
 └────────┬──────────┘         │ + next(): Element      │
          │                    └───────────┬───────────┘
          │                                │
 ┌────────▼──────────┐         ┌───────────▼───────────┐
 │ ConcreteCollection│         │  ConcreteIterator      │
 ├───────────────────┤         ├───────────────────────┤
 │ - items[]         │         │ - collection           │
 │ + iterator()      │         │ - current_index        │
 │ + add(item)       │         │ + has_next(): bool     │
 └───────────────────┘         │ + next(): Element      │
                               └───────────────────────┘
```

---

## 5. Python 구현

### 기본 구현: `__iter__`와 `__next__`

```python
from __future__ import annotations
from typing import Any, Iterator


class NumberRange:
    """커스텀 범위 컬렉션 (Iterable)"""

    def __init__(self, start: int, end: int) -> None:
        self._start = start
        self._end = end

    def __iter__(self) -> NumberRangeIterator:
        """Iterator 객체를 반환한다."""
        return NumberRangeIterator(self._start, self._end)


class NumberRangeIterator:
    """범위 반복자 (Iterator)"""

    def __init__(self, start: int, end: int) -> None:
        self._current = start
        self._end = end

    def __iter__(self) -> NumberRangeIterator:
        return self

    def __next__(self) -> int:
        if self._current >= self._end:
            raise StopIteration
        value = self._current
        self._current += 1
        return value


# 사용 예시
numbers = NumberRange(1, 6)
for num in numbers:
    print(num, end=" ")  # 1 2 3 4 5
```

### Generator를 활용한 간결한 구현

```python
from typing import Generator


class BookShelf:
    """책장 컬렉션"""

    def __init__(self) -> None:
        self._books: list[str] = []

    def add(self, book: str) -> None:
        self._books.append(book)

    def __len__(self) -> int:
        return len(self._books)

    def __iter__(self) -> Generator[str, None, None]:
        """순차 순회 (Generator 사용)"""
        for book in self._books:
            yield book

    def reverse_iter(self) -> Generator[str, None, None]:
        """역순 순회"""
        for book in reversed(self._books):
            yield book

    def filtered_iter(self, keyword: str) -> Generator[str, None, None]:
        """필터링 순회"""
        for book in self._books:
            if keyword.lower() in book.lower():
                yield book


# === 사용 예시 ===
if __name__ == "__main__":
    shelf = BookShelf()
    shelf.add("파이썬 프로그래밍")
    shelf.add("디자인 패턴")
    shelf.add("클린 코드")
    shelf.add("파이썬 쿡북")
    shelf.add("리팩터링")

    print("=== 순차 순회 ===")
    for book in shelf:
        print(f"  {book}")

    print("\n=== 역순 순회 ===")
    for book in shelf.reverse_iter():
        print(f"  {book}")

    print("\n=== '파이썬' 필터링 ===")
    for book in shelf.filtered_iter("파이썬"):
        print(f"  {book}")
```

### 실전 Generator 패턴: 무한 시퀀스와 지연 평가

```python
from typing import Generator


def fibonacci() -> Generator[int, None, None]:
    """피보나치 수열 (무한 시퀀스)"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b


def take(n: int, iterable) -> list:
    """처음 n개만 가져온다 (지연 평가 활용)"""
    result = []
    for i, item in enumerate(iterable):
        if i >= n:
            break
        result.append(item)
    return result


# 메모리를 거의 사용하지 않으면서 무한 시퀀스 처리
print(take(10, fibonacci()))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 6. C# 구현

### 기본 구현: `IEnumerable<T>`와 `IEnumerator<T>`

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

// 책장 컬렉션
public class BookShelf : IEnumerable<string>
{
    private readonly List<string> _books = new();

    public void Add(string book) => _books.Add(book);
    public int Count => _books.Count;

    // IEnumerable<string> 구현
    public IEnumerator<string> GetEnumerator()
    {
        return new BookShelfEnumerator(_books);
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    // 역순 반복 (yield return 활용)
    public IEnumerable<string> ReverseIterator()
    {
        for (int i = _books.Count - 1; i >= 0; i--)
        {
            yield return _books[i];
        }
    }

    // 필터링 반복
    public IEnumerable<string> FilteredIterator(string keyword)
    {
        foreach (var book in _books)
        {
            if (book.Contains(keyword, StringComparison.OrdinalIgnoreCase))
            {
                yield return book;
            }
        }
    }
}

// 수동 구현한 Enumerator
public class BookShelfEnumerator : IEnumerator<string>
{
    private readonly List<string> _books;
    private int _currentIndex = -1;

    public BookShelfEnumerator(List<string> books)
    {
        _books = books;
    }

    public string Current => _books[_currentIndex];
    object IEnumerator.Current => Current;

    public bool MoveNext()
    {
        _currentIndex++;
        return _currentIndex < _books.Count;
    }

    public void Reset() => _currentIndex = -1;
    public void Dispose() { }
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var shelf = new BookShelf();
        shelf.Add("C# 프로그래밍");
        shelf.Add("디자인 패턴");
        shelf.Add("클린 코드");
        shelf.Add("C# 쿡북");
        shelf.Add("리팩터링");

        Console.WriteLine("=== 순차 순회 (foreach) ===");
        foreach (var book in shelf)
        {
            Console.WriteLine($"  {book}");
        }

        Console.WriteLine("\n=== 역순 순회 ===");
        foreach (var book in shelf.ReverseIterator())
        {
            Console.WriteLine($"  {book}");
        }

        Console.WriteLine("\n=== 'C#' 필터링 ===");
        foreach (var book in shelf.FilteredIterator("C#"))
        {
            Console.WriteLine($"  {book}");
        }
    }
}
```

### `yield return`으로 간결하게 구현

```csharp
using System;
using System.Collections.Generic;

public class NumberRange
{
    private readonly int _start;
    private readonly int _end;

    public NumberRange(int start, int end)
    {
        _start = start;
        _end = end;
    }

    // yield return을 사용하면 IEnumerator를 직접 구현할 필요 없다
    public IEnumerable<int> GetNumbers()
    {
        for (int i = _start; i < _end; i++)
        {
            yield return i;
        }
    }

    // 짝수만 반환
    public IEnumerable<int> GetEvenNumbers()
    {
        for (int i = _start; i < _end; i++)
        {
            if (i % 2 == 0)
                yield return i;
        }
    }
}

// 무한 시퀀스 (피보나치)
public static class Sequences
{
    public static IEnumerable<long> Fibonacci()
    {
        long a = 0, b = 1;
        while (true)
        {
            yield return a;
            (a, b) = (b, a + b);
        }
    }

    // LINQ와 함께 사용
    // Sequences.Fibonacci().Take(10).ToList()
    // → [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
}
```

---

## 7. 실무 예제: 커스텀 컬렉션 순회

### Python: 트리 구조 순회

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Generator


@dataclass
class TreeNode:
    """트리 노드"""
    value: str
    children: list[TreeNode] = field(default_factory=list)

    def add_child(self, value: str) -> TreeNode:
        child = TreeNode(value)
        self.children.append(child)
        return child


class Tree:
    """트리 컬렉션: 다양한 순회 방식 제공"""

    def __init__(self, root: TreeNode) -> None:
        self._root = root

    def dfs(self, node: TreeNode | None = None) -> Generator[str, None, None]:
        """깊이 우선 탐색 (DFS)"""
        if node is None:
            node = self._root
        yield node.value
        for child in node.children:
            yield from self.dfs(child)

    def bfs(self) -> Generator[str, None, None]:
        """너비 우선 탐색 (BFS)"""
        queue = [self._root]
        while queue:
            current = queue.pop(0)
            yield current.value
            queue.extend(current.children)

    def __iter__(self) -> Generator[str, None, None]:
        """기본 순회는 DFS"""
        return self.dfs()


# === 사용 예시: 조직도 순회 ===
if __name__ == "__main__":
    # 조직도 생성
    ceo = TreeNode("CEO")
    cto = ceo.add_child("CTO")
    cfo = ceo.add_child("CFO")

    dev_lead = cto.add_child("개발팀장")
    dev_lead.add_child("시니어 개발자")
    dev_lead.add_child("주니어 개발자")

    infra_lead = cto.add_child("인프라팀장")
    infra_lead.add_child("DevOps 엔지니어")

    finance_lead = cfo.add_child("재무팀장")

    org = Tree(ceo)

    print("=== DFS (깊이 우선) ===")
    for name in org.dfs():
        print(f"  {name}")

    print("\n=== BFS (너비 우선) ===")
    for name in org.bfs():
        print(f"  {name}")
```

**출력:**

```
=== DFS (깊이 우선) ===
  CEO
  CTO
  개발팀장
  시니어 개발자
  주니어 개발자
  인프라팀장
  DevOps 엔지니어
  CFO
  재무팀장

=== BFS (너비 우선) ===
  CEO
  CTO
  CFO
  개발팀장
  인프라팀장
  재무팀장
  시니어 개발자
  주니어 개발자
  DevOps 엔지니어
```

### C#: 페이지네이션 Iterator

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

/// <summary>
/// 대량의 데이터를 페이지 단위로 순회하는 Iterator
/// </summary>
public class PaginatedIterator<T>
{
    private readonly IList<T> _items;
    private readonly int _pageSize;

    public PaginatedIterator(IList<T> items, int pageSize)
    {
        _items = items;
        _pageSize = pageSize;
    }

    public int TotalPages => (int)Math.Ceiling((double)_items.Count / _pageSize);

    /// <summary>
    /// 페이지별로 순회
    /// </summary>
    public IEnumerable<Page<T>> GetPages()
    {
        int pageNumber = 1;
        for (int i = 0; i < _items.Count; i += _pageSize)
        {
            var pageItems = _items.Skip(i).Take(_pageSize).ToList();
            yield return new Page<T>
            {
                PageNumber = pageNumber++,
                Items = pageItems,
                TotalPages = TotalPages
            };
        }
    }

    /// <summary>
    /// 모든 아이템을 하나씩 순회 (페이지 경계 무시)
    /// </summary>
    public IEnumerable<T> GetAllItems()
    {
        foreach (var item in _items)
        {
            yield return item;
        }
    }
}

public class Page<T>
{
    public int PageNumber { get; set; }
    public List<T> Items { get; set; } = new();
    public int TotalPages { get; set; }

    public override string ToString()
        => $"페이지 {PageNumber}/{TotalPages} ({Items.Count}건)";
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        // 100개의 상품 데이터
        var products = Enumerable.Range(1, 100)
            .Select(i => $"상품_{i:D3}")
            .ToList();

        var paginator = new PaginatedIterator<string>(products, pageSize: 10);

        Console.WriteLine($"전체 {products.Count}건, {paginator.TotalPages}페이지\n");

        // 처음 3페이지만 출력
        foreach (var page in paginator.GetPages().Take(3))
        {
            Console.WriteLine($"=== {page} ===");
            foreach (var product in page.Items)
            {
                Console.WriteLine($"  {product}");
            }
        }
    }
}
```

---

## 8. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **내부 구조 은닉** | 컬렉션의 내부 표현을 노출하지 않는다 |
| **통일된 인터페이스** | 다양한 컬렉션을 동일한 방식으로 순회할 수 있다 |
| **다중 순회 지원** | 같은 컬렉션에 여러 순회 방식(DFS, BFS 등)을 제공할 수 있다 |
| **지연 평가** | Generator를 활용하면 필요한 시점에만 요소를 생성한다 |
| **단일 책임 원칙** | 순회 로직을 컬렉션에서 분리한다 |

### 단점

| 단점 | 설명 |
|------|------|
| **과도한 설계** | 단순한 컬렉션에는 불필요할 수 있다 (언어 기본 지원 활용) |
| **효율성** | 특정 상황에서는 직접 접근이 더 효율적일 수 있다 |
| **동시 수정 문제** | 순회 중 컬렉션이 수정되면 문제가 발생할 수 있다 |

---

## 9. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Composite** | 트리 구조를 순회할 때 Iterator를 함께 사용 |
| **Factory Method** | 컬렉션이 적절한 Iterator를 생성할 때 사용 |
| **Memento** | Iterator의 현재 위치를 저장/복원하는 데 사용 가능 |
| **Visitor** | Iterator로 순회하면서 Visitor로 작업 수행 |

---

## 10. 정리 및 체크리스트

### 핵심 정리

1. Iterator 패턴은 **컬렉션의 내부 구조를 숨기면서 순차적 접근**을 제공한다.
2. Python에서는 `__iter__()`, `__next__()`, `yield`로, C#에서는 `IEnumerable<T>`, `yield return`으로 구현한다.
3. **Generator/yield**를 활용하면 메모리 효율적인 **지연 평가(Lazy Evaluation)**가 가능하다.
4. 현대 언어에서는 언어 수준에서 지원하므로 **직접 Iterator 클래스를 만들 필요가 줄었다**.

### 적용 체크리스트

- [ ] 컬렉션의 **내부 구조를 숨기면서** 요소에 접근해야 하는가?
- [ ] **여러 가지 순회 방식**(순차, 역순, 필터링, DFS, BFS)을 지원해야 하는가?
- [ ] 대용량 데이터를 **메모리 효율적으로** 처리해야 하는가? (Generator 활용)
- [ ] 서로 다른 컬렉션을 **동일한 인터페이스**로 순회해야 하는가?
- [ ] **무한 시퀀스**를 다뤄야 하는가?

> **2~3년차 개발자를 위한 팁**: Python의 `for ... in` 구문, C#의 `foreach` 구문은 내부적으로 Iterator 패턴을 사용한다. `yield`를 이해하면 메모리 효율적인 데이터 처리가 가능해진다. 예를 들어 10GB 파일을 한 줄씩 처리할 때 Generator를 쓰면 메모리 걱정 없이 처리할 수 있다.
