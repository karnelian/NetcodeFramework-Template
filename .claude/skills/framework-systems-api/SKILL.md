---
name: framework-systems-api
description: "NetcodeFramework 26개 시스템 API 레퍼런스. EventBus, Command, Pool, TimeScale, Scene, UI, Input, Camera 등."
---

# NetcodeFramework 시스템 API 레퍼런스

> 모든 시스템은 Manager(MonoSingleton) + Service(Interface) 구조.
> 아키텍처 패턴 상세는 `framework-architecture` 스킬 참조.

---

## EventSystem — Pub-Sub 이벤트 버스

```csharp
// 이벤트 정의
public struct EnemyKilledEvent : IEvent { public string EnemyId; }

// 발행
EventBus.instance.Publish(new EnemyKilledEvent { EnemyId = "goblin_01" });

// 구독 (반드시 .AddTo())
EventBus.instance.Subscribe<EnemyKilledEvent>()
    .Subscribe(e => Debug.Log($"Enemy killed: {e.EnemyId}"))
    .AddTo(Disposables);
```

연동 시스템: AchievementSystem(달성 이벤트), NotificationSystem(알림), AnalyticsSystem(텔레메트리)

---

## CommandSystem — Undo/Redo

```csharp
// ICommand 인터페이스
public interface ICommand
{
    void Execute();
    void Undo();
    bool CanExecute();
    bool IsUndoable { get; }  // false면 히스토리 미기록
}

// LambdaCommand (간단한 명령)
CommandManager.instance.ExecuteAndRecord(
    new LambdaCommand(
        execute: () => player.Move(newPos),
        undo: () => player.Move(oldPos)
    )
);

// Execute만 (히스토리 없이)
CommandManager.instance.Execute(command);

// Undo / Redo
CommandManager.instance.Undo();
CommandManager.instance.Redo();

// 상태 관찰
CommandManager.instance.UndoCount;  // ReadOnlyReactiveProperty<int>
CommandManager.instance.RedoCount;
CommandManager.instance.OnCommandExecuted;  // Observable<ICommand>
```

---

## PoolSystem — 오브젝트 풀링

```csharp
// Handle 패턴 (자동 반환)
using var handle = PoolManager.instance.SpawnWithHandle("HitVFX", prefab, pos, rot);
handle.GameObject.GetComponent<ParticleSystem>().Play();

// 수동 패턴
var obj = PoolManager.instance.Spawn("HitVFX", prefab, pos, rot);
PoolManager.instance.Despawn("HitVFX", obj);
```

연동: AudioSystem(SFX 풀링), ResourceLoadSystem(Addressable + 풀링)

---

## TimeScaleSystem — 레이어 기반 시간 제어

```csharp
// Handle 패턴 (자동 복원)
using var handle = TimeScaleManager.instance.SetScale(0.3f, (int)TimeLayer.Gameplay);

// 수동 리셋
TimeScaleManager.instance.ResetLayer((int)TimeLayer.Gameplay);

// TimeLayerTarget 컴포넌트: GameObject에 부착 → 해당 레이어 타임스케일 자동 반영
```

---

## SceneSystem — Strategy 패턴 씬 전환

```csharp
// 씬 로드
SceneManager.instance.LoadSceneAsObservable("StageName", sceneData)
    .Subscribe(_ => Debug.Log("씬 로드 완료"))
    .AddTo(_disposables);

// 상태 관찰
SceneManager.instance.CurrentScene;     // ReadOnlyReactiveProperty<string>
SceneManager.instance.IsLoading;        // ReadOnlyReactiveProperty<bool>
SceneManager.instance.LoadingProgress;  // ReadOnlyReactiveProperty<float>
```

Strategy: Initialize 시 `SingleplayerTransitionStrategy` 또는 `MultiplayerTransitionStrategy` 자동 선택.
상세 씬 플로우는 `scene-structure` 스킬 참조.

---

## UISystem — 팝업 스택 + 토스트

```csharp
// 팝업 Push/Pop (PopupStackManager)
UIManager.instance.PopupStack.Push(popup);
UIManager.instance.PopupStack.Pop();

// 상태 관찰
UIManager.instance.PopupStack.PopupCount;   // ReadOnlyReactiveProperty<int>
UIManager.instance.PopupStack.OnPopupOpened; // Observable<IPopup>
UIManager.instance.PopupStack.OnPopupClosed; // Observable<IPopup>

// Toast
UIManager.instance.ShowToast("메시지", ToastType.Success);
```

