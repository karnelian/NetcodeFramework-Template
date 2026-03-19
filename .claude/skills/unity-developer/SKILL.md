---
name: unity-developer
description: "Unity 엔진 실무 가이드. 애니메이션, 물리, 성능 최적화, 셰이더, AI/NavMesh, 일반 Unity 패턴. 아키텍처/네이밍/코딩 스타일/UI 계층은 CLAUDE.md 참조. 성능 프로파일링, 메모리 최적화, Draw Call 배칭, LOD, 오클루전 컬링, Update 최적화 등 심화 내용은 `references/performance-deep-dive.md` 참조."
---

# Unity 개발 가이드 (엔진 실무)

> **아키텍처 패턴, 네이밍 컨벤션, 코딩 스타일, UI 계층 규칙은 프로젝트 루트 `CLAUDE.md` 참조.**
> 이 스킬은 CLAUDE.md가 다루지 않는 Unity 엔진 실무 영역을 커버한다.

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/performance-deep-dive.md` | Profiler 코드 패턴, 메모리/GC 심화, Draw Call 배칭 코드, LOD 코드 설정, 오클루전 컬링, Physics 최적화, Update 분산, 비동기 로딩, 성능 체크리스트 | 성능 최적화 작업, 프로파일링, 프레임 드랍 디버깅 시 |
| `references/unity-patterns.md` | MonoBehaviour 베스트 프랙티스, ScriptableObject 데이터 패턴, Object Pooling, Event System, Coroutine 패턴, Singleton | 일반 Unity 코드 패턴 참조 시 |

---

## 애니메이션

### Animator Controller 구성
- State Machine은 계층적으로 구성 (Sub-State Machine 활용)
- 각 State 이름: `{동작}_{방향/변형}` (예: `Idle`, `Run_Forward`, `Attack_01`)
- Transition 조건: Bool 파라미터 우선, Trigger는 일회성 액션에만
- Any State → 특정 State 전환은 최소화 (디버깅 어려움)

### Animation Type 선택
| 타입 | 용도 |
|------|------|
| Humanoid | 캐릭터 (Mixamo/AI 리깅 호환, Retargeting 가능) |
| Generic | 몬스터, 프롭, 비인간형 |
| Legacy | 사용 금지 (구버전 호환용) |

### Blend Tree
- 1D: 속도 기반 (Idle → Walk → Run)
- 2D Freeform: 방향 + 속도 (이동 블렌딩)
- Direct: 표정/얼굴 블렌딩

### Animation Event
- 이벤트 함수는 Animator가 붙은 GameObject의 컴포넌트에 정의
- 무거운 로직 금지 → 이벤트에서 EventBus 발행만 하고, Manager에서 처리
- 이름 규칙: `OnAnim_{액션}` (예: `OnAnim_FootStep`, `OnAnim_AttackHit`)

### Root Motion
- Apply Root Motion: 자연스러운 이동이 필요할 때만 (전투 대시, 회피)
- 일반 이동: 코드 기반 이동 + 애니메이션 블렌딩 (Root Motion OFF)

---

## 물리

### Rigidbody 설정
| 설정 | 권장값 | 설명 |
|------|--------|------|
| Interpolate | Interpolate | 시각적 떨림 방지 |
| Collision Detection | Continuous (빠른 물체) / Discrete (일반) | 관통 방지 |
| Constraints | 필요한 축만 해제 | 불필요한 회전/이동 잠금 |

### Collider 선택
| 타입 | 용도 | 비용 |
|------|------|------|
| Box | 상자형 오브젝트, 벽, 바닥 | 낮음 |
| Sphere | 원형 감지, 근접 범위 | 낮음 |
| Capsule | 캐릭터, 적 | 낮음 |
| Mesh | 복잡한 지형 (Convex OFF) | 높음 — 최소화 |

### Physics Layer Matrix
- `Project Settings > Physics > Layer Collision Matrix`에서 불필요한 레이어 간 충돌 비활성화
- 레이어 구성 예: `Player`, `Enemy`, `Projectile`, `Environment`, `Trigger`, `IgnoreRaycast`
- 같은 팀 충돌 비활성화: Player↔Player OFF, Enemy↔Enemy OFF

### Update vs FixedUpdate
| 로직 | 사용할 곳 |
|------|-----------| 
| 물리 이동 (Rigidbody.velocity, AddForce) | FixedUpdate |
| 입력 읽기 (Input System) | Update |
| Raycast | Update 또는 FixedUpdate (용도에 따라) |
| 애니메이션 파라미터 갱신 | Update |

### Raycast 패턴
```csharp
// 기본 패턴: LayerMask + maxDistance 필수
if (Physics.Raycast(origin, direction, out var hit, maxDistance, layerMask))
{
    // hit.collider, hit.point, hit.normal 사용
}

