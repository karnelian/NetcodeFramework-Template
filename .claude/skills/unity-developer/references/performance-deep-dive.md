# 성능 최적화 심화 가이드

> 기본 개념은 `SKILL.md`의 성능 최적화 섹션 참조.
> 이 문서는 실전 코드 패턴과 체크리스트를 다룬다.

---

## 1. Profiler 코드 패턴

```csharp
using UnityEngine.Profiling;

public class PerformanceMonitor : MonoBehaviour
{
    private void Update()
    {
        // 커스텀 프로파일 구간 마킹
        Profiler.BeginSample("Enemy AI Update");
        UpdateEnemyAI();
        Profiler.EndSample();

        // 메모리 상태 확인
        long allocatedMemory = Profiler.GetTotalAllocatedMemoryLong();
        long reservedMemory = Profiler.GetTotalReservedMemoryLong();

        // FPS 계산
        float fps = 1.0f / Time.unscaledDeltaTime;
    }
}
```

Coplay MCP 연동:
- `get_worst_cpu_frames`: CPU 병목 프레임 추출
- `get_worst_gc_frames`: GC 스파이크 프레임 추출

---

## 2. 메모리 / GC 심화

```csharp
// === 나쁜 예 — 매 프레임 GC Alloc ===
void Update()
{
    string status = "Health: " + health + " / " + maxHealth; // Boxing + 할당
    var enemies = GameObject.FindGameObjectsWithTag("Enemy");  // 배열 할당
}

// === 좋은 예 — Zero Allocation ===
private StringBuilder _sb = new StringBuilder(50);
private List<Enemy> _enemyCache = new List<Enemy>(100);

void Update()
{
    // StringBuilder 재사용
    _sb.Clear();
    _sb.Append("Health: ").Append(health).Append(" / ").Append(maxHealth);

    // 캐싱된 리스트 사용 (Start에서 채움)
    foreach (var enemy in _enemyCache)
        enemy.UpdateLogic();
}
```

### 추가 GC 팁
- `foreach` → `for (int i = 0; i < list.Count; i++)` (IEnumerator 박싱 방지)
- 람다/클로저: 변수 캡처 시 힙 할당 발생 → 핫 루프에서 회피
- `string.Format` / `$""` 보간 → StringBuilder로 교체
- LINQ → 핫 패스에서 사용 금지 (GC 할당 다수)

---

## 3. Draw Call 배칭 코드

```csharp
// Static Batching — 코드로 수동 설정
public class StaticBatchHelper : MonoBehaviour
{
    void Start()
    {
        GameObject[] staticObjects = GameObject.FindGameObjectsWithTag("StaticProp");
        StaticBatchingUtility.Combine(staticObjects, gameObject);
    }
}

// Dynamic Batching 조건:
// - 같은 Material
// - 정점 300개 이하
// - 같은 Scale (비균일 스케일 시 배칭 깨짐)
// - 라이트맵 없음

// GPU Instancing (대량 동일 오브젝트):
// 셰이더에 #pragma multi_compile_instancing 추가
// Material Inspector에서 Enable GPU Instancing 체크
// Graphics.DrawMeshInstanced 또는 Graphics.RenderMeshInstanced 사용
```

### 머티리얼 공유 주의
```csharp
// 나쁜 예 — Material 인스턴스 생성 (배칭 파괴!)
renderer.material.color = Color.red;

// 좋은 예 — SharedMaterial + MaterialPropertyBlock
var mpb = new MaterialPropertyBlock();
mpb.SetColor("_BaseColor", Color.red);
renderer.SetPropertyBlock(mpb);
```

---

## 4. LOD 코드 설정

```csharp
void SetupLOD()
{
    LODGroup lodGroup = gameObject.AddComponent<LODGroup>();

    LOD[] lods = new LOD[3];
    lods[0] = new LOD(0.6f, GetRenderers("LOD0")); // 0~60% 화면: 고디테일
    lods[1] = new LOD(0.3f, GetRenderers("LOD1")); // 60~30%: 중디테일
    lods[2] = new LOD(0.1f, GetRenderers("LOD2")); // 30~10%: 저디테일

    lodGroup.SetLODs(lods);
    lodGroup.RecalculateBounds();
}

private Renderer[] GetRenderers(string lodName)
{
    return transform.Find(lodName).GetComponentsInChildren<Renderer>();
}
```

