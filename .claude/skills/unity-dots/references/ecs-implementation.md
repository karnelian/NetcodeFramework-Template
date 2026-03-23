# ECS 구현 패턴 레퍼런스

> 기본 개념/선택 기준은 `SKILL.md` 참조. 이 문서는 심화 패턴 코드를 다룬다.

---

## 1. EntityQuery (복잡한 쿼리)

```csharp
[BurstCompile]
public partial struct QueryExamplesSystem : ISystem
{
    private EntityQuery _enemyQuery;

    public void OnCreate(ref SystemState state)
    {
        _enemyQuery = new EntityQueryBuilder(Allocator.Temp)
            .WithAll<EnemyTag, Health, LocalTransform>()
            .WithNone<Dead>()
            .Build(ref state);
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // SystemAPI.Query — 가장 간단한 방법
        foreach (var (health, entity) in
            SystemAPI.Query<RefRW<Health>>()
                .WithAll<EnemyTag>()
                .WithEntityAccess())
        {
            if (health.ValueRO.Current <= 0)
            {
                SystemAPI.GetSingleton<EndSimulationEntityCommandBufferSystem.Singleton>()
                    .CreateCommandBuffer(state.WorldUnmanaged)
                    .DestroyEntity(entity);
            }
        }

        // 엔티티 수 조회
        int enemyCount = _enemyQuery.CalculateEntityCount();

        // 배열로 변환
        var enemies = _enemyQuery.ToEntityArray(Allocator.Temp);
        var healths = _enemyQuery.ToComponentDataArray<Health>(Allocator.Temp);
    }
}
```

---

## 2. Parallel ECB (병렬 구조 변경)

```csharp
[BurstCompile]
public partial struct SpawnSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var ecb = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>()
            .CreateCommandBuffer(state.WorldUnmanaged);

        foreach (var (spawner, transform) in
            SystemAPI.Query<RefRW<Spawner>, RefRO<LocalTransform>>())
        {
            spawner.ValueRW.Timer -= SystemAPI.Time.DeltaTime;

            if (spawner.ValueRO.Timer <= 0)
            {
                spawner.ValueRW.Timer = spawner.ValueRO.Interval;

                Entity e = ecb.Instantiate(spawner.ValueRO.Prefab);
                ecb.SetComponent(e, new LocalTransform
                {
                    Position = transform.ValueRO.Position,
                    Rotation = quaternion.identity,
                    Scale = 1f
                });
                ecb.AddComponent(e, new Speed { Value = 5f });
            }
        }
    }
}

// 병렬 Job에서 ECB 사용
[BurstCompile]
public partial struct ParallelSpawnJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter ECB;

    void Execute([EntityIndexInQuery] int index, in Spawner spawner)
    {
        Entity e = ECB.Instantiate(index, spawner.Prefab);
        ECB.AddComponent(index, e, new Speed { Value = 5f });
    }
}
```

---

## 3. Aspect (컴포넌트 그룹핑)

```csharp
public readonly partial struct CharacterAspect : IAspect
{
    public readonly Entity Entity;

    private readonly RefRW<LocalTransform> _transform;
    private readonly RefRO<Speed> _speed;
    private readonly RefRW<Health> _health;

    [Optional]
    private readonly RefRO<Shield> _shield;

    private readonly DynamicBuffer<InventoryItem> _inventory;

    public float3 Position
    {
        get => _transform.ValueRO.Position;
        set => _transform.ValueRW.Position = value;
    }

    public float CurrentHealth => _health.ValueRO.Current;
    public float MaxHealth => _health.ValueRO.Max;
    public bool HasShield => _shield.IsValid;

    public void TakeDamage(float amount)
    {
        float remaining = amount;
        if (HasShield && _shield.ValueRO.Amount > 0)
            remaining = math.max(0, amount - _shield.ValueRO.Amount);
        _health.ValueRW.Current = math.max(0, _health.ValueRO.Current - remaining);
    }

    public void Move(float3 direction, float deltaTime)
    {
        _transform.ValueRW.Position += direction * _speed.ValueRO.Value * deltaTime;
    }

    public void AddItem(int itemId, int quantity)
    {
        _inventory.Add(new InventoryItem { ItemId = itemId, Quantity = quantity });
    }
}

// 사용
[BurstCompile]
public partial struct CharacterSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float dt = SystemAPI.Time.DeltaTime;
        foreach (var character in SystemAPI.Query<CharacterAspect>())
        {
            character.Move(new float3(1, 0, 0), dt);
        }
    }
}
```

---

## 4. Singleton 컴포넌트

```csharp
public struct GameConfig : IComponentData
{
    public float DifficultyMultiplier;
    public int MaxEnemies;
    public float SpawnRate;
}

public struct GameState : IComponentData
{
    public int Score;
    public int Wave;
    public float TimeRemaining;
}

// 초기화
public partial struct GameInitSystem : ISystem
{
    public void OnCreate(ref SystemState state)
    {
        var entity = state.EntityManager.CreateEntity();
        state.EntityManager.AddComponentData(entity, new GameConfig
        {
            DifficultyMultiplier = 1.0f,
            MaxEnemies = 100,
            SpawnRate = 2.0f
        });
    }
}

// 접근
[BurstCompile]
public partial struct ScoreSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var config = SystemAPI.GetSingleton<GameConfig>();
        ref var gameState = ref SystemAPI.GetSingletonRW<GameState>().ValueRW;
        gameState.TimeRemaining -= SystemAPI.Time.DeltaTime;
    }
}
```