// 여러 대상: NonAlloc으로 GC 방지
private readonly RaycastHit[] _hits = new RaycastHit[16];
int count = Physics.RaycastNonAlloc(origin, direction, _hits, maxDistance, layerMask);
```

### Trigger vs Collision
- **Trigger** (Is Trigger = true): 감지 영역 (아이템 수집, 영역 진입)
- **Collision** (Is Trigger = false): 물리적 충돌 (벽, 바닥, 적 피격)
- OnTriggerEnter/Stay/Exit vs OnCollisionEnter/Stay/Exit

---

## 성능 최적화

> **심화 내용 (코드 패턴, Profiler 활용, 체크리스트)은 `references/performance-deep-dive.md` 참조.**

### 배칭
- **Static Batching**: 움직이지 않는 오브젝트 → Inspector에서 Static 체크
- **Dynamic Batching**: 작은 메시 자동 배칭 (300 vertices 이하)
- **SRP Batcher**: URP에서 자동 활성화 — 같은 셰이더 변형끼리 배칭
- **GPU Instancing**: 같은 메시+머티리얼 대량 렌더링 (풀, 나무, 총알)

### LOD (Level of Detail)
- LOD Group 컴포넌트: 거리별 메시 교체
- LOD 0: 원본 (가까운), LOD 1: 50% 폴리곤, LOD 2: 25%, Culled: 비렌더링
- AI 생성 모델은 폴리곤이 높으므로 LOD 필수 (`ai-3d-pipeline/SKILL.md` 참조)

### 오클루전 컬링
- `Window > Rendering > Occlusion Culling`에서 베이킹
- Static Occluder/Occludee 설정 (큰 벽 = Occluder, 작은 오브젝트 = Occludee)

### GC 최소화
```csharp
// 나쁜 예 — 매 프레임 GC Alloc
foreach (var item in collection) { }  // IEnumerator 박싱
string status = "HP: " + hp;          // 매번 string 할당

// 좋은 예
for (int i = 0; i < list.Count; i++) { }  // 인덱서 사용
_sb.Clear(); _sb.Append("HP: "); _sb.Append(hp); // StringBuilder 재사용
```

### Profiler 활용
- `Window > Analysis > Profiler`
- CPU: 프레임별 메서드 실행 시간 확인
- Memory: GC Alloc 추적 (매 프레임 0이 이상적)
- GPU: Draw Call, 배칭 효율 확인
- Coplay MCP: `get_worst_cpu_frames`, `get_worst_gc_frames` 도구 활용

### 성능 목표 (60 FPS = 16.67ms)

| 영역 | 프레임 예산 |
|------|-------------|
| Game Logic | 5–7ms |
| Rendering | 3–5ms |
| Physics | 2–3ms |
| Scripts | 2–3ms |

### Update 최적화 전략
- **Staggered Update**: 비싼 로직을 N프레임마다 분산 실행
- **Distance-based Update**: 플레이어와의 거리에 따라 업데이트 빈도 조절 (가까운: 20fps, 먼: 2fps)
- **Raycast 간격 제한**: 매 프레임 대신 0.1초 간격으로 실행

---

## 셰이더 / 머티리얼

### URP 셰이더 선택
| 셰이더 | 용도 |
|--------|------|
| URP/Lit | 일반 오브젝트 (PBR: Albedo, Normal, Metallic) |
| URP/Simple Lit | 가벼운 오브젝트 (Blinn-Phong, 모바일) |
| URP/Unlit | UI, 이펙트, 자체 발광 |
| Shader Graph | 커스텀 효과 (물, 용암, 디졸브, 홀로그램) |

### Material Property Block
같은 머티리얼에서 인스턴스별 프로퍼티 변경 시 사용 (배칭 유지):
```csharp
private static readonly int ColorID = Shader.PropertyToID("_BaseColor");
private MaterialPropertyBlock _mpb;

