# Interpreter 패턴 (인터프리터)

> **핵심 의도 한줄 요약**: 언어의 문법 규칙을 클래스 계층으로 표현하고, 그 문법에 따라 문장(표현식)을 해석(평가)한다.

---

## 목차

1. [개요](#1-개요)
2. [문제 상황](#2-문제-상황)
3. [일상 비유](#3-일상-비유)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [Python 구현](#5-python-구현)
6. [C# 구현](#6-c-구현)
7. [실무 예제: 간단한 수식 계산기](#7-실무-예제-간단한-수식-계산기)
8. [사용 빈도가 낮은 이유](#8-사용-빈도가-낮은-이유)
9. [장단점](#9-장단점)
10. [관련 패턴](#10-관련-패턴)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)

---

## 1. 개요

Interpreter 패턴은 **특정 언어의 문법을 클래스 구조로 표현**하고, 그 구조를 사용하여 문장을 해석하는 패턴이다.

간단히 말하면:

```
"3 + 5 * 2"  →  파싱  →  AST(추상 구문 트리)  →  해석  →  결과: 13
```

### 핵심 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| **AbstractExpression** | 모든 표현식의 공통 인터페이스 (interpret 메서드) |
| **TerminalExpression** | 더 이상 분해할 수 없는 최소 단위 (숫자, 변수) |
| **NonterminalExpression** | 다른 표현식을 조합하는 표현식 (덧셈, 곱셈, AND, OR) |
| **Context** | 해석에 필요한 전역 정보 (변수 값 등) |
| **Client** | AST를 구성하고 interpret를 호출 |

---

## 2. 문제 상황

### 문제: 규칙 기반 시스템

할인 정책을 설정 파일이나 관리자 인터페이스에서 **문자열로 정의**하고 싶다.

```python
# 나쁜 예: 규칙을 하드코딩
def apply_discount(user, order):
    if user.grade == "VIP" and order.amount > 50000:
        return order.amount * 0.1
    elif user.age >= 65 or user.has_coupon:
        return order.amount * 0.05
    # 새 규칙 추가마다 코드 수정 & 배포 필요...
```

원하는 것:

```
# 설정으로 규칙을 정의 (코드 수정 없이 변경 가능)
"grade == 'VIP' AND amount > 50000"  → 10% 할인
"age >= 65 OR has_coupon"            → 5% 할인
```

이런 **미니 언어(DSL)**를 해석하려면 Interpreter 패턴이 필요하다.

---

## 3. 일상 비유

**음악 악보 해석**을 떠올려 보자.

```
악보 (Expression):
  ♩ = 4분음표 (TerminalExpression)
  ♪ = 8분음표 (TerminalExpression)
  ♩♩♪♪ = 리듬 패턴 (NonterminalExpression)

연주자 (Interpreter):
  악보를 읽고 → 각 음표를 해석하여 → 소리로 변환

같은 악보를 피아노로 해석하면 → 피아노 소리
같은 악보를 바이올린으로 해석하면 → 바이올린 소리
```

---

## 4. UML 다이어그램

```
 ┌───────────┐       ┌──────────────────────┐
 │  Client   │──────>│  <<interface>>       │
 └───────────┘       │  AbstractExpression  │
                     ├──────────────────────┤
                     │ + interpret(context) │
                     └──────────┬───────────┘
                                │
                   ┌────────────┼────────────┐
                   │                         │
        ┌──────────▼─────────┐    ┌──────────▼─────────┐
        │ TerminalExpression │    │NonterminalExpression│
        ├────────────────────┤    ├─────────────────────┤
        │ + interpret()      │    │ - left: Expression  │
        │   (숫자, 변수 등)  │    │ - right: Expression │
        └────────────────────┘    │ + interpret()       │
                                  │   (조합: +, -, AND) │
                                  └─────────────────────┘

 예: "3 + 5"
    Add(Number(3), Number(5)).interpret()
     = 3 + 5
     = 8
```

---

## 5. Python 구현

### 기본 구현: 불리언 표현식 해석기

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


class Context:
    """해석에 필요한 컨텍스트 (변수 저장소)"""

    def __init__(self) -> None:
        self._variables: dict[str, Any] = {}

    def set_variable(self, name: str, value: Any) -> None:
        self._variables[name] = value

    def get_variable(self, name: str) -> Any:
        if name not in self._variables:
            raise KeyError(f"정의되지 않은 변수: {name}")
        return self._variables[name]


class Expression(ABC):
    """추상 표현식"""

    @abstractmethod
    def interpret(self, context: Context) -> Any:
        pass

    @abstractmethod
    def __str__(self) -> str:
        pass


# === Terminal Expressions ===

class NumberExpression(Expression):
    """숫자 리터럴"""

    def __init__(self, value: float) -> None:
        self._value = value

    def interpret(self, context: Context) -> float:
        return self._value

    def __str__(self) -> str:
        return str(self._value)


class VariableExpression(Expression):
    """변수 참조"""

    def __init__(self, name: str) -> None:
        self._name = name

    def interpret(self, context: Context) -> Any:
        return context.get_variable(self._name)

    def __str__(self) -> str:
        return self._name


class BooleanExpression(Expression):
    """불리언 리터럴"""

    def __init__(self, value: bool) -> None:
        self._value = value

    def interpret(self, context: Context) -> bool:
        return self._value

    def __str__(self) -> str:
        return str(self._value)


# === Nonterminal Expressions: 산술 ===

class AddExpression(Expression):
    """덧셈"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> float:
        return self._left.interpret(context) + self._right.interpret(context)

    def __str__(self) -> str:
        return f"({self._left} + {self._right})"


class SubtractExpression(Expression):
    """뺄셈"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> float:
        return self._left.interpret(context) - self._right.interpret(context)

    def __str__(self) -> str:
        return f"({self._left} - {self._right})"


class MultiplyExpression(Expression):
    """곱셈"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> float:
        return self._left.interpret(context) * self._right.interpret(context)

    def __str__(self) -> str:
        return f"({self._left} * {self._right})"


# === Nonterminal Expressions: 비교 ===

class GreaterThanExpression(Expression):
    """크다 비교 (>)"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> bool:
        return self._left.interpret(context) > self._right.interpret(context)

    def __str__(self) -> str:
        return f"({self._left} > {self._right})"


class EqualsExpression(Expression):
    """같다 비교 (==)"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> bool:
        return self._left.interpret(context) == self._right.interpret(context)

    def __str__(self) -> str:
        return f"({self._left} == {self._right})"


# === Nonterminal Expressions: 논리 ===

class AndExpression(Expression):
    """논리 AND"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> bool:
        return bool(self._left.interpret(context) and self._right.interpret(context))

    def __str__(self) -> str:
        return f"({self._left} AND {self._right})"


class OrExpression(Expression):
    """논리 OR"""

    def __init__(self, left: Expression, right: Expression) -> None:
        self._left = left
        self._right = right

    def interpret(self, context: Context) -> bool:
        return bool(self._left.interpret(context) or self._right.interpret(context))

    def __str__(self) -> str:
        return f"({self._left} OR {self._right})"


class NotExpression(Expression):
    """논리 NOT"""

    def __init__(self, expression: Expression) -> None:
        self._expression = expression

    def interpret(self, context: Context) -> bool:
        return not self._expression.interpret(context)

    def __str__(self) -> str:
        return f"(NOT {self._expression})"


# === 사용 예시 ===
if __name__ == "__main__":
    context = Context()

    # 변수 설정
    context.set_variable("x", 10)
    context.set_variable("y", 20)
    context.set_variable("grade", "VIP")

    # 수식: (x + y) * 2
    expr1 = MultiplyExpression(
        AddExpression(VariableExpression("x"), VariableExpression("y")),
        NumberExpression(2),
    )
    print(f"수식: {expr1}")
    print(f"결과: {expr1.interpret(context)}")

    # 조건: x > 5 AND y > 15
    expr2 = AndExpression(
        GreaterThanExpression(VariableExpression("x"), NumberExpression(5)),
        GreaterThanExpression(VariableExpression("y"), NumberExpression(15)),
    )
    print(f"\n조건: {expr2}")
    print(f"결과: {expr2.interpret(context)}")

    # 복합: (x > 5 AND y > 15) OR NOT (x == 0)
    expr3 = OrExpression(
        expr2,
        NotExpression(
            EqualsExpression(VariableExpression("x"), NumberExpression(0))
        ),
    )
    print(f"\n복합: {expr3}")
    print(f"결과: {expr3.interpret(context)}")
```

---

## 6. C# 구현

```csharp
using System;
using System.Collections.Generic;

// Context
public class Context
{
    private readonly Dictionary<string, object> _variables = new();

    public void SetVariable(string name, object value)
    {
        _variables[name] = value;
    }

    public object GetVariable(string name)
    {
        if (!_variables.ContainsKey(name))
            throw new KeyNotFoundException($"정의되지 않은 변수: {name}");
        return _variables[name];
    }

    public T GetVariable<T>(string name)
    {
        return (T)GetVariable(name);
    }
}

// 추상 표현식
public interface IExpression
{
    object Interpret(Context context);
}

// Terminal: 숫자
public class NumberExpression : IExpression
{
    private readonly double _value;

    public NumberExpression(double value) => _value = value;

    public object Interpret(Context context) => _value;
    public override string ToString() => _value.ToString();
}

// Terminal: 변수
public class VariableExpression : IExpression
{
    private readonly string _name;

    public VariableExpression(string name) => _name = name;

    public object Interpret(Context context) => context.GetVariable(_name);
    public override string ToString() => _name;
}

// Nonterminal: 덧셈
public class AddExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public AddExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public object Interpret(Context context)
    {
        double left = Convert.ToDouble(_left.Interpret(context));
        double right = Convert.ToDouble(_right.Interpret(context));
        return left + right;
    }

    public override string ToString() => $"({_left} + {_right})";
}

// Nonterminal: 곱셈
public class MultiplyExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public MultiplyExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public object Interpret(Context context)
    {
        double left = Convert.ToDouble(_left.Interpret(context));
        double right = Convert.ToDouble(_right.Interpret(context));
        return left * right;
    }

    public override string ToString() => $"({_left} * {_right})";
}

// Nonterminal: 크다 (>)
public class GreaterThanExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public GreaterThanExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public object Interpret(Context context)
    {
        double left = Convert.ToDouble(_left.Interpret(context));
        double right = Convert.ToDouble(_right.Interpret(context));
        return left > right;
    }

    public override string ToString() => $"({_left} > {_right})";
}

// Nonterminal: AND
public class AndExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public AndExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public object Interpret(Context context)
    {
        bool left = Convert.ToBoolean(_left.Interpret(context));
        bool right = Convert.ToBoolean(_right.Interpret(context));
        return left && right;
    }

    public override string ToString() => $"({_left} AND {_right})";
}

// Nonterminal: OR
public class OrExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public OrExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public object Interpret(Context context)
    {
        bool left = Convert.ToBoolean(_left.Interpret(context));
        bool right = Convert.ToBoolean(_right.Interpret(context));
        return left || right;
    }

    public override string ToString() => $"({_left} OR {_right})";
}

// === 사용 예시 ===
public class Program
{
    public static void Main()
    {
        var context = new Context();
        context.SetVariable("x", 10.0);
        context.SetVariable("y", 20.0);

        // (x + y) * 2
        var expr = new MultiplyExpression(
            new AddExpression(
                new VariableExpression("x"),
                new VariableExpression("y")),
            new NumberExpression(2));

        Console.WriteLine($"수식: {expr}");
        Console.WriteLine($"결과: {expr.Interpret(context)}");

        // x > 5 AND y > 15
        var condition = new AndExpression(
            new GreaterThanExpression(
                new VariableExpression("x"),
                new NumberExpression(5)),
            new GreaterThanExpression(
                new VariableExpression("y"),
                new NumberExpression(15)));

        Console.WriteLine($"\n조건: {condition}");
        Console.WriteLine($"결과: {condition.Interpret(context)}");
    }
}
```

---

## 7. 실무 예제: 간단한 수식 계산기

문자열 수식을 파싱하여 AST를 생성하고 해석하는 완전한 예제다.

### Python 구현

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any


# === AST 노드 정의 ===
class ASTNode(ABC):
    @abstractmethod
    def evaluate(self) -> float:
        pass


class NumberNode(ASTNode):
    def __init__(self, value: float) -> None:
        self.value = value

    def evaluate(self) -> float:
        return self.value

    def __repr__(self) -> str:
        return f"{self.value}"


class BinaryOpNode(ASTNode):
    def __init__(self, op: str, left: ASTNode, right: ASTNode) -> None:
        self.op = op
        self.left = left
        self.right = right

    def evaluate(self) -> float:
        left_val = self.left.evaluate()
        right_val = self.right.evaluate()

        match self.op:
            case "+":
                return left_val + right_val
            case "-":
                return left_val - right_val
            case "*":
                return left_val * right_val
            case "/":
                if right_val == 0:
                    raise ZeroDivisionError("0으로 나눌 수 없습니다.")
                return left_val / right_val
            case _:
                raise ValueError(f"알 수 없는 연산자: {self.op}")

    def __repr__(self) -> str:
        return f"({self.left} {self.op} {self.right})"


class UnaryMinusNode(ASTNode):
    def __init__(self, operand: ASTNode) -> None:
        self.operand = operand

    def evaluate(self) -> float:
        return -self.operand.evaluate()

    def __repr__(self) -> str:
        return f"(-{self.operand})"


# === 토크나이저 (Lexer) ===
class Token:
    def __init__(self, type_: str, value: Any) -> None:
        self.type = type_
        self.value = value

    def __repr__(self) -> str:
        return f"Token({self.type}, {self.value})"


class Lexer:
    """문자열을 토큰 목록으로 변환"""

    def __init__(self, text: str) -> None:
        self._text = text
        self._pos = 0

    def tokenize(self) -> list[Token]:
        tokens: list[Token] = []
        while self._pos < len(self._text):
            ch = self._text[self._pos]

            if ch.isspace():
                self._pos += 1
                continue
            elif ch.isdigit() or ch == ".":
                tokens.append(self._read_number())
            elif ch in "+-*/":
                tokens.append(Token("OP", ch))
                self._pos += 1
            elif ch == "(":
                tokens.append(Token("LPAREN", ch))
                self._pos += 1
            elif ch == ")":
                tokens.append(Token("RPAREN", ch))
                self._pos += 1
            else:
                raise SyntaxError(f"알 수 없는 문자: '{ch}' (위치: {self._pos})")

        tokens.append(Token("EOF", None))
        return tokens

    def _read_number(self) -> Token:
        start = self._pos
        has_dot = False
        while self._pos < len(self._text):
            ch = self._text[self._pos]
            if ch.isdigit():
                self._pos += 1
            elif ch == "." and not has_dot:
                has_dot = True
                self._pos += 1
            else:
                break
        return Token("NUMBER", float(self._text[start:self._pos]))


# === 파서 (재귀 하강 파서) ===
class Parser:
    """토큰 목록을 AST로 변환

    문법:
      expression = term (('+' | '-') term)*
      term       = factor (('*' | '/') factor)*
      factor     = NUMBER | '(' expression ')' | '-' factor
    """

    def __init__(self, tokens: list[Token]) -> None:
        self._tokens = tokens
        self._pos = 0

    def parse(self) -> ASTNode:
        node = self._expression()
        if self._current().type != "EOF":
            raise SyntaxError(f"예기치 않은 토큰: {self._current()}")
        return node

    def _current(self) -> Token:
        return self._tokens[self._pos]

    def _eat(self, type_: str) -> Token:
        token = self._current()
        if token.type != type_:
            raise SyntaxError(f"예상: {type_}, 실제: {token}")
        self._pos += 1
        return token

    def _expression(self) -> ASTNode:
        """expression = term (('+' | '-') term)*"""
        node = self._term()
        while self._current().type == "OP" and self._current().value in ("+", "-"):
            op = self._eat("OP").value
            right = self._term()
            node = BinaryOpNode(op, node, right)
        return node

    def _term(self) -> ASTNode:
        """term = factor (('*' | '/') factor)*"""
        node = self._factor()
        while self._current().type == "OP" and self._current().value in ("*", "/"):
            op = self._eat("OP").value
            right = self._factor()
            node = BinaryOpNode(op, node, right)
        return node

    def _factor(self) -> ASTNode:
        """factor = NUMBER | '(' expression ')' | '-' factor"""
        token = self._current()

        if token.type == "NUMBER":
            self._eat("NUMBER")
            return NumberNode(token.value)
        elif token.type == "LPAREN":
            self._eat("LPAREN")
            node = self._expression()
            self._eat("RPAREN")
            return node
        elif token.type == "OP" and token.value == "-":
            self._eat("OP")
            return UnaryMinusNode(self._factor())
        else:
            raise SyntaxError(f"예기치 않은 토큰: {token}")


# === 계산기 (통합) ===
class Calculator:
    """수식 계산기"""

    def calculate(self, expression: str) -> float:
        # 1. 토크나이징
        lexer = Lexer(expression)
        tokens = lexer.tokenize()

        # 2. 파싱 (AST 생성)
        parser = Parser(tokens)
        ast = parser.parse()

        # 3. 평가 (Interpret)
        result = ast.evaluate()

        print(f"  수식: {expression}")
        print(f"  AST:  {ast}")
        print(f"  결과: {result}")
        return result


# === 사용 예시 ===
if __name__ == "__main__":
    calc = Calculator()

    print("=== 기본 연산 ===")
    calc.calculate("3 + 5")

    print("\n=== 연산자 우선순위 ===")
    calc.calculate("3 + 5 * 2")

    print("\n=== 괄호 ===")
    calc.calculate("(3 + 5) * 2")

    print("\n=== 복합 수식 ===")
    calc.calculate("(10 - 3) * (2 + 4) / 7")

    print("\n=== 음수 ===")
    calc.calculate("-5 + 3")

    print("\n=== 소수점 ===")
    calc.calculate("3.14 * 2.0")
```

**출력:**

```
=== 기본 연산 ===
  수식: 3 + 5
  AST:  (3.0 + 5.0)
  결과: 8.0

=== 연산자 우선순위 ===
  수식: 3 + 5 * 2
  AST:  (3.0 + (5.0 * 2.0))
  결과: 13.0

=== 괄호 ===
  수식: (3 + 5) * 2
  AST:  ((3.0 + 5.0) * 2.0)
  결과: 16.0

=== 복합 수식 ===
  수식: (10 - 3) * (2 + 4) / 7
  AST:  (((10.0 - 3.0) * (2.0 + 4.0)) / 7.0)
  결과: 6.0
```

---

## 8. 사용 빈도가 낮은 이유

Interpreter 패턴은 GoF 패턴 중 **실무에서 가장 드물게 사용**되는 패턴이다. 그 이유는 다음과 같다.

### 1. 성능 문제

AST를 매번 순회하면서 해석하므로, 복잡한 문법이나 대량의 입력에는 **성능이 좋지 않다**. 실제 언어 인터프리터/컴파일러는 바이트코드 컴파일 등 더 효율적인 방식을 사용한다.

### 2. 기존 도구의 존재

| 용도 | 대안 도구 |
|------|-----------|
| 수식 계산 | Python `eval()`, Expression 라이브러리 |
| 정규 표현식 | `re` 모듈, `Regex` 클래스 |
| JSON/XML 파싱 | 기존 파서 라이브러리 |
| SQL 해석 | SQL 파서/ORM |
| DSL 구축 | ANTLR, PLY, Lex/Yacc 등 전문 도구 |

### 3. 문법이 복잡해지면 유지보수 어려움

간단한 문법에는 적합하지만, 문법 규칙이 늘어나면 클래스 수가 급증하고 관리가 어려워진다.

### 4. 적합한 사용 사례

그럼에도 Interpreter 패턴이 유용한 경우:

- **간단한 DSL** (설정 파일, 규칙 엔진)
- **수식 파서** (계산기, 검색 쿼리)
- **정규 표현식** 내부 구현
- **교육 목적** (컴파일러 이론 학습)

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **문법 표현** | 문법 규칙을 클래스 구조로 명확히 표현 |
| **확장 용이** | 새 문법 규칙(Expression) 추가가 쉽다 |
| **Visitor와 결합** | AST를 Visitor로 순회하여 다양한 연산 수행 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **성능** | 복잡한 문법에서 성능이 나쁘다 |
| **클래스 폭증** | 문법 규칙마다 클래스가 필요하여 코드량이 급증 |
| **복잡성** | 파서(Lexer + Parser) 구현이 별도로 필요 |
| **유지보수** | 문법이 복잡해지면 관리가 어렵다 |

---

## 10. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Composite** | AST는 Composite 패턴과 동일한 트리 구조 |
| **Visitor** | AST를 순회하면서 다양한 연산을 수행할 때 함께 사용 |
| **Flyweight** | Terminal Expression을 공유하여 메모리 절약 |
| **Iterator** | AST를 순회하는 데 사용 가능 |

---

## 11. 정리 및 체크리스트

### 핵심 정리

1. Interpreter는 **문법을 클래스 계층으로 표현**하고 해석하는 패턴이다.
2. **Terminal Expression**(리프)과 **Nonterminal Expression**(조합)으로 구성된다.
3. 실무에서는 **간단한 DSL이나 규칙 엔진**에 주로 사용된다.
4. 복잡한 문법에는 **ANTLR 등 전문 파서 도구**가 더 적합하다.

### 적용 체크리스트

- [ ] 해석해야 할 **간단한 문법**이 있는가?
- [ ] 문법이 **자주 변경**되지 않는가?
- [ ] **기존 파서 라이브러리**로 해결할 수 없는가?
- [ ] 성능이 **크리티컬하지 않은** 상황인가?
- [ ] **DSL이나 규칙 엔진**을 직접 구현해야 하는가?

> **2~3년차 개발자를 위한 팁**: 실무에서 Interpreter 패턴을 직접 구현할 일은 드물지만, 그 **개념을 이해하는 것은 매우 유용**하다. SQL 쿼리 빌더, ORM의 내부 동작, 정규 표현식 엔진, 템플릿 엔진(Jinja2, Razor) 등이 모두 이 패턴의 변형이다. 또한 이 패턴을 이해하면 컴파일러/인터프리터의 동작 원리를 파악하는 데 큰 도움이 된다.