UI Prefab 네이밍/계층 규칙은 `hierarchy-rules` 스킬 참조.

---

## AudioSystem — BGM/SFX

```csharp
// BGM
AudioManager.instance.PlayBGM(clip);
AudioManager.instance.StopBGM();

// SFX (풀링 기반)
AudioManager.instance.PlaySFX(clip);

// 볼륨 (OptionSystem 연동)
// AudioMixer 그룹: Master, BGM, SFX

// HasInstance 체크 필수
if (AudioManager.HasInstance) AudioManager.instance.PlaySFX(clip);
```

사운드 리소스 전략은 `audio-strategy` 스킬 참조.

---

## InputSystem — Unity Input System 래핑

```csharp
// Action Map 전환
InputManager.instance.SwitchActionMap("UI");

// 상태 관찰
InputManager.instance.CurrentActionMap;   // ReadOnlyReactiveProperty<string>
InputManager.instance.CurrentDevice;      // ReadOnlyReactiveProperty<InputDeviceType>
InputManager.instance.IsRebinding;        // ReadOnlyReactiveProperty<bool>

// 이벤트
InputManager.instance.OnActionMapChanged;   // Observable<string>
InputManager.instance.OnDeviceChanged;      // Observable<InputDeviceType>
InputManager.instance.OnRebindCompleted;    // Observable<string>
InputManager.instance.OnConflictDetected;   // Observable<ConflictInfo>
```

조건부 컴파일: `#if UNITY_INPUT_SYSTEM`

---

## CameraSystem — Cinemachine 3 래핑

```csharp
CameraManager.instance.SwitchPreset("Combat");
CameraManager.instance.BlendTo("Exploration", 1.5f);
CameraManager.instance.ShakeCamera(intensity: 0.5f, duration: 0.3f);
CameraManager.instance.ZoomCamera(targetFOV: 45f, duration: 1f);
CameraManager.instance.SetFollowTarget(playerTransform);
CameraManager.instance.SetLookAtTarget(bossTransform);

// 상태
CameraManager.instance.CurrentPreset;     // ReadOnlyReactiveProperty<string>
CameraManager.instance.IsTransitioning;   // ReadOnlyReactiveProperty<bool>
```

조건부 컴파일: `#if UNITY_CINEMACHINE`

---

## AuthenticationSystem

```csharp
var authService = AuthenticationManager.instance.AuthenticationService;

// 비동기 액션
authService.SignInAsyncAsObservable()
    .Subscribe(_ => Debug.Log("로그인 성공"))
    .AddTo(_disposables);

// 상태
authService.IsSignedIn;   // ReadOnlyReactiveProperty<bool>
authService.PlayerId;     // ReadOnlyReactiveProperty<string>
authService.Username;     // ReadOnlyReactiveProperty<string>

// 이벤트
authService.OnSignInSuccess;  // Observable<Unit>
authService.OnSignInFailed;   // Observable<string>
```

---

## ResourceLoadSystem — Addressables 래핑

```csharp
// 오브젝트 생성 (풀링 자동 적용)
ResourceLoadManager.instance.CreateObject(address, parent)
    .Subscribe(obj => { /* 사용 */ })
    .AddTo(_disposables);

// 이펙트 생성 (자동 반환)
ResourceLoadManager.instance.CreateEffect(address, position);
```

---

## 기타 시스템 요약

| 시스템 | 핵심 API |
|--------|---------|
| **LobbySystem** | CreateLobby, JoinLobby, LeaveLobby (ISessionService) |
| **RelaySystem** | CreateRelay, JoinRelay (IRelayService) |
| **ChatSystem** | SendMessage, OnMessageReceived (UGS only) |
| **SaveSystem** | SaveAsObservable, LoadAsObservable (Cloud/Local) |
| **RemoteConfigSystem** | GetConfig, OnConfigUpdated |
| **LeaderboardSystem** | SubmitScore, GetScores |
| **AchievementSystem** | Unlock, CheckProgress (Toast 자동 연동) |
| **EconomySystem** | GetBalance, Purchase, AddCurrency |
| **Language** | SetLocale, GetLocalizedString |
| **NotificationSystem** | AddNotification, History, EventBus 연동 |
| **OptionSystem** | Audio/Graphics/Input 설정 (PlayerPrefs) |
| **AnalyticsSystem** | TrackEvent, 배치 전송 (UGS/Steam/Local) |
| **DebugSystem** | [DebugCommand] 어트리뷰트, 인게임 콘솔 |
