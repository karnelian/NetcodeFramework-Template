---
name: procedural-generation
description: "코드 기반으로 생성 가능한 리소스 전략. 셰이더, 머티리얼, 터레인, 이펙트, 레벨 생성 가이드."
---

# 프로시저럴 생성 전략

## 목적

아티스트 없이 코드만으로 생성할 수 있는 리소스 영역을 정의한다.
Claude Code + Coplay MCP로 대부분 구현 가능.

---

## 코드로 생성 가능한 영역

### 셰이더 / 머티리얼 (90% 코드 가능)
- Shader Graph 노드 기반 셰이더 생성 (URP)
- 프로시저럴 텍스처: 노이즈 기반 패턴 (Perlin, Voronoi, Simplex)
- 머티리얼 프로퍼티 일괄 설정 스크립트
- 용도: 물, 용암, 에너지 쉴드, 홀로그램, 디졸브 효과

### 파티클 / VFX (80~90% 코드 가능)
- ParticleSystem 컴포넌트 코드 생성 및 파라미터 조정
- VFX Graph 노드 연결
- Sub Emitter 구성
- 부족한 것: 커스텀 파티클 텍스처 → 무료 팩 또는 AI 이미지 생성

### 터레인 (70% 코드 가능)
- Terrain Data 코드 생성: 하이트맵, 스플랫맵
- 노이즈 기반 지형 (산, 계곡, 평원)
- 프로시저럴 오브젝트 배치: 나무, 풀, 바위 랜덤 배치
- Coplay MCP: `create_terrain` 도구 활용
- 부족한 것: 커스텀 터레인 텍스처/브러시 → 에셋 팩

### 레벨 / 던전 (60~80% 코드 가능)
- BSP (Binary Space Partitioning): 룸 기반 던전
- Wave Function Collapse: 타일 기반 맵
- Cellular Automata: 동굴 형태
- ProBuilder API: 구조물 코드 생성
- 부족한 것: 디테일 데코레이션 → 에셋 팩

### UI 애니메이션 (100% 코드 가능)
- DOTween/LeanTween 기반 UI 트랜지션
- 커스텀 이징 커브
- 화면 전환 효과

### 라이팅 (90% 코드 가능)
- Light 컴포넌트 생성 및 설정
- 라이트 프로브 배치
- 포스트 프로세싱 프로파일 코드 생성
- 부족한 것: 라이트맵 베이킹 (에디터 수동 또는 자동화 스크립트)

---

## 기존 시스템 연동

### PoolSystem
프로시저럴 생성된 오브젝트는 PoolSystem으로 관리:
- `PoolManager.instance.Spawn(prefab)`: 풀에서 가져오기
- `PoolManager.instance.Despawn(obj)`: 풀에 반환
- 대량 생성/파괴 시 풀링 필수 (GC 방지)

### ResourceLoadSystem
Addressable 등록된 프리팹은 ResourceLoadManager로 로드:
- `ResourceLoadManager.instance.CreateObject(address, parent)`: 풀링 자동 적용
- `ResourceLoadManager.instance.CreateEffect(address, position)`: 이펙트 생성 + 자동 반환

---

## Coplay MCP 활용

| 도구 | 용도 |
|------|------|
| `create_terrain` | 베이스 터레인 생성 |
| `execute_script` | 에디터 스크립트 실행 (프로시저럴 배치) |
| `create_game_object` | 런타임 오브젝트 생성 |
| `create_material` | 머티리얼 생성 |
| `assign_material` | 머티리얼 할당 |
| `set_property` | 컴포넌트 프로퍼티 설정 |

---

## 구현 원칙

1. "코드로 될까?"를 먼저 판단 → 안 되는 부분만 외부 리소스
2. 프로시저럴 결과물은 프리팹으로 저장하여 재사용
3. **시드(Seed) 기반** 재현 가능하게 구현
4. ScriptableObject로 생성 파라미터 관리 (CLAUDE.md Settings 패턴 따름)
5. 에디터 툴로 만들어서 파라미터 조절 가능하게

### Settings 패턴 예시
```csharp
[CreateAssetMenu(
    fileName = "ProceduralSettings",
    menuName = "NetcodeFramework/Procedural/Settings")]
public class ProceduralSettings : ScriptableObject
{
    [Header("=== Generation ===")]
    [SerializeField] private int seed = 42;
    [SerializeField] private float noiseScale = 0.1f;

    [Header("=== Debug ===")]
    [SerializeField] private bool enableDebugLog = false;

    public int Seed => seed;
    public float NoiseScale => noiseScale;
    public bool EnableDebugLog => enableDebugLog;
}
```
