# Clean Architecture (클린 아키텍처)

> **대상 독자**: 2~3년차 개발자
> **핵심 키워드**: 의존성 규칙, 계층 분리, 유스케이스, 엔티티, 인터페이스 어댑터
> **난이도**: ★★★★☆ (중상급)

---

## 목차

1. [클린 아키텍처 소개](#1-클린-아키텍처-소개)
2. [핵심 원칙: 의존성 규칙](#2-핵심-원칙-의존성-규칙)
3. [4개 레이어 상세 설명](#3-4개-레이어-상세-설명)
4. [원형 다이어그램](#4-원형-다이어그램)
5. [Python 실전 예제: 사용자 관리 시스템](#5-python-실전-예제-사용자-관리-시스템)
6. [C# 실전 예제: 사용자 관리 시스템](#6-c-실전-예제-사용자-관리-시스템)
7. [프로젝트 폴더 구조](#7-프로젝트-폴더-구조)
8. [헥사고날 아키텍처와의 관계](#8-헥사고날-아키텍처와의-관계)
9. [Onion Architecture와의 비교](#9-onion-architecture와의-비교)
10. [장단점](#10-장단점)
11. [언제 사용해야 하는가?](#11-언제-사용해야-하는가)
12. [정리 및 체크리스트](#12-정리-및-체크리스트)
13. [참고 자료](#13-참고-자료)

---

## 1. 클린 아키텍처 소개

### Robert C. Martin (Uncle Bob)

클린 아키텍처는 2012년 Robert C. Martin(Uncle Bob)이 자신의 블로그에서 처음 소개하고, 이후 "Clean Architecture: A Craftsman's Guide to Software Structure and Design" (2017) 책으로 정리한 소프트웨어 아키텍처 원칙입니다.

Uncle Bob은 여러 아키텍처 패턴들(Hexagonal Architecture, Onion Architecture, Screaming Architecture, DCI, BCE)의 **공통적인 핵심 아이디어를 하나로 통합**하여 클린 아키텍처를 제안했습니다.

### 핵심 목표

```
┌─────────────────────────────────────────────────────────────────┐
│                   클린 아키텍처의 핵심 목표                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 프레임워크 독립성 (Framework Independence)                    │
│     → 프레임워크에 의존하지 않음. 프레임워크는 도구일 뿐.           │
│     → Django를 Flask로 바꿔도 핵심 로직은 변하지 않아야 함         │
│                                                                 │
│  2. 테스트 용이성 (Testability)                                  │
│     → 비즈니스 규칙을 UI, DB, 웹 서버 없이 테스트 가능             │
│                                                                 │
│  3. UI 독립성 (UI Independence)                                  │
│     → UI를 쉽게 교체 가능. 비즈니스 규칙은 UI를 모름               │
│     → 콘솔 앱 → 웹 앱으로 전환해도 핵심 로직 불변                  │
│                                                                 │
│  4. 데이터베이스 독립성 (Database Independence)                   │
│     → Oracle → PostgreSQL, SQL → NoSQL 교체 가능                │
│     → 비즈니스 규칙은 DB에 묶이지 않음                            │
│                                                                 │
│  5. 외부 에이전시 독립성 (External Agency Independence)           │
│     → 비즈니스 규칙은 외부 세계에 대해 아무것도 모름               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 탄생 배경: 왜 필요한가?

많은 프로젝트에서 겪는 문제:

```
[ 아키텍처 없는 프로젝트의 현실 ]

  시간 ──────────────────────────────────────►

  처음:    "빠르게 개발하자!"
           Controller에 모든 로직 → 동작함!

  6개월:   "기능 추가가 점점 어려워진다..."
           Controller가 500줄 → 스파게티 코드

  1년:     "DB를 바꿔야 하는데 모든 곳에 SQL이..."
           비즈니스 로직과 DB 코드가 뒤섞임

  2년:     "리팩토링하기엔 너무 위험하다..."
           테스트 없음. 어디를 건드리면 어디가 깨질지 모름

  결론:     처음부터 구조를 잘 잡았어야 했다!
```

---

## 2. 핵심 원칙: 의존성 규칙

### The Dependency Rule (의존성 규칙)

클린 아키텍처의 **가장 중요한 단 하나의 규칙**:

> **"소스 코드의 의존성은 반드시 안쪽(상위 수준 정책)을 향해야 한다."**

```
┌─────────────────────────────────────────────────────────────────┐
│                      의존성 규칙 시각화                           │
│                                                                 │
│                     의존 방향 (바깥 → 안쪽)                      │
│                     ════════════════════                         │
│                                                                 │
│    바깥쪽 (저수준)           →→→           안쪽 (고수준)          │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │ Frameworks   │───►│  Interface   │───►│  Use Cases   │──┐    │
│  │ & Drivers    │    │  Adapters    │    │              │  │    │
│  │              │    │              │    │              │  │    │
│  │ DB, Web, UI  │    │ Controllers  │    │ Application  │  │    │
│  │ 외부 라이브러리│    │ Presenters   │    │ Business     │  │    │
│  └──────────────┘    │ Gateways     │    │ Rules        │  │    │
│                      └──────────────┘    └──────────────┘  │    │
│                                                    │       │    │
│                                                    ▼       │    │
│                                          ┌──────────────┐  │    │
│                                          │  Entities    │◄─┘    │
│                                          │              │       │
│                                          │ Enterprise   │       │
│                                          │ Business     │       │
│                                          │ Rules        │       │
│                                          └──────────────┘       │
│                                                                 │
│  [규칙] 안쪽 원은 바깥쪽 원에 대해 아무것도 알지 못합니다.           │
│         이름, 함수, 클래스 등 어떤 것도 참조해서는 안 됩니다.        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 의존성 역전으로 규칙 지키기

안쪽 레이어가 바깥쪽 레이어의 기능을 사용해야 할 때, **인터페이스(추상화)를 활용**합니다.

```
[ 의존성 역전 원칙 (DIP) 적용 ]

  잘못된 의존:
  ┌──────────┐        ┌──────────────┐
  │ UseCase  │───────►│ MySQL        │    UseCase가 구체적인 DB에 의존
  │          │        │ Repository   │    → DB를 바꾸면 UseCase도 수정
  └──────────┘        └──────────────┘

  올바른 의존 (DIP 적용):
  ┌──────────┐        ┌──────────────────┐
  │ UseCase  │───────►│ IUserRepository  │    UseCase는 인터페이스에만 의존
  │          │        │ (interface)      │
  └──────────┘        └──────────────────┘
                              ▲
                              │ 구현 (implements)
                      ┌──────────────────┐
                      │ MySQLUserRepo    │    구현체는 바깥 레이어에 위치
                      │ (infrastructure) │    → DB를 바꾸면 구현체만 교체
                      └──────────────────┘
```

---

## 3. 4개 레이어 상세 설명

### 3-1. Entities (엔티티) - 가장 안쪽

**핵심 비즈니스 규칙**을 캡슐화합니다. 엔터프라이즈 전체에서 공유되는 가장 일반적인 비즈니스 규칙을 담습니다.

```
┌────────────────────────────────────────────────┐
│  Entities (엔티티 레이어)                        │
│                                                │
│  특징:                                          │
│  - 가장 변하지 않는 핵심 비즈니스 규칙             │
│  - 외부 변화에 영향받지 않음                      │
│  - 어떤 프레임워크에도 의존하지 않음               │
│  - 여러 애플리케이션에서 재사용 가능               │
│                                                │
│  포함하는 것:                                    │
│  - 엔티티 객체 (비즈니스 데이터 + 규칙)           │
│  - Value Object (값 객체)                       │
│  - 도메인 이벤트                                 │
│  - 비즈니스 예외                                 │
│                                                │
│  예시:                                          │
│  - "사용자의 이메일은 유효한 형식이어야 한다"       │
│  - "주문 금액은 0보다 커야 한다"                  │
│  - "비밀번호는 8자 이상이어야 한다"               │
└────────────────────────────────────────────────┘
```

### 3-2. Use Cases (유스케이스) - 두 번째 레이어

**애플리케이션 고유의 비즈니스 규칙**을 담습니다. 시스템의 모든 유스케이스를 캡슐화하고 구현합니다.

```
┌────────────────────────────────────────────────┐
│  Use Cases (유스케이스 레이어)                    │
│                                                │
│  특징:                                          │
│  - 애플리케이션별 비즈니스 규칙                   │
│  - 엔티티로/로부터 데이터 흐름을 조정              │
│  - 엔티티의 비즈니스 규칙을 호출                  │
│  - 입력/출력 포트(인터페이스) 정의                │
│                                                │
│  포함하는 것:                                    │
│  - 유스케이스 인터랙터 (서비스 클래스)             │
│  - 입력 DTO (Input Port)                        │
│  - 출력 DTO (Output Port)                       │
│  - 리포지토리 인터페이스 (Output Port)            │
│                                                │
│  예시:                                          │
│  - "사용자 회원가입" 유스케이스                    │
│  - "주문 생성" 유스케이스                         │
│  - "상품 검색" 유스케이스                         │
│                                                │
│  [주의] 유스케이스는 "어떻게" 저장하는지 모릅니다.  │
│         인터페이스만 알고, 구현은 바깥 레이어 담당    │
└────────────────────────────────────────────────┘
```

### 3-3. Interface Adapters (인터페이스 어댑터) - 세 번째 레이어

**데이터 형식을 변환**하는 어댑터 레이어입니다. 유스케이스/엔티티에 가장 편리한 형식에서 외부 시스템(DB, 웹 등)에 가장 편리한 형식으로 변환합니다.

```
┌────────────────────────────────────────────────┐
│  Interface Adapters (인터페이스 어댑터 레이어)     │
│                                                │
│  특징:                                          │
│  - 데이터 형식 변환의 책임                        │
│  - 안쪽 레이어와 바깥 레이어 사이의 다리 역할       │
│  - MVC 패턴의 대부분이 여기에 해당                │
│                                                │
│  포함하는 것:                                    │
│  - Controller: 외부 요청 → UseCase 입력 변환     │
│  - Presenter: UseCase 출력 → 외부 응답 변환      │
│  - Gateway: UseCase가 정의한 인터페이스의 구현     │
│  - Repository 구현체                             │
│  - DTO / ViewModel                              │
│                                                │
│  예시:                                          │
│  - HTTP 요청을 파싱하여 UseCase에 전달           │
│  - DB 결과를 엔티티 객체로 변환                   │
│  - 엔티티를 JSON 응답으로 변환                    │
└────────────────────────────────────────────────┘
```

### 3-4. Frameworks & Drivers (프레임워크 & 드라이버) - 가장 바깥쪽

**외부 도구와 프레임워크**로 구성됩니다. 가장 세부적인 구현 사항들이 위치합니다.

```
┌────────────────────────────────────────────────┐
│  Frameworks & Drivers (프레임워크 & 드라이버)     │
│                                                │
│  특징:                                          │
│  - 세부 구현 사항                                │
│  - 가장 변하기 쉬운 레이어                       │
│  - 안쪽 레이어에 영향을 주지 않고 교체 가능        │
│                                                │
│  포함하는 것:                                    │
│  - 웹 프레임워크 (Flask, ASP.NET, Django 등)     │
│  - 데이터베이스 (MySQL, PostgreSQL, MongoDB 등)  │
│  - UI 프레임워크                                 │
│  - 외부 API 클라이언트                           │
│  - 디바이스 드라이버                              │
│  - 설정 파일                                     │
│                                                │
│  [원칙] 이 레이어에는 가능한 적은 코드만 작성합니다.│
│         안쪽 레이어와 연결하는 "접착 코드"만 존재    │
└────────────────────────────────────────────────┘
```

---

## 4. 원형 다이어그램

### 클린 아키텍처 동심원 (텍스트 기반)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    Clean Architecture 동심원 다이어그램                   │
│                                                                         │
│    ┌───────────────────────────────────────────────────────────────┐    │
│    │  Frameworks & Drivers                                         │    │
│    │  (Web, DB, UI, Devices, External Interfaces)                  │    │
│    │                                                               │    │
│    │    ┌───────────────────────────────────────────────────┐      │    │
│    │    │  Interface Adapters                                │      │    │
│    │    │  (Controllers, Gateways, Presenters)               │      │    │
│    │    │                                                    │      │    │
│    │    │    ┌──────────────────────────────────────┐        │      │    │
│    │    │    │  Use Cases                            │        │      │    │
│    │    │    │  (Application Business Rules)         │        │      │    │
│    │    │    │                                       │        │      │    │
│    │    │    │    ┌────────────────────────┐          │        │      │    │
│    │    │    │    │                        │          │        │      │    │
│    │    │    │    │      Entities          │          │        │      │    │
│    │    │    │    │  (Enterprise Business  │          │        │      │    │
│    │    │    │    │       Rules)           │          │        │      │    │
│    │    │    │    │                        │          │        │      │    │
│    │    │    │    └────────────────────────┘          │        │      │    │
│    │    │    │                                       │        │      │    │
│    │    │    └──────────────────────────────────────┘        │      │    │
│    │    │                                                    │      │    │
│    │    └───────────────────────────────────────────────────┘      │    │
│    │                                                               │    │
│    └───────────────────────────────────────────────────────────────┘    │
│                                                                         │
│              ══════════════════════════════════                          │
│              의존성 방향:  바깥쪽 ───────► 안쪽                           │
│              ══════════════════════════════════                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 데이터 흐름 다이어그램

```
┌─────────────────────────────────────────────────────────────────────┐
│                     데이터 흐름 (일반적인 요청 처리)                   │
│                                                                     │
│  HTTP 요청                                                          │
│      │                                                              │
│      ▼                                                              │
│  ┌──────────┐     ┌───────────┐     ┌──────────┐     ┌──────────┐  │
│  │ Web      │────►│Controller │────►│ UseCase  │────►│ Entity   │  │
│  │ Framework│     │(Adapter)  │     │(Interac- │     │(Business │  │
│  │          │     │           │     │ tor)     │     │ Rules)   │  │
│  └──────────┘     └───────────┘     └──────────┘     └──────────┘  │
│                                          │                │         │
│                                          │ Repository     │ 검증    │
│                                          │ 인터페이스     │ 결과    │
│                                          ▼                │         │
│                                    ┌───────────┐          │         │
│                                    │ Repository│◄─────────┘         │
│                                    │ 구현체    │                     │
│                                    │ (Adapter) │                    │
│                                    └─────┬─────┘                    │
│                                          │                          │
│                                          ▼                          │
│                                    ┌───────────┐                    │
│                                    │ Database  │                    │
│                                    │ (Driver)  │                    │
│                                    └───────────┘                    │
│                                                                     │
│  응답 흐름은 역방향으로 진행됩니다:                                    │
│  Entity → UseCase → Presenter/Controller → Web Framework → 클라이언트│
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Python 실전 예제: 사용자 관리 시스템

### 5-1. Entities 레이어

```python
# domain/entities/user.py
"""
엔티티 레이어 - 핵심 비즈니스 규칙
어떤 외부 라이브러리에도 의존하지 않는 순수 Python 코드
"""
import re
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional


@dataclass
class User:
    """사용자 엔티티 - 핵심 비즈니스 규칙을 포함"""
    id: Optional[int] = None
    username: str = ""
    email: str = ""
    password_hash: str = ""
    is_active: bool = True
    created_at: datetime = field(default_factory=datetime.now)

    def validate_email(self) -> bool:
        """이메일 형식 검증 - 엔터프라이즈 비즈니스 규칙"""
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, self.email))

    def validate_username(self) -> bool:
        """사용자명 검증 - 엔터프라이즈 비즈니스 규칙"""
        if not self.username or len(self.username) < 3:
            return False
        if len(self.username) > 50:
            return False
        return bool(re.match(r'^[a-zA-Z0-9_]+$', self.username))

    def deactivate(self) -> None:
        """계정 비활성화 - 비즈니스 규칙"""
        self.is_active = False

    def activate(self) -> None:
        """계정 활성화 - 비즈니스 규칙"""
        self.is_active = True


# domain/entities/exceptions.py
"""도메인 예외 - 비즈니스 규칙 위반 시 발생"""


class DomainException(Exception):
    """도메인 예외 기본 클래스"""
    pass


class InvalidEmailError(DomainException):
    """유효하지 않은 이메일"""
    def __init__(self, email: str):
        super().__init__(f"유효하지 않은 이메일 형식입니다: {email}")


class InvalidUsernameError(DomainException):
    """유효하지 않은 사용자명"""
    def __init__(self, username: str):
        super().__init__(f"유효하지 않은 사용자명입니다: {username}")


class UserAlreadyExistsError(DomainException):
    """이미 존재하는 사용자"""
    def __init__(self, identifier: str):
        super().__init__(f"이미 존재하는 사용자입니다: {identifier}")


class UserNotFoundError(DomainException):
    """사용자를 찾을 수 없음"""
    def __init__(self, identifier: str):
        super().__init__(f"사용자를 찾을 수 없습니다: {identifier}")
```

### 5-2. Use Cases 레이어

```python
# domain/use_cases/ports.py
"""
유스케이스 레이어 - 포트(인터페이스) 정의
바깥 레이어가 구현해야 할 인터페이스를 정의합니다.
"""
from abc import ABC, abstractmethod
from typing import List, Optional
from domain.entities.user import User


class UserRepositoryPort(ABC):
    """사용자 저장소 포트 (Output Port)
    유스케이스가 데이터 접근에 사용하는 인터페이스.
    실제 구현은 인프라 레이어에서 합니다.
    """

    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional[User]:
        pass

    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass

    @abstractmethod
    def find_by_username(self, username: str) -> Optional[User]:
        pass

    @abstractmethod
    def save(self, user: User) -> User:
        pass

    @abstractmethod
    def delete(self, user_id: int) -> None:
        pass

    @abstractmethod
    def find_all(self) -> List[User]:
        pass


class PasswordHasherPort(ABC):
    """비밀번호 해시 포트 (Output Port)"""

    @abstractmethod
    def hash(self, password: str) -> str:
        pass

    @abstractmethod
    def verify(self, password: str, password_hash: str) -> bool:
        pass


# domain/use_cases/dtos.py
"""유스케이스 입출력 DTO"""
from dataclasses import dataclass
from datetime import datetime
from typing import Optional


@dataclass
class CreateUserRequest:
    """사용자 생성 요청 DTO (Input)"""
    username: str
    email: str
    password: str


@dataclass
class UserResponse:
    """사용자 응답 DTO (Output)"""
    id: int
    username: str
    email: str
    is_active: bool
    created_at: datetime


@dataclass
class UpdateUserRequest:
    """사용자 수정 요청 DTO (Input)"""
    user_id: int
    username: Optional[str] = None
    email: Optional[str] = None


# domain/use_cases/create_user.py
"""사용자 생성 유스케이스"""
from domain.entities.user import User
from domain.entities.exceptions import (
    InvalidEmailError, InvalidUsernameError, UserAlreadyExistsError
)
from domain.use_cases.ports import UserRepositoryPort, PasswordHasherPort
from domain.use_cases.dtos import CreateUserRequest, UserResponse


class CreateUserUseCase:
    """
    사용자 생성 유스케이스 (Interactor)
    애플리케이션 비즈니스 규칙을 구현합니다.
    """

    def __init__(
        self,
        user_repository: UserRepositoryPort,
        password_hasher: PasswordHasherPort
    ):
        # 의존성 주입: 인터페이스(포트)에만 의존
        self._user_repo = user_repository
        self._password_hasher = password_hasher

    def execute(self, request: CreateUserRequest) -> UserResponse:
        """사용자 생성 실행"""

        # 1. 엔티티 생성 및 비즈니스 규칙 검증
        user = User(
            username=request.username,
            email=request.email
        )

        if not user.validate_email():
            raise InvalidEmailError(request.email)

        if not user.validate_username():
            raise InvalidUsernameError(request.username)

        # 2. 애플리케이션 비즈니스 규칙: 중복 검사
        existing = self._user_repo.find_by_email(request.email)
        if existing:
            raise UserAlreadyExistsError(request.email)

        existing = self._user_repo.find_by_username(request.username)
        if existing:
            raise UserAlreadyExistsError(request.username)

        # 3. 비밀번호 해싱 (구현은 외부 레이어)
        user.password_hash = self._password_hasher.hash(request.password)

        # 4. 저장 (구현은 외부 레이어)
        saved_user = self._user_repo.save(user)

        # 5. 응답 DTO 반환
        return UserResponse(
            id=saved_user.id,
            username=saved_user.username,
            email=saved_user.email,
            is_active=saved_user.is_active,
            created_at=saved_user.created_at
        )


# domain/use_cases/get_user.py
"""사용자 조회 유스케이스"""
from domain.entities.exceptions import UserNotFoundError
from domain.use_cases.ports import UserRepositoryPort
from domain.use_cases.dtos import UserResponse


class GetUserUseCase:
    """사용자 조회 유스케이스"""

    def __init__(self, user_repository: UserRepositoryPort):
        self._user_repo = user_repository

    def execute(self, user_id: int) -> UserResponse:
        """ID로 사용자 조회"""
        user = self._user_repo.find_by_id(user_id)
        if not user:
            raise UserNotFoundError(str(user_id))

        return UserResponse(
            id=user.id,
            username=user.username,
            email=user.email,
            is_active=user.is_active,
            created_at=user.created_at
        )


class GetAllUsersUseCase:
    """전체 사용자 목록 조회 유스케이스"""

    def __init__(self, user_repository: UserRepositoryPort):
        self._user_repo = user_repository

    def execute(self) -> list[UserResponse]:
        """전체 사용자 목록 반환"""
        users = self._user_repo.find_all()
        return [
            UserResponse(
                id=u.id,
                username=u.username,
                email=u.email,
                is_active=u.is_active,
                created_at=u.created_at
            )
            for u in users
        ]
```

### 5-3. Interface Adapters 레이어

```python
# adapters/repositories/in_memory_user_repository.py
"""
인터페이스 어댑터 레이어 - 리포지토리 구현체
유스케이스 레이어의 인터페이스(Port)를 구현합니다.
"""
from typing import List, Optional
from domain.entities.user import User
from domain.use_cases.ports import UserRepositoryPort


class InMemoryUserRepository(UserRepositoryPort):
    """인메모리 사용자 저장소 구현체"""

    def __init__(self):
        self._users: dict[int, User] = {}
        self._next_id = 1

    def find_by_id(self, user_id: int) -> Optional[User]:
        return self._users.get(user_id)

    def find_by_email(self, email: str) -> Optional[User]:
        for user in self._users.values():
            if user.email == email:
                return user
        return None

    def find_by_username(self, username: str) -> Optional[User]:
        for user in self._users.values():
            if user.username == username:
                return user
        return None

    def save(self, user: User) -> User:
        if user.id is None:
            user.id = self._next_id
            self._next_id += 1
        self._users[user.id] = user
        return user

    def delete(self, user_id: int) -> None:
        self._users.pop(user_id, None)

    def find_all(self) -> List[User]:
        return list(self._users.values())


# adapters/security/bcrypt_password_hasher.py
"""비밀번호 해시 어댑터"""
import hashlib
from domain.use_cases.ports import PasswordHasherPort


class SimplePasswordHasher(PasswordHasherPort):
    """간단한 비밀번호 해시 구현 (예제용)
    실무에서는 bcrypt, argon2 등을 사용하세요.
    """

    def hash(self, password: str) -> str:
        return hashlib.sha256(password.encode()).hexdigest()

    def verify(self, password: str, password_hash: str) -> bool:
        return self.hash(password) == password_hash


# adapters/controllers/user_controller.py
"""
사용자 컨트롤러 - HTTP 요청을 유스케이스 입력으로 변환
프레임워크에 독립적인 컨트롤러
"""
from domain.use_cases.create_user import CreateUserUseCase
from domain.use_cases.get_user import GetUserUseCase, GetAllUsersUseCase
from domain.use_cases.dtos import CreateUserRequest, UserResponse
from domain.entities.exceptions import DomainException


class UserController:
    """사용자 관련 요청을 처리하는 컨트롤러"""

    def __init__(
        self,
        create_user_use_case: CreateUserUseCase,
        get_user_use_case: GetUserUseCase,
        get_all_users_use_case: GetAllUsersUseCase
    ):
        self._create_user = create_user_use_case
        self._get_user = get_user_use_case
        self._get_all_users = get_all_users_use_case

    def create_user(self, data: dict) -> dict:
        """사용자 생성 요청 처리"""
        try:
            request = CreateUserRequest(
                username=data.get("username", ""),
                email=data.get("email", ""),
                password=data.get("password", "")
            )
            response = self._create_user.execute(request)
            return {
                "success": True,
                "data": self._to_dict(response)
            }
        except DomainException as e:
            return {
                "success": False,
                "error": str(e)
            }

    def get_user(self, user_id: int) -> dict:
        """사용자 조회 요청 처리"""
        try:
            response = self._get_user.execute(user_id)
            return {
                "success": True,
                "data": self._to_dict(response)
            }
        except DomainException as e:
            return {
                "success": False,
                "error": str(e)
            }

    def get_all_users(self) -> dict:
        """전체 사용자 목록 요청 처리"""
        responses = self._get_all_users.execute()
        return {
            "success": True,
            "data": [self._to_dict(r) for r in responses]
        }

    @staticmethod
    def _to_dict(response: UserResponse) -> dict:
        """응답 DTO를 딕셔너리로 변환"""
        return {
            "id": response.id,
            "username": response.username,
            "email": response.email,
            "is_active": response.is_active,
            "created_at": response.created_at.isoformat()
        }
```

### 5-4. Frameworks & Drivers 레이어

```python
# frameworks/web/flask_app.py
"""
프레임워크 & 드라이버 레이어 - Flask 웹 서버
가장 바깥쪽 레이어로, 최소한의 접착 코드만 포함합니다.
"""
from flask import Flask, request, jsonify

from domain.use_cases.create_user import CreateUserUseCase
from domain.use_cases.get_user import GetUserUseCase, GetAllUsersUseCase
from adapters.repositories.in_memory_user_repository import InMemoryUserRepository
from adapters.security.bcrypt_password_hasher import SimplePasswordHasher
from adapters.controllers.user_controller import UserController


def create_app() -> Flask:
    """Flask 앱 팩토리 - 의존성 조립"""
    app = Flask(__name__)

    # ── 의존성 조립 (Composition Root) ──
    # 바깥 레이어에서 안쪽 레이어의 의존성을 조립합니다
    user_repository = InMemoryUserRepository()
    password_hasher = SimplePasswordHasher()

    create_user_uc = CreateUserUseCase(user_repository, password_hasher)
    get_user_uc = GetUserUseCase(user_repository)
    get_all_users_uc = GetAllUsersUseCase(user_repository)

    controller = UserController(create_user_uc, get_user_uc, get_all_users_uc)

    # ── 라우트 정의 (최소한의 접착 코드) ──
    @app.route("/users", methods=["POST"])
    def create_user():
        data = request.get_json()
        result = controller.create_user(data)
        status_code = 201 if result["success"] else 400
        return jsonify(result), status_code

    @app.route("/users/<int:user_id>", methods=["GET"])
    def get_user(user_id):
        result = controller.get_user(user_id)
        status_code = 200 if result["success"] else 404
        return jsonify(result), status_code

    @app.route("/users", methods=["GET"])
    def get_all_users():
        result = controller.get_all_users()
        return jsonify(result), 200

    return app


if __name__ == "__main__":
    app = create_app()
    app.run(debug=True, port=5000)
```

---

## 6. C# 실전 예제: 사용자 관리 시스템

### 6-1. Entities 레이어

```csharp
// Domain/Entities/User.cs
namespace UserManagement.Domain.Entities
{
    /// <summary>
    /// 사용자 엔티티 - 핵심 비즈니스 규칙
    /// 어떤 프레임워크에도 의존하지 않는 순수 C# 클래스
    /// </summary>
    public class User
    {
        public int? Id { get; set; }
        public string Username { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string PasswordHash { get; set; } = string.Empty;
        public bool IsActive { get; set; } = true;
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

        /// <summary>이메일 형식 검증 - 엔터프라이즈 비즈니스 규칙</summary>
        public bool ValidateEmail()
        {
            if (string.IsNullOrWhiteSpace(Email)) return false;
            var pattern = @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
            return System.Text.RegularExpressions.Regex.IsMatch(Email, pattern);
        }

        /// <summary>사용자명 검증 - 엔터프라이즈 비즈니스 규칙</summary>
        public bool ValidateUsername()
        {
            if (string.IsNullOrWhiteSpace(Username)) return false;
            if (Username.Length < 3 || Username.Length > 50) return false;
            return System.Text.RegularExpressions.Regex.IsMatch(
                Username, @"^[a-zA-Z0-9_]+$");
        }

        public void Deactivate() => IsActive = false;
        public void Activate() => IsActive = true;
    }
}

// Domain/Exceptions/DomainException.cs
namespace UserManagement.Domain.Exceptions
{
    public class DomainException : Exception
    {
        public DomainException(string message) : base(message) { }
    }

    public class InvalidEmailException : DomainException
    {
        public InvalidEmailException(string email)
            : base($"유효하지 않은 이메일 형식입니다: {email}") { }
    }

    public class InvalidUsernameException : DomainException
    {
        public InvalidUsernameException(string username)
            : base($"유효하지 않은 사용자명입니다: {username}") { }
    }

    public class UserAlreadyExistsException : DomainException
    {
        public UserAlreadyExistsException(string identifier)
            : base($"이미 존재하는 사용자입니다: {identifier}") { }
    }

    public class UserNotFoundException : DomainException
    {
        public UserNotFoundException(string identifier)
            : base($"사용자를 찾을 수 없습니다: {identifier}") { }
    }
}
```

### 6-2. Use Cases 레이어

```csharp
// Application/Ports/IUserRepository.cs
namespace UserManagement.Application.Ports
{
    /// <summary>
    /// 사용자 저장소 포트 (Output Port)
    /// 유스케이스 레이어에서 정의하고, 인프라 레이어에서 구현합니다.
    /// </summary>
    public interface IUserRepository
    {
        Task<User?> FindByIdAsync(int userId);
        Task<User?> FindByEmailAsync(string email);
        Task<User?> FindByUsernameAsync(string username);
        Task<User> SaveAsync(User user);
        Task DeleteAsync(int userId);
        Task<IReadOnlyList<User>> FindAllAsync();
    }
}

// Application/Ports/IPasswordHasher.cs
namespace UserManagement.Application.Ports
{
    /// <summary>비밀번호 해시 포트</summary>
    public interface IPasswordHasher
    {
        string Hash(string password);
        bool Verify(string password, string hash);
    }
}

// Application/DTOs/CreateUserRequest.cs
namespace UserManagement.Application.DTOs
{
    public record CreateUserRequest(
        string Username,
        string Email,
        string Password
    );

    public record UserResponse(
        int Id,
        string Username,
        string Email,
        bool IsActive,
        DateTime CreatedAt
    );

    public record UpdateUserRequest(
        int UserId,
        string? Username = null,
        string? Email = null
    );
}

// Application/UseCases/CreateUserUseCase.cs
using UserManagement.Domain.Entities;
using UserManagement.Domain.Exceptions;
using UserManagement.Application.Ports;
using UserManagement.Application.DTOs;

namespace UserManagement.Application.UseCases
{
    /// <summary>
    /// 사용자 생성 유스케이스 (Interactor)
    /// 애플리케이션 비즈니스 규칙을 구현합니다.
    /// </summary>
    public class CreateUserUseCase
    {
        private readonly IUserRepository _userRepository;
        private readonly IPasswordHasher _passwordHasher;

        public CreateUserUseCase(
            IUserRepository userRepository,
            IPasswordHasher passwordHasher)
        {
            _userRepository = userRepository;
            _passwordHasher = passwordHasher;
        }

        public async Task<UserResponse> ExecuteAsync(CreateUserRequest request)
        {
            // 1. 엔티티 생성 및 비즈니스 규칙 검증
            var user = new User
            {
                Username = request.Username,
                Email = request.Email
            };

            if (!user.ValidateEmail())
                throw new InvalidEmailException(request.Email);

            if (!user.ValidateUsername())
                throw new InvalidUsernameException(request.Username);

            // 2. 애플리케이션 규칙: 중복 검사
            var existingByEmail = await _userRepository.FindByEmailAsync(request.Email);
            if (existingByEmail != null)
                throw new UserAlreadyExistsException(request.Email);

            var existingByUsername = await _userRepository.FindByUsernameAsync(request.Username);
            if (existingByUsername != null)
                throw new UserAlreadyExistsException(request.Username);

            // 3. 비밀번호 해싱
            user.PasswordHash = _passwordHasher.Hash(request.Password);

            // 4. 저장
            var savedUser = await _userRepository.SaveAsync(user);

            // 5. 응답 DTO 반환
            return new UserResponse(
                Id: savedUser.Id ?? 0,
                Username: savedUser.Username,
                Email: savedUser.Email,
                IsActive: savedUser.IsActive,
                CreatedAt: savedUser.CreatedAt
            );
        }
    }
}

// Application/UseCases/GetUserUseCase.cs
namespace UserManagement.Application.UseCases
{
    public class GetUserUseCase
    {
        private readonly IUserRepository _userRepository;

        public GetUserUseCase(IUserRepository userRepository)
        {
            _userRepository = userRepository;
        }

        public async Task<UserResponse> ExecuteAsync(int userId)
        {
            var user = await _userRepository.FindByIdAsync(userId);
            if (user == null)
                throw new UserNotFoundException(userId.ToString());

            return new UserResponse(
                Id: user.Id ?? 0,
                Username: user.Username,
                Email: user.Email,
                IsActive: user.IsActive,
                CreatedAt: user.CreatedAt
            );
        }
    }

    public class GetAllUsersUseCase
    {
        private readonly IUserRepository _userRepository;

        public GetAllUsersUseCase(IUserRepository userRepository)
        {
            _userRepository = userRepository;
        }

        public async Task<IReadOnlyList<UserResponse>> ExecuteAsync()
        {
            var users = await _userRepository.FindAllAsync();
            return users.Select(u => new UserResponse(
                Id: u.Id ?? 0,
                Username: u.Username,
                Email: u.Email,
                IsActive: u.IsActive,
                CreatedAt: u.CreatedAt
            )).ToList().AsReadOnly();
        }
    }
}
```

### 6-3. Interface Adapters 레이어

```csharp
// Infrastructure/Repositories/InMemoryUserRepository.cs
using UserManagement.Application.Ports;
using UserManagement.Domain.Entities;

namespace UserManagement.Infrastructure.Repositories
{
    /// <summary>
    /// 인메모리 사용자 저장소 - IUserRepository 구현체
    /// 유스케이스 레이어의 인터페이스를 구현합니다.
    /// </summary>
    public class InMemoryUserRepository : IUserRepository
    {
        private readonly Dictionary<int, User> _users = new();
        private int _nextId = 1;

        public Task<User?> FindByIdAsync(int userId)
        {
            _users.TryGetValue(userId, out var user);
            return Task.FromResult(user);
        }

        public Task<User?> FindByEmailAsync(string email)
        {
            var user = _users.Values.FirstOrDefault(u => u.Email == email);
            return Task.FromResult(user);
        }

        public Task<User?> FindByUsernameAsync(string username)
        {
            var user = _users.Values.FirstOrDefault(u => u.Username == username);
            return Task.FromResult(user);
        }

        public Task<User> SaveAsync(User user)
        {
            if (user.Id == null)
            {
                user.Id = _nextId++;
            }
            _users[user.Id.Value] = user;
            return Task.FromResult(user);
        }

        public Task DeleteAsync(int userId)
        {
            _users.Remove(userId);
            return Task.CompletedTask;
        }

        public Task<IReadOnlyList<User>> FindAllAsync()
        {
            IReadOnlyList<User> users = _users.Values.ToList().AsReadOnly();
            return Task.FromResult(users);
        }
    }
}

// Infrastructure/Security/Sha256PasswordHasher.cs
using System.Security.Cryptography;
using System.Text;
using UserManagement.Application.Ports;

namespace UserManagement.Infrastructure.Security
{
    /// <summary>비밀번호 해시 구현 (예제용)</summary>
    public class Sha256PasswordHasher : IPasswordHasher
    {
        public string Hash(string password)
        {
            var bytes = SHA256.HashData(Encoding.UTF8.GetBytes(password));
            return Convert.ToHexString(bytes).ToLower();
        }

        public bool Verify(string password, string hash)
        {
            return Hash(password) == hash;
        }
    }
}
```

### 6-4. Frameworks & Drivers 레이어

```csharp
// WebApi/Controllers/UsersController.cs
using Microsoft.AspNetCore.Mvc;
using UserManagement.Application.UseCases;
using UserManagement.Application.DTOs;
using UserManagement.Domain.Exceptions;

namespace UserManagement.WebApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UsersController : ControllerBase
    {
        private readonly CreateUserUseCase _createUser;
        private readonly GetUserUseCase _getUser;
        private readonly GetAllUsersUseCase _getAllUsers;

        public UsersController(
            CreateUserUseCase createUser,
            GetUserUseCase getUser,
            GetAllUsersUseCase getAllUsers)
        {
            _createUser = createUser;
            _getUser = getUser;
            _getAllUsers = getAllUsers;
        }

        [HttpPost]
        public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
        {
            try
            {
                var response = await _createUser.ExecuteAsync(request);
                return CreatedAtAction(
                    nameof(GetUser),
                    new { id = response.Id },
                    response);
            }
            catch (DomainException ex)
            {
                return BadRequest(new { error = ex.Message });
            }
        }

        [HttpGet("{id}")]
        public async Task<IActionResult> GetUser(int id)
        {
            try
            {
                var response = await _getUser.ExecuteAsync(id);
                return Ok(response);
            }
            catch (UserNotFoundException)
            {
                return NotFound();
            }
        }

        [HttpGet]
        public async Task<IActionResult> GetAllUsers()
        {
            var response = await _getAllUsers.ExecuteAsync();
            return Ok(response);
        }
    }
}

// WebApi/Program.cs - 의존성 조립 (Composition Root)
using UserManagement.Application.Ports;
using UserManagement.Application.UseCases;
using UserManagement.Infrastructure.Repositories;
using UserManagement.Infrastructure.Security;

var builder = WebApplication.CreateBuilder(args);

// ── 의존성 등록 ──
// Infrastructure (바깥 레이어)의 구현체를 Application (안쪽 레이어)의 인터페이스에 등록
builder.Services.AddSingleton<IUserRepository, InMemoryUserRepository>();
builder.Services.AddSingleton<IPasswordHasher, Sha256PasswordHasher>();

// Use Cases 등록
builder.Services.AddScoped<CreateUserUseCase>();
builder.Services.AddScoped<GetUserUseCase>();
builder.Services.AddScoped<GetAllUsersUseCase>();

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();
app.Run();
```

---

## 7. 프로젝트 폴더 구조

### Python 프로젝트 구조

```
user-management/
│
├── domain/                          # 안쪽 레이어 (Entities + Use Cases)
│   ├── __init__.py
│   ├── entities/                    # 엔티티 레이어
│   │   ├── __init__.py
│   │   ├── user.py                  # User 엔티티
│   │   └── exceptions.py           # 도메인 예외
│   │
│   └── use_cases/                   # 유스케이스 레이어
│       ├── __init__.py
│       ├── ports.py                 # 포트(인터페이스) 정의
│       ├── dtos.py                  # 입출력 DTO
│       ├── create_user.py           # 사용자 생성 유스케이스
│       ├── get_user.py              # 사용자 조회 유스케이스
│       ├── update_user.py           # 사용자 수정 유스케이스
│       └── delete_user.py           # 사용자 삭제 유스케이스
│
├── adapters/                        # 인터페이스 어댑터 레이어
│   ├── __init__.py
│   ├── repositories/                # 저장소 구현
│   │   ├── __init__.py
│   │   ├── in_memory_user_repository.py
│   │   └── sqlalchemy_user_repository.py
│   ├── security/                    # 보안 구현
│   │   ├── __init__.py
│   │   └── bcrypt_password_hasher.py
│   └── controllers/                 # 컨트롤러
│       ├── __init__.py
│       └── user_controller.py
│
├── frameworks/                      # 프레임워크 & 드라이버 레이어
│   ├── __init__.py
│   └── web/
│       ├── __init__.py
│       └── flask_app.py             # Flask 앱 (접착 코드)
│
├── tests/                           # 테스트
│   ├── unit/
│   │   ├── test_user_entity.py
│   │   ├── test_create_user_uc.py
│   │   └── test_get_user_uc.py
│   └── integration/
│       └── test_flask_api.py
│
├── requirements.txt
└── main.py                          # 진입점
```

### C# 프로젝트 구조 (솔루션)

```
UserManagement.sln
│
├── src/
│   ├── UserManagement.Domain/           # 엔티티 레이어 (프로젝트)
│   │   ├── Entities/
│   │   │   └── User.cs
│   │   ├── Exceptions/
│   │   │   └── DomainException.cs
│   │   └── UserManagement.Domain.csproj  # 의존성: 없음!
│   │
│   ├── UserManagement.Application/      # 유스케이스 레이어 (프로젝트)
│   │   ├── Ports/
│   │   │   ├── IUserRepository.cs
│   │   │   └── IPasswordHasher.cs
│   │   ├── DTOs/
│   │   │   ├── CreateUserRequest.cs
│   │   │   └── UserResponse.cs
│   │   ├── UseCases/
│   │   │   ├── CreateUserUseCase.cs
│   │   │   ├── GetUserUseCase.cs
│   │   │   └── GetAllUsersUseCase.cs
│   │   └── UserManagement.Application.csproj
│   │       # 의존성: Domain만 참조
│   │
│   ├── UserManagement.Infrastructure/   # 인터페이스 어댑터 (프로젝트)
│   │   ├── Repositories/
│   │   │   ├── InMemoryUserRepository.cs
│   │   │   └── EfCoreUserRepository.cs
│   │   ├── Security/
│   │   │   └── Sha256PasswordHasher.cs
│   │   └── UserManagement.Infrastructure.csproj
│   │       # 의존성: Application 참조 (Domain은 간접 참조)
│   │
│   └── UserManagement.WebApi/           # 프레임워크 & 드라이버 (프로젝트)
│       ├── Controllers/
│       │   └── UsersController.cs
│       ├── Program.cs                    # Composition Root
│       └── UserManagement.WebApi.csproj
│           # 의존성: Application + Infrastructure 참조
│
└── tests/
    ├── UserManagement.Domain.Tests/
    ├── UserManagement.Application.Tests/
    └── UserManagement.WebApi.Tests/
```

### 프로젝트 참조(의존성) 구조

```
┌────────────────────────────────────────────────────────┐
│               C# 프로젝트 참조 관계                      │
│                                                        │
│  WebApi ──────► Application ──────► Domain              │
│    │                                   ▲               │
│    │                                   │               │
│    └──────► Infrastructure ────────────┘               │
│                   │                                    │
│                   └──────► Application (간접)           │
│                                                        │
│  [핵심] Domain 프로젝트는 아무것도 참조하지 않습니다.      │
│         Application은 Domain만 참조합니다.              │
│         Infrastructure는 Application을 참조하여         │
│         인터페이스를 구현합니다.                          │
│         WebApi는 모든 것을 알고 조립합니다.               │
│         (Composition Root)                             │
└────────────────────────────────────────────────────────┘
```

---

## 8. 헥사고날 아키텍처와의 관계

### 헥사고날 아키텍처 (Ports and Adapters)

헥사고날 아키텍처는 Alistair Cockburn이 제안한 패턴으로, 클린 아키텍처의 핵심 영감 중 하나입니다.

```
┌──────────────────────────────────────────────────────────────┐
│                  헥사고날 아키텍처 다이어그램                    │
│                                                              │
│                   Driving Adapters                            │
│                   (입력 측)                                   │
│              ┌────────────────────┐                           │
│              │  REST Controller   │                           │
│              │  CLI Interface     │                           │
│              │  GUI               │                           │
│              └────────┬───────────┘                           │
│                       │                                      │
│                  Input Port                                   │
│                  (인터페이스)                                  │
│                       │                                      │
│              ┌────────▼──────────────────┐                    │
│              │                           │                    │
│              │     Application Core      │                    │
│              │    (Domain + Use Cases)   │                    │
│              │                           │                    │
│              └────────┬──────────────────┘                    │
│                       │                                      │
│                  Output Port                                  │
│                  (인터페이스)                                  │
│                       │                                      │
│              ┌────────▼───────────┐                           │
│              │  DB Repository     │                           │
│              │  Email Service     │                           │
│              │  Message Queue     │                           │
│              └────────────────────┘                           │
│                   Driven Adapters                             │
│                   (출력 측)                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 클린 아키텍처와 헥사고날의 매핑

| 헥사고날 아키텍처 | 클린 아키텍처 |
|-----------------|-------------|
| Application Core | Entities + Use Cases |
| Input Ports | Use Case Input Boundary (인터페이스) |
| Output Ports | Use Case Output Boundary (인터페이스) |
| Driving Adapters | Controllers (Interface Adapters) |
| Driven Adapters | Gateways, Repositories (Interface Adapters) |
| 외부 시스템 | Frameworks & Drivers |

> **결론**: 헥사고날 아키텍처와 클린 아키텍처는 **본질적으로 같은 원칙**을 다른 관점에서 표현한 것입니다. 둘 다 핵심 비즈니스 로직을 외부 세계로부터 격리하는 것이 목표입니다.

---

## 9. Onion Architecture와의 비교

### Onion Architecture (양파 아키텍처)

Jeffrey Palermo가 2008년에 제안한 아키텍처로, 클린 아키텍처와 매우 유사합니다.

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Onion Architecture          Clean Architecture                 │
│                                                                  │
│   ┌─────────────────┐         ┌─────────────────┐               │
│   │ Infrastructure  │         │ Frameworks &    │               │
│   │  ┌───────────┐  │         │ Drivers         │               │
│   │  │Application│  │         │  ┌───────────┐  │               │
│   │  │ Services  │  │         │  │ Interface │  │               │
│   │  │ ┌───────┐ │  │         │  │ Adapters  │  │               │
│   │  │ │Domain │ │  │         │  │ ┌───────┐ │  │               │
│   │  │ │ Model │ │  │         │  │ │  Use  │ │  │               │
│   │  │ │       │ │  │         │  │ │ Cases │ │  │               │
│   │  │ │       │ │  │         │  │ │┌─────┐│ │  │               │
│   │  │ │       │ │  │         │  │ ││Enti-││ │  │               │
│   │  │ │       │ │  │         │  │ ││ties ││ │  │               │
│   │  │ └───────┘ │  │         │  │ │└─────┘│ │  │               │
│   │  └───────────┘  │         │  │ └───────┘ │  │               │
│   └─────────────────┘         │  └───────────┘  │               │
│                               └─────────────────┘               │
│                                                                  │
│   매핑 관계:                                                      │
│   Domain Model    ≈  Entities                                    │
│   App Services    ≈  Use Cases                                   │
│   Infrastructure  ≈  Interface Adapters + Frameworks & Drivers   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 세 가지 아키텍처 비교표

| 특성 | Layered | Onion | Clean |
|------|---------|-------|-------|
| **레이어 수** | 3~4 | 3~4 (동심원) | 4 (동심원) |
| **의존성 방향** | 위→아래 | 바깥→안 | 바깥→안 |
| **핵심 개념** | 계층 분리 | 도메인 중심 | 의존성 규칙 |
| **도메인 위치** | 중간 레이어 | 가장 안쪽 | 가장 안쪽 |
| **DB 의존성** | 있을 수 있음 | 없음 | 없음 |
| **제안자** | 전통적 | Jeffrey Palermo | Robert C. Martin |
| **제안 시기** | 1990년대~ | 2008년 | 2012년 |
| **복잡도** | 낮음 | 중간 | 높음 |

---

## 10. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **프레임워크 독립** | Django→Flask, ASP.NET→Minimal API 교체 용이 |
| **테스트 용이** | 비즈니스 로직을 DB/UI 없이 단위 테스트 가능 |
| **DB 독립** | MySQL→PostgreSQL 교체 시 구현체만 교체 |
| **UI 독립** | 웹→모바일→CLI 전환이 핵심 로직 변경 없이 가능 |
| **유지보수성** | 각 레이어가 독립적으로 변경 가능 |
| **확장성** | 새 기능 추가 시 기존 코드 영향 최소화 |
| **팀 분업** | 레이어별로 팀을 나눠 개발 가능 |

### 단점

| 단점 | 설명 |
|------|------|
| **초기 복잡도** | 간단한 CRUD에도 많은 파일과 클래스 필요 |
| **학습 곡선** | 의존성 역전, 포트/어댑터 등 개념 이해 필요 |
| **코드량 증가** | DTO 변환, 인터페이스 정의 등 보일러플레이트 |
| **과도한 추상화** | 작은 프로젝트에서는 오버엔지니어링 |
| **간접 참조** | 코드 추적이 어려울 수 있음 (인터페이스 → 구현체) |
| **초기 설계 비용** | 올바른 경계 설정에 경험 필요 |

---

## 11. 언제 사용해야 하는가?

### 적합한 경우

```
[O] 적합한 프로젝트 유형
────────────────────────
- 중~대규모 프로젝트 (개발자 5명 이상)
- 장기 유지보수가 예상되는 프로젝트 (2년+)
- 비즈니스 로직이 복잡한 도메인
- 외부 의존성(DB, API 등)이 변경될 가능성이 있는 경우
- 높은 테스트 커버리지가 요구되는 경우
- 마이크로서비스 아키텍처의 개별 서비스
```

### 부적합한 경우

```
[X] 부적합한 프로젝트 유형
────────────────────────
- 프로토타입 또는 MVP
- 단순 CRUD 애플리케이션
- 짧은 수명의 프로젝트
- 1인 개발의 소규모 프로젝트
- 비즈니스 로직이 거의 없는 프로젝트
- 빠른 출시가 최우선인 스타트업 초기
```

### 실무 조언

```
┌─────────────────────────────────────────────────────────────────┐
│                    실무에서의 단계적 적용                          │
│                                                                 │
│  Level 1: 기본 계층 분리                                         │
│  ─────────────────────                                          │
│  Controller / Service / Repository 분리부터 시작                  │
│  → 가장 기본적인 관심사 분리를 먼저 익히세요                       │
│                                                                 │
│  Level 2: 인터페이스 도입                                        │
│  ─────────────────────                                          │
│  Repository를 인터페이스로 추상화                                 │
│  → 테스트 시 Mock 사용이 가능해집니다                              │
│                                                                 │
│  Level 3: 유스케이스 분리                                        │
│  ─────────────────────                                          │
│  Service를 개별 UseCase로 분리                                   │
│  → 각 기능이 독립적으로 관리됩니다                                 │
│                                                                 │
│  Level 4: 완전한 클린 아키텍처                                    │
│  ─────────────────────                                          │
│  4개 레이어를 별도 프로젝트/패키지로 분리                          │
│  → 컴파일 타임에 의존성 규칙이 강제됩니다                          │
│                                                                 │
│  [팁] 처음부터 Level 4를 적용하지 마세요.                         │
│       프로젝트 복잡도에 맞춰 단계적으로 적용하는 것이 현실적입니다.   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12. 정리 및 체크리스트

### 클린 아키텍처 핵심 정리

```
┌─────────────────────────────────────────────────────────────┐
│                클린 아키텍처 핵심 원칙 정리                    │
│                                                             │
│  1. 의존성은 항상 안쪽(고수준)을 향한다.                       │
│  2. 안쪽 레이어는 바깥 레이어의 존재를 모른다.                  │
│  3. 의존성 역전(DIP)으로 안쪽→바깥 의존을 해결한다.            │
│  4. Entity는 가장 안정적이고 변하지 않는 코어다.               │
│  5. UseCase는 애플리케이션의 "무엇을 하는지"를 정의한다.       │
│  6. 구현 세부사항(DB, 웹 등)은 가장 바깥에 위치한다.           │
│  7. 프레임워크는 도구이지, 아키텍처가 아니다.                  │
└─────────────────────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] Entity에 프레임워크 의존성(ORM 어노테이션 등)이 없는가?
- [ ] UseCase가 구체적인 DB/웹 프레임워크를 참조하지 않는가?
- [ ] 외부 의존성은 인터페이스(Port)를 통해 접근하는가?
- [ ] 각 레이어가 독립적으로 단위 테스트 가능한가?
- [ ] Composition Root에서 의존성을 올바르게 조립하는가?
- [ ] DTO를 통해 레이어 간 데이터를 전달하는가?
- [ ] 의존성 방향이 항상 안쪽을 향하는가?
- [ ] DB를 교체해도 UseCase 코드를 수정할 필요가 없는가?
- [ ] 프레임워크를 교체해도 비즈니스 로직이 변하지 않는가?
- [ ] 과도한 추상화 없이 프로젝트 규모에 맞는 수준인가?

---

## 13. 참고 자료

### 공식 자료
- [The Clean Architecture - Robert C. Martin (원문)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Clean Architecture Book - Robert C. Martin](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)

### 추천 도서
- "Clean Architecture" - Robert C. Martin (2017)
- "Implementing Domain-Driven Design" - Vaughn Vernon
- "Get Your Hands Dirty on Clean Architecture" - Tom Hombergs (Spring Boot 기반)

### 추천 영상
- [Robert C. Martin - Clean Architecture (NDC 2012)](https://www.youtube.com/watch?v=2dKZ-dWaCiU)
- [Clean Architecture with ASP.NET Core](https://www.youtube.com/results?search_query=clean+architecture+aspnet+core)

### 관련 패턴
- **Hexagonal Architecture**: Alistair Cockburn의 포트와 어댑터 패턴
- **Onion Architecture**: Jeffrey Palermo의 양파 아키텍처
- **Domain-Driven Design**: Eric Evans의 도메인 주도 설계
- **CQRS**: Command Query Responsibility Segregation

---

> **이전 문서**: [MVVM 패턴](./01-mvvm.md)
> **다음 문서**: [Layered Architecture (계층형 아키텍처)](./03-layered-architecture.md)
