---
name: netcode-framework
description: "NetcodeFramework 통합 가이드. 아키텍처(Manager/Service/Init Chain), 코드 패턴(R3 Observable, MVVM, Handle/Using, Extensions), 26개 시스템 API. 프레임워크 코드 작성, 새 시스템 추가, Manager/Service 구현, R3 구독, UI MVVM, 씬 전환, 이벤트 발행, 풀링, 에셋 로딩 등 NetcodeFramework 관련 모든 작업에 사용. 네이밍/코딩 스타일은 CLAUDE.md 참조."
---

# NetcodeFramework

> **네이밍 컨벤션, 코딩 스타일, 금지 사항은 프로젝트 루트 `CLAUDE.md` 참조.**

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/code-patterns.md` | R3 Observable 규칙, MVVM 패턴, Handle/Using 패턴, 조건부 컴파일, Extension Methods 전체 목록 | R3 구독, ViewModel/View 작성, Handle 사용, 확장 메서드 활용 시 |
| `references/systems-api.md` | 26개 시스템별 핵심 API (EventBus, Command, Pool, TimeScale, Scene, UI, Audio, Input, Camera, Auth, ResourceLoad 등) | 시스템 API 호출, 시스템 간 연동 코드 작성 시 |

---

## 프로젝트 구조

```
Assets/
├── NetcodeFramework/              ← 서브모듈 (karnelian/NetcodeFramework-Core)
│   ├── Shared/                    ← MonoSingleton, PlatformManager, Extensions
│   ├── Core/                      ← GameSessionManager, ServiceType, UI/ (MVVM)
│   ├── Plugins/                   ← 26개 시스템
│   ├── Editor/                    ← DependencyInstaller (자동 패키지 설치)
│   ├── Addressables/              ← Manager 프리팹 + Settings SO
│   ├── Scenes/                    ← Bootstrap, Mock, UGS, Steam
│   ├── Tests/                     ← NUnit Edit Mode 테스트 (112개)
│   └── Fonts/
└── _Game/                         ← 게임별 코드 (여기에 작성)
    └── ExampleGame/
```

### 시스템 폴더 구조 (Plugins 내부)
```
Plugins/{SystemName}System/
├── Core/
│   ├── {SystemName}Manager.cs
│   ├── I{SystemName}Service.cs
│   ├── {SystemName}Settings.cs
│   └── (Data 클래스)
├── UGS/
│   └── UGS{SystemName}Service.cs
├── Steam/
│   └── Steam{SystemName}Service.cs
└── Local/
    └── Local{SystemName}Service.cs
```

---

## 초기화 체인 (GameSessionManager)

```
GameSessionManager.Start()
  ↓
PlatformManager.SelectedPlatform → selectedSessionType
  ↓
InitializeSessionAsync(GameSessionType)
  ↓
InitializePluginsAsync()
  ├── ServiceType를 Priority 순으로 정렬
  ├── 같은 Priority: Observable.CombineLatest (병렬 초기화)
  ├── 다른 Priority: Observable.SelectMany (순차 체이닝)
  └── 각 ServiceType → CreateManagerByServiceType()
      ├── Addressables로 프리팹 로드
      ├── Instantiate (GameSessionManager.transform 하위)
      └── IManagerInitializable.Initialize() 호출
```

### 초기화 우선순위

| Priority | 시스템 |
|----------|--------|
| 1 | ResourceLoadManager (에셋 로딩 기반) |
| 2 | SceneManager, UIManager, AudioManager, OptionSystem, InputManager, CameraManager, NotificationManager |
| 3 | AuthenticationManager, SaveManager, RelayManager, LobbyManager, ChatManager, RemoteConfigManager, LeaderboardManager, AchievementManager, EconomyManager |
| 4 | AnalyticsManager (텔레메트리 — 마지막) |

---

## Manager 패턴

모든 Manager는 `MonoSingleton<T>` + `IManagerInitializable` 상속.

```csharp
public class XxxManager : MonoSingleton<XxxManager>, IManagerInitializable
{
    [SerializeField] private XxxSettings settings;
    private IXxxService _service;
    private CompositeDisposable _disposables = new();
    private bool _isInitialized;

    public void Initialize()
    {
        if (_isInitialized) return;

        if (settings == null)
        {
            Debug.LogError("[XxxManager] Settings null, creating default.");
            settings = ScriptableObject.CreateInstance<XxxSettings>();
        }

        var sessionType = FindSessionType();
        _service = sessionType switch
        {
            GameSessionType.UGS   => new UGSXxxService(settings),
            GameSessionType.Steam => new SteamXxxService(settings),
            GameSessionType.Local => new LocalXxxService(settings),
            _                     => new LocalXxxService(settings)
        };

        _isInitialized = true;
    }

    private GameSessionType FindSessionType()
    {
        var allBehaviours = Object.FindObjectsByType<MonoBehaviour>(FindObjectsSortMode.None);
        foreach (var behaviour in allBehaviours)
        {
            if (behaviour is IGameSessionTypeProvider provider)
                return provider.CurrentSessionType;
        }
        return GameSessionType.Local;
    }

