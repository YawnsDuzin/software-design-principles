# 높은 응집도 (High Cohesion)

> **"하나의 클래스는 하나의 명확한 목적을 가져야 한다. 양말 서랍에는 양말만 넣자."**

## 목차

1. [정의와 개념](#1-정의와-개념)
2. [비유로 이해하기](#2-비유로-이해하기)
3. [응집도의 종류](#3-응집도의-종류)
4. [나쁜 예제: 낮은 응집도](#4-나쁜-예제-낮은-응집도)
5. [좋은 예제: 높은 응집도](#5-좋은-예제-높은-응집도)
6. [응집도와 결합도의 관계](#6-응집도와-결합도의-관계)
7. [Python 코드 예제](#7-python-코드-예제)
8. [C# 코드 예제](#8-c-코드-예제)
9. [응집도를 높이는 리팩토링 기법](#9-응집도를-높이는-리팩토링-기법)
10. [실무 팁](#10-실무-팁)
11. [정리 및 체크리스트](#11-정리-및-체크리스트)
12. [관련 개념 연결](#12-관련-개념-연결)

---

## 1. 정의와 개념

**높은 응집도(High Cohesion)** 란 하나의 모듈 또는 클래스 내부의 요소(메서드, 속성)들이 서로 밀접하게 관련되어 있고, 하나의 명확한 책임에 집중하는 것을 의미합니다.

### 핵심 질문

> "이 클래스의 모든 메서드와 속성이 하나의 목적을 위해 존재하는가?"

- **YES** -> 높은 응집도 -> 좋은 설계
- **NO** -> 낮은 응집도 -> 리팩토링 필요

### 왜 중요한가?

| 관점 | 낮은 응집도 | 높은 응집도 |
|------|-----------|-----------|
| 이해하기 | 클래스가 여러 일을 해서 파악 어려움 | 클래스 이름만 봐도 역할 파악 가능 |
| 수정하기 | 하나 바꾸면 관련 없는 기능에 영향 | 변경 범위가 명확하고 제한적 |
| 재사용 | 불필요한 기능까지 같이 가져와야 함 | 필요한 기능만 깔끔하게 사용 |
| 테스트 | 테스트 케이스가 복잡하고 방대 | 테스트가 단순하고 명확 |
| 네이밍 | `UserManager`, `DataHelper` 같은 모호한 이름 | `PasswordHasher`, `EmailValidator` 같은 명확한 이름 |

---

## 2. 비유로 이해하기

### 잘 정리된 서랍 vs 잡동사니 서랍

```
┌─────────────────────────────────────────────────────────────┐
│                  ✅ 높은 응집도 = 잘 정리된 서랍               │
│                                                             │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│   │ 양말     │   │ 속옷     │   │ 셔츠     │   │ 바지     │   │
│   │ 서랍     │   │ 서랍     │   │ 서랍     │   │ 서랍     │   │
│   │         │   │         │   │         │   │         │   │
│   │ 양말만   │   │ 속옷만   │   │ 셔츠만   │   │ 바지만   │   │
│   │ 들어있음  │   │ 들어있음  │   │ 들어있음  │   │ 들어있음  │   │
│   └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│                                                             │
│   -> 양말을 찾으려면? 양말 서랍만 열면 됨!                      │
├─────────────────────────────────────────────────────────────┤
│                  ❌ 낮은 응집도 = 잡동사니 서랍                 │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  양말 + 충전기 + 과자 + 열쇠 + 속옷 + 리모컨 + ...     │   │
│   │                                                     │   │
│   │  모든 것이 한 서랍에! 뭐가 어딨는지 모름                 │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   -> 양말을 찾으려면? 서랍 전체를 뒤져야 함!                    │
└─────────────────────────────────────────────────────────────┘
```

### 현실 세계 비유: 병원

- **높은 응집도**: 안과(눈), 이비인후과(귀코목), 정형외과(뼈) - 각 과가 전문 분야에 집중
- **낮은 응집도**: 모든 질환을 다 보는 "만능 클리닉" - 전문성이 떨어지고 관리 어려움

---

## 3. 응집도의 종류

응집도는 높은 것(좋은 것)부터 낮은 것(나쁜 것) 순으로 다음과 같이 분류됩니다.

```
좋음 ◀──────────────────────────────────────────────────────────▶ 나쁨
 │                                                               │
 ▼                                                               ▼
기능적   순차적    통신적    절차적    시간적    논리적    우연적
(Func)  (Seq)   (Comm)   (Proc)   (Temp)   (Log)   (Coin)
```

### 3.1 기능적 응집 (Functional Cohesion) - 가장 좋음

모듈의 모든 요소가 **하나의 잘 정의된 기능**을 수행합니다.

```python
# 기능적 응집: 비밀번호 해싱이라는 단일 기능에 집중
class PasswordHasher:
    def __init__(self, salt_rounds: int = 12):
        self._salt_rounds = salt_rounds

    def hash(self, password: str) -> str:
        """비밀번호를 해싱"""
        salt = self._generate_salt()
        return self._apply_hash(password, salt)

    def verify(self, password: str, hashed: str) -> bool:
        """비밀번호 검증"""
        return self._apply_hash(password, self._extract_salt(hashed)) == hashed

    def _generate_salt(self) -> str:
        """솔트 생성 (내부 헬퍼)"""
        ...

    def _apply_hash(self, password: str, salt: str) -> str:
        """해시 적용 (내부 헬퍼)"""
        ...

    def _extract_salt(self, hashed: str) -> str:
        """해시에서 솔트 추출 (내부 헬퍼)"""
        ...

# 모든 메서드가 "비밀번호 해싱"이라는 하나의 목적을 위해 존재
```

### 3.2 순차적 응집 (Sequential Cohesion)

한 요소의 **출력이 다음 요소의 입력**이 되는 형태입니다.

```python
# 순차적 응집: 파이프라인처럼 데이터가 순차적으로 흐름
class DataProcessor:
    def process(self, raw_data: str) -> dict:
        cleaned = self._clean(raw_data)         # 1단계 출력 ->
        parsed = self._parse(cleaned)           # 2단계 입력/출력 ->
        validated = self._validate(parsed)      # 3단계 입력/출력 ->
        return self._transform(validated)       # 4단계 입력

    def _clean(self, data: str) -> str: ...
    def _parse(self, data: str) -> dict: ...
    def _validate(self, data: dict) -> dict: ...
    def _transform(self, data: dict) -> dict: ...
```

### 3.3 통신적 응집 (Communicational Cohesion)

여러 요소가 **같은 데이터를 조작**하지만, 순서는 중요하지 않습니다.

```python
# 통신적 응집: 같은 Report 데이터를 다양한 방식으로 출력
class ReportPrinter:
    def __init__(self, report_data: dict):
        self._data = report_data

    def print_summary(self) -> str: ...
    def print_details(self) -> str: ...
    def print_chart(self) -> str: ...
    # 모두 같은 report_data를 사용하지만 독립적
```

### 3.4 절차적 응집 (Procedural Cohesion)

요소들이 **특정 순서로 실행되어야** 하지만, 기능적으로는 관련이 약합니다.

```python
# 절차적 응집: 순서가 중요하지만 기능적 관련성은 약함
class UserOnboarding:
    def onboard(self, user_data: dict):
        self._validate_email(user_data["email"])   # 이메일 검증
        self._create_account(user_data)             # 계정 생성
        self._send_welcome_email(user_data["email"]) # 환영 이메일
        self._setup_default_preferences(user_data)   # 기본 설정
        # 순서가 중요하지만, 각 단계는 서로 다른 관심사
```

### 3.5 시간적 응집 (Temporal Cohesion)

**같은 시점에 실행되는** 요소들이 모여있습니다.

```python
# 시간적 응집: "앱 시작 시"라는 시간적 이유로 묶임
class AppInitializer:
    def initialize(self):
        self._load_config()       # 설정 로드
        self._connect_database()  # DB 연결
        self._start_logger()      # 로거 시작
        self._warm_cache()        # 캐시 워밍
        # 기능적 관련성 없음, 단지 "시작 시" 실행될 뿐
```

### 3.6 논리적 응집 (Logical Cohesion)

**논리적으로 비슷한 카테고리**에 속하지만, 기능이 다른 요소들의 모음입니다.

```python
# 논리적 응집: "입력"이라는 카테고리로만 묶임
class InputHandler:
    def read_from_keyboard(self) -> str: ...
    def read_from_file(self, path: str) -> str: ...
    def read_from_network(self, url: str) -> str: ...
    def read_from_database(self, query: str) -> str: ...
    # "읽기"라는 카테고리만 공통, 구체적 기능은 제각각
```

### 3.7 우연적 응집 (Coincidental Cohesion) - 가장 나쁨

요소들 사이에 **어떤 관련성도 없습니다**. 단지 같은 파일에 있을 뿐입니다.

```python
# 우연적 응집: 아무 관련 없는 것들이 한 클래스에
class Utils:
    @staticmethod
    def format_date(date): ...        # 날짜 포맷
    @staticmethod
    def calculate_tax(amount): ...    # 세금 계산
    @staticmethod
    def send_email(to, body): ...     # 이메일 전송
    @staticmethod
    def compress_image(path): ...     # 이미지 압축
    @staticmethod
    def generate_uuid(): ...          # UUID 생성
    # 공통점: 없음! 그냥 "유틸리티"라는 이름으로 던져넣음
```

---

## 4. 나쁜 예제: 낮은 응집도

### 문제 상황: God Object (신 객체)

```
❌ 낮은 응집도: God Object

┌──────────────────────────────────────────┐
│              UserManager                  │
│                                          │
│  - 사용자 CRUD                            │
│  - 비밀번호 해싱                           │
│  - 이메일 발송                             │
│  - 로그인/로그아웃                         │
│  - 권한 관리                              │
│  - 프로필 이미지 처리                      │
│  - 알림 설정                              │
│  - 결제 정보 관리                          │
│  - 보고서 생성                             │
│  - CSV 내보내기                            │
│                                          │
│  메서드: 50개 이상                         │
│  줄 수: 2000줄 이상                       │
│  책임: 모든 것                            │
└──────────────────────────────────────────┘

문제: 이 클래스를 이해하려면 2000줄을 다 읽어야 함
     이메일 기능을 고치면 결제 기능에 영향 가능
     테스트 케이스가 수백 개 필요
```

### Python - 낮은 응집도 예제

```python
# ❌ 나쁜 예: God Object - 너무 많은 책임
class UserManager:
    """사용자와 관련된 모든 것을 처리하는 만능 클래스"""

    def __init__(self):
        self.db_connection = self._connect_db()
        self.email_config = self._load_email_config()
        self.image_processor = self._init_image_processor()

    # --- 사용자 CRUD ---
    def create_user(self, name: str, email: str) -> dict:
        user = {"name": name, "email": email}
        self._save_to_db(user)
        self._send_welcome_email(email)  # 이메일 발송과 혼재
        return user

    def get_user(self, user_id: int) -> dict: ...
    def update_user(self, user_id: int, data: dict) -> dict: ...
    def delete_user(self, user_id: int) -> None: ...

    # --- 인증 (다른 관심사!) ---
    def login(self, email: str, password: str) -> str: ...
    def logout(self, token: str) -> None: ...
    def hash_password(self, password: str) -> str: ...
    def verify_password(self, password: str, hashed: str) -> bool: ...

    # --- 이메일 (또 다른 관심사!) ---
    def _send_welcome_email(self, email: str) -> None: ...
    def send_password_reset_email(self, email: str) -> None: ...
    def send_notification_email(self, email: str, message: str) -> None: ...
    def _load_email_config(self): ...

    # --- 프로필 이미지 (또 다른 관심사!) ---
    def upload_profile_image(self, user_id: int, image_data: bytes) -> str: ...
    def resize_image(self, image_data: bytes, width: int, height: int) -> bytes: ...
    def _init_image_processor(self): ...

    # --- 권한 관리 (또 다른 관심사!) ---
    def assign_role(self, user_id: int, role: str) -> None: ...
    def check_permission(self, user_id: int, permission: str) -> bool: ...
    def get_user_roles(self, user_id: int) -> list: ...

    # --- 보고서 (정말 여기에 있어야 하나?) ---
    def generate_user_report(self) -> str: ...
    def export_users_csv(self) -> bytes: ...

    # --- DB 접근 (인프라 관심사!) ---
    def _connect_db(self): ...
    def _save_to_db(self, data: dict) -> None: ...
    def _query_db(self, query: str) -> list: ...


# 문제점:
# 1. 한 클래스가 6개 이상의 책임을 가짐
# 2. 이메일 로직 변경 시 사용자 CRUD 코드에 영향 가능
# 3. 테스트하려면 DB, 이메일, 이미지 처리 등 모두 설정 필요
# 4. "UserManager"라는 이름이 모호함 - 정확히 뭘 관리하는가?
```

### C# - 낮은 응집도 예제

```csharp
// ❌ 나쁜 예: God Object
public class UserManager
{
    private readonly SqlConnection _dbConnection;
    private readonly SmtpClient _smtpClient;

    public UserManager()
    {
        _dbConnection = new SqlConnection("...");
        _smtpClient = new SmtpClient("smtp.example.com");
    }

    // 사용자 CRUD
    public User CreateUser(string name, string email) { ... }
    public User GetUser(int userId) { ... }
    public void UpdateUser(int userId, UserUpdateDto data) { ... }
    public void DeleteUser(int userId) { ... }

    // 인증 (다른 책임!)
    public string Login(string email, string password) { ... }
    public void Logout(string token) { ... }
    public string HashPassword(string password) { ... }
    public bool VerifyPassword(string password, string hashed) { ... }

    // 이메일 (또 다른 책임!)
    public void SendWelcomeEmail(string email) { ... }
    public void SendPasswordResetEmail(string email) { ... }

    // 프로필 이미지 (또 다른 책임!)
    public string UploadProfileImage(int userId, byte[] imageData) { ... }
    public byte[] ResizeImage(byte[] imageData, int width, int height) { ... }

    // 권한 관리 (또 다른 책임!)
    public void AssignRole(int userId, string role) { ... }
    public bool CheckPermission(int userId, string permission) { ... }

    // 보고서 (또 다른 책임!)
    public string GenerateUserReport() { ... }
    public byte[] ExportUsersCsv() { ... }

    // 이 클래스는 수정해야 할 이유가 너무 많다!
}
```

---

## 5. 좋은 예제: 높은 응집도

### 개선된 구조: 단일 책임에 집중

```
✅ 높은 응집도: 각 클래스가 하나의 책임에 집중

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ UserService  │  │ AuthService  │  │ EmailService │
│              │  │              │  │              │
│ - create     │  │ - login      │  │ - sendWelcome│
│ - get        │  │ - logout     │  │ - sendReset  │
│ - update     │  │ - hashPwd    │  │ - sendNotify │
│ - delete     │  │ - verifyPwd  │  │              │
└──────────────┘  └──────────────┘  └──────────────┘

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ ImageService │  │ RoleService  │  │ ReportService│
│              │  │              │  │              │
│ - upload     │  │ - assign     │  │ - generate   │
│ - resize     │  │ - check      │  │ - exportCsv  │
│ - compress   │  │ - getRoles   │  │ - exportPdf  │
└──────────────┘  └──────────────┘  └──────────────┘

각 클래스: 50~150줄, 명확한 이름, 쉬운 테스트
```

### Python - 높은 응집도 예제

```python
# ✅ 좋은 예: 각 클래스가 하나의 책임에 집중
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional


# --- 도메인 모델 ---
@dataclass
class User:
    id: Optional[int] = None
    name: str = ""
    email: str = ""
    password_hash: str = ""
    created_at: datetime = field(default_factory=datetime.now)


# --- 사용자 저장소: 사용자 데이터 접근만 담당 ---
class UserRepository:
    """사용자 데이터의 저장과 조회만 담당 (높은 응집도)"""

    def __init__(self, db_connection):
        self._db = db_connection

    def save(self, user: User) -> User:
        """사용자 저장"""
        print(f"[DB] 사용자 저장: {user.name}")
        user.id = 1  # 실제로는 DB에서 생성된 ID
        return user

    def find_by_id(self, user_id: int) -> Optional[User]:
        """ID로 사용자 조회"""
        print(f"[DB] 사용자 조회: {user_id}")
        return User(id=user_id, name="테스트", email="test@test.com")

    def find_by_email(self, email: str) -> Optional[User]:
        """이메일로 사용자 조회"""
        print(f"[DB] 이메일로 조회: {email}")
        return None

    def update(self, user: User) -> User:
        """사용자 정보 수정"""
        print(f"[DB] 사용자 수정: {user.id}")
        return user

    def delete(self, user_id: int) -> None:
        """사용자 삭제"""
        print(f"[DB] 사용자 삭제: {user_id}")


# --- 비밀번호 서비스: 비밀번호 관련 기능만 담당 ---
class PasswordService:
    """비밀번호 해싱과 검증만 담당 (높은 응집도)"""

    def __init__(self, salt_rounds: int = 12):
        self._salt_rounds = salt_rounds

    def hash(self, password: str) -> str:
        """비밀번호 해싱"""
        print(f"[Password] 해싱 처리 (rounds={self._salt_rounds})")
        return f"hashed_{password}"

    def verify(self, password: str, hashed: str) -> bool:
        """비밀번호 검증"""
        return self.hash(password) == hashed

    def is_strong(self, password: str) -> bool:
        """비밀번호 강도 검사"""
        return (
            len(password) >= 8
            and any(c.isupper() for c in password)
            and any(c.isdigit() for c in password)
        )


# --- 이메일 서비스: 이메일 발송만 담당 ---
class EmailService:
    """이메일 발송만 담당 (높은 응집도)"""

    def __init__(self, smtp_host: str, smtp_port: int):
        self._host = smtp_host
        self._port = smtp_port

    def send_welcome(self, email: str, name: str) -> None:
        """환영 이메일 전송"""
        print(f"[Email] {email}에게 환영 이메일 전송")

    def send_password_reset(self, email: str, reset_token: str) -> None:
        """비밀번호 재설정 이메일 전송"""
        print(f"[Email] {email}에게 비밀번호 재설정 링크 전송")

    def send_notification(self, email: str, subject: str, body: str) -> None:
        """일반 알림 이메일 전송"""
        print(f"[Email] {email}에게 '{subject}' 전송")


# --- 인증 서비스: 인증 관련 기능만 담당 ---
class AuthService:
    """로그인/로그아웃/토큰 관리만 담당 (높은 응집도)"""

    def __init__(self, user_repo: UserRepository, password_service: PasswordService):
        self._user_repo = user_repo
        self._password_service = password_service
        self._active_tokens: dict[str, int] = {}

    def login(self, email: str, password: str) -> Optional[str]:
        """로그인 처리"""
        user = self._user_repo.find_by_email(email)
        if user and self._password_service.verify(password, user.password_hash):
            token = f"token_{user.id}"
            self._active_tokens[token] = user.id
            return token
        return None

    def logout(self, token: str) -> None:
        """로그아웃 처리"""
        self._active_tokens.pop(token, None)

    def validate_token(self, token: str) -> Optional[int]:
        """토큰 검증"""
        return self._active_tokens.get(token)


# --- 사용자 등록 유스케이스: 각 서비스를 조합 ---
class UserRegistrationUseCase:
    """사용자 등록 프로세스를 조율하는 유스케이스"""

    def __init__(
        self,
        user_repo: UserRepository,
        password_service: PasswordService,
        email_service: EmailService
    ):
        self._user_repo = user_repo
        self._password_service = password_service
        self._email_service = email_service

    def register(self, name: str, email: str, password: str) -> User:
        # 비밀번호 강도 확인
        if not self._password_service.is_strong(password):
            raise ValueError("비밀번호가 너무 약합니다")

        # 중복 이메일 확인
        if self._user_repo.find_by_email(email):
            raise ValueError("이미 등록된 이메일입니다")

        # 사용자 생성
        user = User(
            name=name,
            email=email,
            password_hash=self._password_service.hash(password)
        )
        saved_user = self._user_repo.save(user)

        # 환영 이메일 전송
        self._email_service.send_welcome(email, name)

        return saved_user


# --- 사용 예시 ---
if __name__ == "__main__":
    # 각 서비스를 조립
    user_repo = UserRepository(db_connection=None)
    password_service = PasswordService(salt_rounds=12)
    email_service = EmailService(smtp_host="smtp.example.com", smtp_port=587)

    # 유스케이스에 주입
    registration = UserRegistrationUseCase(
        user_repo=user_repo,
        password_service=password_service,
        email_service=email_service
    )

    # 사용자 등록
    user = registration.register("홍길동", "hong@example.com", "Secure1234")
    print(f"등록 완료: {user.name}")
```

### C# - 높은 응집도 예제

```csharp
// ✅ 좋은 예: 각 클래스가 하나의 책임에 집중

// --- 도메인 모델 ---
public class User
{
    public int? Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public string PasswordHash { get; set; } = "";
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

// --- 인터페이스 ---
public interface IUserRepository
{
    User Save(User user);
    User? FindById(int userId);
    User? FindByEmail(string email);
    User Update(User user);
    void Delete(int userId);
}

public interface IPasswordService
{
    string Hash(string password);
    bool Verify(string password, string hashed);
    bool IsStrong(string password);
}

public interface IEmailService
{
    void SendWelcome(string email, string name);
    void SendPasswordReset(string email, string resetToken);
    void SendNotification(string email, string subject, string body);
}

// --- 사용자 저장소: 데이터 접근만 담당 ---
public class UserRepository : IUserRepository
{
    private readonly DbContext _context;

    public UserRepository(DbContext context)
    {
        _context = context;
    }

    public User Save(User user)
    {
        Console.WriteLine($"[DB] 사용자 저장: {user.Name}");
        user.Id = 1;
        return user;
    }

    public User? FindById(int userId)
    {
        Console.WriteLine($"[DB] 사용자 조회: {userId}");
        return new User { Id = userId, Name = "테스트" };
    }

    public User? FindByEmail(string email)
    {
        Console.WriteLine($"[DB] 이메일로 조회: {email}");
        return null;
    }

    public User Update(User user) { ... }
    public void Delete(int userId) { ... }
}

// --- 비밀번호 서비스: 비밀번호 관련만 담당 ---
public class PasswordService : IPasswordService
{
    private readonly int _saltRounds;

    public PasswordService(int saltRounds = 12)
    {
        _saltRounds = saltRounds;
    }

    public string Hash(string password)
    {
        Console.WriteLine($"[Password] 해싱 처리 (rounds={_saltRounds})");
        return $"hashed_{password}";
    }

    public bool Verify(string password, string hashed)
        => Hash(password) == hashed;

    public bool IsStrong(string password)
        => password.Length >= 8
           && password.Any(char.IsUpper)
           && password.Any(char.IsDigit);
}

// --- 이메일 서비스: 이메일 발송만 담당 ---
public class EmailService : IEmailService
{
    private readonly string _smtpHost;
    private readonly int _smtpPort;

    public EmailService(string smtpHost, int smtpPort)
    {
        _smtpHost = smtpHost;
        _smtpPort = smtpPort;
    }

    public void SendWelcome(string email, string name)
        => Console.WriteLine($"[Email] {email}에게 환영 이메일 전송");

    public void SendPasswordReset(string email, string resetToken)
        => Console.WriteLine($"[Email] {email}에게 비밀번호 재설정 링크 전송");

    public void SendNotification(string email, string subject, string body)
        => Console.WriteLine($"[Email] {email}에게 '{subject}' 전송");
}

// --- 사용자 등록 유스케이스 ---
public class UserRegistrationUseCase
{
    private readonly IUserRepository _userRepo;
    private readonly IPasswordService _passwordService;
    private readonly IEmailService _emailService;

    public UserRegistrationUseCase(
        IUserRepository userRepo,
        IPasswordService passwordService,
        IEmailService emailService)
    {
        _userRepo = userRepo;
        _passwordService = passwordService;
        _emailService = emailService;
    }

    public User Register(string name, string email, string password)
    {
        if (!_passwordService.IsStrong(password))
            throw new ArgumentException("비밀번호가 너무 약합니다");

        if (_userRepo.FindByEmail(email) != null)
            throw new InvalidOperationException("이미 등록된 이메일입니다");

        var user = new User
        {
            Name = name,
            Email = email,
            PasswordHash = _passwordService.Hash(password)
        };

        var savedUser = _userRepo.Save(user);
        _emailService.SendWelcome(email, name);

        return savedUser;
    }
}
```

---

## 6. 응집도와 결합도의 관계

응집도와 결합도는 **반비례 관계**입니다. 응집도를 높이면 자연스럽게 결합도가 낮아집니다.

```
┌──────────────────────────────────────────────────────┐
│              응집도와 결합도의 관계                      │
│                                                      │
│   높은 응집도                    낮은 응집도            │
│   + 낮은 결합도                  + 높은 결합도          │
│   = 좋은 설계                    = 나쁜 설계           │
│                                                      │
│                                                      │
│  ┌─────┐  ┌─────┐           ┌──────────────────┐    │
│  │  A  │  │  B  │           │   A + B + C      │    │
│  │(인증)│  │(메일)│           │ (모든 것이 한 곳에) │    │
│  └──┬──┘  └──┬──┘           └───────┬──────────┘    │
│     │ 약한   │                       │ 강한           │
│     │ 연결   │                       │ 내부 결합       │
│     ▼       ▼                       ▼               │
│  인터페이스 통신              직접적인 내부 참조         │
│                                                      │
│  각 모듈 독립 테스트 가능     전체를 함께 테스트해야 함    │
└──────────────────────────────────────────────────────┘
```

### 핵심 원리

| 상태 | 응집도 | 결합도 | 결과 |
|------|--------|--------|------|
| 이상적 | 높음 | 낮음 | 유지보수 쉬움, 테스트 용이 |
| 최악 | 낮음 | 높음 | 유지보수 어려움, 테스트 어려움 |
| God Object | 낮음 | 높음 | 모든 것이 한 곳에, 변경 위험 |
| 잘 분리된 모듈 | 높음 | 낮음 | 각각 독립적, 교체 용이 |

### 왜 반비례 관계인가?

```python
# God Object (낮은 응집도 -> 높은 결합도)
class UserManager:
    def save_user(self): ...
    def send_email(self): ...      # 이메일 기능이 사용자와 결합
    def process_payment(self): ... # 결제 기능이 사용자와 결합

# 분리된 클래스 (높은 응집도 -> 낮은 결합도)
class UserService:     # 사용자 기능만
    def save_user(self): ...

class EmailService:    # 이메일 기능만 -> UserService와 분리됨
    def send_email(self): ...

class PaymentService:  # 결제 기능만 -> UserService와 분리됨
    def process_payment(self): ...
```

---

## 7. Python 코드 예제

### 종합 예제: 전자상거래 주문 처리

```python
"""
높은 응집도를 적용한 전자상거래 주문 처리 시스템
각 클래스가 명확한 단일 책임을 가짐
"""
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from datetime import datetime
from decimal import Decimal
from enum import Enum
from typing import Optional


# --- 도메인 모델 ---
class OrderStatus(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

@dataclass
class OrderItem:
    product_id: int
    product_name: str
    quantity: int
    unit_price: Decimal

@dataclass
class Order:
    id: Optional[int] = None
    customer_email: str = ""
    items: list[OrderItem] = field(default_factory=list)
    status: OrderStatus = OrderStatus.PENDING
    created_at: datetime = field(default_factory=datetime.now)


# --- 가격 계산기: 가격 계산만 담당 (기능적 응집) ---
class OrderPriceCalculator:
    """주문 가격 계산에만 집중하는 클래스"""

    TAX_RATE = Decimal("0.1")
    FREE_SHIPPING_THRESHOLD = Decimal("50000")
    SHIPPING_FEE = Decimal("3000")

    def calculate_subtotal(self, items: list[OrderItem]) -> Decimal:
        """소계 계산"""
        return sum(item.unit_price * item.quantity for item in items)

    def calculate_tax(self, subtotal: Decimal) -> Decimal:
        """세금 계산"""
        return subtotal * self.TAX_RATE

    def calculate_shipping(self, subtotal: Decimal) -> Decimal:
        """배송비 계산"""
        if subtotal >= self.FREE_SHIPPING_THRESHOLD:
            return Decimal("0")
        return self.SHIPPING_FEE

    def calculate_total(self, items: list[OrderItem]) -> dict:
        """총액 계산"""
        subtotal = self.calculate_subtotal(items)
        tax = self.calculate_tax(subtotal)
        shipping = self.calculate_shipping(subtotal)
        return {
            "subtotal": subtotal,
            "tax": tax,
            "shipping": shipping,
            "total": subtotal + tax + shipping
        }


# --- 재고 관리: 재고 확인과 차감만 담당 (기능적 응집) ---
class InventoryService:
    """재고 관리에만 집중하는 클래스"""

    def __init__(self):
        self._stock: dict[int, int] = {}

    def set_stock(self, product_id: int, quantity: int) -> None:
        """재고 설정"""
        self._stock[product_id] = quantity

    def check_availability(self, product_id: int, quantity: int) -> bool:
        """재고 확인"""
        return self._stock.get(product_id, 0) >= quantity

    def reserve(self, product_id: int, quantity: int) -> bool:
        """재고 예약 (차감)"""
        if not self.check_availability(product_id, quantity):
            return False
        self._stock[product_id] -= quantity
        return True

    def release(self, product_id: int, quantity: int) -> None:
        """재고 복구"""
        self._stock[product_id] = self._stock.get(product_id, 0) + quantity

    def get_stock(self, product_id: int) -> int:
        """현재 재고 조회"""
        return self._stock.get(product_id, 0)


# --- 주문 검증기: 주문 유효성 검사만 담당 (기능적 응집) ---
class OrderValidator:
    """주문 유효성 검사에만 집중하는 클래스"""

    def __init__(self, inventory_service: InventoryService):
        self._inventory = inventory_service

    def validate(self, order: Order) -> list[str]:
        """주문 유효성 검사, 오류 목록 반환"""
        errors = []
        errors.extend(self._validate_customer(order))
        errors.extend(self._validate_items(order))
        errors.extend(self._validate_inventory(order))
        return errors

    def _validate_customer(self, order: Order) -> list[str]:
        errors = []
        if not order.customer_email:
            errors.append("고객 이메일이 필요합니다")
        if "@" not in order.customer_email:
            errors.append("유효하지 않은 이메일 형식입니다")
        return errors

    def _validate_items(self, order: Order) -> list[str]:
        errors = []
        if not order.items:
            errors.append("주문 항목이 비어있습니다")
        for item in order.items:
            if item.quantity <= 0:
                errors.append(f"{item.product_name}: 수량은 1 이상이어야 합니다")
            if item.unit_price <= 0:
                errors.append(f"{item.product_name}: 가격이 유효하지 않습니다")
        return errors

    def _validate_inventory(self, order: Order) -> list[str]:
        errors = []
        for item in order.items:
            if not self._inventory.check_availability(item.product_id, item.quantity):
                stock = self._inventory.get_stock(item.product_id)
                errors.append(
                    f"{item.product_name}: 재고 부족 (요청: {item.quantity}, 재고: {stock})"
                )
        return errors


# --- 주문 처리 유스케이스: 각 서비스를 조율 ---
class PlaceOrderUseCase:
    """주문 처리 프로세스를 조율하는 유스케이스"""

    def __init__(
        self,
        validator: OrderValidator,
        calculator: OrderPriceCalculator,
        inventory: InventoryService
    ):
        self._validator = validator
        self._calculator = calculator
        self._inventory = inventory

    def execute(self, order: Order) -> dict:
        # 1. 검증
        errors = self._validator.validate(order)
        if errors:
            return {"success": False, "errors": errors}

        # 2. 가격 계산
        pricing = self._calculator.calculate_total(order.items)

        # 3. 재고 차감
        for item in order.items:
            if not self._inventory.reserve(item.product_id, item.quantity):
                return {"success": False, "errors": ["재고 예약 실패"]}

        # 4. 주문 확정
        order.status = OrderStatus.CONFIRMED

        return {
            "success": True,
            "order_id": order.id,
            "pricing": pricing,
            "status": order.status.value
        }


# --- 사용 예시 ---
if __name__ == "__main__":
    # 서비스 조립
    inventory = InventoryService()
    inventory.set_stock(101, 50)
    inventory.set_stock(102, 10)

    calculator = OrderPriceCalculator()
    validator = OrderValidator(inventory)
    use_case = PlaceOrderUseCase(validator, calculator, inventory)

    # 주문 생성
    order = Order(
        id=1,
        customer_email="customer@example.com",
        items=[
            OrderItem(101, "키보드", 2, Decimal("35000")),
            OrderItem(102, "마우스", 1, Decimal("25000")),
        ]
    )

    # 주문 처리
    result = use_case.execute(order)
    print(f"주문 결과: {result}")
```

---

## 8. C# 코드 예제

### 종합 예제: 전자상거래 주문 처리

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// --- 도메인 모델 ---
public enum OrderStatus
{
    Pending, Confirmed, Shipped, Delivered, Cancelled
}

public record OrderItem(
    int ProductId,
    string ProductName,
    int Quantity,
    decimal UnitPrice
);

public class Order
{
    public int? Id { get; set; }
    public string CustomerEmail { get; set; } = "";
    public List<OrderItem> Items { get; set; } = new();
    public OrderStatus Status { get; set; } = OrderStatus.Pending;
    public DateTime CreatedAt { get; set; } = DateTime.Now;
}

// --- 인터페이스 ---
public interface IOrderPriceCalculator
{
    decimal CalculateSubtotal(List<OrderItem> items);
    decimal CalculateTax(decimal subtotal);
    decimal CalculateShipping(decimal subtotal);
    PricingResult CalculateTotal(List<OrderItem> items);
}

public interface IInventoryService
{
    bool CheckAvailability(int productId, int quantity);
    bool Reserve(int productId, int quantity);
    void Release(int productId, int quantity);
    int GetStock(int productId);
}

public interface IOrderValidator
{
    List<string> Validate(Order order);
}

public record PricingResult(
    decimal Subtotal,
    decimal Tax,
    decimal Shipping,
    decimal Total
);

// --- 가격 계산기 (기능적 응집) ---
public class OrderPriceCalculator : IOrderPriceCalculator
{
    private const decimal TaxRate = 0.1m;
    private const decimal FreeShippingThreshold = 50000m;
    private const decimal ShippingFee = 3000m;

    public decimal CalculateSubtotal(List<OrderItem> items)
        => items.Sum(item => item.UnitPrice * item.Quantity);

    public decimal CalculateTax(decimal subtotal)
        => subtotal * TaxRate;

    public decimal CalculateShipping(decimal subtotal)
        => subtotal >= FreeShippingThreshold ? 0m : ShippingFee;

    public PricingResult CalculateTotal(List<OrderItem> items)
    {
        var subtotal = CalculateSubtotal(items);
        var tax = CalculateTax(subtotal);
        var shipping = CalculateShipping(subtotal);
        return new PricingResult(subtotal, tax, shipping, subtotal + tax + shipping);
    }
}

// --- 재고 서비스 (기능적 응집) ---
public class InventoryService : IInventoryService
{
    private readonly Dictionary<int, int> _stock = new();

    public void SetStock(int productId, int quantity)
        => _stock[productId] = quantity;

    public bool CheckAvailability(int productId, int quantity)
        => _stock.GetValueOrDefault(productId, 0) >= quantity;

    public bool Reserve(int productId, int quantity)
    {
        if (!CheckAvailability(productId, quantity)) return false;
        _stock[productId] -= quantity;
        return true;
    }

    public void Release(int productId, int quantity)
        => _stock[productId] = _stock.GetValueOrDefault(productId, 0) + quantity;

    public int GetStock(int productId)
        => _stock.GetValueOrDefault(productId, 0);
}

// --- 주문 검증기 (기능적 응집) ---
public class OrderValidator : IOrderValidator
{
    private readonly IInventoryService _inventory;

    public OrderValidator(IInventoryService inventory)
    {
        _inventory = inventory;
    }

    public List<string> Validate(Order order)
    {
        var errors = new List<string>();
        errors.AddRange(ValidateCustomer(order));
        errors.AddRange(ValidateItems(order));
        errors.AddRange(ValidateInventory(order));
        return errors;
    }

    private List<string> ValidateCustomer(Order order)
    {
        var errors = new List<string>();
        if (string.IsNullOrEmpty(order.CustomerEmail))
            errors.Add("고객 이메일이 필요합니다");
        else if (!order.CustomerEmail.Contains('@'))
            errors.Add("유효하지 않은 이메일 형식입니다");
        return errors;
    }

    private List<string> ValidateItems(Order order)
    {
        var errors = new List<string>();
        if (!order.Items.Any())
            errors.Add("주문 항목이 비어있습니다");

        foreach (var item in order.Items)
        {
            if (item.Quantity <= 0)
                errors.Add($"{item.ProductName}: 수량은 1 이상이어야 합니다");
            if (item.UnitPrice <= 0)
                errors.Add($"{item.ProductName}: 가격이 유효하지 않습니다");
        }
        return errors;
    }

    private List<string> ValidateInventory(Order order)
    {
        var errors = new List<string>();
        foreach (var item in order.Items)
        {
            if (!_inventory.CheckAvailability(item.ProductId, item.Quantity))
            {
                var stock = _inventory.GetStock(item.ProductId);
                errors.Add($"{item.ProductName}: 재고 부족 (요청: {item.Quantity}, 재고: {stock})");
            }
        }
        return errors;
    }
}

// --- 주문 처리 유스케이스 ---
public class PlaceOrderUseCase
{
    private readonly IOrderValidator _validator;
    private readonly IOrderPriceCalculator _calculator;
    private readonly IInventoryService _inventory;

    public PlaceOrderUseCase(
        IOrderValidator validator,
        IOrderPriceCalculator calculator,
        IInventoryService inventory)
    {
        _validator = validator;
        _calculator = calculator;
        _inventory = inventory;
    }

    public OrderResult Execute(Order order)
    {
        // 1. 검증
        var errors = _validator.Validate(order);
        if (errors.Any())
            return OrderResult.Failure(errors);

        // 2. 가격 계산
        var pricing = _calculator.CalculateTotal(order.Items);

        // 3. 재고 차감
        foreach (var item in order.Items)
        {
            if (!_inventory.Reserve(item.ProductId, item.Quantity))
                return OrderResult.Failure(new List<string> { "재고 예약 실패" });
        }

        // 4. 주문 확정
        order.Status = OrderStatus.Confirmed;

        return OrderResult.Success(order.Id, pricing);
    }
}

public record OrderResult(
    bool IsSuccess,
    int? OrderId,
    PricingResult? Pricing,
    List<string>? Errors)
{
    public static OrderResult Success(int? orderId, PricingResult pricing)
        => new(true, orderId, pricing, null);

    public static OrderResult Failure(List<string> errors)
        => new(false, null, null, errors);
}
```

---

## 9. 응집도를 높이는 리팩토링 기법

### 9.1 클래스 추출 (Extract Class)

God Object를 여러 작은 클래스로 분리합니다.

```python
# Before: 하나의 큰 클래스
class Report:
    def __init__(self, data):
        self.data = data

    def calculate_statistics(self): ...  # 통계 계산
    def format_as_html(self): ...        # HTML 포맷
    def format_as_pdf(self): ...         # PDF 포맷
    def send_by_email(self): ...         # 이메일 전송
    def save_to_file(self): ...          # 파일 저장

# After: 책임별로 분리
class StatisticsCalculator:
    def calculate(self, data) -> dict: ...

class ReportFormatter:
    def to_html(self, data: dict) -> str: ...
    def to_pdf(self, data: dict) -> bytes: ...

class ReportDistributor:
    def send_by_email(self, report, recipient: str): ...
    def save_to_file(self, report, path: str): ...
```

### 9.2 메서드 추출 (Extract Method)

긴 메서드를 의미 있는 단위로 분리합니다.

```python
# Before: 하나의 긴 메서드
def process_order(order):
    # 검증 (20줄)
    if not order.items: ...
    if not order.email: ...
    # ... 검증 로직 20줄 ...

    # 가격 계산 (15줄)
    subtotal = 0
    for item in order.items: ...
    # ... 계산 로직 15줄 ...

    # DB 저장 (10줄)
    connection = get_connection()
    # ... 저장 로직 10줄 ...

    # 이메일 전송 (10줄)
    # ... 이메일 로직 10줄 ...

# After: 의미 있는 단위로 분리
def process_order(order):
    validate_order(order)
    pricing = calculate_pricing(order)
    save_order(order, pricing)
    send_confirmation_email(order, pricing)
```

### 9.3 인라인 클래스 제거 (Inline Class for Over-Splitting)

지나치게 분리된 경우 다시 합칩니다.

```python
# 지나친 분리 (응집도가 의미 없을 정도로 작은 클래스)
class NameValidator:
    def validate(self, name: str) -> bool:
        return len(name) > 0

class EmailValidator:
    def validate(self, email: str) -> bool:
        return "@" in email

class AgeValidator:
    def validate(self, age: int) -> bool:
        return 0 < age < 150

# 적절한 수준으로 합치기
class UserInputValidator:
    def validate_name(self, name: str) -> bool:
        return len(name) > 0

    def validate_email(self, email: str) -> bool:
        return "@" in email

    def validate_age(self, age: int) -> bool:
        return 0 < age < 150

    # 모두 "사용자 입력 검증"이라는 하나의 책임
```

---

## 10. 실무 팁

### 응집도 자가 진단

다음 질문에 "예"가 많으면 응집도가 낮습니다.

| # | 질문 | 경고 |
|---|------|------|
| 1 | 클래스 이름에 `Manager`, `Handler`, `Processor`, `Utils`가 들어가는가? | 모호한 이름 = 모호한 책임 |
| 2 | 클래스의 메서드 중 서로 호출하지 않는 그룹이 2개 이상인가? | 클래스를 분리해야 할 신호 |
| 3 | 클래스를 한 문장으로 설명할 때 "그리고(and)"가 들어가는가? | 두 개 이상의 책임 |
| 4 | 클래스의 일부 메서드만 특정 필드를 사용하는가? | LCOM(응집도 부족 메트릭) 높음 |
| 5 | 클래스가 200줄 이상인가? | 과도한 책임의 신호 |
| 6 | 테스트를 작성할 때 setUp에서 많은 Mock이 필요한가? | 의존성 과다 = 책임 과다 |

### 클래스 네이밍 가이드

```
❌ 나쁜 이름 (무엇을 하는지 불명확)     ✅ 좋은 이름 (역할이 명확)
──────────────────────────────        ────────────────────────────
UserManager                          UserRepository (저장/조회)
                                     UserAuthenticator (인증)
                                     UserProfileEditor (프로필 수정)

DataHelper                           CsvParser (CSV 파싱)
                                     JsonSerializer (JSON 직렬화)
                                     DataValidator (데이터 검증)

CommonUtils                          StringFormatter (문자열 포맷)
                                     DateCalculator (날짜 계산)
                                     FilePathResolver (경로 처리)
```

### LCOM (Lack of Cohesion of Methods) 간이 진단

```
클래스의 메서드가 어떤 필드를 사용하는지 확인:

class Example:
    field_a, field_b, field_c

    method_1() -> field_a, field_b 사용
    method_2() -> field_a, field_b 사용
    method_3() -> field_c 사용          <- 이 메서드만 field_c 사용!

    -> method_3과 field_c를 별도 클래스로 분리 고려
```

---

## 11. 정리 및 체크리스트

### 핵심 요약

```
┌───────────────────────────────────────────────┐
│            높은 응집도 핵심 원칙                  │
│                                               │
│  1. 하나의 클래스는 하나의 명확한 책임을 가져라    │
│  2. 클래스 이름이 역할을 정확히 설명해야 한다      │
│  3. 모든 메서드가 같은 필드/데이터를 다뤄야 한다   │
│  4. "그리고(and)"가 필요하면 분리하라             │
└───────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] 클래스를 한 문장으로 설명할 수 있는가? ("이 클래스는 __를 한다")
- [ ] 클래스 이름이 구체적이고 명확한가? (`Manager`, `Utils` 아닌)
- [ ] 모든 메서드가 클래스의 핵심 책임과 관련 있는가?
- [ ] 클래스가 200줄 이하인가?
- [ ] 메서드들이 대부분 같은 필드를 사용하는가?
- [ ] 클래스를 수정해야 하는 이유가 하나뿐인가?
- [ ] 불필요한 Utility 클래스를 만들지 않았는가?

### 응집도 수준 판단 가이드

| 수준 | 설명 | 예시 | 권장 |
|------|------|------|------|
| 기능적 | 단일 기능에 집중 | `PasswordHasher` | 최고 |
| 순차적 | 파이프라인 처리 | `DataProcessor` | 좋음 |
| 통신적 | 같은 데이터 처리 | `UserProfileView` | 괜찮음 |
| 절차적 | 순서만 공유 | `StartupSequence` | 주의 |
| 시간적 | 실행 시점만 공유 | `AppInitializer` | 리팩토링 고려 |
| 논리적 | 카테고리만 공유 | `InputHandler` | 리팩토링 필요 |
| 우연적 | 관련 없음 | `Utils` | 즉시 리팩토링 |

---

## 12. 관련 개념 연결

| 관련 개념 | 관계 |
|-----------|------|
| [느슨한 결합](./01-loose-coupling.md) | 응집도를 높이면 자연스럽게 결합도가 낮아짐 (반비례 관계) |
| [관심사의 분리](./03-separation-of-concerns.md) | SoC를 적용하면 각 모듈의 응집도가 높아짐 |
| [의존성 주입](./04-dependency-injection.md) | 응집도 높은 작은 클래스들을 DI로 조립 |
| SOLID - SRP | 단일 책임 원칙은 높은 응집도의 다른 표현 |
| SOLID - ISP | 인터페이스 분리도 응집도와 밀접한 관련 |
| Extract Class | 낮은 응집도를 높이는 핵심 리팩토링 기법 |
| God Object | 낮은 응집도의 대표적 안티패턴 |

---

> **이전 문서**: [느슨한 결합 (Loose Coupling)](./01-loose-coupling.md)
>
> **다음 문서**: [관심사의 분리 (Separation of Concerns)](./03-separation-of-concerns.md)
