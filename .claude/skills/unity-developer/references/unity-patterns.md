# Unity 일반 패턴 레퍼런스

> 프레임워크(R3, MVVM, EventBus 등) 패턴은 `netcode-framework` 스킬 참조.
> 이 문서는 프레임워크 밖 일반 Unity MonoBehaviour/SO/Coroutine 패턴을 다룬다.

---

## MonoBehaviour 베스트 프랙티스

```csharp
public class EnemyController : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private Transform target;

    // 컴포넌트 캐싱 (Awake에서)
    private Rigidbody _rb;
    private Animator _animator;

    private void Awake()
    {
        _rb = GetComponent<Rigidbody>();
        _animator = GetComponent<Animator>();
    }

    private void Start()
    {
        if (target == null)
            target = GameObject.FindGameObjectWithTag("Player").transform;
    }

    private void FixedUpdate()
    {
        Vector3 direction = (target.position - transform.position).normalized;
        _rb.MovePosition(transform.position + direction * moveSpeed * Time.fixedDeltaTime);
    }

    private void OnDisable()
    {
        StopAllCoroutines();
    }
}
```

### 핵심 규칙
- `[SerializeField] private` > `public` (캡슐화 유지)
- Awake: 자기 자신 컴포넌트 캐싱
- Start: 다른 오브젝트 참조 초기화 (Awake 완료 후)
- OnDisable: Coroutine 정리, 이벤트 구독 해제

---

## ScriptableObject 데이터 패턴

```csharp
[CreateAssetMenu(fileName = "WeaponData", menuName = "Game/Weapon")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public int damage;
    public float fireRate;
    public GameObject projectilePrefab;
    public AudioClip fireSound;

    // 순수 함수 메서드 허용
    public float GetDamageMultiplier(float distance)
    {
        return Mathf.Max(0.5f, 1f - (distance / 100f));
    }
}

// 사용
public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponData _data;
    private float _nextFireTime;

    public void Fire()
    {
        if (Time.time < _nextFireTime) return;
        Instantiate(_data.projectilePrefab, transform.position, transform.rotation);
        _nextFireTime = Time.time + 1f / _data.fireRate;
    }
}
```

> **에디터 스크립트로 SO 일괄 생성은 `unity-so-builder` 스킬 참조.**

---

## Object Pooling (일반 패턴)

> **프레임워크 프로젝트에서는 `PoolManager.instance.Spawn/Despawn` 사용 권장.**
> 아래는 프레임워크 밖이거나 커스텀 풀이 필요할 때 참고.

```csharp
public class ObjectPool : MonoBehaviour
{
    [SerializeField] private GameObject prefab;
    [SerializeField] private int poolSize = 20;
    private Queue<GameObject> _pool = new Queue<GameObject>();

    private void Start()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject obj = Instantiate(prefab);
            obj.SetActive(false);
            _pool.Enqueue(obj);
        }
    }

    public GameObject Get()
    {
        if (_pool.Count > 0)
        {
            GameObject obj = _pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(prefab); // 풀 소진 시 확장
    }

    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

---

## Event 패턴 (프레임워크 밖)

```csharp
// C# event (가볍고 빠름)
public static class GameEvents
{
    public static event Action<int> OnScoreChanged;
    public static event Action<string> OnGameOver;

    public static void TriggerScoreChanged(int score) => OnScoreChanged?.Invoke(score);
    public static void TriggerGameOver(string reason) => OnGameOver?.Invoke(reason);
}
```

> **프레임워크에서는 `EventBus.instance.Publish/Subscribe` 사용.**
> 위 static event는 프레임워크 밖 프로토타입이나 독립 시스템에서만 사용.

---

## Coroutine 패턴

```csharp
public class TimedAbility : MonoBehaviour
{
    // WaitForSeconds 캐싱 (GC 방지)
    private WaitForSeconds _cooldownWait = new WaitForSeconds(5f);
    private Coroutine _current;

    public void Activate()
    {
        if (_current != null)
            StopCoroutine(_current);
        _current = StartCoroutine(AbilityRoutine());
    }

    private IEnumerator AbilityRoutine()
    {
        Debug.Log("Ability activated");
        yield return _cooldownWait;
        Debug.Log("Ability ready");
        _current = null;
    }

    // 위치 보간
    private IEnumerator LerpPosition(Vector3 target, float duration)
    {
        Vector3 start = transform.position;
        float elapsed = 0f;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            transform.position = Vector3.Lerp(start, target, elapsed / duration);
            yield return null;
        }
        transform.position = target;
    }
}
```

---

## Performance 요약 규칙

- `GetComponent<T>()` → Awake에서 캐싱
- `CompareTag()` 사용 (string 비교 금지)
- `Camera.main` → Start에서 캐싱
- `renderer.material` 금지 → `sharedMaterial` + MaterialPropertyBlock
- 컴포넌트 비활성화 > GameObject 비활성화 (가능한 경우)
- `StringBuilder` 사용 (string 연결 금지)