    protected override void OnOnDestroy()
    {
        base.OnOnDestroy();
        _disposables?.Dispose();
    }
}
```

## Service Interface 패턴

```csharp
public interface IXxxService
{
    Observable<Unit> OnActionCompleted { get; }              // 이벤트
    ReadOnlyReactiveProperty<bool> IsActive { get; }         // 상태
    Observable<ResultType> DoActionAsObservable(params);      // 비동기 메서드
}
```

---

## Platform Strategy

```
PlatformManager (Shared/PlatformManager.cs)
  └── ReactiveProperty<GameSessionType> _selectedPlatform
  └── SelectPlatform(type, remember) → PlayerPrefs 저장

GameSessionType enum: { Steam, UGS, Mock, Local }
  - UGS: Unity Gaming Services (온라인 멀티플레이어)
  - Steam: Facepunch.Steamworks
  - Local: 오프라인 싱글플레이어
  - Mock: 개발/테스트용
```

---

## ServiceType Registry

`Core/ServiceType.cs` — 19개 enum 값
`Core/GameSessionUtils.cs` — 3개 핵심 메서드:
- `GetManagerInfoByServiceType(ServiceType)` → `(managerName, addressableAddress)`
- `GetPriorityByServiceType(ServiceType)` → `int` (1~4)
- `CreateManagerByServiceType(ServiceType, parent)` → `Observable<object>`

## Addressable 경로 규칙

```
Assets/NetcodeFramework/Addressables/Manager/{ManagerName}/{ManagerName}.prefab
Assets/NetcodeFramework/Addressables/Manager/{ManagerName}/{ManagerName}Settings.asset
```

---

## 시스템 간 의존 관계

```
PlatformManager → GameSessionManager → 모든 Manager.Initialize()

SaveSystem → LocalLeaderboard, LocalAchievement, LocalEconomy, LocalAnalytics
EventSystem (EventBus) → AchievementSystem, NotificationSystem, AnalyticsSystem
PoolSystem → AudioSystem (SFX 풀링), 게임 오브젝트 풀링
UISystem → NotificationSystem (Toast), AchievementSystem (달성 Toast)
TimeScaleSystem → TimeLayerTarget 컴포넌트 (자동 스케일 반영)
InputSystem → OptionSystem (InputSettings/InputRebinding 위임)
RemoteConfigSystem → OptionSystem (서버 설정 반영)
```

---

## 새 시스템 추가 체크리스트

1. `Plugins/{SystemName}System/Core/` → Manager + Interface + Settings
2. `Plugins/{SystemName}System/UGS/` → UGS 구현체
3. `Plugins/{SystemName}System/Steam/` → Steam 구현체
4. `Plugins/{SystemName}System/Local/` → Local 구현체
5. `Core/ServiceType.cs` → enum 값 추가
6. `Core/GameSessionUtils.cs` → 3개 switch 케이스 추가 (Info, Priority, Create)
7. `Addressables/Manager/{ManagerName}/` → 프리팹 + Settings SO
8. GameSessionManager 서비스 목록에 추가

---

## 핵심 코드 패턴 요약 (상세: references/code-patterns.md)

### R3 Observable 핵심 규칙
| 상황 | 사용할 것 |
|------|-----------|
| async/await 필요 | `Observable.FromAsync(async ct => { ... })` |
| 동기 결과 반환 | `Observable.Return(value)` |
| 상태 저장 | `ReactiveProperty<T>` (내부), `ReadOnlyReactiveProperty<T>` (공개) |
| 구독 해제 | **반드시** `.AddTo(disposable)` |
| 병렬 대기 | `Observable.CombineLatest(...)` |
| 순차 체이닝 | `Observable.SelectMany(...)` |

### 확장 메서드 필수 활용
| 금지 패턴 | 사용할 것 |
|-----------|-----------|
| `GetComponent<T>()` + null 체크 + `AddComponent<T>()` | `GetOrAddComponent<T>()` |
| `foreach (Transform child in t) { Destroy(...); }` | `transform.DestroyAllChildren()` |
| `new Color(c.r, c.g, c.b, alpha)` | `color.WithAlpha(alpha)` |
| `collection == null \|\| collection.Count == 0` | `collection.IsNullOrEmpty()` |

---

## 시스템 API 요약 (상세: references/systems-api.md)

| 시스템 | 핵심 API |
|--------|---------|
| **EventSystem** | `EventBus.instance.Publish(event)` / `Subscribe<T>()` |
| **CommandSystem** | `CommandManager.instance.ExecuteAndRecord(cmd)` / `Undo()` / `Redo()` |
| **PoolSystem** | `PoolManager.instance.Spawn/Despawn` / `SpawnWithHandle` |
| **TimeScaleSystem** | `TimeScaleManager.instance.SetScale(scale, layer)` |
| **SceneSystem** | `SceneManager.instance.LoadSceneAsObservable(name, data)` |
| **UISystem** | `UIManager.instance.PopupStack.Push/Pop` / `ShowToast()` |
| **AudioSystem** | `AudioManager.instance.PlayBGM/PlaySFX` |
| **InputSystem** | `InputManager.instance.SwitchActionMap()` |
| **CameraSystem** | `CameraManager.instance.SwitchPreset/ShakeCamera/ZoomCamera` |
| **ResourceLoad** | `ResourceLoadManager.instance.CreateObject(address, parent)` |
| **AuthSystem** | `AuthService.SignInAsyncAsObservable()` |
| **SaveSystem** | `SaveAsObservable` / `LoadAsObservable` |
