# Layered Architecture (계층형 아키텍처)

> **대상 독자**: 2~3년차 개발자
> **핵심 키워드**: 계층 분리, Presentation Layer, Business Logic Layer, Data Access Layer, N-Tier
> **난이도**: ★★☆☆☆ (초중급)

---

## 목차

1. [계층형 아키텍처 개요](#1-계층형-아키텍처-개요)
2. [3-Tier / N-Tier 구조](#2-3-tier--n-tier-구조)
3. [각 계층의 역할과 책임](#3-각-계층의-역할과-책임)
4. [계층 간 통신 규칙](#4-계층-간-통신-규칙)
5. [Python 구현: FastAPI 기반 예제](#5-python-구현-fastapi-기반-예제)
6. [C# 구현: ASP.NET 기반 예제](#6-c-구현-aspnet-기반-예제)
7. [폴더 구조 예시](#7-폴더-구조-예시)
8. [Layered vs Clean Architecture 비교](#8-layered-vs-clean-architecture-비교)
9. [장단점](#9-장단점)
10. [실무 팁: 계층을 나누는 기준](#10-실무-팁-계층을-나누는-기준)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [참고 자료](#12-참고-자료)

---

## 1. 계층형 아키텍처 개요

### 가장 전통적이고 기본적인 아키텍처 패턴

계층형 아키텍처(Layered Architecture)는 소프트웨어 설계에서 **가장 널리 사용되고 오래된 아키텍처 패턴**입니다. 대부분의 개발자가 처음 접하는 아키텍처이며, 많은 프레임워크(Spring, ASP.NET, Django 등)의 기본 구조이기도 합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                    계층형 아키텍처의 핵심 아이디어                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "관련된 기능을 수평적 계층(Layer)으로 묶고,                       │
│   각 계층은 바로 아래 계층만 사용한다."                             │
│                                                                 │
│  ┌───────────────────────────────────────┐  ◄─ 사용자 요청       │
│  │        Presentation Layer            │     (HTTP, UI)        │
│  │        (표현 계층)                     │                      │
│  └───────────────────┬───────────────────┘                      │
│                      │ 호출                                     │
│                      ▼                                          │
│  ┌───────────────────────────────────────┐                      │
│  │       Business Logic Layer           │                      │
│  │       (비즈니스 로직 계층)              │                      │
│  └───────────────────┬───────────────────┘                      │
│                      │ 호출                                     │
│                      ▼                                          │
│  ┌───────────────────────────────────────┐                      │
│  │        Data Access Layer             │                      │
│  │        (데이터 접근 계층)              │                      │
│  └───────────────────┬───────────────────┘                      │
│                      │                                          │
│                      ▼                                          │
│                  Database                                       │
│                                                                 │
│  [핵심] 각 계층은 자기 바로 아래 계층만 알고 있습니다.               │
│         위의 계층이 무엇인지는 모릅니다.                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 역사적 배경

```
┌─────────────────────────────────────────────────────────────┐
│                     계층형 아키텍처 역사                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1990년대 ─── Client-Server 아키텍처 등장                    │
│               2-Tier: 클라이언트 + 서버(DB)                  │
│               │                                             │
│  1990년대 중반 ─── 3-Tier 아키텍처 표준화                    │
│               Presentation + Logic + Data                   │
│               J2EE, .NET에서 표준 패턴으로 채택               │
│               │                                             │
│  2000년대 ─── N-Tier로 확장                                  │
│               더 세분화된 계층 구조                           │
│               SOA(Service-Oriented Architecture)와 결합      │
│               │                                             │
│  2010년대+ ── 여전히 가장 보편적인 기본 아키텍처               │
│               Clean Architecture 등 진화된 패턴 등장         │
│               하지만 단순한 프로젝트에서는 여전히 최선           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 3-Tier / N-Tier 구조

### 3-Tier 구조 (가장 기본)

```
┌─────────────────────────────────────────────────────────────────┐
│                      3-Tier 아키텍처                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                 Presentation Layer                       │    │
│  │                 (표현 계층)                               │    │
│  │                                                         │    │
│  │  - HTTP 요청/응답 처리                                   │    │
│  │  - 입력 데이터 검증 (형식 검증)                           │    │
│  │  - 응답 데이터 포매팅 (JSON, HTML 등)                    │    │
│  │  - 인증/인가 처리                                        │    │
│  │                                                         │    │
│  │  예: Controller, API Endpoint, View                     │    │
│  └──────────────────────┬──────────────────────────────────┘    │
│                         │                                       │
│                    DTO / Request Model                           │
│                         │                                       │
│  ┌──────────────────────▼──────────────────────────────────┐    │
│  │              Business Logic Layer                        │    │
│  │              (비즈니스 로직 계층)                          │    │
│  │                                                         │    │
│  │  - 핵심 비즈니스 규칙 구현                                │    │
│  │  - 트랜잭션 관리                                         │    │
│  │  - 비즈니스 데이터 검증                                   │    │
│  │  - 비즈니스 프로세스 조율                                 │    │
│  │                                                         │    │
│  │  예: Service, Manager, Processor                        │    │
│  └──────────────────────┬──────────────────────────────────┘    │
│                         │                                       │
│                    Entity / Domain Model                         │
│                         │                                       │
│  ┌──────────────────────▼──────────────────────────────────┐    │
│  │               Data Access Layer                          │    │
│  │               (데이터 접근 계층)                           │    │
│  │                                                         │    │
│  │  - 데이터베이스 CRUD 연산                                 │    │
│  │  - ORM 매핑                                              │    │
│  │  - 쿼리 작성 및 실행                                     │    │
│  │  - 외부 API 호출                                         │    │
│  │                                                         │    │
│  │  예: Repository, DAO, Gateway                           │    │
│  └──────────────────────┬──────────────────────────────────┘    │
│                         │                                       │
│                         ▼                                       │
│                   ┌──────────┐                                  │
│                   │ Database │                                  │
│                   └──────────┘                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### N-Tier 확장 구조

실제 프로젝트에서는 3개 이상의 계층을 사용하기도 합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                     N-Tier 확장 구조 예시                         │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Presentation Layer (API / UI)                          │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Application Layer (유스케이스 조율, DTO 변환)             │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Domain Layer (핵심 비즈니스 규칙, 엔티티)                 │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Infrastructure Layer (DB, 외부 API, 파일 시스템)         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  [참고] 4개 레이어로 확장하면 Clean Architecture와                │
│         유사해지기 시작합니다.                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 각 계층의 역할과 책임

### 3-1. Presentation Layer (표현 계층)

사용자와의 **상호작용을 담당**합니다. 요청을 받고 응답을 반환하는 진입점입니다.

| 책임 | 하면 안 되는 것 |
|------|---------------|
| HTTP 요청 파싱 | SQL 쿼리 직접 실행 |
| 입력 형식 검증 (필수값, 타입) | 복잡한 비즈니스 규칙 구현 |
| 응답 포매팅 (JSON, XML) | DB 커넥션 직접 관리 |
| 라우팅 | 다른 서비스 직접 호출 |
| 인증 토큰 검증 | 비즈니스 로직과 혼합된 코드 |
| 에러 응답 변환 | 엔티티 직접 노출 |

### 3-2. Business Logic Layer (비즈니스 로직 계층)

**애플리케이션의 핵심 가치**인 비즈니스 규칙을 구현합니다. 가장 중요한 계층입니다.

| 책임 | 하면 안 되는 것 |
|------|---------------|
| 비즈니스 규칙 구현 | HTTP 요청/응답 직접 처리 |
| 데이터 비즈니스 검증 | SQL 쿼리 직접 작성 |
| 트랜잭션 범위 결정 | HTML/JSON 생성 |
| 여러 Repository 조율 | UI 관련 로직 |
| 비즈니스 이벤트 발행 | 프레임워크 종속 코드 |
| 권한 검사 (비즈니스 수준) | 인프라 세부사항 의존 |

### 3-3. Data Access Layer (데이터 접근 계층)

**데이터 저장소와의 통신**을 담당합니다. DB, 파일, 외부 API 등 모든 외부 데이터 소스 접근을 캡슐화합니다.

| 책임 | 하면 안 되는 것 |
|------|---------------|
| CRUD 연산 구현 | 비즈니스 규칙 구현 |
| ORM 매핑 설정 | HTTP 요청 처리 |
| SQL/NoSQL 쿼리 작성 | 입력 검증 |
| 커넥션 풀 관리 | 응답 포매팅 |
| 캐싱 (데이터 수준) | 트랜잭션 범위 결정 |
| 외부 API 호출 래핑 | 직접 사용자와 상호작용 |

---

## 4. 계층 간 통신 규칙

### 기본 규칙

```
┌─────────────────────────────────────────────────────────────────┐
│                    계층 간 통신 규칙                               │
│                                                                 │
│  규칙 1: 상위 계층은 바로 아래 계층만 호출한다 (Strict Layering)    │
│  ─────────────────────────────────────────────                   │
│                                                                 │
│    Presentation ──► Business Logic ──► Data Access              │
│         O               O                O                      │
│                                                                 │
│    Presentation ─────────────────────► Data Access              │
│         X  (계층 건너뛰기 금지!)                                  │
│                                                                 │
│                                                                 │
│  규칙 2: 하위 계층은 상위 계층을 알지 못한다                       │
│  ─────────────────────────────────────────                       │
│                                                                 │
│    Data Access ──► Business Logic                               │
│         X  (역방향 의존 금지!)                                    │
│                                                                 │
│                                                                 │
│  규칙 3: 계층 간 데이터 전달은 DTO를 사용한다                      │
│  ─────────────────────────────────────────                       │
│                                                                 │
│    Controller          Service           Repository             │
│    ┌──────┐           ┌──────┐          ┌──────┐               │
│    │ API  │──Request──│ 처리 │──Entity──│  DB  │               │
│    │ DTO  │  DTO      │  후  │  Model   │ 조회 │               │
│    │      │◄─Response─│      │◄─Entity──│      │               │
│    │      │  DTO      │      │  Model   │      │               │
│    └──────┘           └──────┘          └──────┘               │
│                                                                 │
│                                                                 │
│  규칙 4: 인터페이스를 통한 느슨한 결합 (권장)                      │
│  ─────────────────────────────────────────                       │
│                                                                 │
│    Service ──► IRepository (인터페이스)                           │
│                    ▲                                             │
│                    │ 구현                                        │
│                SqlRepository                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Strict vs Relaxed Layering

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [ Strict Layering (엄격한 계층화) ]                      │
│                                                          │
│    Presentation → Business → Data Access                 │
│                                                          │
│    - 바로 아래 계층만 접근 가능                             │
│    - 장점: 계층 독립성 극대화                               │
│    - 단점: 단순 조회에도 모든 계층을 거쳐야 함               │
│                                                          │
│  ─────────────────────────────────────────                │
│                                                          │
│  [ Relaxed Layering (느슨한 계층화) ]                     │
│                                                          │
│    Presentation → Business → Data Access                 │
│    Presentation ──────────→ Data Access (허용)            │
│                                                          │
│    - 아래 계층이면 건너뛸 수 있음                           │
│    - 장점: 단순 CRUD에서 불필요한 패스스루 제거             │
│    - 단점: 계층 경계가 모호해질 위험                        │
│                                                          │
│  [실무 권장] 기본적으로 Strict Layering을 따르되,           │
│             정말 단순한 조회(pass-through)에 한해서만        │
│             Relaxed를 허용하세요.                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Python 구현: FastAPI 기반 예제

### 상품 관리 시스템 (Product Management)

#### 5-1. Data Access Layer

```python
# dal/models.py
"""데이터 접근 계층 - 데이터 모델"""
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional


@dataclass
class Product:
    """상품 엔티티"""
    id: Optional[int] = None
    name: str = ""
    description: str = ""
    price: float = 0.0
    stock: int = 0
    category: str = ""
    is_active: bool = True
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: Optional[datetime] = None


# dal/repositories.py
"""데이터 접근 계층 - Repository 구현"""
from abc import ABC, abstractmethod
from typing import List, Optional
from dal.models import Product


class IProductRepository(ABC):
    """상품 저장소 인터페이스"""

    @abstractmethod
    def find_by_id(self, product_id: int) -> Optional[Product]:
        pass

    @abstractmethod
    def find_all(self, category: Optional[str] = None) -> List[Product]:
        pass

    @abstractmethod
    def save(self, product: Product) -> Product:
        pass

    @abstractmethod
    def update(self, product: Product) -> Product:
        pass

    @abstractmethod
    def delete(self, product_id: int) -> bool:
        pass

    @abstractmethod
    def find_by_name(self, name: str) -> Optional[Product]:
        pass


class InMemoryProductRepository(IProductRepository):
    """인메모리 상품 저장소 (테스트/개발용)"""

    def __init__(self):
        self._products: dict[int, Product] = {}
        self._next_id = 1

    def find_by_id(self, product_id: int) -> Optional[Product]:
        return self._products.get(product_id)

    def find_all(self, category: Optional[str] = None) -> List[Product]:
        products = list(self._products.values())
        if category:
            products = [p for p in products if p.category == category]
        return [p for p in products if p.is_active]

    def save(self, product: Product) -> Product:
        product.id = self._next_id
        self._next_id += 1
        self._products[product.id] = product
        return product

    def update(self, product: Product) -> Product:
        if product.id and product.id in self._products:
            product.updated_at = datetime.now()
            self._products[product.id] = product
        return product

    def delete(self, product_id: int) -> bool:
        if product_id in self._products:
            del self._products[product_id]
            return True
        return False

    def find_by_name(self, name: str) -> Optional[Product]:
        for product in self._products.values():
            if product.name == name:
                return product
        return None
```

#### 5-2. Business Logic Layer

```python
# bll/dtos.py
"""비즈니스 로직 계층 - DTO 정의"""
from dataclasses import dataclass
from datetime import datetime
from typing import Optional


@dataclass
class CreateProductRequest:
    """상품 생성 요청"""
    name: str
    description: str
    price: float
    stock: int
    category: str


@dataclass
class UpdateProductRequest:
    """상품 수정 요청"""
    name: Optional[str] = None
    description: Optional[str] = None
    price: Optional[float] = None
    stock: Optional[int] = None
    category: Optional[str] = None


@dataclass
class ProductResponse:
    """상품 응답"""
    id: int
    name: str
    description: str
    price: float
    stock: int
    category: str
    is_active: bool
    created_at: str
    stock_status: str  # 재고 상태 (비즈니스 로직으로 계산)


# bll/exceptions.py
"""비즈니스 로직 계층 - 비즈니스 예외"""


class BusinessException(Exception):
    """비즈니스 예외 기본 클래스"""
    pass


class ProductNotFoundError(BusinessException):
    def __init__(self, product_id: int):
        super().__init__(f"상품을 찾을 수 없습니다 (ID: {product_id})")
        self.product_id = product_id


class DuplicateProductError(BusinessException):
    def __init__(self, name: str):
        super().__init__(f"이미 존재하는 상품명입니다: {name}")


class InvalidProductError(BusinessException):
    def __init__(self, message: str):
        super().__init__(f"유효하지 않은 상품 정보: {message}")


class InsufficientStockError(BusinessException):
    def __init__(self, product_id: int, requested: int, available: int):
        super().__init__(
            f"재고 부족 (상품 ID: {product_id}, "
            f"요청: {requested}, 가용: {available})"
        )


# bll/services.py
"""비즈니스 로직 계층 - 서비스"""
from typing import List
from dal.models import Product
from dal.repositories import IProductRepository
from bll.dtos import (
    CreateProductRequest, UpdateProductRequest, ProductResponse
)
from bll.exceptions import (
    ProductNotFoundError, DuplicateProductError,
    InvalidProductError, InsufficientStockError
)


class ProductService:
    """
    상품 서비스 - 비즈니스 로직 담당
    이 클래스가 계층형 아키텍처에서 가장 중요한 부분입니다.
    """

    def __init__(self, product_repository: IProductRepository):
        self._repo = product_repository

    def create_product(self, request: CreateProductRequest) -> ProductResponse:
        """상품 생성"""
        # 비즈니스 검증
        self._validate_product_data(request.name, request.price, request.stock)

        # 중복 검사
        existing = self._repo.find_by_name(request.name)
        if existing:
            raise DuplicateProductError(request.name)

        # 엔티티 생성 및 저장
        product = Product(
            name=request.name,
            description=request.description,
            price=request.price,
            stock=request.stock,
            category=request.category
        )

        saved = self._repo.save(product)
        return self._to_response(saved)

    def get_product(self, product_id: int) -> ProductResponse:
        """상품 조회"""
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ProductNotFoundError(product_id)
        return self._to_response(product)

    def get_all_products(self, category: str = None) -> List[ProductResponse]:
        """전체 상품 목록 조회"""
        products = self._repo.find_all(category)
        return [self._to_response(p) for p in products]

    def update_product(
        self, product_id: int, request: UpdateProductRequest
    ) -> ProductResponse:
        """상품 수정"""
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ProductNotFoundError(product_id)

        # 변경할 필드만 업데이트
        if request.name is not None:
            # 이름 변경 시 중복 검사
            existing = self._repo.find_by_name(request.name)
            if existing and existing.id != product_id:
                raise DuplicateProductError(request.name)
            product.name = request.name

        if request.description is not None:
            product.description = request.description

        if request.price is not None:
            if request.price < 0:
                raise InvalidProductError("가격은 0 이상이어야 합니다.")
            product.price = request.price

        if request.stock is not None:
            if request.stock < 0:
                raise InvalidProductError("재고는 0 이상이어야 합니다.")
            product.stock = request.stock

        if request.category is not None:
            product.category = request.category

        updated = self._repo.update(product)
        return self._to_response(updated)

    def delete_product(self, product_id: int) -> bool:
        """상품 삭제"""
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ProductNotFoundError(product_id)
        return self._repo.delete(product_id)

    def decrease_stock(self, product_id: int, quantity: int) -> ProductResponse:
        """재고 차감 (주문 시)"""
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ProductNotFoundError(product_id)

        if product.stock < quantity:
            raise InsufficientStockError(product_id, quantity, product.stock)

        product.stock -= quantity
        updated = self._repo.update(product)
        return self._to_response(updated)

    # ── 내부 헬퍼 메서드 ──

    def _validate_product_data(
        self, name: str, price: float, stock: int
    ) -> None:
        """상품 데이터 비즈니스 검증"""
        if not name or len(name.strip()) < 2:
            raise InvalidProductError("상품명은 2자 이상이어야 합니다.")
        if len(name) > 100:
            raise InvalidProductError("상품명은 100자 이하여야 합니다.")
        if price < 0:
            raise InvalidProductError("가격은 0 이상이어야 합니다.")
        if price > 99_999_999:
            raise InvalidProductError("가격이 한도를 초과했습니다.")
        if stock < 0:
            raise InvalidProductError("재고는 0 이상이어야 합니다.")

    def _get_stock_status(self, stock: int) -> str:
        """재고 상태 계산 - 비즈니스 로직"""
        if stock == 0:
            return "품절"
        elif stock <= 10:
            return "재고 부족"
        elif stock <= 50:
            return "보통"
        else:
            return "충분"

    def _to_response(self, product: Product) -> ProductResponse:
        """엔티티를 응답 DTO로 변환"""
        return ProductResponse(
            id=product.id,
            name=product.name,
            description=product.description,
            price=product.price,
            stock=product.stock,
            category=product.category,
            is_active=product.is_active,
            created_at=product.created_at.isoformat(),
            stock_status=self._get_stock_status(product.stock)
        )
```

#### 5-3. Presentation Layer (FastAPI)

```python
# api/routes.py
"""표현 계층 - FastAPI 라우트"""
from fastapi import FastAPI, HTTPException, Query
from pydantic import BaseModel, Field
from typing import Optional, List

from bll.services import ProductService
from bll.dtos import CreateProductRequest, UpdateProductRequest
from bll.exceptions import (
    BusinessException, ProductNotFoundError,
    DuplicateProductError, InvalidProductError
)
from dal.repositories import InMemoryProductRepository


# ── Pydantic 모델 (API 요청/응답 스키마) ──

class CreateProductSchema(BaseModel):
    """API 요청 스키마 - 상품 생성"""
    name: str = Field(..., min_length=2, max_length=100, examples=["무선 마우스"])
    description: str = Field(default="", max_length=500)
    price: float = Field(..., ge=0, examples=[29900])
    stock: int = Field(..., ge=0, examples=[100])
    category: str = Field(..., examples=["전자기기"])


class UpdateProductSchema(BaseModel):
    """API 요청 스키마 - 상품 수정"""
    name: Optional[str] = Field(None, min_length=2, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    price: Optional[float] = Field(None, ge=0)
    stock: Optional[int] = Field(None, ge=0)
    category: Optional[str] = None


class ProductResponseSchema(BaseModel):
    """API 응답 스키마"""
    id: int
    name: str
    description: str
    price: float
    stock: int
    category: str
    is_active: bool
    created_at: str
    stock_status: str


class ApiResponse(BaseModel):
    """통일된 API 응답 형식"""
    success: bool
    message: str = ""
    data: Optional[dict | list] = None


# ── 앱 생성 및 의존성 주입 ──

app = FastAPI(title="상품 관리 API", version="1.0.0")

# 의존성 조립 (실무에서는 DI 컨테이너 사용)
repository = InMemoryProductRepository()
product_service = ProductService(repository)


# ── API 엔드포인트 ──

@app.post("/api/products", response_model=ApiResponse, status_code=201)
async def create_product(schema: CreateProductSchema):
    """상품 생성 API"""
    try:
        # Pydantic 스키마 -> BLL DTO 변환
        request = CreateProductRequest(
            name=schema.name,
            description=schema.description,
            price=schema.price,
            stock=schema.stock,
            category=schema.category
        )
        result = product_service.create_product(request)
        return ApiResponse(
            success=True,
            message="상품이 생성되었습니다.",
            data=result.__dict__
        )
    except DuplicateProductError as e:
        raise HTTPException(status_code=409, detail=str(e))
    except InvalidProductError as e:
        raise HTTPException(status_code=400, detail=str(e))


@app.get("/api/products/{product_id}", response_model=ApiResponse)
async def get_product(product_id: int):
    """상품 조회 API"""
    try:
        result = product_service.get_product(product_id)
        return ApiResponse(success=True, data=result.__dict__)
    except ProductNotFoundError:
        raise HTTPException(status_code=404, detail="상품을 찾을 수 없습니다.")


@app.get("/api/products", response_model=ApiResponse)
async def get_all_products(
    category: Optional[str] = Query(None, description="카테고리 필터")
):
    """전체 상품 목록 조회 API"""
    results = product_service.get_all_products(category)
    return ApiResponse(
        success=True,
        data=[r.__dict__ for r in results]
    )


@app.put("/api/products/{product_id}", response_model=ApiResponse)
async def update_product(product_id: int, schema: UpdateProductSchema):
    """상품 수정 API"""
    try:
        request = UpdateProductRequest(
            name=schema.name,
            description=schema.description,
            price=schema.price,
            stock=schema.stock,
            category=schema.category
        )
        result = product_service.update_product(product_id, request)
        return ApiResponse(
            success=True,
            message="상품이 수정되었습니다.",
            data=result.__dict__
        )
    except ProductNotFoundError:
        raise HTTPException(status_code=404, detail="상품을 찾을 수 없습니다.")
    except (DuplicateProductError, InvalidProductError) as e:
        raise HTTPException(status_code=400, detail=str(e))


@app.delete("/api/products/{product_id}", response_model=ApiResponse)
async def delete_product(product_id: int):
    """상품 삭제 API"""
    try:
        product_service.delete_product(product_id)
        return ApiResponse(success=True, message="상품이 삭제되었습니다.")
    except ProductNotFoundError:
        raise HTTPException(status_code=404, detail="상품을 찾을 수 없습니다.")
```

---

## 6. C# 구현: ASP.NET 기반 예제

### 동일한 상품 관리 시스템

#### 6-1. Data Access Layer

```csharp
// DAL/Models/Product.cs
namespace ProductManagement.DAL.Models
{
    /// <summary>상품 엔티티</summary>
    public class Product
    {
        public int? Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public int Stock { get; set; }
        public string Category { get; set; } = string.Empty;
        public bool IsActive { get; set; } = true;
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public DateTime? UpdatedAt { get; set; }
    }
}

// DAL/Repositories/IProductRepository.cs
namespace ProductManagement.DAL.Repositories
{
    /// <summary>상품 저장소 인터페이스</summary>
    public interface IProductRepository
    {
        Task<Product?> FindByIdAsync(int productId);
        Task<IReadOnlyList<Product>> FindAllAsync(string? category = null);
        Task<Product> SaveAsync(Product product);
        Task<Product> UpdateAsync(Product product);
        Task<bool> DeleteAsync(int productId);
        Task<Product?> FindByNameAsync(string name);
    }
}

// DAL/Repositories/InMemoryProductRepository.cs
namespace ProductManagement.DAL.Repositories
{
    /// <summary>인메모리 상품 저장소 구현</summary>
    public class InMemoryProductRepository : IProductRepository
    {
        private readonly Dictionary<int, Product> _products = new();
        private int _nextId = 1;

        public Task<Product?> FindByIdAsync(int productId)
        {
            _products.TryGetValue(productId, out var product);
            return Task.FromResult(product);
        }

        public Task<IReadOnlyList<Product>> FindAllAsync(string? category = null)
        {
            var query = _products.Values.Where(p => p.IsActive);
            if (!string.IsNullOrEmpty(category))
                query = query.Where(p => p.Category == category);

            IReadOnlyList<Product> result = query.ToList().AsReadOnly();
            return Task.FromResult(result);
        }

        public Task<Product> SaveAsync(Product product)
        {
            product.Id = _nextId++;
            _products[product.Id.Value] = product;
            return Task.FromResult(product);
        }

        public Task<Product> UpdateAsync(Product product)
        {
            if (product.Id.HasValue && _products.ContainsKey(product.Id.Value))
            {
                product.UpdatedAt = DateTime.UtcNow;
                _products[product.Id.Value] = product;
            }
            return Task.FromResult(product);
        }

        public Task<bool> DeleteAsync(int productId)
        {
            var result = _products.Remove(productId);
            return Task.FromResult(result);
        }

        public Task<Product?> FindByNameAsync(string name)
        {
            var product = _products.Values.FirstOrDefault(
                p => p.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
            return Task.FromResult(product);
        }
    }
}
```

#### 6-2. Business Logic Layer

```csharp
// BLL/DTOs/ProductDTOs.cs
namespace ProductManagement.BLL.DTOs
{
    public record CreateProductRequest(
        string Name,
        string Description,
        decimal Price,
        int Stock,
        string Category
    );

    public record UpdateProductRequest(
        string? Name = null,
        string? Description = null,
        decimal? Price = null,
        int? Stock = null,
        string? Category = null
    );

    public record ProductResponse(
        int Id,
        string Name,
        string Description,
        decimal Price,
        int Stock,
        string Category,
        bool IsActive,
        string CreatedAt,
        string StockStatus
    );
}

// BLL/Exceptions/BusinessExceptions.cs
namespace ProductManagement.BLL.Exceptions
{
    public class BusinessException : Exception
    {
        public BusinessException(string message) : base(message) { }
    }

    public class ProductNotFoundException : BusinessException
    {
        public int ProductId { get; }
        public ProductNotFoundException(int productId)
            : base($"상품을 찾을 수 없습니다 (ID: {productId})")
        {
            ProductId = productId;
        }
    }

    public class DuplicateProductException : BusinessException
    {
        public DuplicateProductException(string name)
            : base($"이미 존재하는 상품명입니다: {name}") { }
    }

    public class InvalidProductException : BusinessException
    {
        public InvalidProductException(string message)
            : base($"유효하지 않은 상품 정보: {message}") { }
    }

    public class InsufficientStockException : BusinessException
    {
        public InsufficientStockException(int productId, int requested, int available)
            : base($"재고 부족 (상품 ID: {productId}, 요청: {requested}, 가용: {available})")
        { }
    }
}

// BLL/Services/ProductService.cs
using ProductManagement.DAL.Models;
using ProductManagement.DAL.Repositories;
using ProductManagement.BLL.DTOs;
using ProductManagement.BLL.Exceptions;

namespace ProductManagement.BLL.Services
{
    /// <summary>
    /// 상품 서비스 - 비즈니스 로직 담당
    /// 계층형 아키텍처에서 가장 핵심적인 계층입니다.
    /// </summary>
    public class ProductService
    {
        private readonly IProductRepository _repository;

        public ProductService(IProductRepository repository)
        {
            _repository = repository;
        }

        public async Task<ProductResponse> CreateProductAsync(
            CreateProductRequest request)
        {
            // 비즈니스 검증
            ValidateProductData(request.Name, request.Price, request.Stock);

            // 중복 검사
            var existing = await _repository.FindByNameAsync(request.Name);
            if (existing != null)
                throw new DuplicateProductException(request.Name);

            // 엔티티 생성 및 저장
            var product = new Product
            {
                Name = request.Name,
                Description = request.Description,
                Price = request.Price,
                Stock = request.Stock,
                Category = request.Category
            };

            var saved = await _repository.SaveAsync(product);
            return ToResponse(saved);
        }

        public async Task<ProductResponse> GetProductAsync(int productId)
        {
            var product = await _repository.FindByIdAsync(productId)
                ?? throw new ProductNotFoundException(productId);
            return ToResponse(product);
        }

        public async Task<IReadOnlyList<ProductResponse>> GetAllProductsAsync(
            string? category = null)
        {
            var products = await _repository.FindAllAsync(category);
            return products.Select(ToResponse).ToList().AsReadOnly();
        }

        public async Task<ProductResponse> UpdateProductAsync(
            int productId, UpdateProductRequest request)
        {
            var product = await _repository.FindByIdAsync(productId)
                ?? throw new ProductNotFoundException(productId);

            if (request.Name != null)
            {
                var existing = await _repository.FindByNameAsync(request.Name);
                if (existing != null && existing.Id != productId)
                    throw new DuplicateProductException(request.Name);
                product.Name = request.Name;
            }

            if (request.Description != null)
                product.Description = request.Description;

            if (request.Price.HasValue)
            {
                if (request.Price.Value < 0)
                    throw new InvalidProductException("가격은 0 이상이어야 합니다.");
                product.Price = request.Price.Value;
            }

            if (request.Stock.HasValue)
            {
                if (request.Stock.Value < 0)
                    throw new InvalidProductException("재고는 0 이상이어야 합니다.");
                product.Stock = request.Stock.Value;
            }

            if (request.Category != null)
                product.Category = request.Category;

            var updated = await _repository.UpdateAsync(product);
            return ToResponse(updated);
        }

        public async Task DeleteProductAsync(int productId)
        {
            _ = await _repository.FindByIdAsync(productId)
                ?? throw new ProductNotFoundException(productId);
            await _repository.DeleteAsync(productId);
        }

        public async Task<ProductResponse> DecreaseStockAsync(
            int productId, int quantity)
        {
            var product = await _repository.FindByIdAsync(productId)
                ?? throw new ProductNotFoundException(productId);

            if (product.Stock < quantity)
                throw new InsufficientStockException(
                    productId, quantity, product.Stock);

            product.Stock -= quantity;
            var updated = await _repository.UpdateAsync(product);
            return ToResponse(updated);
        }

        // ── 내부 헬퍼 메서드 ──

        private static void ValidateProductData(
            string name, decimal price, int stock)
        {
            if (string.IsNullOrWhiteSpace(name) || name.Trim().Length < 2)
                throw new InvalidProductException("상품명은 2자 이상이어야 합니다.");
            if (name.Length > 100)
                throw new InvalidProductException("상품명은 100자 이하여야 합니다.");
            if (price < 0)
                throw new InvalidProductException("가격은 0 이상이어야 합니다.");
            if (price > 99_999_999)
                throw new InvalidProductException("가격이 한도를 초과했습니다.");
            if (stock < 0)
                throw new InvalidProductException("재고는 0 이상이어야 합니다.");
        }

        private static string GetStockStatus(int stock) => stock switch
        {
            0 => "품절",
            <= 10 => "재고 부족",
            <= 50 => "보통",
            _ => "충분"
        };

        private static ProductResponse ToResponse(Product product) => new(
            Id: product.Id ?? 0,
            Name: product.Name,
            Description: product.Description,
            Price: product.Price,
            Stock: product.Stock,
            Category: product.Category,
            IsActive: product.IsActive,
            CreatedAt: product.CreatedAt.ToString("o"),
            StockStatus: GetStockStatus(product.Stock)
        );
    }
}
```

#### 6-3. Presentation Layer (ASP.NET)

```csharp
// API/Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;
using ProductManagement.BLL.Services;
using ProductManagement.BLL.DTOs;
using ProductManagement.BLL.Exceptions;

namespace ProductManagement.API.Controllers
{
    /// <summary>
    /// 상품 API 컨트롤러 (Presentation Layer)
    /// HTTP 요청을 받아 서비스(BLL)에 전달하고 응답을 반환합니다.
    /// </summary>
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly ProductService _productService;

        public ProductsController(ProductService productService)
        {
            _productService = productService;
        }

        /// <summary>상품 생성</summary>
        [HttpPost]
        [ProducesResponseType(typeof(ApiResponse<ProductResponse>), 201)]
        [ProducesResponseType(typeof(ApiResponse<object>), 400)]
        public async Task<IActionResult> CreateProduct(
            [FromBody] CreateProductRequest request)
        {
            try
            {
                var result = await _productService.CreateProductAsync(request);
                return CreatedAtAction(
                    nameof(GetProduct),
                    new { id = result.Id },
                    ApiResponse<ProductResponse>.Ok(result, "상품이 생성되었습니다.")
                );
            }
            catch (DuplicateProductException ex)
            {
                return Conflict(ApiResponse<object>.Fail(ex.Message));
            }
            catch (InvalidProductException ex)
            {
                return BadRequest(ApiResponse<object>.Fail(ex.Message));
            }
        }

        /// <summary>상품 조회</summary>
        [HttpGet("{id}")]
        [ProducesResponseType(typeof(ApiResponse<ProductResponse>), 200)]
        [ProducesResponseType(404)]
        public async Task<IActionResult> GetProduct(int id)
        {
            try
            {
                var result = await _productService.GetProductAsync(id);
                return Ok(ApiResponse<ProductResponse>.Ok(result));
            }
            catch (ProductNotFoundException)
            {
                return NotFound(ApiResponse<object>.Fail("상품을 찾을 수 없습니다."));
            }
        }

        /// <summary>전체 상품 목록 조회</summary>
        [HttpGet]
        [ProducesResponseType(typeof(ApiResponse<IReadOnlyList<ProductResponse>>), 200)]
        public async Task<IActionResult> GetAllProducts(
            [FromQuery] string? category = null)
        {
            var results = await _productService.GetAllProductsAsync(category);
            return Ok(ApiResponse<IReadOnlyList<ProductResponse>>.Ok(results));
        }

        /// <summary>상품 수정</summary>
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateProduct(
            int id, [FromBody] UpdateProductRequest request)
        {
            try
            {
                var result = await _productService.UpdateProductAsync(id, request);
                return Ok(ApiResponse<ProductResponse>.Ok(
                    result, "상품이 수정되었습니다."));
            }
            catch (ProductNotFoundException)
            {
                return NotFound(ApiResponse<object>.Fail("상품을 찾을 수 없습니다."));
            }
            catch (BusinessException ex)
            {
                return BadRequest(ApiResponse<object>.Fail(ex.Message));
            }
        }

        /// <summary>상품 삭제</summary>
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteProduct(int id)
        {
            try
            {
                await _productService.DeleteProductAsync(id);
                return Ok(ApiResponse<object>.Ok(null, "상품이 삭제되었습니다."));
            }
            catch (ProductNotFoundException)
            {
                return NotFound(ApiResponse<object>.Fail("상품을 찾을 수 없습니다."));
            }
        }
    }

    /// <summary>통일된 API 응답 형식</summary>
    public record ApiResponse<T>(bool Success, string Message, T? Data)
    {
        public static ApiResponse<T> Ok(T? data, string message = "")
            => new(true, message, data);
        public static ApiResponse<T> Fail(string message)
            => new(false, message, default);
    }
}

// Program.cs - 의존성 등록
using ProductManagement.DAL.Repositories;
using ProductManagement.BLL.Services;

var builder = WebApplication.CreateBuilder(args);

// 의존성 등록 - 계층 간 연결
builder.Services.AddSingleton<IProductRepository, InMemoryProductRepository>();
builder.Services.AddScoped<ProductService>();
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapControllers();
app.Run();
```

---

## 7. 폴더 구조 예시

### Python (FastAPI) 프로젝트

```
product-management/
│
├── dal/                             # Data Access Layer
│   ├── __init__.py
│   ├── models.py                    # 데이터 모델 (엔티티)
│   ├── repositories.py              # 저장소 인터페이스 + 구현
│   └── database.py                  # DB 연결 설정
│
├── bll/                             # Business Logic Layer
│   ├── __init__.py
│   ├── dtos.py                      # 입출력 DTO
│   ├── exceptions.py                # 비즈니스 예외
│   └── services.py                  # 비즈니스 서비스
│
├── api/                             # Presentation Layer
│   ├── __init__.py
│   ├── routes.py                    # API 라우트 (엔드포인트)
│   ├── schemas.py                   # Pydantic 요청/응답 스키마
│   └── middleware.py                # 미들웨어 (인증, 로깅 등)
│
├── tests/
│   ├── test_repositories.py
│   ├── test_services.py
│   └── test_routes.py
│
├── config.py                        # 설정
├── main.py                          # 앱 진입점
└── requirements.txt
```

### C# (ASP.NET) 프로젝트

```
ProductManagement.sln
│
├── ProductManagement.DAL/            # Data Access Layer 프로젝트
│   ├── Models/
│   │   └── Product.cs
│   ├── Repositories/
│   │   ├── IProductRepository.cs
│   │   └── InMemoryProductRepository.cs
│   ├── Data/
│   │   └── AppDbContext.cs           # EF Core DbContext
│   └── ProductManagement.DAL.csproj
│       # 의존성: EF Core 등 DB 패키지만
│
├── ProductManagement.BLL/            # Business Logic Layer 프로젝트
│   ├── DTOs/
│   │   └── ProductDTOs.cs
│   ├── Exceptions/
│   │   └── BusinessExceptions.cs
│   ├── Services/
│   │   ├── IProductService.cs
│   │   └── ProductService.cs
│   └── ProductManagement.BLL.csproj
│       # 의존성: DAL 프로젝트 참조
│
├── ProductManagement.API/            # Presentation Layer 프로젝트
│   ├── Controllers/
│   │   └── ProductsController.cs
│   ├── Middleware/
│   │   └── ExceptionHandlerMiddleware.cs
│   ├── Program.cs                    # 앱 진입점 + DI 설정
│   ├── appsettings.json
│   └── ProductManagement.API.csproj
│       # 의존성: BLL + DAL 프로젝트 참조
│
└── ProductManagement.Tests/          # 테스트 프로젝트
    ├── DAL/
    │   └── ProductRepositoryTests.cs
    ├── BLL/
    │   └── ProductServiceTests.cs
    └── API/
        └── ProductsControllerTests.cs
```

### 프로젝트 참조 관계

```
┌──────────────────────────────────────────────────────┐
│            C# 프로젝트 참조 관계 (Layered)              │
│                                                      │
│  API (Presentation) ──► BLL (Business) ──► DAL (Data)│
│      │                                      ▲        │
│      └──────────────────────────────────────┘         │
│                   (직접 참조도 가능)                    │
│                                                      │
│  [비교] Clean Architecture에서는:                     │
│                                                      │
│  WebApi ──► Application ──► Domain                   │
│    │                          ▲                      │
│    └──► Infrastructure ───────┘                      │
│                                                      │
│  Layered에서는 의존성이 위→아래로 흐르지만,              │
│  Clean에서는 바깥→안쪽으로 흐릅니다.                    │
│  이 차이가 두 아키텍처의 핵심적인 구분점입니다.           │
└──────────────────────────────────────────────────────┘
```

---

## 8. Layered vs Clean Architecture 비교

```
┌─────────────────────────────────────────────────────────────────────┐
│              Layered Architecture vs Clean Architecture             │
├──────────────────┬──────────────────────┬───────────────────────────┤
│      항목         │  Layered             │  Clean                    │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 의존성 방향       │ 위 → 아래            │ 바깥 → 안쪽               │
│                  │ (Presentation→DAL)   │ (Framework→Entity)        │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 도메인 위치       │ 중간 계층             │ 가장 안쪽 (핵심)          │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ DB 의존성        │ DAL에 직접 의존 가능   │ 인터페이스로 추상화 필수   │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 프레임워크 독립   │ 보통 프레임워크에      │ 핵심 로직은               │
│                  │ 종속됨               │ 프레임워크 독립            │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 테스트 용이성     │ 보통                 │ 높음                      │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 복잡도           │ 낮음                 │ 높음                      │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 학습 곡선        │ 완만                 │ 가파름                    │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 초기 설정 비용    │ 적음                 │ 많음                      │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 유지보수성       │ 보통                 │ 높음                      │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 적합한 규모      │ 소~중규모            │ 중~대규모                  │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ DB 교체 용이성   │ 어려움               │ 쉬움 (구현체만 교체)       │
├──────────────────┼──────────────────────┼───────────────────────────┤
│ 핵심 원칙        │ 관심사 분리          │ 의존성 규칙               │
└──────────────────┴──────────────────────┴───────────────────────────┘
```

### 시각적 비교

```
  [ Layered Architecture ]              [ Clean Architecture ]

  ┌──────────────────────┐           ┌─────────────────────────┐
  │   Presentation       │           │  Frameworks & Drivers   │
  │   (Controller, View) │           │  ┌───────────────────┐  │
  ├──────────────────────┤           │  │ Interface Adapters│  │
  │   Business Logic     │           │  │ ┌───────────────┐ │  │
  │   (Service)          │           │  │ │  Use Cases    │ │  │
  ├──────────────────────┤           │  │ │ ┌───────────┐ │ │  │
  │   Data Access        │           │  │ │ │ Entities  │ │ │  │
  │   (Repository)       │           │  │ │ └───────────┘ │ │  │
  └──────────────────────┘           │  │ └───────────────┘ │  │
         │                           │  └───────────────────┘  │
         ▼                           └─────────────────────────┘
      Database
                                     의존성: 바깥 → 안쪽
  의존성: 위 → 아래                   (동심원 구조)
  (수직 계층 구조)
```

### 언제 어떤 것을 선택할까?

```
┌─────────────────────────────────────────────────────────────────┐
│                    아키텍처 선택 가이드                            │
│                                                                 │
│  Layered Architecture를 선택:                                    │
│  ─────────────────────────────                                  │
│  [O] 팀이 3명 이하의 소규모 프로젝트                               │
│  [O] 단순한 CRUD 중심 애플리케이션                                 │
│  [O] 빠른 MVP/프로토타입 개발이 필요한 경우                        │
│  [O] 팀의 아키텍처 경험이 적은 경우                                │
│  [O] 프레임워크(Django, Spring Boot)의 기본 구조를 따르는 경우      │
│  [O] 프로젝트 수명이 2년 미만                                     │
│                                                                 │
│  Clean Architecture를 선택:                                      │
│  ─────────────────────────                                      │
│  [O] 중~대규모 프로젝트 (5명+)                                    │
│  [O] 복잡한 비즈니스 도메인                                       │
│  [O] 높은 테스트 커버리지가 요구되는 경우                          │
│  [O] 프레임워크/DB 교체 가능성이 있는 경우                         │
│  [O] 장기 유지보수가 예상되는 경우 (3년+)                          │
│  [O] 마이크로서비스의 개별 서비스 설계                             │
│                                                                 │
│  [팁] 확실하지 않으면 Layered로 시작하고,                         │
│       프로젝트가 커지면 Clean으로 진화시키세요.                     │
│       처음부터 완벽한 아키텍처를 선택할 필요는 없습니다.             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **이해하기 쉬움** | 직관적인 수직 구조로 누구나 빠르게 이해 |
| **널리 알려짐** | 대부분의 개발자가 이미 익숙한 패턴 |
| **프레임워크 지원** | Django, Spring, ASP.NET 등이 기본 제공 |
| **빠른 개발** | 초기 설정이 적고 개발 속도가 빠름 |
| **관심사 분리** | 기본적인 역할 분리가 가능 |
| **독립적 개발** | 계층별로 나눠서 개발 가능 |
| **교체 용이성** | 같은 계층 내 구현체 교체가 비교적 쉬움 |

### 단점

| 단점 | 설명 |
|------|------|
| **Sink Hole 문제** | 단순 패스스루(통과만 하는) 계층이 생김 |
| **모놀리식 경향** | 계층이 서로 강하게 결합될 수 있음 |
| **DB 중심 설계** | DAL이 기반이 되어 DB 스키마에 종속되기 쉬움 |
| **테스트 어려움** | 계층 간 강한 결합 시 단위 테스트 어려움 |
| **횡단 관심사** | 로깅, 인증 등이 여러 계층에 걸쳐 분산 |
| **확장의 한계** | 대규모 프로젝트에서 구조적 한계 |

### Sink Hole 안티패턴

```
┌─────────────────────────────────────────────────────────────────┐
│                  Sink Hole 안티패턴 예시                          │
│                                                                 │
│  [문제 상황] 단순 조회에서 모든 계층을 통과만 함                    │
│                                                                 │
│  Controller                                                     │
│  ┌──────────────────────────────────────────────┐               │
│  │  def get_user(id):                           │               │
│  │      return service.get_user(id)  # 그냥 전달  │               │
│  └──────────────────────┬───────────────────────┘               │
│                         ▼                                       │
│  Service (아무 로직도 없이 통과만!)                                │
│  ┌──────────────────────────────────────────────┐               │
│  │  def get_user(id):                           │               │
│  │      return repository.find_by_id(id)  # 전달  │              │
│  └──────────────────────┬───────────────────────┘               │
│                         ▼                                       │
│  Repository                                                     │
│  ┌──────────────────────────────────────────────┐               │
│  │  def find_by_id(id):                         │               │
│  │      return db.query(User).get(id)           │               │
│  └──────────────────────────────────────────────┘               │
│                                                                 │
│  [해결] 20% 규칙을 적용하세요.                                   │
│  전체 요청의 80%는 비즈니스 로직을 포함하고,                       │
│  20% 이하만 단순 패스스루라면 정상입니다.                          │
│  80% 이상이 패스스루라면 아키텍처 재검토가 필요합니다.              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. 실무 팁: 계층을 나누는 기준

### 핵심 질문 체크리스트

```
계층을 나눌 때 스스로에게 물어보세요:

  Q1. "이 코드는 UI가 바뀌어도 변하지 않는가?"
      → Yes: Business Logic Layer 또는 Data Access Layer
      → No:  Presentation Layer

  Q2. "이 코드는 DB가 바뀌어도 변하지 않는가?"
      → Yes: Business Logic Layer 또는 Presentation Layer
      → No:  Data Access Layer

  Q3. "이 코드는 비즈니스 규칙을 구현하는가?"
      → Yes: Business Logic Layer
      → No:  Presentation 또는 Data Access Layer

  Q4. "이 코드는 외부 시스템과 통신하는가?"
      → Yes: Data Access Layer
      → No:  Business Logic 또는 Presentation Layer
```

### 실무에서 자주 하는 실수와 해결책

```
┌─────────────────────────────────────────────────────────────────┐
│                    흔한 실수와 해결책                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  실수 1: Controller에 비즈니스 로직 작성                          │
│  ─────────────────────────────────────                          │
│  [X] Controller에서 가격 계산, 재고 검증 등                      │
│  [O] Service로 이동. Controller는 요청 파싱만                    │
│                                                                 │
│  실수 2: Service에서 SQL 직접 작성                               │
│  ─────────────────────────────────────                          │
│  [X] service.py에서 cursor.execute("SELECT ...")                │
│  [O] Repository에서 SQL 실행, Service는 Repository만 호출       │
│                                                                 │
│  실수 3: Entity를 API 응답으로 직접 반환                          │
│  ─────────────────────────────────────                          │
│  [X] return user (DB 엔티티를 그대로 반환)                       │
│  [O] DTO로 변환 후 반환 (민감 정보 제외, 형식 통일)               │
│                                                                 │
│  실수 4: 순환 참조                                               │
│  ─────────────────────────────────────                          │
│  [X] Service A → Service B → Service A                         │
│  [O] 공통 로직을 별도 서비스로 추출하거나 이벤트 활용             │
│                                                                 │
│  실수 5: Repository에 비즈니스 로직 포함                          │
│  ─────────────────────────────────────                          │
│  [X] Repository에서 할인 계산, 권한 검사 등                      │
│  [O] Repository는 순수 CRUD만. 로직은 Service에서               │
│                                                                 │
│  실수 6: 하나의 거대한 Service 클래스                             │
│  ─────────────────────────────────────                          │
│  [X] ProductService 하나에 30개 메서드                           │
│  [O] ProductQueryService, ProductCommandService 등으로 분리     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 점진적 개선 전략

```
┌─────────────────────────────────────────────────────────────────┐
│               계층형 아키텍처 점진적 개선 로드맵                    │
│                                                                 │
│  Step 1: 기본 3계층 분리                                         │
│  ────────────────────────                                       │
│  Controller / Service / Repository로 분리                        │
│  → 일단 코드를 기능별로 나누는 것부터 시작                         │
│                                                                 │
│  Step 2: 인터페이스 도입                                         │
│  ────────────────────────                                       │
│  Repository를 인터페이스로 추상화                                 │
│  → Mock을 활용한 단위 테스트 작성 가능                             │
│                                                                 │
│  Step 3: DTO 적용                                               │
│  ────────────────────────                                       │
│  API 요청/응답용 DTO와 내부 Entity 분리                           │
│  → API 변경이 내부 모델에 영향을 주지 않음                        │
│                                                                 │
│  Step 4: 도메인 모델 분리                                        │
│  ────────────────────────                                       │
│  비즈니스 규칙을 Entity 자체에 포함                               │
│  → Rich Domain Model 적용                                       │
│                                                                 │
│  Step 5: (선택) Clean Architecture로 진화                        │
│  ────────────────────────                                       │
│  의존성 방향을 역전시키고, 도메인을 핵심으로 재배치                 │
│  → 필요한 경우에만. 모든 프로젝트에 필요하진 않음                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. 정리 및 체크리스트

### 계층형 아키텍처 핵심 정리

```
┌─────────────────────────────────────────────────────────────┐
│               계층형 아키텍처 핵심 원칙 정리                   │
│                                                             │
│  1. 관련 기능을 수평 계층으로 분리한다.                        │
│  2. 각 계층은 바로 아래 계층만 사용한다.                       │
│  3. 하위 계층은 상위 계층을 알지 못한다.                      │
│  4. 계층 간 데이터는 DTO로 전달한다.                         │
│  5. Presentation은 요청/응답만, Business는 규칙만,           │
│     Data Access는 저장/조회만 담당한다.                       │
│  6. 인터페이스를 통해 계층 간 결합도를 낮춘다.                 │
│  7. 가장 중요한 것은 비즈니스 로직의 올바른 위치 배정이다.     │
└─────────────────────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] Controller에 비즈니스 로직이 포함되어 있지 않은가?
- [ ] Service에 SQL이나 DB 접근 코드가 직접 있지 않은가?
- [ ] Repository에 비즈니스 규칙이 포함되어 있지 않은가?
- [ ] Entity를 API 응답으로 직접 반환하고 있지 않은가?
- [ ] 계층 간 순환 참조가 없는가?
- [ ] 각 계층이 독립적으로 테스트 가능한가?
- [ ] Sink Hole (단순 패스스루) 비율이 20% 이하인가?
- [ ] 인터페이스를 통해 Repository를 추상화했는가?
- [ ] DTO를 통해 계층 간 데이터를 전달하는가?
- [ ] 한 Service 클래스가 너무 비대하지 않은가?
- [ ] 횡단 관심사(로깅, 인증)를 적절히 처리하고 있는가?
- [ ] 프로젝트 규모에 맞는 적절한 계층 수를 사용하는가?

---

## 12. 참고 자료

### 공식 문서
- [Microsoft - N계층 아키텍처 스타일](https://learn.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/n-tier)
- [Martin Fowler - PresentationDomainDataLayering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)
- [FastAPI 공식 문서 - 프로젝트 구조](https://fastapi.tiangolo.com/tutorial/)

### 추천 도서
- "Patterns of Enterprise Application Architecture" - Martin Fowler (2002)
- "Software Architecture Patterns" - Mark Richards (O'Reilly)
- "Fundamentals of Software Architecture" - Mark Richards & Neal Ford

### 관련 패턴
- **Clean Architecture**: 의존성 규칙 기반의 진화된 계층 구조
- **Hexagonal Architecture**: 포트와 어댑터 기반 격리
- **CQRS**: 명령과 조회 책임 분리
- **Microservices**: 각 서비스 내부에 Layered 적용 가능

### 관련 문서
- **[MVVM 패턴](./01-mvvm.md)**: UI 아키텍처 패턴
- **[Clean Architecture](./02-clean-architecture.md)**: 의존성 규칙 기반 아키텍처

---

> **이전 문서**: [Clean Architecture (클린 아키텍처)](./02-clean-architecture.md)
