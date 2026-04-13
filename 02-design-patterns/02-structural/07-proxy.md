# Proxy 패턴 (프록시)

> **핵심 의도 한줄 요약:** 다른 객체에 대한 접근을 제어하는 대리자(대변인)를 제공하여, 추가 기능을 수행하는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [프록시의 종류](#5-프록시의-종류)
6. [패턴 적용 전 (Before)](#6-패턴-적용-전-before)
7. [패턴 적용 후 (After)](#7-패턴-적용-후-after)
8. [Python 구현](#8-python-구현)
9. [C# 구현](#9-c-구현)
10. [Proxy vs Decorator 차이점](#10-proxy-vs-decorator-차이점)
11. [실무 예제: 이미지 지연 로딩 프록시](#11-실무-예제-이미지-지연-로딩-프록시)
12. [장단점](#12-장단점)
13. [관련 패턴](#13-관련-패턴)
14. [정리 및 체크리스트](#14-정리-및-체크리스트)

---

## 1. 개요

Proxy 패턴은 **GoF 구조 패턴** 중 하나로, 다른 객체에 대한 **대리자 또는 자리표시자(placeholder)**를 제공합니다. 프록시는 원래 객체와 같은 인터페이스를 가지면서, 원래 객체에 대한 접근을 제어합니다.

핵심 아이디어: "원래 객체를 직접 사용하는 대신, 대리자를 통해 접근함으로써 추가적인 제어를 수행한다."

실무에서 마주치는 상황:
- 무거운 객체의 지연 초기화 (Lazy Loading)
- 접근 권한 제어 (보호 프록시)
- 원격 서비스 호출 감싸기 (원격 프록시)
- 요청 결과 캐싱 (캐싱 프록시)
- 요청 로깅 (로깅 프록시)

---

## 2. 왜 필요한가? - 문제 상황

### 상황 1: 메모리 낭비

고해상도 이미지 뷰어에서 1,000개의 이미지를 로드합니다:

```
각 이미지: ~10MB
1,000개 = ~10GB 메모리!

하지만 화면에 보이는 이미지는 10개뿐...
나머지 990개는 메모리만 차지하고 있음!
```

### 상황 2: 접근 제어

민감한 데이터에 누구나 접근할 수 있습니다:

```python
# 아무나 호출 가능!
admin_service.delete_user("user-123")
admin_service.change_salary("emp-456", 10000000)
```

### 상황 3: 원격 서비스

원격 서비스를 호출할 때 네트워크 에러 처리, 재시도, 타임아웃이 필요합니다:

```python
# 매번 이런 코드를 반복해야 함
try:
    result = remote_service.call(data)
except TimeoutError:
    # 재시도 로직...
except ConnectionError:
    # 폴백 로직...
```

---

## 3. 비유로 이해하기

### 비서와 사장

```
방문자가 사장을 만나고 싶을 때:

직접 접근 (프록시 없음):
[방문자] ──직접──> [사장]
→ 아무나, 아무 때나, 제한 없이 접근

비서를 통한 접근 (프록시):
[방문자] ──요청──> [비서(Proxy)] ──전달──> [사장(Real)]

비서가 하는 일:
1. 일정 확인 (사장이 바쁜지 체크)
2. 방문자 신원 확인 (접근 권한 체크)
3. 요청 내용 기록 (로깅)
4. 반복 문의는 직접 답변 (캐싱)
5. 사장이 부재 시 대리 답변 (폴백)

비서는 사장과 같은 "업무 인터페이스"를 가짐!
방문자는 비서에게 말하든 사장에게 말하든 동일한 방식으로 소통
```

### 신용카드 (프록시) vs 현금 (실체)

```
[신용카드] = 은행 계좌에 대한 프록시

- 같은 인터페이스: "결제하기"
- 접근 제어: 한도 초과 시 거부
- 지연 처리: 즉시 현금을 꺼내지 않고, 나중에 정산
- 로깅: 모든 거래 내역 기록
- 보안: PIN 번호 확인
```

---

## 4. UML 다이어그램

```
┌──────────┐        ┌────────────────────────┐
│  Client  │        │   <<interface>>         │
│          │───────>│      Subject            │
│          │        │────────────────────────│
└──────────┘        │ + request()             │
                    └────────────────────────┘
                                △
                                │ implements
                   ┌────────────┴────────────┐
                   │                          │
          ┌────────────────┐       ┌─────────────────────┐
          │  RealSubject   │       │       Proxy          │
          │────────────────│       │─────────────────────│
          │ + request()    │       │ - realSubject        │──> RealSubject
          │   (실제 로직)  │       │─────────────────────│
          │                │       │ + request()           │
          └────────────────┘       │   // 접근 제어        │
                                   │   // 추가 처리        │
                                   │   realSubject.request()│
                                   │   // 후처리           │
                                   └─────────────────────┘
```

**핵심:**
- Proxy와 RealSubject가 **같은 인터페이스**를 구현
- Proxy가 RealSubject를 **참조**하여 요청을 위임
- 위임 전후에 **추가 로직** 수행

---

## 5. 프록시의 종류

### 1. 가상 프록시 (Virtual Proxy)

```
목적: 무거운 객체의 생성을 지연시킴 (Lazy Loading)
시점: 실제로 필요할 때까지 객체 생성을 미룸

예시: 이미지 로딩
- 썸네일만 먼저 보여주고
- 사용자가 클릭하면 실제 고해상도 이미지 로드
```

### 2. 보호 프록시 (Protection Proxy)

```
목적: 접근 권한에 따라 요청을 필터링
시점: 요청이 들어올 때 권한 확인

예시: 관리자만 삭제 가능
- 일반 사용자: 조회만 허용
- 관리자: 모든 작업 허용
```

### 3. 원격 프록시 (Remote Proxy)

```
목적: 원격 객체에 대한 로컬 대리자
시점: 네트워크 호출 시

예시: 마이크로서비스 클라이언트
- 로컬 메서드처럼 호출하지만
- 실제로는 네트워크를 통해 원격 서비스 호출
```

### 4. 캐싱 프록시 (Caching Proxy)

```
목적: 비용이 큰 연산 결과를 캐시
시점: 동일한 요청이 반복될 때

예시: API 응답 캐싱
- 같은 요청이면 캐시에서 즉시 반환
- 새로운 요청이면 실제 서비스 호출 후 캐시 저장
```

### 5. 로깅 프록시 (Logging Proxy)

```
목적: 서비스 요청 이력을 기록
시점: 모든 요청 전후

예시: API 호출 로깅
- 요청 내용, 응답 결과, 소요 시간 기록
```

---

## 6. 패턴 적용 전 (Before)

```python
# ❌ Before: 클라이언트가 접근 제어, 캐싱, 로깅을 직접 관리

import time


class DatabaseService:
    """데이터베이스 서비스"""

    def query(self, sql: str) -> list[dict]:
        print(f"[DB] 쿼리 실행: {sql}")
        time.sleep(0.5)  # 느린 쿼리 시뮬레이션
        return [{"id": 1, "name": "Alice"}]


# 클라이언트 코드: 매번 직접 관리해야 함
class UserController:
    def __init__(self):
        self.db = DatabaseService()
        self.cache = {}  # 캐시 직접 관리

    def get_users(self, current_user_role: str):
        # 접근 제어: 직접 체크
        if current_user_role not in ("admin", "manager"):
            raise PermissionError("접근 권한이 없습니다")

        # 캐싱: 직접 관리
        cache_key = "all_users"
        if cache_key in self.cache:
            return self.cache[cache_key]

        # 로깅: 직접 기록
        print(f"[LOG] get_users 호출 by {current_user_role}")
        start = time.time()

        # 실제 쿼리
        result = self.db.query("SELECT * FROM users")

        # 로깅: 시간 기록
        elapsed = time.time() - start
        print(f"[LOG] get_users 완료: {elapsed:.2f}s")

        # 캐시 저장
        self.cache[cache_key] = result
        return result

# 문제:
# 1. 비즈니스 로직이 접근제어/캐싱/로깅과 섞임
# 2. 다른 서비스에서도 같은 코드를 반복해야 함
# 3. 횡단 관심사(cross-cutting concerns)가 분리되지 않음
```

---

## 7. 패턴 적용 후 (After)

```python
# ✅ After: Proxy 패턴으로 관심사 분리

from abc import ABC, abstractmethod


class DatabaseService(ABC):
    """Subject 인터페이스"""

    @abstractmethod
    def query(self, sql: str) -> list[dict]:
        pass


class RealDatabaseService(DatabaseService):
    """Real Subject: 실제 DB 서비스"""

    def query(self, sql: str) -> list[dict]:
        print(f"[DB] 쿼리 실행: {sql}")
        return [{"id": 1, "name": "Alice"}]


class CachingProxy(DatabaseService):
    """캐싱 프록시"""

    def __init__(self, service: DatabaseService):
        self._service = service
        self._cache = {}

    def query(self, sql: str) -> list[dict]:
        if sql in self._cache:
            print(f"[CACHE] 히트: {sql}")
            return self._cache[sql]
        result = self._service.query(sql)
        self._cache[sql] = result
        return result


class AccessControlProxy(DatabaseService):
    """보호 프록시"""

    def __init__(self, service: DatabaseService, user_role: str):
        self._service = service
        self._role = user_role

    def query(self, sql: str) -> list[dict]:
        if self._role not in ("admin", "manager"):
            raise PermissionError("접근 권한이 없습니다")
        return self._service.query(sql)


# 클라이언트: 깔끔!
service = RealDatabaseService()
service = CachingProxy(service)
service = AccessControlProxy(service, "admin")
result = service.query("SELECT * FROM users")
```

---

## 8. Python 구현

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import Optional, Any
import time
import hashlib


# ===== Subject 인터페이스 =====
class DataService(ABC):
    """데이터 서비스 인터페이스 (Subject)"""

    @abstractmethod
    def get(self, key: str) -> Optional[dict]:
        pass

    @abstractmethod
    def set(self, key: str, value: dict) -> bool:
        pass

    @abstractmethod
    def delete(self, key: str) -> bool:
        pass

    @abstractmethod
    def list_all(self) -> list[dict]:
        pass


# ===== Real Subject =====
class DatabaseService(DataService):
    """실제 데이터베이스 서비스 (Real Subject)"""

    def __init__(self):
        self._data: dict[str, dict] = {
            "user:1": {"id": 1, "name": "Alice", "role": "admin"},
            "user:2": {"id": 2, "name": "Bob", "role": "user"},
            "user:3": {"id": 3, "name": "Charlie", "role": "user"},
        }

    def get(self, key: str) -> Optional[dict]:
        time.sleep(0.1)  # DB 지연 시뮬레이션
        print(f"  [DB] GET {key}")
        return self._data.get(key)

    def set(self, key: str, value: dict) -> bool:
        time.sleep(0.1)
        print(f"  [DB] SET {key}")
        self._data[key] = value
        return True

    def delete(self, key: str) -> bool:
        time.sleep(0.1)
        print(f"  [DB] DELETE {key}")
        if key in self._data:
            del self._data[key]
            return True
        return False

    def list_all(self) -> list[dict]:
        time.sleep(0.2)
        print(f"  [DB] LIST ALL ({len(self._data)} items)")
        return list(self._data.values())


# ===== Proxy 1: 캐싱 프록시 =====
@dataclass
class CacheEntry:
    value: Any
    expires_at: datetime


class CachingProxy(DataService):
    """캐싱 프록시: 자주 조회되는 데이터를 캐시"""

    def __init__(self, service: DataService,
                 ttl_seconds: int = 60):
        self._service = service
        self._cache: dict[str, CacheEntry] = {}
        self._ttl = timedelta(seconds=ttl_seconds)
        self._stats = {"hits": 0, "misses": 0}

    def get(self, key: str) -> Optional[dict]:
        # 캐시 확인
        if key in self._cache:
            entry = self._cache[key]
            if datetime.now() < entry.expires_at:
                self._stats["hits"] += 1
                print(f"  [CACHE] 히트: {key}")
                return entry.value
            else:
                del self._cache[key]
                print(f"  [CACHE] 만료: {key}")

        # 캐시 미스 -> 실제 서비스 호출
        self._stats["misses"] += 1
        result = self._service.get(key)

        if result is not None:
            self._cache[key] = CacheEntry(
                value=result,
                expires_at=datetime.now() + self._ttl
            )
        return result

    def set(self, key: str, value: dict) -> bool:
        # 쓰기 시 캐시 무효화
        if key in self._cache:
            del self._cache[key]
            print(f"  [CACHE] 무효화: {key}")
        return self._service.set(key, value)

    def delete(self, key: str) -> bool:
        if key in self._cache:
            del self._cache[key]
            print(f"  [CACHE] 무효화: {key}")
        return self._service.delete(key)

    def list_all(self) -> list[dict]:
        return self._service.list_all()

    @property
    def stats(self) -> dict:
        total = self._stats["hits"] + self._stats["misses"]
        hit_rate = (self._stats["hits"] / total * 100
                    if total > 0 else 0)
        return {**self._stats, "hit_rate": f"{hit_rate:.1f}%"}


# ===== Proxy 2: 보호 프록시 =====
@dataclass
class User:
    id: str
    name: str
    role: str  # "admin", "manager", "user"


class ProtectionProxy(DataService):
    """보호 프록시: 권한에 따라 접근을 제한"""

    # 역할별 허용 작업
    PERMISSIONS = {
        "admin": {"get", "set", "delete", "list_all"},
        "manager": {"get", "set", "list_all"},
        "user": {"get", "list_all"},
    }

    def __init__(self, service: DataService, current_user: User):
        self._service = service
        self._user = current_user

    def _check_permission(self, operation: str) -> None:
        allowed = self.PERMISSIONS.get(self._user.role, set())
        if operation not in allowed:
            raise PermissionError(
                f"[AUTH] 권한 없음: {self._user.name}"
                f"({self._user.role})이 '{operation}' 시도"
            )
        print(f"  [AUTH] 허용: {self._user.name}"
              f"({self._user.role}) -> {operation}")

    def get(self, key: str) -> Optional[dict]:
        self._check_permission("get")
        return self._service.get(key)

    def set(self, key: str, value: dict) -> bool:
        self._check_permission("set")
        return self._service.set(key, value)

    def delete(self, key: str) -> bool:
        self._check_permission("delete")
        return self._service.delete(key)

    def list_all(self) -> list[dict]:
        self._check_permission("list_all")
        return self._service.list_all()


# ===== Proxy 3: 로깅 프록시 =====
class LoggingProxy(DataService):
    """로깅 프록시: 모든 작업을 기록"""

    def __init__(self, service: DataService):
        self._service = service
        self._log: list[dict] = []

    def _log_operation(self, operation: str, key: str = "",
                       elapsed: float = 0, success: bool = True):
        entry = {
            "timestamp": datetime.now().isoformat(),
            "operation": operation,
            "key": key,
            "elapsed_ms": round(elapsed * 1000, 2),
            "success": success,
        }
        self._log.append(entry)
        status = "OK" if success else "FAIL"
        print(f"  [LOG] {operation} {key} -> "
              f"{status} ({elapsed * 1000:.1f}ms)")

    def get(self, key: str) -> Optional[dict]:
        start = time.perf_counter()
        try:
            result = self._service.get(key)
            self._log_operation("GET", key,
                               time.perf_counter() - start,
                               result is not None)
            return result
        except Exception as e:
            self._log_operation("GET", key,
                               time.perf_counter() - start, False)
            raise

    def set(self, key: str, value: dict) -> bool:
        start = time.perf_counter()
        try:
            result = self._service.set(key, value)
            self._log_operation("SET", key,
                               time.perf_counter() - start, result)
            return result
        except Exception as e:
            self._log_operation("SET", key,
                               time.perf_counter() - start, False)
            raise

    def delete(self, key: str) -> bool:
        start = time.perf_counter()
        try:
            result = self._service.delete(key)
            self._log_operation("DELETE", key,
                               time.perf_counter() - start, result)
            return result
        except Exception as e:
            self._log_operation("DELETE", key,
                               time.perf_counter() - start, False)
            raise

    def list_all(self) -> list[dict]:
        start = time.perf_counter()
        result = self._service.list_all()
        self._log_operation("LIST_ALL", "",
                           time.perf_counter() - start)
        return result

    @property
    def access_log(self) -> list[dict]:
        return self._log.copy()


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 프록시 체인 구성 (안에서 바깥으로)
    # 실제 DB -> 캐싱 -> 보호 -> 로깅
    admin_user = User("u1", "Alice", "admin")
    normal_user = User("u2", "Bob", "user")

    # 관리자용 서비스
    print("=== 관리자 서비스 구성 ===")
    db = DatabaseService()
    cached = CachingProxy(db, ttl_seconds=300)
    protected = ProtectionProxy(cached, admin_user)
    service = LoggingProxy(protected)

    # GET: 캐시 미스 -> DB 조회
    print("\n--- GET (캐시 미스) ---")
    result = service.get("user:1")
    print(f"결과: {result}")

    # GET: 캐시 히트
    print("\n--- GET (캐시 히트) ---")
    result = service.get("user:1")
    print(f"결과: {result}")

    # SET: 캐시 무효화 + DB 저장
    print("\n--- SET ---")
    service.set("user:4", {"id": 4, "name": "Dave", "role": "user"})

    # DELETE: 관리자만 가능
    print("\n--- DELETE (관리자) ---")
    service.delete("user:4")

    # 일반 사용자로 교체
    print("\n=== 일반 사용자 서비스 ===")
    user_service = LoggingProxy(
        ProtectionProxy(cached, normal_user)
    )

    # GET: 일반 사용자도 가능
    print("\n--- GET (일반 사용자) ---")
    result = user_service.get("user:2")

    # DELETE: 일반 사용자는 불가!
    print("\n--- DELETE (일반 사용자) ---")
    try:
        user_service.delete("user:2")
    except PermissionError as e:
        print(f"  권한 오류: {e}")

    # 캐시 통계
    print(f"\n=== 캐시 통계 ===")
    print(cached.stats)
```

---

## 9. C# 구현

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;

// ===== Subject 인터페이스 =====
public interface IDataService
{
    Dictionary<string, object> Get(string key);
    bool Set(string key, Dictionary<string, object> value);
    bool Delete(string key);
    List<Dictionary<string, object>> ListAll();
}

// ===== Real Subject =====
public class DatabaseService : IDataService
{
    private readonly Dictionary<string, Dictionary<string, object>> _data = new()
    {
        ["user:1"] = new() { ["id"] = 1, ["name"] = "Alice", ["role"] = "admin" },
        ["user:2"] = new() { ["id"] = 2, ["name"] = "Bob", ["role"] = "user" },
    };

    public Dictionary<string, object> Get(string key)
    {
        System.Threading.Thread.Sleep(100);
        Console.WriteLine($"  [DB] GET {key}");
        return _data.ContainsKey(key) ? _data[key] : null;
    }

    public bool Set(string key, Dictionary<string, object> value)
    {
        System.Threading.Thread.Sleep(100);
        Console.WriteLine($"  [DB] SET {key}");
        _data[key] = value;
        return true;
    }

    public bool Delete(string key)
    {
        System.Threading.Thread.Sleep(100);
        Console.WriteLine($"  [DB] DELETE {key}");
        return _data.Remove(key);
    }

    public List<Dictionary<string, object>> ListAll()
    {
        System.Threading.Thread.Sleep(200);
        Console.WriteLine($"  [DB] LIST ALL ({_data.Count} items)");
        return _data.Values.ToList();
    }
}

// ===== 캐싱 프록시 =====
public class CachingProxy : IDataService
{
    private readonly IDataService _service;
    private readonly Dictionary<string, (Dictionary<string, object> Value,
                                         DateTime ExpiresAt)> _cache = new();
    private readonly TimeSpan _ttl;
    private int _hits, _misses;

    public CachingProxy(IDataService service, int ttlSeconds = 60)
    {
        _service = service;
        _ttl = TimeSpan.FromSeconds(ttlSeconds);
    }

    public Dictionary<string, object> Get(string key)
    {
        if (_cache.TryGetValue(key, out var entry)
            && DateTime.Now < entry.ExpiresAt)
        {
            _hits++;
            Console.WriteLine($"  [CACHE] 히트: {key}");
            return entry.Value;
        }

        _misses++;
        var result = _service.Get(key);
        if (result != null)
        {
            _cache[key] = (result, DateTime.Now + _ttl);
        }
        return result;
    }

    public bool Set(string key, Dictionary<string, object> value)
    {
        _cache.Remove(key);
        Console.WriteLine($"  [CACHE] 무효화: {key}");
        return _service.Set(key, value);
    }

    public bool Delete(string key)
    {
        _cache.Remove(key);
        Console.WriteLine($"  [CACHE] 무효화: {key}");
        return _service.Delete(key);
    }

    public List<Dictionary<string, object>> ListAll()
    {
        return _service.ListAll();
    }

    public string GetStats()
    {
        var total = _hits + _misses;
        var rate = total > 0 ? (double)_hits / total * 100 : 0;
        return $"Hits: {_hits}, Misses: {_misses}, Rate: {rate:F1}%";
    }
}

// ===== 보호 프록시 =====
public record UserContext(string Id, string Name, string Role);

public class ProtectionProxy : IDataService
{
    private readonly IDataService _service;
    private readonly UserContext _user;

    private static readonly Dictionary<string, HashSet<string>> Permissions = new()
    {
        ["admin"] = new() { "Get", "Set", "Delete", "ListAll" },
        ["manager"] = new() { "Get", "Set", "ListAll" },
        ["user"] = new() { "Get", "ListAll" },
    };

    public ProtectionProxy(IDataService service, UserContext user)
    {
        _service = service;
        _user = user;
    }

    private void CheckPermission(string operation)
    {
        if (!Permissions.TryGetValue(_user.Role, out var allowed)
            || !allowed.Contains(operation))
        {
            throw new UnauthorizedAccessException(
                $"[AUTH] {_user.Name}({_user.Role}): " +
                $"'{operation}' 권한 없음");
        }
        Console.WriteLine(
            $"  [AUTH] 허용: {_user.Name}({_user.Role}) -> {operation}");
    }

    public Dictionary<string, object> Get(string key)
    {
        CheckPermission("Get");
        return _service.Get(key);
    }

    public bool Set(string key, Dictionary<string, object> value)
    {
        CheckPermission("Set");
        return _service.Set(key, value);
    }

    public bool Delete(string key)
    {
        CheckPermission("Delete");
        return _service.Delete(key);
    }

    public List<Dictionary<string, object>> ListAll()
    {
        CheckPermission("ListAll");
        return _service.ListAll();
    }
}

// ===== 로깅 프록시 =====
public class LoggingProxy : IDataService
{
    private readonly IDataService _service;

    public LoggingProxy(IDataService service)
    {
        _service = service;
    }

    private T LogOperation<T>(string operation, string key,
                               Func<T> action)
    {
        var sw = Stopwatch.StartNew();
        try
        {
            var result = action();
            sw.Stop();
            Console.WriteLine(
                $"  [LOG] {operation} {key} -> OK ({sw.ElapsedMilliseconds}ms)");
            return result;
        }
        catch (Exception ex)
        {
            sw.Stop();
            Console.WriteLine(
                $"  [LOG] {operation} {key} -> FAIL ({sw.ElapsedMilliseconds}ms): {ex.Message}");
            throw;
        }
    }

    public Dictionary<string, object> Get(string key)
        => LogOperation("GET", key, () => _service.Get(key));

    public bool Set(string key, Dictionary<string, object> value)
        => LogOperation("SET", key, () => _service.Set(key, value));

    public bool Delete(string key)
        => LogOperation("DELETE", key, () => _service.Delete(key));

    public List<Dictionary<string, object>> ListAll()
        => LogOperation("LIST_ALL", "*", () => _service.ListAll());
}

// ===== 가상 프록시 (Lazy Loading) =====
public class LazyServiceProxy : IDataService
{
    private IDataService _service;
    private readonly Func<IDataService> _factory;

    public LazyServiceProxy(Func<IDataService> factory)
    {
        _factory = factory;
    }

    private IDataService GetService()
    {
        if (_service == null)
        {
            Console.WriteLine("  [LAZY] 서비스 초기화 중...");
            _service = _factory();
        }
        return _service;
    }

    public Dictionary<string, object> Get(string key)
        => GetService().Get(key);

    public bool Set(string key, Dictionary<string, object> value)
        => GetService().Set(key, value);

    public bool Delete(string key)
        => GetService().Delete(key);

    public List<Dictionary<string, object>> ListAll()
        => GetService().ListAll();
}

// ===== 사용 =====
public class Program
{
    public static void Main()
    {
        var admin = new UserContext("1", "Alice", "admin");

        // 프록시 체인: Lazy -> Caching -> Protection -> Logging
        IDataService service = new LazyServiceProxy(
            () => new DatabaseService());
        var cachingProxy = new CachingProxy(service, ttlSeconds: 300);
        IDataService protectedService = new ProtectionProxy(cachingProxy, admin);
        IDataService finalService = new LoggingProxy(protectedService);

        Console.WriteLine("=== GET (첫 번째: lazy init + cache miss) ===");
        var result = finalService.Get("user:1");

        Console.WriteLine("\n=== GET (두 번째: cache hit) ===");
        result = finalService.Get("user:1");

        Console.WriteLine($"\n캐시 통계: {cachingProxy.GetStats()}");
    }
}
```

---

## 10. Proxy vs Decorator 차이점

Proxy와 Decorator는 구조적으로 매우 유사하지만 **의도**가 다릅니다.

| 구분 | Proxy | Decorator |
|------|-------|-----------|
| **의도** | 접근 **제어** | 기능 **추가** |
| **대상 생명주기** | Proxy가 대상 생성/관리 가능 | 클라이언트가 대상 생성 후 전달 |
| **대상 인식** | Proxy가 대상의 구체 클래스를 알 수 있음 | Decorator는 인터페이스만 알면 됨 |
| **목적** | 접근 제어, 지연 로딩, 캐싱 | 동적 기능 확장 |
| **대상과의 관계** | 1:1 (특정 서비스의 대리) | N:1 (여러 데코레이터 자유 조합) |

### 코드로 보는 차이

```python
# Proxy: 대상 객체의 생성을 직접 제어할 수 있음
class ImageProxy:
    def __init__(self, filename: str):
        self._filename = filename
        self._real_image = None  # 아직 생성하지 않음!

    def display(self):
        if self._real_image is None:
            self._real_image = RealImage(self._filename)  # 직접 생성
        self._real_image.display()


# Decorator: 이미 생성된 대상 객체를 받아서 기능 추가
class BorderDecorator:
    def __init__(self, image: Image):  # 외부에서 전달받음
        self._image = image

    def display(self):
        self._draw_border()
        self._image.display()
```

```
핵심 차이:
Proxy  → "대상에 대한 접근을 제어한다" (대리인)
Decorator → "대상에 새로운 기능을 추가한다" (장식자)
```

---

## 11. 실무 예제: 이미지 지연 로딩 프록시

### Python - 이미지 뷰어

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional
import time


# ===== Subject =====
class Image(ABC):
    """이미지 인터페이스"""

    @abstractmethod
    def display(self) -> str:
        pass

    @abstractmethod
    def get_info(self) -> dict:
        pass

    @property
    @abstractmethod
    def width(self) -> int:
        pass

    @property
    @abstractmethod
    def height(self) -> int:
        pass


# ===== Real Subject =====
class HighResImage(Image):
    """고해상도 이미지 (Real Subject)

    생성 시 파일을 로드하므로 시간과 메모리가 많이 소요됩니다.
    """

    def __init__(self, filename: str):
        self._filename = filename
        self._data: Optional[bytes] = None
        self._width = 0
        self._height = 0
        self._load()  # 생성 즉시 로드!

    def _load(self):
        """파일에서 이미지 로드 (무거운 작업)"""
        print(f"  [HighRes] '{self._filename}' 로딩 중... "
              f"(수 MB 데이터)")
        time.sleep(0.5)  # 파일 I/O 시뮬레이션
        self._data = b"x" * 10_000_000  # 10MB 시뮬레이션
        self._width = 3840
        self._height = 2160
        print(f"  [HighRes] 로딩 완료: "
              f"{self._width}x{self._height}")

    def display(self) -> str:
        return (f"[이미지 표시] {self._filename} "
                f"({self._width}x{self._height})")

    def get_info(self) -> dict:
        return {
            "filename": self._filename,
            "width": self._width,
            "height": self._height,
            "size_bytes": len(self._data) if self._data else 0,
            "loaded": True,
        }

    @property
    def width(self) -> int:
        return self._width

    @property
    def height(self) -> int:
        return self._height


# ===== Virtual Proxy =====
class ImageProxy(Image):
    """이미지 프록시 (Virtual Proxy)

    실제 이미지는 display()가 호출될 때까지 로드하지 않습니다.
    썸네일이나 메타데이터는 바로 제공합니다.
    """

    def __init__(self, filename: str):
        self._filename = filename
        self._real_image: Optional[HighResImage] = None
        # 메타데이터만 미리 로드 (가벼운 작업)
        self._metadata = self._load_metadata()

    def _load_metadata(self) -> dict:
        """메타데이터만 로드 (가벼운 작업)"""
        # 실제로는 파일 헤더만 읽어서 크기 정보 추출
        return {
            "filename": self._filename,
            "width": 3840,
            "height": 2160,
            "size_bytes": 10_000_000,
            "loaded": False,
        }

    def _ensure_loaded(self):
        """필요할 때만 실제 이미지 로드"""
        if self._real_image is None:
            print(f"  [Proxy] 실제 이미지 로딩 시작...")
            self._real_image = HighResImage(self._filename)

    def display(self) -> str:
        """실제 표시가 필요할 때만 로드"""
        self._ensure_loaded()
        return self._real_image.display()

    def get_info(self) -> dict:
        """메타데이터는 로드 없이 즉시 반환"""
        if self._real_image:
            return self._real_image.get_info()
        return self._metadata

    @property
    def width(self) -> int:
        return self._metadata["width"]

    @property
    def height(self) -> int:
        return self._metadata["height"]

    @property
    def is_loaded(self) -> bool:
        return self._real_image is not None


# ===== 이미지 갤러리 =====
class ImageGallery:
    """이미지 갤러리: Proxy를 사용한 지연 로딩"""

    def __init__(self):
        self._images: list[ImageProxy] = []
        self._current_index = 0

    def add_image(self, filename: str):
        # 프록시만 생성 (실제 이미지 로드 X)
        self._images.append(ImageProxy(filename))

    def show_thumbnails(self):
        """썸네일 목록 표시 (실제 이미지 로드 없이 메타데이터만 사용)"""
        print("\n=== 썸네일 목록 ===")
        for i, image in enumerate(self._images):
            info = image.get_info()
            loaded = "로드됨" if info["loaded"] else "미로드"
            marker = " >" if i == self._current_index else "  "
            print(f"{marker}[{i}] {info['filename']} "
                  f"({info['width']}x{info['height']}) [{loaded}]")

    def view_image(self, index: int) -> str:
        """특정 이미지를 전체 크기로 보기 (이때 실제 로드!)"""
        if 0 <= index < len(self._images):
            self._current_index = index
            print(f"\n=== 이미지 보기: [{index}] ===")
            return self._images[index].display()
        return "이미지를 찾을 수 없습니다."

    def get_memory_stats(self) -> dict:
        """메모리 사용 통계"""
        total = len(self._images)
        loaded = sum(1 for img in self._images if img.is_loaded)
        unloaded = total - loaded

        # 프록시만 있으면 메타데이터 크기만 사용
        proxy_memory = total * 100  # 메타데이터 ~100bytes
        loaded_memory = loaded * 10_000_000  # 로드된 이미지 ~10MB

        return {
            "total_images": total,
            "loaded": loaded,
            "unloaded": unloaded,
            "proxy_memory": proxy_memory,
            "loaded_memory": loaded_memory,
            "total_memory": proxy_memory + loaded_memory,
            "saved_memory": unloaded * 10_000_000,  # 미로드 이미지 절감량
        }


# ===== 사용 예시 =====
if __name__ == "__main__":
    gallery = ImageGallery()

    # 1,000개의 이미지 등록 (프록시만 생성, 실제 로드 X)
    print("=== 이미지 등록 (프록시만 생성) ===")
    start = time.perf_counter()

    for i in range(1000):
        gallery.add_image(f"photo_{i:04d}.jpg")

    elapsed = time.perf_counter() - start
    print(f"1,000개 이미지 등록: {elapsed:.4f}초 (실제 로드 없음!)")

    # 썸네일 표시 (처음 5개만 표시)
    gallery._images = gallery._images[:10]  # 데모용 축소
    gallery.show_thumbnails()

    # 특정 이미지 보기 (이때만 실제 로드!)
    result = gallery.view_image(3)
    print(result)

    # 다시 썸네일 보기 (3번은 이미 로드됨)
    gallery.show_thumbnails()

    # 같은 이미지 다시 보기 (이미 로드되어 있으므로 즉시 표시)
    print("\n=== 같은 이미지 다시 보기 ===")
    result = gallery.view_image(3)
    print(result)

    # 메모리 통계
    stats = gallery.get_memory_stats()
    print(f"\n{'=' * 50}")
    print("메모리 통계")
    print(f"{'=' * 50}")
    print(f"총 이미지: {stats['total_images']}")
    print(f"로드됨: {stats['loaded']}")
    print(f"미로드: {stats['unloaded']}")
    print(f"사용 메모리: {stats['total_memory']:,} bytes")
    print(f"절감 메모리: {stats['saved_memory']:,} bytes "
          f"({stats['saved_memory'] / 1_000_000:.0f} MB)")
```

### C# - Lazy Loading Proxy

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Threading;

public interface IImage
{
    string Display();
    ImageInfo GetInfo();
    int Width { get; }
    int Height { get; }
}

public record ImageInfo(
    string Filename, int Width, int Height,
    long SizeBytes, bool IsLoaded);

public class HighResImage : IImage
{
    private readonly string _filename;
    private byte[] _data;

    public int Width { get; private set; }
    public int Height { get; private set; }

    public HighResImage(string filename)
    {
        _filename = filename;
        Load();
    }

    private void Load()
    {
        Console.WriteLine($"  [HighRes] '{_filename}' 로딩 중...");
        Thread.Sleep(500); // 파일 I/O 시뮬레이션
        _data = new byte[10_000_000]; // 10MB
        Width = 3840;
        Height = 2160;
        Console.WriteLine($"  [HighRes] 로딩 완료");
    }

    public string Display()
        => $"[이미지 표시] {_filename} ({Width}x{Height})";

    public ImageInfo GetInfo()
        => new(_filename, Width, Height, _data?.Length ?? 0, true);
}

public class ImageProxy : IImage
{
    private readonly string _filename;
    private HighResImage _realImage;

    public int Width => 3840;  // 메타데이터에서 가져옴
    public int Height => 2160;
    public bool IsLoaded => _realImage != null;

    public ImageProxy(string filename)
    {
        _filename = filename;
        // 실제 이미지는 로드하지 않음!
    }

    private void EnsureLoaded()
    {
        if (_realImage == null)
        {
            Console.WriteLine($"  [Proxy] 지연 로딩 시작...");
            _realImage = new HighResImage(_filename);
        }
    }

    public string Display()
    {
        EnsureLoaded(); // 필요할 때만 로드!
        return _realImage.Display();
    }

    public ImageInfo GetInfo()
    {
        if (_realImage != null)
            return _realImage.GetInfo();
        return new(_filename, Width, Height, 10_000_000, false);
    }
}

public class ImageGallery
{
    private readonly List<ImageProxy> _images = new();

    public void AddImage(string filename)
    {
        _images.Add(new ImageProxy(filename)); // 프록시만 생성
    }

    public void ShowThumbnails()
    {
        Console.WriteLine("\n=== 썸네일 목록 ===");
        for (int i = 0; i < _images.Count; i++)
        {
            var info = _images[i].GetInfo();
            var status = info.IsLoaded ? "로드됨" : "미로드";
            Console.WriteLine(
                $"  [{i}] {info.Filename} " +
                $"({info.Width}x{info.Height}) [{status}]");
        }
    }

    public string ViewImage(int index)
    {
        if (index >= 0 && index < _images.Count)
        {
            Console.WriteLine($"\n=== 이미지 보기: [{index}] ===");
            return _images[index].Display(); // 이때 로드!
        }
        return "이미지를 찾을 수 없습니다.";
    }

    public (int loaded, int total) GetLoadStats()
    {
        int loaded = 0;
        foreach (var img in _images)
            if (img.IsLoaded) loaded++;
        return (loaded, _images.Count);
    }
}

public class Program
{
    public static void Main()
    {
        var gallery = new ImageGallery();

        // 100개 이미지 등록 (프록시만 생성)
        var sw = Stopwatch.StartNew();
        for (int i = 0; i < 100; i++)
            gallery.AddImage($"photo_{i:D4}.jpg");
        sw.Stop();
        Console.WriteLine(
            $"100개 이미지 등록: {sw.ElapsedMilliseconds}ms (로드 없음)");

        // 썸네일 (로드 없음)
        gallery.ShowThumbnails();

        // 하나만 실제 보기 (이때만 로드)
        var result = gallery.ViewImage(5);
        Console.WriteLine(result);

        // 통계
        var (loaded, total) = gallery.GetLoadStats();
        Console.WriteLine($"\n로드: {loaded}/{total} " +
                          $"(절감: {(total - loaded) * 10}MB)");
    }
}
```

---

## 12. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **접근 제어** | 보안, 권한 관리를 투명하게 추가 |
| **지연 초기화** | 무거운 객체를 필요할 때만 생성하여 메모리/시간 절약 |
| **캐싱** | 반복 요청의 성능을 대폭 향상 |
| **투명성** | 클라이언트가 프록시와 실체를 구분하지 않음 |
| **관심사 분리** | 비즈니스 로직과 횡단 관심사를 분리 |

### 단점

| 단점 | 설명 |
|------|------|
| **복잡도 증가** | 새로운 클래스 추가로 코드가 복잡해짐 |
| **응답 지연** | 프록시 레이어를 거치므로 약간의 오버헤드 |
| **디버깅 어려움** | 여러 프록시 레이어에서 문제 추적이 어려울 수 있음 |

---

## 13. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Adapter** | 다른 인터페이스 제공 vs Proxy는 같은 인터페이스 |
| **Decorator** | 기능 추가 vs Proxy는 접근 제어 |
| **Facade** | 복잡한 서브시스템 단순화 vs Proxy는 동일 인터페이스의 대리 |
| **Flyweight** | Proxy가 Flyweight의 생성을 지연시킬 수 있음 |

---

## 14. 정리 및 체크리스트

### 핵심 정리

```
Proxy 패턴 = "대상에 대한 접근을 제어하는 대리자"

When: 객체에 대한 접근을 제어하거나 추가 처리가 필요할 때
How:  동일한 인터페이스를 가진 프록시 클래스가 실제 객체를 감싸기
Why:  실제 객체를 수정하지 않고 접근 제어/지연 로딩/캐싱 등 추가
```

### 적용 체크리스트

- [ ] 어떤 종류의 프록시가 필요한가? (가상/보호/원격/캐싱/로깅)
- [ ] 프록시와 실체가 같은 인터페이스를 공유하는가?
- [ ] 프록시가 실체의 생명주기를 관리하는가? (가상 프록시)
- [ ] Proxy와 Decorator를 혼동하지 않았는가?
- [ ] 프록시 체인의 순서가 올바른가?

### 프록시 종류별 사용 시나리오

| 종류 | 시나리오 |
|------|---------|
| **가상 프록시** | 무거운 객체 지연 로딩, 이미지/비디오 로드 |
| **보호 프록시** | API 접근 제어, 권한 관리, 인증 |
| **원격 프록시** | 마이크로서비스 클라이언트, gRPC, REST 클라이언트 |
| **캐싱 프록시** | DB 쿼리 캐싱, API 응답 캐싱 |
| **로깅 프록시** | API 호출 기록, 감사 로그(audit log) |

### Python의 프록시 관련 기능

```python
# 1. __getattr__을 활용한 동적 프록시
class DynamicProxy:
    def __init__(self, target):
        self._target = target

    def __getattr__(self, name):
        # 모든 속성 접근을 target에 위임
        attr = getattr(self._target, name)
        if callable(attr):
            def wrapper(*args, **kwargs):
                print(f"[Proxy] {name}() 호출")
                return attr(*args, **kwargs)
            return wrapper
        return attr


# 2. property를 활용한 지연 로딩
class LazyProperty:
    def __init__(self, func):
        self.func = func

    def __set_name__(self, owner, name):
        self.attr_name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.func(obj)
        setattr(obj, self.attr_name, value)  # 캐시
        return value

class ExpensiveResource:
    @LazyProperty
    def data(self):
        print("무거운 데이터 로딩...")
        return list(range(1000000))
```

> **기억하세요:** "Proxy는 같은 인터페이스로 접근을 제어하고, Decorator는 같은 인터페이스로 기능을 추가합니다. 둘 다 래핑하지만, 목적이 다릅니다."
