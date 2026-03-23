---
name: unity-dots
description: "Unity DOTS/ECS 패턴 가이드. Entity Component System, Job System, Burst Compiler. 대량 엔티티 관리, 데이터 지향 최적화, OOP→ECS 전환 시 참조. ECS, DOTS, Job System, Burst, ISystem, IJobEntity, Archetype, Chunk 관련 작업에 사용."
---

# Unity DOTS / ECS 패턴

> **일반 Unity 패턴(MonoBehaviour, SO 등)은 `unity-developer` 스킬 참조.**
> **오브젝트 풀링은 프레임워크 `PoolSystem` 우선 사용. DOTS 엔티티는 이 스킬 참조.**

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/ecs-implementation.md` | ISystem/IJobEntity 전체 패턴, EntityQuery, ECB, Aspect, Singleton, Baking, Native Collections, SpatialHash Job | ECS 시스템 구현 시 |

---

## 언제 DOTS를 쓸 것인가

| 조건 | 권장 |
|------|------|
| 수백~수천 유사 엔티티 (총알, 유닛, 파티클) | DOTS |
| 복잡한 개별 행동 (NPC 대화, UI) | MonoBehaviour |
| CPU 병목 + 병렬화 가능 | Jobs + Burst |
| 프로토타입 / 빠른 이터레이션 | MonoBehaviour → 나중에 DOTS 전환 |

**전환 규칙**: State Machine으로 시작. 성능이 요구될 때 ECS로 전환.

---

## 핵심 개념

### ECS vs OOP

| 측면 | 전통 OOP | ECS/DOTS |
|------|----------|----------|
| 데이터 배치 | 객체 지향 (분산) | 데이터 지향 (연속) |
| 메모리 | 흩어짐 | 연속 Chunk |
| 처리 | 객체 단위 | 일괄 처리 |
| 스케일링 | 엔티티 수에 비례 성능 저하 | 선형 스케일링 |

### DOTS 구성요소

```
Entity:    경량 ID (데이터 없음)
Component: 순수 데이터 (로직 없음)
System:    컴포넌트를 처리하는 로직
World:     엔티티 컨테이너
Archetype: 컴포넌트 조합의 고유 식별자
Chunk:     같은 Archetype 엔티티의 메모리 블록 (16KB)
```

---

## 기본 패턴

### Component 정의
```csharp
using Unity.Entities;
using Unity.Mathematics;

// 일반 컴포넌트
public struct Speed : IComponentData { public float Value; }
public struct Health : IComponentData { public float Current; public float Max; }

// 태그 (제로 사이즈 마커)
public struct EnemyTag : IComponentData { }

// 버퍼 (가변 크기 배열)
[InternalBufferCapacity(8)]
public struct InventoryItem : IBufferElementData
{
    public int ItemId;
    public int Quantity;
}

// Shared 컴포넌트 (엔티티 그룹핑)
public struct TeamId : ISharedComponentData { public int Value; }
```

### ISystem (권장 — Burst 호환)
```csharp
[BurstCompile]
public partial struct MovementSystem : ISystem
{
    [BurstCompile]
    public void OnCreate(ref SystemState state)
    {
        state.RequireForUpdate<Speed>();
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float dt = SystemAPI.Time.DeltaTime;

        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<Speed>>())
        {
            transform.ValueRW.Position +=
                new float3(0, 0, speed.ValueRO.Value * dt);
        }
    }
}
```

### IJobEntity (병렬 처리)
```csharp
[BurstCompile]
public partial struct MoveJob : IJobEntity
{
    public float DeltaTime;

    void Execute(ref LocalTransform transform, in Speed speed)
    {
        transform.Position += new float3(0, 0, speed.Value * DeltaTime);
    }
}

// System에서 스케줄링
state.Dependency = new MoveJob { DeltaTime = SystemAPI.Time.DeltaTime }
    .ScheduleParallel(state.Dependency);
```

### Entity Command Buffer (구조 변경)
```csharp
// 생성/파괴/컴포넌트 추가·제거는 ECB 사용
var ecb = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>()
    .CreateCommandBuffer(state.WorldUnmanaged);

Entity e = ecb.Instantiate(prefabEntity);
ecb.SetComponent(e, new Speed { Value = 5f });
ecb.AddComponent(e, new EnemyTag());
ecb.DestroyEntity(deadEntity);
```

---

## 성능 팁

| 규칙 | 이유 |
|------|------|
| `[BurstCompile]` 모든 ISystem/Job에 적용 | 10~50x 성능 향상 |
| `ISystem` > `SystemBase` | Unmanaged, Burst 호환 |
| `ScheduleParallel` 우선 사용 | 멀티코어 활용 |
| 구조 변경은 ECB로 모아서 처리 | Sync Point 최소화 |
| Enableable Component로 활성/비활성 | Add/Remove 대신 (구조 변경 회피) |
| Aspect로 컴포넌트 그룹핑 | 코드 가독성 + 재사용 |
| Native Collections는 반드시 Dispose | 메모리 누수 방지 |

---

## MUST DO / MUST NOT

### MUST DO
- Burst Compile 모든 시스템과 Job
- Profile 후 최적화 (추측 금지)
- Aspect로 관련 컴포넌트 묶기
- ECB로 구조 변경 일괄 처리

### MUST NOT
- Managed 타입(string, class) 컴포넌트에 사용 (Burst 불가)
- Job 내부에서 구조 변경 (ECB 사용)
- Native Collection Dispose 누락
- 과도 설계 (단순한 것부터 시작)