---

## 5. 오클루전 컬링

설정 순서:
1. Static 오브젝트에 "Occluder Static" / "Occludee Static" 마킹
2. `Window > Rendering > Occlusion Culling` → Bake
3. 큰 벽/건물 = Occluder, 작은 소품 = Occludee

런타임 Frustum 체크 (커스텀):
```csharp
Plane[] planes = GeometryUtility.CalculateFrustumPlanes(Camera.main);
Bounds bounds = GetComponent<Renderer>().bounds;
if (GeometryUtility.TestPlanesAABB(planes, bounds))
{
    // 카메라 뷰 안에 있음 → 업데이트
}
```

---

## 6. Physics 최적화

```csharp
void Start()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    rb.sleepThreshold = 0.1f;  // Sleep 허용 (비활성 물체 연산 절감)
    rb.interpolation = RigidbodyInterpolation.None; // 필요할 때만 Interpolate

    // Fixed Timestep: Project Settings > Time (기본 0.02 = 50fps)
}

// Raycast 간격 제한
private float _raycastInterval = 0.1f;
private float _nextRaycast;
private RaycastHit _hitCache;

void Update()
{
    if (Time.time < _nextRaycast) return;

    int layerMask = 1 << LayerMask.NameToLayer("Ground");
    if (Physics.Raycast(transform.position, Vector3.down, out _hitCache, 10f, layerMask))
    {
        // 처리
    }
    _nextRaycast = Time.time + _raycastInterval;
}
```

---

## 7. Update 분산 실행

### Staggered Update
```csharp
private static int _updateOffset = 0;
private int _myOffset;

void Start() { _myOffset = _updateOffset++; }

void Update()
{
    // 5프레임마다 한 번, 오브젝트별 오프셋으로 분산
    if ((Time.frameCount + _myOffset) % 5 == 0)
        ExpensiveUpdate();
}
```

### Distance-based Update Rate
```csharp
private Transform _player;
private float _updateInterval;
private float _nextUpdate;

void Update()
{
    if (Time.time < _nextUpdate) return;

    float distance = Vector3.Distance(transform.position, _player.position);

    _updateInterval = distance < 10f ? 0.05f   // 가까움: 20fps
                    : distance < 50f ? 0.1f     // 보통: 10fps
                    : 0.5f;                      // 멀리: 2fps

    PerformUpdate();
    _nextUpdate = Time.time + _updateInterval;
}
```

---

## 8. 비동기 로딩

```csharp
// Addressable 기반 (프레임워크 패턴)
// ResourceLoadManager.instance.CreateObject(address, parent)

// 직접 로딩 시:
public IEnumerator LoadSceneAsync(string sceneName)
{
    AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);
    op.allowSceneActivation = false;

    while (!op.isDone)
    {
        float progress = Mathf.Clamp01(op.progress / 0.9f);
        // 로딩 UI 업데이트

        if (op.progress >= 0.9f)
        {
            yield return new WaitForSeconds(1f);
            op.allowSceneActivation = true;
        }
        yield return null;
    }
}
```

---

## 9. 성능 체크리스트

**목표: 60 FPS (16.67ms/프레임)**

| 우선순위 | 항목 | 확인 |
|:--------:|------|------|
| 1 | Profiler로 병목 확인 (CPU/GPU/Memory) | ☐ |
| 2 | Draw Call 절감 (배칭, 인스턴싱) | ☐ |
| 3 | Update 루프 최적화 (분산, 거리 기반) | ☐ |
| 4 | Object Pooling (PoolManager 사용) | ☐ |
| 5 | LOD 시스템 구축 | ☐ |
| 6 | Occlusion Culling 베이킹 | ☐ |
| 7 | 텍스처 사이즈/압축 최적화 | ☐ |
| 8 | GC Alloc 제거 (매 프레임 0 목표) | ☐ |
| 9 | Async 로딩 (Addressable) | ☐ |
| 10 | Distance-based Update Rate | ☐ |
