# S - 단일 책임 원칙 (Single Responsibility Principle)

> **SOLID 원칙 1/5**
> **대상 독자**: 2~3년차 개발자
> **핵심 문장**: "클래스는 하나의 이유로만 변경되어야 한다." — Robert C. Martin

---

## 목차

1. [SRP란 무엇인가?](#1-srp란-무엇인가)
2. [왜 SRP가 중요한가?](#2-왜-srp가-중요한가)
3. [SRP 위반 사례 (Before)](#3-srp-위반-사례-before)
4. [SRP 적용 사례 (After)](#4-srp-적용-사례-after)
5. [실무 팁: SRP 위반 징후](#5-실무-팁-srp-위반-징후)
6. [정리 및 체크리스트](#6-정리-및-체크리스트)
7. [다음 단계](#7-다음-단계)

---

## 1. SRP란 무엇인가?

### 정의

> **"클래스는 오직 하나의 책임만 가져야 하며, 변경할 이유가 오직 하나여야 한다."**

여기서 "책임"이란 **변경의 이유(Reason to Change)** 를 의미합니다. 클래스가 두 가지 이유로 변경될 수 있다면, 그 클래스는 두 가지 책임을 갖고 있는 것입니다.

### 비유

레스토랑을 생각해 보세요:

- **셰프**는 요리를 합니다 (요리 책임)
- **웨이터**는 서빙을 합니다 (서비스 책임)
- **캐셔**는 결제를 처리합니다 (결제 책임)

만약 한 사람이 요리, 서빙, 결제를 모두 담당한다면? 요리법이 바뀌어도, 서비스 규정이 바뀌어도, 결제 시스템이 바뀌어도 그 사람이 영향을 받습니다. 각 역할을 분리하면, 변경의 영향 범위가 명확해집니다.

### "책임"의 올바른 해석

```
"책임 = 변경의 이유"

하나의 클래스가 변경되어야 하는 이유가 여러 개라면,
그 클래스는 여러 책임을 갖고 있는 것이다.
```

| 변경의 이유 | 해당 책임 | 분리 대상 |
|------------|----------|----------|
| 유효성 검사 규칙 변경 | 검증 로직 | Validator |
| 데이터 저장 방식 변경 | 영속성 로직 | Repository |
| 알림 채널 변경 | 알림 로직 | Notifier |
| 비즈니스 규칙 변경 | 도메인 로직 | Service |

---

## 2. 왜 SRP가 중요한가?

### SRP를 지키면

- **변경 영향 최소화**: 하나의 변경이 하나의 클래스에만 영향
- **테스트 용이**: 작은 클래스는 단위 테스트가 쉬움
- **재사용성 향상**: 단일 책임 클래스는 다른 곳에서도 사용 가능
- **코드 이해도 향상**: 클래스 이름만으로 역할 파악 가능

### SRP를 무시하면

- **산탄총 수술(Shotgun Surgery)**: 하나의 변경이 여러 곳에 영향
- **높은 결합도**: 관련 없는 기능들이 하나의 클래스에 뭉쳐 있음
- **테스트 어려움**: 하나를 테스트하려면 다른 의존성도 모두 필요
- **코드 충돌**: 여러 개발자가 같은 파일을 동시에 수정

---

## 3. SRP 위반 사례 (Before)

### ❌ Python - 나쁜 예: 모든 것을 하는 UserManager

```python
import re
import json
import smtplib
from email.mime.text import MIMEText


class UserManager:
    """
    ❌ SRP 위반!
    이 클래스는 최소 4가지 이유로 변경될 수 있다:
    1. 유효성 검사 규칙 변경
    2. 데이터 저장 방식 변경
    3. 이메일 전송 방식 변경
    4. 비밀번호 정책 변경
    """

    def __init__(self):
        self.users_file = "users.json"

    # 책임 1: 유효성 검사
    def validate_email(self, email: str) -> bool:
        pattern = r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$'
        return bool(re.match(pattern, email))

    def validate_password(self, password: str) -> bool:
        if len(password) < 8:
            return False
        if not re.search(r'[A-Z]', password):
            return False
        if not re.search(r'[0-9]', password):
            return False
        return True

    # 책임 2: 데이터 저장/조회
    def save_user(self, user_data: dict) -> None:
        try:
            with open(self.users_file, 'r') as f:
                users = json.load(f)
        except FileNotFoundError:
            users = []

        users.append(user_data)
        with open(self.users_file, 'w') as f:
            json.dump(users, f)

    def find_user(self, email: str) -> dict | None:
        try:
            with open(self.users_file, 'r') as f:
                users = json.load(f)
        except FileNotFoundError:
            return None

        for user in users:
            if user.get('email') == email:
                return user
        return None

    # 책임 3: 이메일 알림
    def send_welcome_email(self, email: str, name: str) -> None:
        msg = MIMEText(f"안녕하세요 {name}님, 가입을 환영합니다!")
        msg['Subject'] = '회원가입 완료'
        msg['From'] = 'noreply@example.com'
        msg['To'] = email

        with smtplib.SMTP('smtp.example.com', 587) as server:
            server.send_message(msg)

    def send_password_reset_email(self, email: str, token: str) -> None:
        msg = MIMEText(f"비밀번호 재설정 링크: https://example.com/reset?token={token}")
        msg['Subject'] = '비밀번호 재설정'
        msg['From'] = 'noreply@example.com'
        msg['To'] = email

        with smtplib.SMTP('smtp.example.com', 587) as server:
            server.send_message(msg)

    # 책임 4: 비밀번호 해싱
    def hash_password(self, password: str) -> str:
        import hashlib
        return hashlib.sha256(password.encode()).hexdigest()

    # 모든 책임이 뒤섞인 회원가입 메서드
    def register_user(self, name: str, email: str, password: str) -> bool:
        if not self.validate_email(email):
            print("유효하지 않은 이메일입니다.")
            return False

        if not self.validate_password(password):
            print("비밀번호가 정책에 맞지 않습니다.")
            return False

        if self.find_user(email):
            print("이미 존재하는 이메일입니다.")
            return False

        hashed = self.hash_password(password)
        self.save_user({"name": name, "email": email, "password": hashed})
        self.send_welcome_email(email, name)

        return True
```

**문제점 분석**:
- 이메일 유효성 검사 규칙이 바뀌면? → `UserManager` 수정
- DB로 마이그레이션하면? → `UserManager` 수정
- 이메일 대신 SMS 알림을 보내려면? → `UserManager` 수정
- 비밀번호 해싱 알고리즘을 바꾸면? → `UserManager` 수정
- 하나의 클래스가 **4가지 이유로 변경**될 수 있음!

### ❌ C# - 나쁜 예: 모든 것을 하는 UserManager

```csharp
public class UserManager
{
    // ❌ SRP 위반! 여러 책임이 하나의 클래스에 혼합
    private readonly string _usersFilePath = "users.json";

    // 책임 1: 유효성 검사
    public bool ValidateEmail(string email)
    {
        var pattern = @"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$";
        return Regex.IsMatch(email, pattern);
    }

    public bool ValidatePassword(string password)
    {
        if (password.Length < 8) return false;
        if (!password.Any(char.IsUpper)) return false;
        if (!password.Any(char.IsDigit)) return false;
        return true;
    }

    // 책임 2: 데이터 저장
    public void SaveUser(UserData userData)
    {
        var users = LoadUsers();
        users.Add(userData);
        var json = JsonSerializer.Serialize(users);
        File.WriteAllText(_usersFilePath, json);
    }

    public UserData? FindUser(string email)
    {
        var users = LoadUsers();
        return users.FirstOrDefault(u => u.Email == email);
    }

    private List<UserData> LoadUsers()
    {
        if (!File.Exists(_usersFilePath))
            return new List<UserData>();
        var json = File.ReadAllText(_usersFilePath);
        return JsonSerializer.Deserialize<List<UserData>>(json) ?? new();
    }

    // 책임 3: 이메일 전송
    public void SendWelcomeEmail(string email, string name)
    {
        using var client = new SmtpClient("smtp.example.com");
        var msg = new MailMessage("noreply@example.com", email)
        {
            Subject = "회원가입 완료",
            Body = $"안녕하세요 {name}님, 가입을 환영합니다!"
        };
        client.Send(msg);
    }

    // 책임 4: 비밀번호 해싱
    public string HashPassword(string password)
    {
        using var sha256 = SHA256.Create();
        var bytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
        return Convert.ToBase64String(bytes);
    }

    // 모든 책임이 뒤섞인 메서드
    public bool RegisterUser(string name, string email, string password)
    {
        if (!ValidateEmail(email)) return false;
        if (!ValidatePassword(password)) return false;
        if (FindUser(email) != null) return false;

        var hashed = HashPassword(password);
        SaveUser(new UserData { Name = name, Email = email, Password = hashed });
        SendWelcomeEmail(email, name);
        return true;
    }
}
```

---

## 4. SRP 적용 사례 (After)

### ✅ Python - 좋은 예: 책임별 분리

```python
import re
import json
import hashlib
from abc import ABC, abstractmethod
from dataclasses import dataclass


# ──────────────────────────────────────────────
# 데이터 모델
# ──────────────────────────────────────────────
@dataclass
class User:
    name: str
    email: str
    password_hash: str


# ──────────────────────────────────────────────
# 책임 1: 유효성 검사 (검증 규칙이 바뀌면 여기만 수정)
# ──────────────────────────────────────────────
class UserValidator:
    """사용자 입력 유효성 검사만 담당"""

    EMAIL_PATTERN = r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$'
    MIN_PASSWORD_LENGTH = 8

    def validate_email(self, email: str) -> bool:
        return bool(re.match(self.EMAIL_PATTERN, email))

    def validate_password(self, password: str) -> bool:
        if len(password) < self.MIN_PASSWORD_LENGTH:
            return False
        if not re.search(r'[A-Z]', password):
            return False
        if not re.search(r'[0-9]', password):
            return False
        return True

    def validate_registration(self, name: str, email: str, password: str) -> list[str]:
        """모든 검증을 수행하고 에러 목록 반환"""
        errors = []
        if not name.strip():
            errors.append("이름은 필수입니다.")
        if not self.validate_email(email):
            errors.append("유효하지 않은 이메일 형식입니다.")
        if not self.validate_password(password):
            errors.append("비밀번호는 8자 이상, 대문자와 숫자를 포함해야 합니다.")
        return errors


# ──────────────────────────────────────────────
# 책임 2: 데이터 저장/조회 (저장 방식이 바뀌면 여기만 수정)
# ──────────────────────────────────────────────
class UserRepository(ABC):
    """사용자 데이터 접근 추상화"""

    @abstractmethod
    def save(self, user: User) -> None:
        pass

    @abstractmethod
    def find_by_email(self, email: str) -> User | None:
        pass

    @abstractmethod
    def exists(self, email: str) -> bool:
        pass


class JsonUserRepository(UserRepository):
    """JSON 파일 기반 저장소 구현"""

    def __init__(self, file_path: str = "users.json"):
        self._file_path = file_path

    def save(self, user: User) -> None:
        users = self._load_all()
        users.append({
            "name": user.name,
            "email": user.email,
            "password_hash": user.password_hash,
        })
        with open(self._file_path, 'w') as f:
            json.dump(users, f, ensure_ascii=False, indent=2)

    def find_by_email(self, email: str) -> User | None:
        for data in self._load_all():
            if data["email"] == email:
                return User(**data)
        return None

    def exists(self, email: str) -> bool:
        return self.find_by_email(email) is not None

    def _load_all(self) -> list[dict]:
        try:
            with open(self._file_path, 'r') as f:
                return json.load(f)
        except (FileNotFoundError, json.JSONDecodeError):
            return []


# ──────────────────────────────────────────────
# 책임 3: 알림 (알림 방식이 바뀌면 여기만 수정)
# ──────────────────────────────────────────────
class NotificationService(ABC):
    """알림 발송 추상화"""

    @abstractmethod
    def send_welcome(self, email: str, name: str) -> None:
        pass


class EmailNotificationService(NotificationService):
    """이메일 알림 구현"""

    def __init__(self, smtp_host: str = "smtp.example.com"):
        self._smtp_host = smtp_host

    def send_welcome(self, email: str, name: str) -> None:
        # 실제 SMTP 발송 로직
        print(f"[이메일 발송] {email}으로 환영 메일 전송 완료")


# ──────────────────────────────────────────────
# 책임 4: 비밀번호 해싱 (해싱 정책이 바뀌면 여기만 수정)
# ──────────────────────────────────────────────
class PasswordHasher:
    """비밀번호 해싱만 담당"""

    def hash(self, password: str) -> str:
        return hashlib.sha256(password.encode()).hexdigest()

    def verify(self, password: str, hashed: str) -> bool:
        return self.hash(password) == hashed


# ──────────────────────────────────────────────
# 조합: UserService - 각 책임을 조율하는 역할만 담당
# ──────────────────────────────────────────────
class UserRegistrationService:
    """
    회원가입 흐름을 조율하는 서비스.
    각 단계의 실제 구현은 주입받은 객체에 위임한다.
    변경 이유: 회원가입 '흐름' 자체가 바뀔 때만.
    """

    def __init__(
        self,
        validator: UserValidator,
        repository: UserRepository,
        notifier: NotificationService,
        hasher: PasswordHasher,
    ):
        self._validator = validator
        self._repository = repository
        self._notifier = notifier
        self._hasher = hasher

    def register(self, name: str, email: str, password: str) -> bool:
        # 1. 유효성 검사 (Validator에 위임)
        errors = self._validator.validate_registration(name, email, password)
        if errors:
            for error in errors:
                print(f"  검증 실패: {error}")
            return False

        # 2. 중복 확인 (Repository에 위임)
        if self._repository.exists(email):
            print("  이미 등록된 이메일입니다.")
            return False

        # 3. 비밀번호 해싱 (Hasher에 위임)
        password_hash = self._hasher.hash(password)

        # 4. 저장 (Repository에 위임)
        user = User(name=name, email=email, password_hash=password_hash)
        self._repository.save(user)

        # 5. 알림 (Notifier에 위임)
        self._notifier.send_welcome(email, name)

        print(f"  '{name}'님 회원가입 완료!")
        return True


# ──────────────────────────────────────────────
# 사용 예시
# ──────────────────────────────────────────────
if __name__ == "__main__":
    service = UserRegistrationService(
        validator=UserValidator(),
        repository=JsonUserRepository("users.json"),
        notifier=EmailNotificationService(),
        hasher=PasswordHasher(),
    )

    service.register("홍길동", "hong@example.com", "SecurePass1")
    service.register("김철수", "invalid-email", "weak")  # 실패 케이스
```

### ✅ C# - 좋은 예: 책임별 분리

```csharp
// ── 데이터 모델 ──
public record User(string Name, string Email, string PasswordHash);

// ── 책임 1: 유효성 검사 ──
public class UserValidator
{
    private const string EmailPattern = @"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$";
    private const int MinPasswordLength = 8;

    public bool ValidateEmail(string email)
        => Regex.IsMatch(email, EmailPattern);

    public bool ValidatePassword(string password)
        => password.Length >= MinPasswordLength
           && password.Any(char.IsUpper)
           && password.Any(char.IsDigit);

    public List<string> ValidateRegistration(string name, string email, string password)
    {
        var errors = new List<string>();
        if (string.IsNullOrWhiteSpace(name))
            errors.Add("이름은 필수입니다.");
        if (!ValidateEmail(email))
            errors.Add("유효하지 않은 이메일 형식입니다.");
        if (!ValidatePassword(password))
            errors.Add("비밀번호는 8자 이상, 대문자와 숫자를 포함해야 합니다.");
        return errors;
    }
}

// ── 책임 2: 데이터 저장 ──
public interface IUserRepository
{
    void Save(User user);
    User? FindByEmail(string email);
    bool Exists(string email);
}

public class JsonUserRepository : IUserRepository
{
    private readonly string _filePath;

    public JsonUserRepository(string filePath = "users.json")
        => _filePath = filePath;

    public void Save(User user)
    {
        var users = LoadAll();
        users.Add(user);
        var json = JsonSerializer.Serialize(users, new JsonSerializerOptions { WriteIndented = true });
        File.WriteAllText(_filePath, json);
    }

    public User? FindByEmail(string email)
        => LoadAll().FirstOrDefault(u => u.Email == email);

    public bool Exists(string email)
        => FindByEmail(email) != null;

    private List<User> LoadAll()
    {
        if (!File.Exists(_filePath)) return new();
        var json = File.ReadAllText(_filePath);
        return JsonSerializer.Deserialize<List<User>>(json) ?? new();
    }
}

// ── 책임 3: 알림 ──
public interface INotificationService
{
    void SendWelcome(string email, string name);
}

public class EmailNotificationService : INotificationService
{
    public void SendWelcome(string email, string name)
    {
        Console.WriteLine($"[이메일 발송] {email}으로 환영 메일 전송 완료");
    }
}

// ── 책임 4: 비밀번호 해싱 ──
public class PasswordHasher
{
    public string Hash(string password)
    {
        using var sha256 = SHA256.Create();
        var bytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
        return Convert.ToBase64String(bytes);
    }

    public bool Verify(string password, string hash)
        => Hash(password) == hash;
}

// ── 조합: 흐름 조율만 담당하는 서비스 ──
public class UserRegistrationService
{
    private readonly UserValidator _validator;
    private readonly IUserRepository _repository;
    private readonly INotificationService _notifier;
    private readonly PasswordHasher _hasher;

    public UserRegistrationService(
        UserValidator validator,
        IUserRepository repository,
        INotificationService notifier,
        PasswordHasher hasher)
    {
        _validator = validator;
        _repository = repository;
        _notifier = notifier;
        _hasher = hasher;
    }

    public bool Register(string name, string email, string password)
    {
        var errors = _validator.ValidateRegistration(name, email, password);
        if (errors.Any())
        {
            errors.ForEach(e => Console.WriteLine($"  검증 실패: {e}"));
            return false;
        }

        if (_repository.Exists(email))
        {
            Console.WriteLine("  이미 등록된 이메일입니다.");
            return false;
        }

        var hash = _hasher.Hash(password);
        _repository.Save(new User(name, email, hash));
        _notifier.SendWelcome(email, name);

        Console.WriteLine($"  '{name}'님 회원가입 완료!");
        return true;
    }
}
```

---

## 5. 실무 팁: SRP 위반 징후

### 클래스 이름에서 냄새 맡기

| 클래스 이름 | 냄새 | 분리 방향 |
|------------|------|----------|
| `UserManager` | Manager = 모든 것을 관리 | UserValidator, UserRepository, UserService |
| `DataHandler` | Handler = 모든 것을 처리 | DataParser, DataWriter, DataTransformer |
| `OrderProcessor` | Processor = 처리가 너무 넓음 | OrderValidator, PaymentService, ShippingService |
| `FileUtility` | Utility = 잡다한 것의 모음 | FileReader, FileWriter, PathResolver |

### SRP 위반 감지 질문

1. **"이 클래스를 설명할 때 '그리고(and)'가 필요한가?"**
   - "이 클래스는 사용자를 검증하**고**, 저장하**고**, 이메일을 보낸다" → 위반!

2. **"이 클래스의 변경 이유가 두 가지 이상인가?"**
   - DB 스키마 변경 시에도, 이메일 템플릿 변경 시에도 수정 → 위반!

3. **"이 클래스의 import/using이 너무 다양한가?"**
   - `smtplib`, `json`, `hashlib`, `re`를 모두 import → 책임이 여러 개일 가능성

4. **"이 클래스를 테스트하려면 mock이 몇 개 필요한가?"**
   - DB mock, SMTP mock, 파일 시스템 mock... → 책임이 여러 개

### SRP를 적용할 때 주의할 점

> **과도한 분리에 주의하세요!**
>
> SRP는 "모든 메서드를 별도 클래스로 분리하라"는 뜻이 아닙니다.
> 관련된 책임은 하나의 클래스에 있는 것이 자연스럽습니다.
>
> ```python
> # ❌ 과도한 분리 - 이건 너무 나간 것
> class UserNameValidator:
>     def validate(self, name): ...
>
> class UserEmailValidator:
>     def validate(self, email): ...
>
> class UserPasswordValidator:
>     def validate(self, password): ...
>
> # ✅ 적절한 수준의 분리
> class UserValidator:
>     """사용자 입력 검증이라는 하나의 책임"""
>     def validate_name(self, name): ...
>     def validate_email(self, email): ...
>     def validate_password(self, password): ...
> ```
>
> **핵심**: "함께 변경되는 것은 함께 두고, 다른 이유로 변경되는 것은 분리한다."

---

## 6. 정리 및 체크리스트

### 핵심 요약

| 항목 | 설명 |
|------|------|
| **원칙** | 클래스는 하나의 변경 이유만 가져야 한다 |
| **장점** | 유지보수 용이, 테스트 용이, 재사용성 향상 |
| **위반 징후** | Manager/Handler/Processor 이름, 다양한 import, "그리고" 설명 |
| **적용 방법** | 변경 이유별로 클래스 분리, 조율 클래스로 결합 |

### 셀프 체크리스트

- [ ] 각 클래스를 한 문장으로 설명할 수 있는가? ("그리고" 없이)
- [ ] 클래스의 변경 이유가 오직 하나인가?
- [ ] 클래스 이름이 Manager/Handler/Utility 같은 모호한 이름이 아닌가?
- [ ] 클래스의 메서드들이 동일한 데이터를 다루고 있는가?
- [ ] 테스트 작성 시 불필요한 mock 없이 테스트 가능한가?
- [ ] 과도한 분리로 오히려 복잡해지지 않았는가?

---

## 7. 다음 단계

SRP로 클래스의 책임을 분리했다면, 이제 **변경에 유연하게 대응하는 방법**을 배울 차례입니다.

**다음 문서**: [03-solid-ocp.md - O: 개방-폐쇄 원칙 (Open/Closed Principle)](./03-solid-ocp.md)

> SRP로 분리한 클래스들이 새로운 요구사항에 어떻게 확장될 수 있는지, 기존 코드를 수정하지 않고도 기능을 추가하는 방법을 알아봅니다.
