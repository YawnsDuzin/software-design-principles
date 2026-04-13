# MVVM 패턴 (Model-View-ViewModel)

> **대상 독자**: 2~3년차 개발자
> **핵심 키워드**: MVVM, 데이터 바인딩, INotifyPropertyChanged, Command 패턴, UI 분리
> **난이도**: ★★★☆☆ (중급)

---

## 목차

1. [MVVM 개요 및 역사](#1-mvvm-개요-및-역사)
2. [왜 MVVM인가?](#2-왜-mvvm인가)
3. [3가지 구성 요소 상세 설명](#3-3가지-구성-요소-상세-설명)
4. [데이터 바인딩 개념](#4-데이터-바인딩-개념)
5. [INotifyPropertyChanged 구현 (C#)](#5-inotifypropertychanged-구현-c)
6. [Command 패턴과 MVVM](#6-command-패턴과-mvvm)
7. [Python에서의 MVVM 적용](#7-python에서의-mvvm-적용)
8. [C#/WPF MVVM 실전 예제 - 할일 목록 앱](#8-cwpf-mvvm-실전-예제---할일-목록-앱)
9. [MVC vs MVP vs MVVM 비교표](#9-mvc-vs-mvp-vs-mvvm-비교표)
10. [MVVM 프레임워크 소개](#10-mvvm-프레임워크-소개)
11. [장단점](#11-장단점)
12. [폴더 구조 예시](#12-폴더-구조-예시)
13. [정리 및 체크리스트](#13-정리-및-체크리스트)
14. [참고 자료](#14-참고-자료)

---

## 1. MVVM 개요 및 역사

### MVVM이란?

MVVM(Model-View-ViewModel)은 **UI 애플리케이션의 구조를 세 가지 핵심 구성 요소로 분리**하는 아키텍처 패턴입니다. 사용자 인터페이스(View)와 비즈니스 로직(Model)을 깔끔하게 분리하여 유지보수성과 테스트 용이성을 극대화합니다.

### 역사적 배경

```
┌─────────────────────────────────────────────────────────────────┐
│                     MVVM의 탄생 타임라인                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  2004  ──── Martin Fowler가 "Presentation Model" 패턴 제안      │
│              (MVVM의 원형)                                       │
│              │                                                  │
│  2005  ──── Microsoft의 John Gossman이 MVVM 패턴 발표           │
│              WPF(Windows Presentation Foundation) 개발 중 고안    │
│              │                                                  │
│  2006  ──── WPF 정식 출시와 함께 MVVM이 표준 패턴으로 자리잡음     │
│              XAML의 데이터 바인딩이 핵심 동력                      │
│              │                                                  │
│  2010+ ──── 다양한 플랫폼으로 확산                                │
│              - Silverlight                                      │
│              - Xamarin (모바일)                                   │
│              - Angular (웹 - 유사 개념)                           │
│              - SwiftUI (iOS)                                    │
│              - Jetpack Compose (Android)                        │
│              │                                                  │
│  현재  ──── .NET MAUI, WinUI 3, Avalonia UI 등에서 표준 패턴     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

MVVM은 WPF의 **강력한 데이터 바인딩 엔진**을 최대한 활용하기 위해 태어난 패턴입니다. 기존 MVC 패턴에서는 Controller가 View를 직접 조작했지만, MVVM에서는 **데이터 바인딩을 통해 View가 ViewModel의 상태를 자동으로 반영**합니다.

---

## 2. 왜 MVVM인가?

### UI와 비즈니스 로직 분리의 필요성

실무에서 흔히 겪는 문제 상황을 살펴봅시다.

```
[ 분리하지 않은 코드의 문제점 ]

┌──────────────────────────────────────────────┐
│              하나의 코드 파일에 모든 것          │
│  ┌─────────────────────────────────────────┐  │
│  │  UI 이벤트 핸들러                         │  │
│  │  + 데이터 검증 로직                       │  │
│  │  + DB 접근 코드                          │  │
│  │  + 비즈니스 규칙                          │  │
│  │  + UI 업데이트 코드                       │  │
│  │  = 스파게티 코드!                         │  │
│  └─────────────────────────────────────────┘  │
│                                              │
│  결과:                                        │
│  - 테스트 불가능 (UI 없이 로직 테스트 불가)     │
│  - 수정 시 사이드 이펙트 빈발                  │
│  - 디자이너와 개발자 동시 작업 불가             │
│  - 코드 재사용 불가                            │
└──────────────────────────────────────────────┘
```

```
[ MVVM으로 분리한 코드 ]

┌──────────┐    데이터      ┌──────────────┐    메서드     ┌──────────┐
│          │    바인딩      │              │    호출      │          │
│   View   │◄═════════════►│  ViewModel   │─────────────►│  Model   │
│  (XAML)  │               │  (C# Class)  │◄─────────────│ (Data +  │
│          │               │              │    결과반환   │  Logic)  │
└──────────┘               └──────────────┘              └──────────┘
  UI 표현에만                 상태 관리와                   순수한
  집중                       변환 로직                    비즈니스 로직

  결과:
  - 각 부분을 독립적으로 테스트 가능
  - 역할이 명확하여 유지보수 용이
  - 디자이너(View)와 개발자(ViewModel/Model) 동시 작업 가능
```

### 핵심 동기

| 문제 | MVVM의 해결 방법 |
|------|-----------------|
| UI 코드와 로직이 뒤섞임 | View는 표시만, ViewModel이 로직 담당 |
| 단위 테스트가 어려움 | ViewModel은 UI 의존 없이 테스트 가능 |
| 코드 재사용이 안 됨 | 같은 ViewModel을 다른 View에서 재사용 |
| 디자이너/개발자 협업 어려움 | XAML(디자이너) / C#(개발자) 분리 |
| 플랫폼 종속성 | ViewModel과 Model은 플랫폼 독립적 |

---

## 3. 3가지 구성 요소 상세 설명

### 전체 구조 다이어그램

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MVVM 패턴 구조도                              │
│                                                                     │
│  ┌─────────────┐      ┌───────────────────┐      ┌──────────────┐  │
│  │             │      │                   │      │              │  │
│  │    VIEW     │      │    VIEWMODEL      │      │    MODEL     │  │
│  │             │      │                   │      │              │  │
│  │  ┌───────┐  │ 바인딩│  ┌─────────────┐  │      │  ┌────────┐  │  │
│  │  │ XAML  │  │◄════►│  │ Properties  │  │      │  │ Entity │  │  │
│  │  │ HTML  │  │      │  │ Commands    │  │      │  │ DTO    │  │  │
│  │  │ UI    │  │      │  │ Methods     │  │─────►│  │ Service│  │  │
│  │  └───────┘  │      │  └─────────────┘  │      │  └────────┘  │  │
│  │             │      │                   │◄─────│              │  │
│  │  사용자가    │      │  INotifyProperty  │      │  비즈니스     │  │
│  │  보는 화면   │      │  Changed 구현     │      │  규칙/데이터  │  │
│  │             │      │  ICommand 구현    │      │              │  │
│  └─────────────┘      └───────────────────┘      └──────────────┘  │
│                                                                     │
│  ◄══► : 데이터 바인딩 (양방향)                                       │
│  ───► : 직접 참조                                                    │
│  ◄─── : 데이터 반환                                                  │
│                                                                     │
│  [중요] View는 ViewModel만 알고, ViewModel은 Model만 압니다.          │
│         Model은 ViewModel을 모르고, ViewModel은 View를 모릅니다.      │
└─────────────────────────────────────────────────────────────────────┘
```

### 3-1. Model (모델)

Model은 **데이터 구조와 비즈니스 로직**을 담당합니다. UI와는 완전히 독립적인 순수한 도메인 영역입니다.

**책임 범위:**
- 데이터의 구조 정의 (Entity, DTO)
- 비즈니스 규칙 구현
- 데이터 유효성 검증
- 외부 서비스/DB 접근 (Repository 패턴 활용)

```python
# Python - Model 예제
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional


@dataclass
class TodoItem:
    """할일 항목 엔티티 (Model)"""
    id: int
    title: str
    description: str = ""
    is_completed: bool = False
    created_at: datetime = field(default_factory=datetime.now)
    completed_at: Optional[datetime] = None

    def complete(self) -> None:
        """할일 완료 처리 - 비즈니스 로직"""
        if self.is_completed:
            raise ValueError("이미 완료된 항목입니다.")
        self.is_completed = True
        self.completed_at = datetime.now()

    def reopen(self) -> None:
        """할일 재개 처리 - 비즈니스 로직"""
        self.is_completed = False
        self.completed_at = None

    def validate(self) -> bool:
        """유효성 검증 - 비즈니스 규칙"""
        if not self.title or len(self.title.strip()) == 0:
            return False
        if len(self.title) > 200:
            return False
        return True
```

```csharp
// C# - Model 예제
using System;

namespace TodoApp.Models
{
    /// <summary>
    /// 할일 항목 엔티티 (Model)
    /// UI와 완전히 독립적인 순수 도메인 객체
    /// </summary>
    public class TodoItem
    {
        public int Id { get; set; }
        public string Title { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public bool IsCompleted { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.Now;
        public DateTime? CompletedAt { get; set; }

        /// <summary>
        /// 할일 완료 처리 - 비즈니스 로직
        /// </summary>
        public void Complete()
        {
            if (IsCompleted)
                throw new InvalidOperationException("이미 완료된 항목입니다.");

            IsCompleted = true;
            CompletedAt = DateTime.Now;
        }

        /// <summary>
        /// 할일 재개 처리 - 비즈니스 로직
        /// </summary>
        public void Reopen()
        {
            IsCompleted = false;
            CompletedAt = null;
        }

        /// <summary>
        /// 유효성 검증 - 비즈니스 규칙
        /// </summary>
        public bool Validate()
        {
            if (string.IsNullOrWhiteSpace(Title))
                return false;
            if (Title.Length > 200)
                return false;
            return true;
        }
    }
}
```

### 3-2. View (뷰)

View는 **사용자에게 보이는 UI 화면**입니다. 가능한 한 로직을 포함하지 않으며, ViewModel에 데이터 바인딩으로 연결됩니다.

**책임 범위:**
- UI 요소 배치 및 스타일링
- 사용자 입력 수신
- ViewModel의 데이터를 화면에 표시
- 애니메이션, 트랜지션 등 순수 UI 동작

```xml
<!-- C#/WPF - View 예제 (XAML) -->
<!-- TodoListView.xaml -->
<Window x:Class="TodoApp.Views.TodoListView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="할일 목록" Height="500" Width="400">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- 제목 -->
        <TextBlock Grid.Row="0" Text="나의 할일 목록"
                   FontSize="24" FontWeight="Bold" Margin="0,0,0,10"/>

        <!-- 새 할일 입력 영역 -->
        <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,0,0,10">
            <!-- ViewModel의 NewTodoTitle 속성에 양방향 바인딩 -->
            <TextBox Text="{Binding NewTodoTitle, UpdateSourceTrigger=PropertyChanged}"
                     Width="280" Margin="0,0,5,0"
                     PlaceholderText="새 할일을 입력하세요..."/>
            <!-- ViewModel의 AddTodoCommand에 바인딩 -->
            <Button Content="추가"
                    Command="{Binding AddTodoCommand}"
                    Width="80"/>
        </StackPanel>

        <!-- 할일 목록 -->
        <ListBox Grid.Row="2"
                 ItemsSource="{Binding TodoItems}"
                 SelectedItem="{Binding SelectedTodo}">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <CheckBox IsChecked="{Binding IsCompleted}"
                                  Margin="0,0,8,0"/>
                        <TextBlock Text="{Binding Title}"/>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>

        <!-- 상태 표시 -->
        <TextBlock Grid.Row="3"
                   Text="{Binding StatusMessage}"
                   Margin="0,10,0,0"/>
    </Grid>
</Window>
```

> **핵심 포인트**: View의 XAML 코드를 보면, 직접적인 이벤트 핸들러(Click="..." 등)가 없습니다. 모든 상호작용은 **데이터 바인딩과 Command**를 통해 ViewModel에 전달됩니다. 이것이 MVVM의 핵심입니다.

### 3-3. ViewModel (뷰모델)

ViewModel은 **View와 Model 사이의 중재자**입니다. View가 필요로 하는 데이터와 명령을 제공하며, Model의 데이터를 View에 맞게 변환합니다.

**책임 범위:**
- View에 표시할 데이터(속성)를 노출
- View에서 실행할 명령(Command)을 노출
- Model의 데이터를 View에 맞게 변환
- 속성 변경 시 View에 알림 (INotifyPropertyChanged)
- 입력 데이터의 UI 수준 유효성 검증

```python
# Python - ViewModel 개념적 예제
from typing import Callable, List, Optional


class Observable:
    """속성 변경 알림을 위한 기본 클래스 (INotifyPropertyChanged 대응)"""

    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}

    def add_listener(self, property_name: str, callback: Callable) -> None:
        if property_name not in self._listeners:
            self._listeners[property_name] = []
        self._listeners[property_name].append(callback)

    def notify(self, property_name: str) -> None:
        for callback in self._listeners.get(property_name, []):
            callback()


class TodoListViewModel(Observable):
    """할일 목록 ViewModel"""

    def __init__(self, todo_service):
        super().__init__()
        self._todo_service = todo_service  # Model (서비스)
        self._todo_items: List[TodoItem] = []
        self._new_todo_title: str = ""
        self._selected_todo: Optional[TodoItem] = None
        self._status_message: str = "준비됨"

        # 초기 데이터 로드
        self._load_todos()

    # ── 속성(Properties) ──
    @property
    def todo_items(self) -> List[TodoItem]:
        return self._todo_items

    @property
    def new_todo_title(self) -> str:
        return self._new_todo_title

    @new_todo_title.setter
    def new_todo_title(self, value: str) -> None:
        if self._new_todo_title != value:
            self._new_todo_title = value
            self.notify("new_todo_title")

    @property
    def status_message(self) -> str:
        return self._status_message

    # ── 명령(Commands) ──
    def add_todo(self) -> None:
        """새 할일 추가 커맨드"""
        if not self._new_todo_title.strip():
            self._status_message = "제목을 입력해주세요."
            self.notify("status_message")
            return

        new_item = TodoItem(
            id=len(self._todo_items) + 1,
            title=self._new_todo_title
        )

        if not new_item.validate():
            self._status_message = "유효하지 않은 할일입니다."
            self.notify("status_message")
            return

        self._todo_service.add(new_item)
        self._todo_items.append(new_item)
        self.new_todo_title = ""  # 입력 필드 초기화

        completed = sum(1 for t in self._todo_items if t.is_completed)
        total = len(self._todo_items)
        self._status_message = f"총 {total}개 중 {completed}개 완료"
        self.notify("todo_items")
        self.notify("status_message")

    def can_add_todo(self) -> bool:
        """추가 가능 여부 확인"""
        return bool(self._new_todo_title.strip())

    def _load_todos(self) -> None:
        """Model에서 데이터 로드"""
        self._todo_items = self._todo_service.get_all()
        self.notify("todo_items")
```

---

## 4. 데이터 바인딩 개념

데이터 바인딩은 **View의 UI 요소와 ViewModel의 속성을 자동으로 동기화**하는 메커니즘입니다. MVVM 패턴의 핵심 동력이라 할 수 있습니다.

### 바인딩 방식 비교

```
┌─────────────────────────────────────────────────────────────────┐
│                    데이터 바인딩 방식 비교                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [ One-Way Binding (단방향) ]                                    │
│                                                                 │
│    ViewModel ──────────────► View                               │
│    (Source)     데이터 흐름    (Target)                           │
│                                                                 │
│    예: 레이블에 텍스트 표시                                       │
│    <TextBlock Text="{Binding UserName}"/>                       │
│    ViewModel의 UserName이 변경되면 자동으로 화면 갱신              │
│                                                                 │
│  ───────────────────────────────────────────────────────────    │
│                                                                 │
│  [ Two-Way Binding (양방향) ]                                    │
│                                                                 │
│    ViewModel ◄──────────────► View                              │
│    (Source)     데이터 흐름     (Target)                          │
│                                                                 │
│    예: 입력 필드와 속성 연결                                      │
│    <TextBox Text="{Binding UserName, Mode=TwoWay}"/>            │
│    사용자가 입력하면 ViewModel 갱신, ViewModel이 바뀌면 UI 갱신    │
│                                                                 │
│  ───────────────────────────────────────────────────────────    │
│                                                                 │
│  [ One-Way to Source (소스 방향 단방향) ]                         │
│                                                                 │
│    ViewModel ◄─────────────── View                              │
│    (Source)     데이터 흐름     (Target)                          │
│                                                                 │
│    예: View의 상태를 ViewModel에 전달만                           │
│    <Slider Value="{Binding Volume, Mode=OneWayToSource}"/>      │
│                                                                 │
│  ───────────────────────────────────────────────────────────    │
│                                                                 │
│  [ One-Time (1회성) ]                                            │
│                                                                 │
│    ViewModel ─ ─ ─ ─ ─ ─ ─► View                               │
│    (Source)     최초 1회만    (Target)                            │
│                                                                 │
│    예: 초기 설정값 표시 (이후 변경 불필요)                         │
│    <TextBlock Text="{Binding AppVersion, Mode=OneTime}"/>       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 바인딩 동작 원리

```
    사용자가 TextBox에 "새 할일" 입력
              │
              ▼
    ┌────────────────────┐
    │   View (TextBox)   │
    │   Text 속성 변경    │
    └────────┬───────────┘
             │  Two-Way 바인딩 엔진이 감지
             ▼
    ┌────────────────────────────┐
    │   ViewModel                │
    │   NewTodoTitle = "새 할일"  │
    │   setter 호출됨            │
    └────────┬───────────────────┘
             │  PropertyChanged 이벤트 발생
             ▼
    ┌────────────────────────────┐
    │   바인딩 엔진이 이벤트 수신  │
    │   연결된 모든 UI 갱신       │
    └────────┬───────────────────┘
             │
             ▼
    다른 곳에서 NewTodoTitle을
    바인딩한 UI 요소도 자동 갱신
```

---

## 5. INotifyPropertyChanged 구현 (C#)

`INotifyPropertyChanged`는 MVVM에서 **ViewModel이 View에게 속성 변경을 알리기 위한 핵심 인터페이스**입니다.

### 기본 구현

```csharp
// C# - INotifyPropertyChanged 기본 구현
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace TodoApp.ViewModels
{
    /// <summary>
    /// 모든 ViewModel의 기본 클래스
    /// INotifyPropertyChanged를 구현하여 속성 변경 알림 제공
    /// </summary>
    public abstract class ViewModelBase : INotifyPropertyChanged
    {
        // 속성이 변경될 때 발생하는 이벤트
        public event PropertyChangedEventHandler? PropertyChanged;

        /// <summary>
        /// 속성 변경을 View에 알립니다.
        /// [CallerMemberName]을 사용하면 호출한 속성의 이름이 자동으로 전달됩니다.
        /// </summary>
        protected virtual void OnPropertyChanged(
            [CallerMemberName] string? propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        /// <summary>
        /// 속성 값을 설정하고, 값이 변경되었으면 알림을 발생시킵니다.
        /// 반환값: 값이 실제로 변경되었으면 true
        /// </summary>
        protected bool SetProperty<T>(
            ref T field,
            T value,
            [CallerMemberName] string? propertyName = null)
        {
            // 같은 값이면 아무것도 하지 않음 (불필요한 UI 갱신 방지)
            if (EqualityComparer<T>.Default.Equals(field, value))
                return false;

            field = value;
            OnPropertyChanged(propertyName);
            return true;
        }
    }
}
```

### 실전 사용 예제

```csharp
// C# - ViewModel에서 INotifyPropertyChanged 활용
using System.Collections.ObjectModel;

namespace TodoApp.ViewModels
{
    public class TodoListViewModel : ViewModelBase
    {
        // ── 내부 필드 ──
        private string _newTodoTitle = string.Empty;
        private string _statusMessage = "준비됨";
        private TodoItem? _selectedTodo;

        // ── 속성(Properties) - View와 바인딩됨 ──

        /// <summary>
        /// 새 할일 제목 (Two-Way 바인딩 대상)
        /// SetProperty가 자동으로 PropertyChanged 이벤트를 발생시킵니다.
        /// </summary>
        public string NewTodoTitle
        {
            get => _newTodoTitle;
            set => SetProperty(ref _newTodoTitle, value);
            // SetProperty 내부에서:
            // 1. _newTodoTitle = value 설정
            // 2. PropertyChanged("NewTodoTitle") 이벤트 발생
            // 3. View의 TextBox가 자동 갱신
        }

        /// <summary>
        /// 상태 메시지 (One-Way 바인딩 대상)
        /// </summary>
        public string StatusMessage
        {
            get => _statusMessage;
            set => SetProperty(ref _statusMessage, value);
        }

        /// <summary>
        /// 선택된 할일 항목
        /// </summary>
        public TodoItem? SelectedTodo
        {
            get => _selectedTodo;
            set => SetProperty(ref _selectedTodo, value);
        }

        /// <summary>
        /// 할일 목록 (ObservableCollection은 자체적으로 변경 알림 제공)
        /// 항목 추가/삭제 시 View의 ListBox가 자동 갱신됨
        /// </summary>
        public ObservableCollection<TodoItem> TodoItems { get; }
            = new ObservableCollection<TodoItem>();
    }
}
```

> **실무 팁**: `ObservableCollection<T>`은 컬렉션의 추가/삭제를 자동으로 알려주므로, 리스트 바인딩에 `List<T>` 대신 반드시 사용하세요. 다만, 컬렉션 내부 항목의 속성 변경은 각 항목이 별도로 `INotifyPropertyChanged`를 구현해야 합니다.

---

## 6. Command 패턴과 MVVM

### ICommand 인터페이스

MVVM에서 **사용자의 액션(버튼 클릭 등)을 처리하는 표준 방법**이 Command 패턴입니다. View는 이벤트 핸들러 대신 Command에 바인딩합니다.

```
┌─────────────────────────────────────────────────────────────┐
│               Command 패턴 동작 흐름                          │
│                                                             │
│  사용자가 "추가" 버튼 클릭                                    │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                            │
│  │ Button      │   Command="{Binding AddTodoCommand}"       │
│  │ (View)      │                                            │
│  └──────┬──────┘                                            │
│         │  CanExecute() 호출 → true/false                   │
│         │  (false면 버튼 비활성화)                            │
│         │                                                   │
│         │  true면 Execute() 호출                             │
│         ▼                                                   │
│  ┌─────────────────┐                                        │
│  │ ICommand        │                                        │
│  │ (AddTodoCommand)│─────► ViewModel의 AddTodo() 실행       │
│  └─────────────────┘                                        │
│                                                             │
│  결과: View에 이벤트 핸들러(코드비하인드) 없이                  │
│        ViewModel의 메서드가 실행됨!                           │
└─────────────────────────────────────────────────────────────┘
```

### RelayCommand 구현

```csharp
// C# - RelayCommand 구현
using System;
using System.Windows.Input;

namespace TodoApp.Commands
{
    /// <summary>
    /// 범용 Command 구현체
    /// ViewModel의 메서드를 Command로 감싸서 View에 노출합니다.
    /// </summary>
    public class RelayCommand : ICommand
    {
        private readonly Action<object?> _execute;
        private readonly Predicate<object?>? _canExecute;

        public RelayCommand(Action<object?> execute, Predicate<object?>? canExecute = null)
        {
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        // 매개변수 없는 버전의 편의 생성자
        public RelayCommand(Action execute, Func<bool>? canExecute = null)
            : this(
                _ => execute(),
                canExecute != null ? _ => canExecute() : null)
        {
        }

        /// <summary>
        /// Command 실행 가능 여부를 반환합니다.
        /// false를 반환하면 바인딩된 Button이 자동으로 비활성화됩니다.
        /// </summary>
        public bool CanExecute(object? parameter)
        {
            return _canExecute == null || _canExecute(parameter);
        }

        /// <summary>
        /// Command를 실행합니다.
        /// </summary>
        public void Execute(object? parameter)
        {
            _execute(parameter);
        }

        /// <summary>
        /// CanExecute의 결과가 변경될 수 있음을 알립니다.
        /// WPF의 CommandManager가 이를 감지하여 UI를 갱신합니다.
        /// </summary>
        public event EventHandler? CanExecuteChanged
        {
            add => CommandManager.RequerySuggested += value;
            remove => CommandManager.RequerySuggested -= value;
        }
    }
}
```

### ViewModel에서 Command 사용

```csharp
// C# - ViewModel에서 Command 활용
namespace TodoApp.ViewModels
{
    public class TodoListViewModel : ViewModelBase
    {
        private readonly ITodoService _todoService;

        // ── Commands ──
        public ICommand AddTodoCommand { get; }
        public ICommand DeleteTodoCommand { get; }
        public ICommand ToggleCompleteCommand { get; }

        public TodoListViewModel(ITodoService todoService)
        {
            _todoService = todoService;

            // Command 초기화 - 실행할 메서드와 실행 가능 조건을 전달
            AddTodoCommand = new RelayCommand(
                execute: AddTodo,                            // 실행할 메서드
                canExecute: () => !string.IsNullOrWhiteSpace(NewTodoTitle)  // 조건
            );

            DeleteTodoCommand = new RelayCommand(
                execute: DeleteTodo,
                canExecute: () => SelectedTodo != null
            );

            ToggleCompleteCommand = new RelayCommand(
                execute: ToggleComplete,
                canExecute: () => SelectedTodo != null
            );
        }

        private void AddTodo()
        {
            var newItem = new TodoItem
            {
                Id = TodoItems.Count + 1,
                Title = NewTodoTitle
            };

            if (!newItem.Validate())
            {
                StatusMessage = "유효하지 않은 할일입니다.";
                return;
            }

            _todoService.Add(newItem);
            TodoItems.Add(newItem);
            NewTodoTitle = string.Empty;  // 입력 필드 초기화

            UpdateStatusMessage();
        }

        private void DeleteTodo()
        {
            if (SelectedTodo == null) return;

            _todoService.Delete(SelectedTodo.Id);
            TodoItems.Remove(SelectedTodo);
            SelectedTodo = null;

            UpdateStatusMessage();
        }

        private void ToggleComplete()
        {
            if (SelectedTodo == null) return;

            if (SelectedTodo.IsCompleted)
                SelectedTodo.Reopen();
            else
                SelectedTodo.Complete();

            UpdateStatusMessage();
        }

        private void UpdateStatusMessage()
        {
            int completed = TodoItems.Count(t => t.IsCompleted);
            StatusMessage = $"총 {TodoItems.Count}개 중 {completed}개 완료";
        }
    }
}
```

---

## 7. Python에서의 MVVM 적용

Python 생태계에는 WPF 같은 강력한 데이터 바인딩 프레임워크가 기본 제공되지 않습니다. 하지만 MVVM 개념을 적용하는 것은 충분히 가능합니다.

### tkinter를 활용한 MVVM 구현

```python
# Python - tkinter 기반 MVVM 구현

import tkinter as tk
from tkinter import messagebox
from dataclasses import dataclass, field
from datetime import datetime
from typing import List, Optional, Callable


# ══════════════════════════════════════════════════════
# MODEL 레이어
# ══════════════════════════════════════════════════════

@dataclass
class TodoItem:
    """할일 항목 모델"""
    id: int
    title: str
    is_completed: bool = False
    created_at: datetime = field(default_factory=datetime.now)

    def complete(self) -> None:
        self.is_completed = True

    def reopen(self) -> None:
        self.is_completed = False

    def validate(self) -> bool:
        return bool(self.title and self.title.strip())


class TodoRepository:
    """할일 저장소 (인메모리)"""

    def __init__(self):
        self._items: List[TodoItem] = []
        self._next_id = 1

    def get_all(self) -> List[TodoItem]:
        return list(self._items)

    def add(self, item: TodoItem) -> TodoItem:
        item.id = self._next_id
        self._next_id += 1
        self._items.append(item)
        return item

    def delete(self, item_id: int) -> None:
        self._items = [i for i in self._items if i.id != item_id]


# ══════════════════════════════════════════════════════
# VIEWMODEL 레이어
# ══════════════════════════════════════════════════════

class ObservableProperty:
    """데이터 바인딩을 모방하는 관찰 가능 속성"""

    def __init__(self, initial_value=None):
        self._value = initial_value
        self._callbacks: List[Callable] = []

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, new_value):
        if self._value != new_value:
            self._value = new_value
            self._notify()

    def bind(self, callback: Callable) -> None:
        """변경 시 호출될 콜백 등록 (바인딩)"""
        self._callbacks.append(callback)

    def _notify(self) -> None:
        for callback in self._callbacks:
            callback(self._value)


class TodoListViewModel:
    """할일 목록 ViewModel"""

    def __init__(self, repository: TodoRepository):
        self._repository = repository

        # Observable 속성들 - View와 바인딩됨
        self.todo_items = ObservableProperty([])
        self.new_todo_title = ObservableProperty("")
        self.status_message = ObservableProperty("준비됨")
        self.selected_index = ObservableProperty(-1)

        # 초기 데이터 로드
        self._refresh_items()

    def add_todo(self) -> None:
        """새 할일 추가 (Command)"""
        title = self.new_todo_title.value
        if not title or not title.strip():
            self.status_message.value = "제목을 입력해주세요."
            return

        item = TodoItem(id=0, title=title.strip())
        if not item.validate():
            self.status_message.value = "유효하지 않은 할일입니다."
            return

        self._repository.add(item)
        self.new_todo_title.value = ""
        self._refresh_items()

    def delete_todo(self) -> None:
        """선택된 할일 삭제 (Command)"""
        idx = self.selected_index.value
        items = self.todo_items.value
        if idx < 0 or idx >= len(items):
            self.status_message.value = "삭제할 항목을 선택해주세요."
            return

        self._repository.delete(items[idx].id)
        self.selected_index.value = -1
        self._refresh_items()

    def toggle_complete(self) -> None:
        """완료 상태 토글 (Command)"""
        idx = self.selected_index.value
        items = self.todo_items.value
        if idx < 0 or idx >= len(items):
            return

        item = items[idx]
        if item.is_completed:
            item.reopen()
        else:
            item.complete()
        self._refresh_items()

    def _refresh_items(self) -> None:
        """항목 목록 새로고침 및 상태 메시지 갱신"""
        items = self._repository.get_all()
        self.todo_items.value = items
        completed = sum(1 for i in items if i.is_completed)
        self.status_message.value = f"총 {len(items)}개 중 {completed}개 완료"


# ══════════════════════════════════════════════════════
# VIEW 레이어
# ══════════════════════════════════════════════════════

class TodoListView:
    """할일 목록 View (tkinter)"""

    def __init__(self, root: tk.Tk, view_model: TodoListViewModel):
        self._vm = view_model
        self._root = root
        self._root.title("할일 목록 - MVVM 예제")
        self._root.geometry("400x500")

        self._setup_ui()
        self._setup_bindings()

    def _setup_ui(self) -> None:
        """UI 구성 요소 생성"""
        # 제목
        tk.Label(self._root, text="나의 할일 목록",
                 font=("Arial", 18, "bold")).pack(pady=10)

        # 입력 영역
        input_frame = tk.Frame(self._root)
        input_frame.pack(fill=tk.X, padx=10)

        self._entry = tk.Entry(input_frame, font=("Arial", 12))
        self._entry.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=(0, 5))

        self._add_btn = tk.Button(input_frame, text="추가", width=8)
        self._add_btn.pack(side=tk.RIGHT)

        # 할일 목록
        self._listbox = tk.Listbox(self._root, font=("Arial", 11),
                                    selectmode=tk.SINGLE)
        self._listbox.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        # 버튼 영역
        btn_frame = tk.Frame(self._root)
        btn_frame.pack(fill=tk.X, padx=10)

        self._complete_btn = tk.Button(btn_frame, text="완료/해제", width=12)
        self._complete_btn.pack(side=tk.LEFT, padx=(0, 5))

        self._delete_btn = tk.Button(btn_frame, text="삭제", width=8)
        self._delete_btn.pack(side=tk.LEFT)

        # 상태 표시
        self._status_label = tk.Label(self._root, text="",
                                       font=("Arial", 10), anchor=tk.W)
        self._status_label.pack(fill=tk.X, padx=10, pady=10)

    def _setup_bindings(self) -> None:
        """ViewModel 속성과 View 요소를 바인딩"""

        # Command 바인딩 (Button -> ViewModel 메서드)
        self._add_btn.config(command=self._vm.add_todo)
        self._delete_btn.config(command=self._vm.delete_todo)
        self._complete_btn.config(command=self._vm.toggle_complete)
        self._entry.bind("<Return>", lambda e: self._vm.add_todo())

        # 데이터 바인딩: Entry <-> ViewModel.new_todo_title (Two-Way)
        self._entry_var = tk.StringVar()
        self._entry.config(textvariable=self._entry_var)
        self._entry_var.trace_add("write",
            lambda *_: setattr(self._vm.new_todo_title, 'value',
                               self._entry_var.get()))
        self._vm.new_todo_title.bind(
            lambda val: self._entry_var.set(val))

        # 데이터 바인딩: Listbox <- ViewModel.todo_items (One-Way)
        self._vm.todo_items.bind(self._update_listbox)

        # 데이터 바인딩: Label <- ViewModel.status_message (One-Way)
        self._vm.status_message.bind(
            lambda val: self._status_label.config(text=val))

        # 선택 변경 -> ViewModel.selected_index
        self._listbox.bind("<<ListboxSelect>>", self._on_select)

    def _update_listbox(self, items: List[TodoItem]) -> None:
        """Listbox UI 갱신"""
        self._listbox.delete(0, tk.END)
        for item in items:
            prefix = "[v]" if item.is_completed else "[ ]"
            self._listbox.insert(tk.END, f"{prefix} {item.title}")

    def _on_select(self, event) -> None:
        """Listbox 선택 변경 시 ViewModel에 알림"""
        selection = self._listbox.curselection()
        self._vm.selected_index.value = selection[0] if selection else -1


# ══════════════════════════════════════════════════════
# 앱 실행 (의존성 조립)
# ══════════════════════════════════════════════════════

if __name__ == "__main__":
    # 의존성 조립
    repository = TodoRepository()
    view_model = TodoListViewModel(repository)

    # View 생성 및 실행
    root = tk.Tk()
    view = TodoListView(root, view_model)
    root.mainloop()
```

---

## 8. C#/WPF MVVM 실전 예제 - 할일 목록 앱

### 완전한 예제 프로젝트 구조

```
TodoApp/
├── Models/
│   ├── TodoItem.cs              # 엔티티
│   └── ITodoService.cs          # 서비스 인터페이스
├── Services/
│   └── InMemoryTodoService.cs   # 서비스 구현
├── ViewModels/
│   ├── ViewModelBase.cs         # 기본 ViewModel
│   ├── RelayCommand.cs          # Command 구현
│   └── TodoListViewModel.cs     # 메인 ViewModel
├── Views/
│   └── TodoListView.xaml        # 메인 View
│   └── TodoListView.xaml.cs     # 코드 비하인드 (최소)
├── App.xaml
└── App.xaml.cs
```

### 서비스 인터페이스와 구현

```csharp
// Models/ITodoService.cs
namespace TodoApp.Models
{
    public interface ITodoService
    {
        List<TodoItem> GetAll();
        TodoItem Add(TodoItem item);
        void Delete(int id);
        void Update(TodoItem item);
    }
}

// Services/InMemoryTodoService.cs
namespace TodoApp.Services
{
    public class InMemoryTodoService : ITodoService
    {
        private readonly List<TodoItem> _items = new();
        private int _nextId = 1;

        public List<TodoItem> GetAll() => new(_items);

        public TodoItem Add(TodoItem item)
        {
            item.Id = _nextId++;
            _items.Add(item);
            return item;
        }

        public void Delete(int id)
        {
            _items.RemoveAll(i => i.Id == id);
        }

        public void Update(TodoItem item)
        {
            var existing = _items.FirstOrDefault(i => i.Id == item.Id);
            if (existing != null)
            {
                existing.Title = item.Title;
                existing.IsCompleted = item.IsCompleted;
                existing.CompletedAt = item.CompletedAt;
            }
        }
    }
}
```

### 완성된 ViewModel

```csharp
// ViewModels/TodoListViewModel.cs
using System.Collections.ObjectModel;
using System.Linq;
using System.Windows.Input;
using TodoApp.Commands;
using TodoApp.Models;

namespace TodoApp.ViewModels
{
    public class TodoListViewModel : ViewModelBase
    {
        private readonly ITodoService _todoService;
        private string _newTodoTitle = string.Empty;
        private string _statusMessage = "준비됨";
        private TodoItem? _selectedTodo;

        // ── 생성자: 의존성 주입 ──
        public TodoListViewModel(ITodoService todoService)
        {
            _todoService = todoService;

            // ObservableCollection: 항목 추가/삭제 시 자동 UI 갱신
            TodoItems = new ObservableCollection<TodoItem>();

            // Command 초기화
            AddTodoCommand = new RelayCommand(AddTodo,
                () => !string.IsNullOrWhiteSpace(NewTodoTitle));
            DeleteTodoCommand = new RelayCommand(DeleteTodo,
                () => SelectedTodo != null);
            ToggleCompleteCommand = new RelayCommand(ToggleComplete,
                () => SelectedTodo != null);
            ClearCompletedCommand = new RelayCommand(ClearCompleted,
                () => TodoItems.Any(t => t.IsCompleted));

            LoadTodos();
        }

        // ── 속성 ──
        public ObservableCollection<TodoItem> TodoItems { get; }

        public string NewTodoTitle
        {
            get => _newTodoTitle;
            set => SetProperty(ref _newTodoTitle, value);
        }

        public string StatusMessage
        {
            get => _statusMessage;
            set => SetProperty(ref _statusMessage, value);
        }

        public TodoItem? SelectedTodo
        {
            get => _selectedTodo;
            set => SetProperty(ref _selectedTodo, value);
        }

        // ── Commands ──
        public ICommand AddTodoCommand { get; }
        public ICommand DeleteTodoCommand { get; }
        public ICommand ToggleCompleteCommand { get; }
        public ICommand ClearCompletedCommand { get; }

        // ── 비공개 메서드 ──
        private void LoadTodos()
        {
            TodoItems.Clear();
            foreach (var item in _todoService.GetAll())
            {
                TodoItems.Add(item);
            }
            UpdateStatus();
        }

        private void AddTodo()
        {
            var item = new TodoItem { Title = NewTodoTitle.Trim() };

            if (!item.Validate())
            {
                StatusMessage = "유효하지 않은 제목입니다.";
                return;
            }

            var added = _todoService.Add(item);
            TodoItems.Add(added);
            NewTodoTitle = string.Empty;
            UpdateStatus();
        }

        private void DeleteTodo()
        {
            if (SelectedTodo == null) return;
            _todoService.Delete(SelectedTodo.Id);
            TodoItems.Remove(SelectedTodo);
            SelectedTodo = null;
            UpdateStatus();
        }

        private void ToggleComplete()
        {
            if (SelectedTodo == null) return;

            if (SelectedTodo.IsCompleted)
                SelectedTodo.Reopen();
            else
                SelectedTodo.Complete();

            _todoService.Update(SelectedTodo);
            UpdateStatus();
        }

        private void ClearCompleted()
        {
            var completed = TodoItems.Where(t => t.IsCompleted).ToList();
            foreach (var item in completed)
            {
                _todoService.Delete(item.Id);
                TodoItems.Remove(item);
            }
            UpdateStatus();
        }

        private void UpdateStatus()
        {
            int total = TodoItems.Count;
            int done = TodoItems.Count(t => t.IsCompleted);
            StatusMessage = $"총 {total}개 중 {done}개 완료";
        }
    }
}
```

### View - XAML

```xml
<!-- Views/TodoListView.xaml -->
<Window x:Class="TodoApp.Views.TodoListView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="할일 목록 - MVVM" Height="550" Width="450"
        WindowStartupLocation="CenterScreen">

    <Window.Resources>
        <!-- 완료된 항목에 취소선 적용하는 스타일 -->
        <Style x:Key="CompletedStyle" TargetType="TextBlock">
            <Style.Triggers>
                <DataTrigger Binding="{Binding IsCompleted}" Value="True">
                    <Setter Property="TextDecorations" Value="Strikethrough"/>
                    <Setter Property="Foreground" Value="Gray"/>
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </Window.Resources>

    <Grid Margin="15">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- 제목 -->
        <TextBlock Grid.Row="0" Text="나의 할일 목록"
                   FontSize="24" FontWeight="Bold" Margin="0,0,0,15"/>

        <!-- 입력 영역 -->
        <Grid Grid.Row="1" Margin="0,0,0,10">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>

            <TextBox Grid.Column="0"
                     Text="{Binding NewTodoTitle, UpdateSourceTrigger=PropertyChanged}"
                     FontSize="14" Padding="5"
                     Margin="0,0,8,0"/>

            <Button Grid.Column="1"
                    Content="추가"
                    Command="{Binding AddTodoCommand}"
                    Width="80" FontSize="14" Padding="5"/>
        </Grid>

        <!-- 할일 목록 -->
        <ListBox Grid.Row="2"
                 ItemsSource="{Binding TodoItems}"
                 SelectedItem="{Binding SelectedTodo}"
                 BorderThickness="1" Padding="5">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="3">
                        <CheckBox IsChecked="{Binding IsCompleted}"
                                  VerticalAlignment="Center"
                                  Margin="0,0,10,0"/>
                        <TextBlock Text="{Binding Title}"
                                   Style="{StaticResource CompletedStyle}"
                                   FontSize="14"
                                   VerticalAlignment="Center"/>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>

        <!-- 액션 버튼들 -->
        <StackPanel Grid.Row="3" Orientation="Horizontal"
                    Margin="0,10,0,0">
            <Button Content="완료/해제"
                    Command="{Binding ToggleCompleteCommand}"
                    Width="90" Margin="0,0,5,0"/>
            <Button Content="삭제"
                    Command="{Binding DeleteTodoCommand}"
                    Width="70" Margin="0,0,5,0"/>
            <Button Content="완료 항목 정리"
                    Command="{Binding ClearCompletedCommand}"
                    Width="110"/>
        </StackPanel>

        <!-- 상태 바 -->
        <TextBlock Grid.Row="4"
                   Text="{Binding StatusMessage}"
                   FontSize="12" Foreground="DarkGray"
                   Margin="0,10,0,0"/>
    </Grid>
</Window>
```

### View - 코드 비하인드 (최소한)

```csharp
// Views/TodoListView.xaml.cs
using System.Windows;
using TodoApp.Services;
using TodoApp.ViewModels;

namespace TodoApp.Views
{
    public partial class TodoListView : Window
    {
        public TodoListView()
        {
            InitializeComponent();

            // ViewModel을 DataContext에 설정
            // 실무에서는 DI 컨테이너를 통해 주입
            var service = new InMemoryTodoService();
            DataContext = new TodoListViewModel(service);
        }
    }
}
```

---

## 9. MVC vs MVP vs MVVM 비교표

```
┌──────────────┬─────────────────┬─────────────────┬──────────────────┐
│    항목       │      MVC        │      MVP        │      MVVM        │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 중재자        │ Controller      │ Presenter       │ ViewModel        │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ View-중재자   │ Controller가    │ Presenter가     │ 데이터 바인딩으로  │
│ 연결 방식     │ View를 선택     │ View 인터페이스  │ 자동 연결         │
│              │                 │ 를 통해 갱신     │                  │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ View의       │ Controller를    │ Presenter를     │ ViewModel을      │
│ 중재자 인식   │ 모름 (느슨)     │ 직접 참조       │ 바인딩으로 참조   │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 1:N 관계     │ Controller :    │ Presenter :     │ ViewModel :      │
│              │ 여러 View       │ 1 View (보통)   │ 여러 View 가능   │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 데이터 흐름   │ 단방향          │ Presenter 경유   │ 양방향 바인딩     │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 테스트 용이성 │ 보통            │ 좋음            │ 매우 좋음         │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 주요 사용처   │ 웹 (ASP.NET     │ Android         │ WPF, MAUI,      │
│              │ MVC, Spring)    │ (전통적), Win   │ Xamarin, SwiftUI │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 복잡도       │ 낮음            │ 중간            │ 높음 (초기)       │
├──────────────┼─────────────────┼─────────────────┼──────────────────┤
│ 학습 곡선    │ 완만            │ 중간            │ 가파름            │
└──────────────┴─────────────────┴─────────────────┴──────────────────┘
```

### 시각적 비교

```
[ MVC ]                    [ MVP ]                    [ MVVM ]

   View                      View                      View
    │  ▲                      │▲                        ▲║
    │  │                      ││                        ║║ 데이터
    │  │ 갱신                  ││ 인터페이스              ║║ 바인딩
    ▼  │                      ▼│ 통해 갱신              ║▼
 Controller ──►Model     Presenter ──►Model       ViewModel──►Model
                                                    (INotify
                                                     PropertyChanged)
```

---

## 10. MVVM 프레임워크 소개

### CommunityToolkit.Mvvm (권장)

Microsoft에서 공식 지원하는 경량 MVVM 툴킷입니다. 보일러플레이트 코드를 획기적으로 줄여줍니다.

```csharp
// 기존 방식 (보일러플레이트가 많음)
public class TodoListViewModel : ViewModelBase
{
    private string _newTodoTitle = string.Empty;

    public string NewTodoTitle
    {
        get => _newTodoTitle;
        set => SetProperty(ref _newTodoTitle, value);
    }
}

// CommunityToolkit.Mvvm 사용 시 (Source Generator 활용)
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

// partial 키워드 필수! Source Generator가 나머지 코드를 자동 생성
public partial class TodoListViewModel : ObservableObject
{
    // [ObservableProperty] → 자동으로 NewTodoTitle 속성 생성
    // get/set, PropertyChanged 이벤트 모두 자동 처리
    [ObservableProperty]
    private string _newTodoTitle = string.Empty;

    [ObservableProperty]
    private string _statusMessage = "준비됨";

    [ObservableProperty]
    private TodoItem? _selectedTodo;

    // [RelayCommand] → 자동으로 AddTodoCommand 속성 생성
    [RelayCommand(CanExecute = nameof(CanAddTodo))]
    private void AddTodo()
    {
        // 비즈니스 로직...
    }

    private bool CanAddTodo() => !string.IsNullOrWhiteSpace(NewTodoTitle);
}
```

### 주요 MVVM 프레임워크 비교

| 프레임워크 | 특징 | 추천 대상 |
|-----------|------|----------|
| **CommunityToolkit.Mvvm** | Microsoft 공식, 경량, Source Generator | 신규 프로젝트 (가장 추천) |
| **Prism** | 풀기능, 모듈화, DI 내장, Navigation | 대규모 엔터프라이즈 앱 |
| **ReactiveUI** | Reactive Extensions 기반, 반응형 | RX에 익숙한 팀 |
| **Caliburn.Micro** | 컨벤션 기반 자동 바인딩 | 빠른 프로토타이핑 |
| **MvvmLight** | 경량 (현재 유지보수 중단) | 레거시 프로젝트만 |

---

## 11. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **테스트 용이성** | ViewModel을 UI 없이 단위 테스트 가능 |
| **관심사 분리** | UI, 로직, 데이터가 명확하게 분리 |
| **코드 재사용** | 동일 ViewModel을 다른 View에서 재사용 |
| **병렬 개발** | 디자이너(XAML)와 개발자(C#) 동시 작업 |
| **유지보수성** | 변경 영향 범위가 제한적 |
| **데이터 바인딩** | 수동 UI 갱신 코드 불필요 |

### 단점

| 단점 | 설명 |
|------|------|
| **초기 복잡도** | 간단한 화면에도 많은 코드가 필요 |
| **학습 곡선** | 바인딩, Command 등 개념 이해 필요 |
| **디버깅 어려움** | 바인딩 오류는 런타임에서만 발견 |
| **과도한 추상화** | 작은 프로젝트에서는 오버엔지니어링 |
| **메모리** | 바인딩 인프라로 인한 메모리 사용 증가 |
| **성능** | 대량 바인딩 시 성능 이슈 가능 |

### 언제 사용하면 좋을까?

```
적합한 경우:                          부적합한 경우:
───────────────                      ───────────────
[O] 중간~대규모 데스크톱/모바일 앱     [X] 간단한 프로토타입
[O] 복잡한 UI 상호작용이 있는 앱       [X] 단순 CRUD 페이지
[O] 단위 테스트가 중요한 프로젝트      [X] 일회성 도구/스크립트
[O] 장기 유지보수가 필요한 프로젝트     [X] 학습 목적의 작은 프로젝트
[O] 팀 규모가 3인 이상               [X] 1인 개발의 소규모 앱
```

---

## 12. 폴더 구조 예시

### C#/WPF 프로젝트

```
TodoApp.sln
│
├── TodoApp/                         # 메인 WPF 프로젝트
│   ├── App.xaml                     # 앱 진입점
│   ├── App.xaml.cs
│   │
│   ├── Models/                      # Model 레이어
│   │   ├── TodoItem.cs              # 엔티티
│   │   ├── Category.cs
│   │   └── Interfaces/
│   │       └── ITodoService.cs
│   │
│   ├── ViewModels/                  # ViewModel 레이어
│   │   ├── Base/
│   │   │   ├── ViewModelBase.cs     # 기본 ViewModel
│   │   │   └── RelayCommand.cs      # Command 구현
│   │   ├── TodoListViewModel.cs     # 할일 목록 VM
│   │   ├── TodoDetailViewModel.cs   # 할일 상세 VM
│   │   └── MainViewModel.cs         # 메인 윈도우 VM
│   │
│   ├── Views/                       # View 레이어
│   │   ├── TodoListView.xaml        # 할일 목록 화면
│   │   ├── TodoDetailView.xaml      # 할일 상세 화면
│   │   ├── MainWindow.xaml          # 메인 윈도우
│   │   └── Converters/              # 값 변환기
│   │       └── BoolToVisibilityConverter.cs
│   │
│   └── Services/                    # 서비스 구현
│       ├── InMemoryTodoService.cs
│       └── SqliteTodoService.cs
│
└── TodoApp.Tests/                   # 테스트 프로젝트
    ├── ViewModels/
    │   ├── TodoListViewModelTests.cs
    │   └── TodoDetailViewModelTests.cs
    └── Models/
        └── TodoItemTests.cs
```

### Python 프로젝트

```
todo_app/
│
├── main.py                          # 앱 진입점
│
├── models/                          # Model 레이어
│   ├── __init__.py
│   ├── todo_item.py                 # 엔티티
│   ├── category.py
│   └── interfaces/
│       └── todo_repository.py       # 저장소 인터페이스 (ABC)
│
├── viewmodels/                      # ViewModel 레이어
│   ├── __init__.py
│   ├── base/
│   │   ├── __init__.py
│   │   ├── observable.py            # Observable 기본 클래스
│   │   └── command.py               # Command 구현
│   ├── todo_list_viewmodel.py       # 할일 목록 VM
│   └── todo_detail_viewmodel.py     # 할일 상세 VM
│
├── views/                           # View 레이어
│   ├── __init__.py
│   ├── todo_list_view.py            # 할일 목록 화면 (tkinter)
│   └── todo_detail_view.py          # 할일 상세 화면
│
├── services/                        # 서비스 구현
│   ├── __init__.py
│   ├── memory_todo_repository.py
│   └── sqlite_todo_repository.py
│
└── tests/                           # 테스트
    ├── test_todo_list_viewmodel.py
    └── test_todo_item.py
```

---

## 13. 정리 및 체크리스트

### MVVM 핵심 정리

```
┌─────────────────────────────────────────────────────────────┐
│                    MVVM 핵심 원칙 정리                        │
│                                                             │
│  1. View는 ViewModel만 알고, Model을 직접 참조하지 않는다.     │
│  2. ViewModel은 View를 알지 못한다. (View 타입을 참조 안 함)   │
│  3. Model은 ViewModel을 알지 못한다.                         │
│  4. 데이터 바인딩이 View와 ViewModel을 연결한다.               │
│  5. Command 패턴으로 사용자 액션을 처리한다.                   │
│  6. INotifyPropertyChanged로 변경을 통지한다.                │
│  7. 각 구성 요소는 독립적으로 테스트 가능해야 한다.              │
└─────────────────────────────────────────────────────────────┘
```

### 실무 체크리스트

- [ ] Model에 UI 관련 코드가 포함되어 있지 않은가?
- [ ] ViewModel이 View의 구체적인 타입(Window, Button 등)을 참조하지 않는가?
- [ ] View의 코드 비하인드에 비즈니스 로직이 없는가?
- [ ] ViewModel의 속성 변경 시 PropertyChanged가 발생하는가?
- [ ] 컬렉션 바인딩에 ObservableCollection을 사용하는가?
- [ ] Command의 CanExecute가 적절히 구현되어 있는가?
- [ ] ViewModel을 UI 없이 단위 테스트할 수 있는가?
- [ ] 의존성 주입(DI)을 활용하고 있는가?
- [ ] MVVM 프레임워크(CommunityToolkit.Mvvm)를 검토했는가?
- [ ] 과도한 추상화 없이 적절한 수준으로 분리했는가?

---

## 14. 참고 자료

### 공식 문서
- [Microsoft - MVVM 패턴](https://learn.microsoft.com/ko-kr/dotnet/architecture/maui/mvvm)
- [CommunityToolkit.Mvvm](https://learn.microsoft.com/ko-kr/dotnet/communitytoolkit/mvvm/)
- [WPF 데이터 바인딩 개요](https://learn.microsoft.com/ko-kr/dotnet/desktop/wpf/data/)

### 추천 도서
- "WPF 4.5 Unleashed" - Adam Nathan
- "Advanced MVVM" - Josh Smith
- "Prism 가이드" - Microsoft Patterns & Practices

### 추천 영상/강좌
- [MVVM 패턴 소개 - Microsoft Channel 9](https://channel9.msdn.com/)
- [WPF MVVM Tutorial - AngelSix (YouTube)](https://www.youtube.com/c/AngelSix)
- [CommunityToolkit.Mvvm Getting Started](https://learn.microsoft.com/ko-kr/dotnet/communitytoolkit/mvvm/generators/overview)

### 관련 패턴
- **MVC**: 웹 애플리케이션의 표준 패턴
- **MVP**: Android 전통적 아키텍처
- **Clean Architecture**: 계층형 아키텍처와 결합 가능
- **Repository 패턴**: Model 레이어의 데이터 접근 추상화

---

> **다음 문서**: [Clean Architecture (클린 아키텍처)](./02-clean-architecture.md)
