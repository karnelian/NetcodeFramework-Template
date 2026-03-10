---
name: framework-patterns
description: "NetcodeFramework 코드 패턴. R3 Observable, MVVM, Handle/Using, Extension Methods, 조건부 컴파일."
---

# NetcodeFramework 코드 패턴

> **네이밍 컨벤션, 코딩 스타일은 CLAUDE.md 참조.**
> **시스템 아키텍처는 framework-architecture 스킬 참조.**

---

## R3 Observable 규칙

| 상황 | 사용할 것 |
|------|-----------|
| 실제 async/await 필요 | `Observable.FromAsync(async ct => { ... })` |
| 동기 결과 반환 | `Observable.Return(value)` |
| 동기 에러 | `Observable.Throw<T>(exception)` |
| 상태 저장 | `ReactiveProperty<T>` (내부), `ReadOnlyReactiveProperty<T>` (공개) |
| 일회성 이벤트 | `Subject<T>` (내부), `Observable<T>` (공개) |
| 구독 해제 | 반드시 `.AddTo(disposable)` |
| 핫 Observable | `.Share()` (여러 구독자 공유) |
| 병렬 대기 | `Observable.CombineLatest(...)` |
| 순차 체이닝 | `Observable.SelectMany(...)` |

```csharp
// 내부 상태
private readonly ReactiveProperty<string> _status = new(string.Empty);
// 외부 노출
public ReadOnlyReactiveProperty<string> Status => _status;

// 구독 (반드시 AddTo)
_service.OnEvent
    .Subscribe(v => HandleEvent(v))
    .AddTo(_disposables);
```

### Dispose 패턴
```csharp
private CompositeDisposable _disposables = new();

// 구독 시
subscription.AddTo(_disposables);

// 정리 시 (OnDestroy 또는 Dispose)
_disposables?.Dispose();
```

---

## MVVM 패턴

### ViewModel
```csharp
public class XxxViewModel : GameSessionBaseViewModel
{
    private readonly ReactiveProperty<string> _message = new("");
    public ReadOnlyReactiveProperty<string> Message => _message;

    // 서비스 접근 (Base에서 제공)
    // protected GameSessionManager GameSessionManager => ...
    // protected IAuthenticationService AuthService => ...
    // protected ISessionService SessionService => ...
    // protected IRelayService RelayService => ...
    // protected ISceneLoader SceneLoader => ...

    public override void Initialize()
    {
        base.Initialize();
        // R3 바인딩, .AddTo(Disposables)
    }
}
```

### View
```csharp
public class XxxView : GameSessionBaseView<XxxViewModel>
{
    [SerializeField] private TextMeshProUGUI messageText;

    protected override void OnViewModelBound()
    {
        ViewModel.Message
            .Subscribe(msg => messageText.text = msg)
            .AddTo(Disposables);
    }
}
```

View의 `Awake()`에서 자동으로 ViewModel 생성 + `Initialize()` + `OnViewModelBound()` 호출.

---

## Handle / Using 패턴

프레임워크의 핵심 리소스 관리 패턴. IDisposable 기반 자동 정리.

### PoolHandle
```csharp
// Scope 기반 자동 반환
using var handle = PoolManager.instance.SpawnWithHandle("HitVFX", prefab, pos, rot);
handle.GameObject.GetComponent<ParticleSystem>().Play();
// 블록 종료 → 자동 Despawn

// 수동 사용
var obj = PoolManager.instance.Spawn("HitVFX", prefab, pos, rot);
// ... 사용 후
PoolManager.instance.Despawn("HitVFX", obj);
```

### TimeScaleHandle
```csharp
using var handle = TimeScaleManager.instance.SetScale(0.3f, (int)TimeLayer.Gameplay);
// Gameplay 레이어 0.3x 속도
// 블록 종료 → 자동 리셋

// TimeLayerTarget 컴포넌트: GameObject에 부착하면 해당 레이어 타임스케일 자동 반영
```

---

## 조건부 컴파일

UGS 패키지 의존성은 반드시 `#if` 래핑:

```csharp
#if UNITY_SERVICES_AUTHENTICATION
using Unity.Services.Authentication;
#endif

public Observable<Unit> SignInAsObservable()
{
#if UNITY_SERVICES_AUTHENTICATION
    return Observable.FromAsync(async ct => {
        await AuthenticationService.Instance.SignInAnonymouslyAsync();
        return Unit.Default;
    });
#else
    Debug.LogWarning("[Service] Package not installed.");
    return Observable.Return(Unit.Default);
#endif
}
```

주요 심볼: `UNITY_SERVICES_AUTHENTICATION`, `UNITY_SERVICES_MULTIPLAYER`, `UNITY_INPUT_SYSTEM`, `UNITY_CINEMACHINE`

---

## 확장 메서드 (Extension Methods)

`NetcodeFramework.Extensions` 네임스페이스 (8개 클래스, 46개 메서드).
코드 작성 시 반드시 기존 확장 메서드를 우선 활용할 것.
모든 메서드 `[AggressiveInlining]` 적용 → 성능 오버헤드 없음.

```csharp
using NetcodeFramework.Extensions;
```

### 필수 활용 패턴

| 기존 패턴 (금지) | 확장 메서드 (사용) |
|-------------------|---------------------|
| `GetComponent<T>()` + null 체크 + `AddComponent<T>()` | `gameObject.GetOrAddComponent<T>()` |
| `foreach (Transform child in t) { Destroy(child.gameObject); }` | `transform.DestroyAllChildren()` |
| `new Color(c.r, c.g, c.b, alpha)` | `color.WithAlpha(alpha)` |
| `collection == null \|\| collection.Count == 0` | `collection.IsNullOrEmpty()` |

### 전체 목록

**GameObjectExtensions** — 컴포넌트 관리
- `GetOrAddComponent<T>()` (GameObject/Component), `HasComponent<T>()`, `SetLayerRecursively(int)`

**TransformExtensions** — 트랜스폼 조작
- `ResetLocal()`, `SetPositionX/Y/Z()`, `SetLocalPositionX/Y/Z()`
- `SetLocalScaleUniform(float)`, `GetChildren()`, `DestroyAllChildren()`, `LookAt2D(Vector3)`

**VectorExtensions** — 벡터 유틸리티
- `WithX/Y/Z(float)` (Vector3/Vector2)
- `Flat()`, `DirectionTo()`, `DistanceTo()`, `IsWithinDistance()`, `ToVector3XZ()`

**CollectionExtensions** — 컬렉션 유틸리티
- `IsNullOrEmpty<T>()` (ICollection/T[])
- `GetOrDefault<T>()` (IList/IDictionary), `Shuffle()`, `RandomElement()`, `ForEach()`

**ColorExtensions** — 색상 유틸리티
- `WithAlpha(float)`, `ToHexString()`, `TryParseHex()`

**StringExtensions** — 문자열 변환
- `Truncate()`, `ToSnakeCase()`, `ToCamelCase()`

**NumberExtensions** — 수치 유틸리티
- `Remap()`, `IsBetween()` (float/int), `Approximately()`

**RectTransformExtensions** — UI 레이아웃
- `SetAnchorStretchAll()`, `SetAnchorCenter()`, `SetSize()`, `SetWidth()`, `SetHeight()`