void SetColor(Color color)
{
    _mpb ??= new MaterialPropertyBlock();
    _mpb.SetColor(ColorID, color);
    renderer.SetPropertyBlock(_mpb);
}
```

### Shader Graph 패턴
- 노드 그래프로 셰이더 제작 (코드 없이)
- 자주 쓰는 패턴: Dissolve (Step + Noise), Outline (Fresnel), Tint (Color Multiply)
- Sub Graph로 재사용 가능한 노드 그룹 추출

### 머티리얼 최적화
- `renderer.material` 사용 금지 (인스턴스 복제 → 배칭 파괴)
- `renderer.sharedMaterial`로 공유, 인스턴스별 차이는 MaterialPropertyBlock으로
- 텍스처 아틀라스: 여러 텍스처를 하나로 합쳐 Draw Call 절감

---

## AI / Navigation

### NavMesh 설정
- `Window > AI > Navigation`에서 베이킹
- Agent Radius/Height: 캐릭터 크기에 맞춤
- Step Height: 계단 높이
- Max Slope: 오를 수 있는 최대 경사

### NavMeshAgent 패턴
```csharp
// 이동
agent.SetDestination(targetPosition);

// 도착 체크
bool arrived = !agent.pathPending && agent.remainingDistance <= agent.stoppingDistance;

// 속도 동기화 (애니메이션)
float speed = agent.velocity.magnitude;
animator.SetFloat("Speed", speed);
```

### NavMesh Obstacle
- 동적 장애물: NavMesh Obstacle (Carve = true)
- 문, 파괴 가능 벽 등에 사용
- Carve는 비용이 높으므로 이동하는 오브젝트에는 OFF

---

## Coroutine vs R3 Observable

| 상황 | 사용할 것 |
|------|-----------| 
| 프레임워크 시스템 (Manager/Service) | R3 Observable (CLAUDE.md 패턴 따름) |
| 게임플레이 타이머, 딜레이 | R3 Observable.Timer / Observable.Interval |
| MonoBehaviour 내부 간단한 시퀀스 | Coroutine 허용 (단, 3줄 이하) |
| 네트워크 비동기 | Observable.FromAsync (CLAUDE.md 규칙) |

Coroutine을 쓸 때:
- `yield return null` (매 프레임), `yield return new WaitForSeconds(t)`
- WaitForSeconds는 캐싱하여 재사용 (GC 방지)
- `StopCoroutine` 반드시 호출 (메모리 누수 방지)
- 복잡한 상태 머신은 Coroutine 대신 R3 또는 State 패턴 사용

---

## 일반 Unity 패턴 요약

> **상세 코드는 `references/unity-patterns.md` 참조.**

### 핵심 규칙
- `GetComponent<T>()` → Awake/Start에서 캐싱, Update에서 호출 금지
- `CompareTag()` 사용 (`tag == "TagName"` 비교 금지)
- `Camera.main` → Start에서 캐싱
- `Destroy` 대신 Object Pooling (프레임워크: `PoolManager.instance.Spawn/Despawn`)
- `Instantiate` 루프 금지 → 풀링 또는 사전 생성

### ScriptableObject 활용
- 게임 데이터(무기, 적, 스킬 등)는 SO에 정의
- `[CreateAssetMenu]` 어트리뷰트로 에디터 메뉴 등록
- SO에 메서드 포함 가능 (데미지 계산 등 순수 함수)
- 프레임워크 연동: `unity-so-builder` 스킬로 에디터 스크립트 자동화

---

## 비동기 씬 로딩

```csharp
// SceneSystem을 통해 로딩 (프레임워크 패턴)
// 직접 SceneManager를 쓸 경우:
Observable.FromAsync(async ct =>
{
    var op = SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Single);
    op.allowSceneActivation = false;

    while (op.progress < 0.9f)
    {
        // 로딩 UI 업데이트
        await Task.Yield();
    }

    op.allowSceneActivation = true;
    return Unit.Default;
});
```
