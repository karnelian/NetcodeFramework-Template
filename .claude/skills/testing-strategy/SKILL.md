---
name: testing-strategy
description: "NetcodeFramework 테스트 규칙. Edit Mode NUnit, R3 오버로드 회피, async 교착 방지, MonoBehaviour 정리."
---

# 테스트 전략

---

## 테스트 환경

- **프레임워크**: NUnit (Edit Mode only)
- **위치**: `Assets/NetcodeFramework/Tests/`
- **현재 테스트**: 112개 (10개 시스템)
- **asmdef 금지** → Assembly-CSharp 단일 어셈블리

### 테스트 커버리지 (10개 시스템)

| 시스템 | 테스트 수 |
|--------|----------|
| PoolSystem | 10 |
| UISystem | 13 |
| SaveSystem | 9 |
| EventSystem | 7 |
| TimeScaleSystem | 10 |
| OptionSystem | 7 |
| ResourceLoadSystem | 12 |
| DebugSystem | 14 |
| SceneSystem | 14 |
| NotificationSystem | 16 |

---

## 프레임워크 고유 패턴

### R3 Subscribe 오버로드 충돌
R3의 `Subscribe()` 메서드 오버로드가 NUnit 컨텍스트에서 모호해짐.
**반드시 명시적 호출**:
```csharp
// 나쁜 예 — 오버로드 충돌
observable.Subscribe(v => { });

// 좋은 예 — 명시적 호출
ObservableSubscribeExtensions.Subscribe(observable, v => { });
```

### async 교착 방지
Unity 메인 스레드에서 `.GetAwaiter().GetResult()` 직접 호출 시 교착.
**반드시 Task.Run 래핑**:
```csharp
// 나쁜 예 — 교착
var result = asyncMethod().GetAwaiter().GetResult();

// 좋은 예 — Task.Run 래핑
var result = Task.Run(() => asyncMethod()).GetAwaiter().GetResult();
```

### MonoBehaviour 정리
Edit Mode에서 `Object.Destroy()` 동작 안 함.
**반드시 DestroyImmediate 사용**:
```csharp
private readonly List<GameObject> _createdObjects = new();

[TearDown]
public void TearDown()
{
    foreach (var obj in _createdObjects)
        Object.DestroyImmediate(obj);
    _createdObjects.Clear();
}
```

---

## 테스트 작성 패턴

### AAA (Arrange-Act-Assert)
```csharp
[Test]
public void Health_WhenDamageExceedsMax_ClampsToZero()
{
    // Arrange
    var character = new Character(health: 100);

    // Act
    character.TakeDamage(150);

    // Assert
    Assert.AreEqual(0, character.Health);
}
```

### 네이밍 컨벤션
`[대상]_[조건]_[기대결과]`
- `Health_WhenDamageExceedsMax_ClampsToZero`
- `Pool_WhenDespawned_ReturnsToPool`
- `EventBus_WhenPublished_NotifiesSubscribers`

---

## 주의사항

- 배치 실행 시 격리 문제 있음 → 개별 실행 권장
- PlayMode 테스트 미사용 (Edit Mode만)
- 테스트 대상 시스템의 Settings SO가 null일 경우 기본값 생성 로직 테스트 포함