---

## 5. Baking (GameObject → Entity 변환)

```csharp
// Authoring 컴포넌트 (에디터에서 MonoBehaviour)
public class EnemyAuthoring : MonoBehaviour
{
    public float Speed = 5f;
    public float Health = 100f;
    public GameObject ProjectilePrefab;

    class Baker : Baker<EnemyAuthoring>
    {
        public override void Bake(EnemyAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);

            AddComponent(entity, new Speed { Value = authoring.Speed });
            AddComponent(entity, new Health
            {
                Current = authoring.Health,
                Max = authoring.Health
            });
            AddComponent(entity, new EnemyTag());

            if (authoring.ProjectilePrefab != null)
            {
                AddComponent(entity, new ProjectilePrefab
                {
                    Value = GetEntity(authoring.ProjectilePrefab, TransformUsageFlags.Dynamic)
                });
            }
        }
    }
}

// 버퍼 Baking
public class SpawnerAuthoring : MonoBehaviour
{
    public GameObject[] Prefabs;
    public float Interval = 1f;

    class Baker : Baker<SpawnerAuthoring>
    {
        public override void Bake(SpawnerAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);

            AddComponent(entity, new Spawner
            {
                Interval = authoring.Interval,
                Timer = 0f
            });

            var buffer = AddBuffer<SpawnPrefabElement>(entity);
            foreach (var prefab in authoring.Prefabs)
            {
                buffer.Add(new SpawnPrefabElement
                {
                    Prefab = GetEntity(prefab, TransformUsageFlags.Dynamic)
                });
            }

            DependsOn(authoring.Prefabs);
        }
    }
}
```

---

## 6. Spatial Hash Job (Native Collections)

```csharp
[BurstCompile]
public struct SpatialHashJob : IJobParallelFor
{
    [ReadOnly] public NativeArray<float3> Positions;
    public NativeParallelMultiHashMap<int, int>.ParallelWriter HashMap;
    public float CellSize;

    public void Execute(int index)
    {
        float3 pos = Positions[index];
        int hash = GetHash(pos);
        HashMap.Add(hash, index);
    }

    int GetHash(float3 pos)
    {
        int x = (int)math.floor(pos.x / CellSize);
        int y = (int)math.floor(pos.y / CellSize);
        int z = (int)math.floor(pos.z / CellSize);
        return x * 73856093 ^ y * 19349663 ^ z * 83492791;
    }
}

[BurstCompile]
public partial struct SpatialHashSystem : ISystem
{
    private NativeParallelMultiHashMap<int, int> _hashMap;

    public void OnCreate(ref SystemState state)
    {
        _hashMap = new NativeParallelMultiHashMap<int, int>(10000, Allocator.Persistent);
    }

    public void OnDestroy(ref SystemState state)
    {
        _hashMap.Dispose();
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var query = SystemAPI.QueryBuilder().WithAll<LocalTransform>().Build();
        int count = query.CalculateEntityCount();

        if (_hashMap.Capacity < count)
            _hashMap.Capacity = count * 2;
        _hashMap.Clear();

        var positions = query.ToComponentDataArray<LocalTransform>(Allocator.TempJob);
        var posFloat3 = new NativeArray<float3>(count, Allocator.TempJob);
        for (int i = 0; i < count; i++)
            posFloat3[i] = positions[i].Position;

        var hashJob = new SpatialHashJob
        {
            Positions = posFloat3,
            HashMap = _hashMap.AsParallelWriter(),
            CellSize = 10f
        };
        state.Dependency = hashJob.Schedule(count, 64, state.Dependency);

        positions.Dispose(state.Dependency);
        posFloat3.Dispose(state.Dependency);
    }
}
```

---

## 7. IJobChunk (Chunk 단위 병렬)

```csharp
[BurstCompile]
partial struct HealthProcessJob : IJobChunk
{
    public ComponentTypeHandle<Health> HealthHandle;

    public void Execute(in ArchetypeChunk chunk, int unfilteredChunkIndex,
        bool useEnabledMask, in v128 chunkEnabledMask)
    {
        var healths = chunk.GetNativeArray(ref HealthHandle);
        for (int i = 0; i < chunk.Count; i++)
        {
            var h = healths[i];
            // 처리
        }
    }
}
```

---

## 8. Enableable Component (구조 변경 회피)

```csharp
// Add/Remove 대신 Enable/Disable로 상태 전환
public struct Disabled : IComponentData, IEnableableComponent { }

// 사용:
// ecb.SetComponentEnabled<Disabled>(entity, true);  // 비활성화
// ecb.SetComponentEnabled<Disabled>(entity, false); // 활성화

// 쿼리에서 자동 필터링됨
foreach (var (health, entity) in
    SystemAPI.Query<RefRW<Health>>()
        .WithDisabled<Disabled>()  // 비활성화된 것만
        .WithEntityAccess())
{
    // ...
}
```
