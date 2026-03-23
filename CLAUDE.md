# NetcodeFramework - 개발 규칙

Unity 멀티플레이어 게임 프레임워크. 이 파일의 규칙을 반드시 따를 것.
상세 아키텍처/패턴/시스템 API는 `.claude/skills/` 스킬 파일 참조.

---

## 네이밍 컨벤션

### 클래스/파일명
| 종류 | 패턴 | 예시 |
|------|------|------|
| Manager | `{System}Manager` | `AudioManager` |
| Service 인터페이스 | `I{System}Service` | `IAuthenticationService` |
| 플랫폼 구현 | `{Platform}{System}Service` | `UGSAuthenticationService` |
| Settings SO | `{System}Settings` | `AudioSystemSettings` |
| ViewModel | `{Screen}ViewModel` | `MainMenuViewModel` |
| View | `{Screen}View` | `MainMenuView` |
| Handler | `{Purpose}Handler` | `LocalSaveHandler` |

### 메서드/프로퍼티
| 종류 | 패턴 | 예시 |
|------|------|------|
| Observable 반환 비동기 | `{Action}AsObservable` | `LoadSceneAsObservable()` |
| Observable 반환 async | `{Action}AsyncAsObservable` | `InitializeAsyncAsObservable()` |
| 이벤트 콜백 | `On{Event}` | `OnSignInSuccess` |
| private 필드 | `_{camelCase}` | `_masterVolume` |
| public 프로퍼티 | `PascalCase` | `MasterVolume` |

### 로그 포맷
```csharp
Debug.Log($"[{GetType().Name}] 설명: {정보}");
Debug.LogWarning($"[{GetType().Name}] 경고: {내용}");
Debug.LogError($"[{GetType().Name}] 에러: {메시지}");
```

---

## ScriptableObject Settings 패턴

```csharp
[CreateAssetMenu(
    fileName = "XxxSettings",
    menuName = "NetcodeFramework/Category/Settings")]
public class XxxSettings : ScriptableObject
{
    [Header("=== 섹션명 ===")]
    [Tooltip("설명")]
    [SerializeField] private float value = 0.5f;

    [Header("=== Debug ===")]
    [SerializeField] private bool enableDebugLog = false;

    public float Value => value;
    public bool EnableDebugLog => enableDebugLog;
}
```

---

## 코딩 스타일

### 필수 규칙
- switch 표현식 사용 (switch문 대신)
- null 조건 연산자: `_service?.Initialize()`
- null 병합 연산자: `_pool?.Count ?? 0`
- 가드 절: `if (_isInitialized) return;`
- HasInstance 체크: `if (AudioManager.HasInstance) AudioManager.instance.PlaySFX(clip);`
- `[SerializeField]` + `[Header]` + `[Tooltip]` 조합

### using 순서
```csharp
using UnityEngine;
using R3;
using System;
using System.Collections.Generic;
using NetcodeFramework.Extensions;
```

---

## 스킬 사용 규칙

### 필수 원칙
- **스킬 파일이 존재하는 영역의 작업은 해당 스킬을 읽지 않고 진행하지 말 것**
- **작업 시작 전에 "참조 스킬: xxx" 형태로 읽은 스킬을 응답에 명시할 것**
- 스킬을 읽지 않은 작업은 품질 미달로 간주

### 스킬 목록 (13개)

| 스킬 | 역할 |
|------|------|
| `netcode-framework` | 프레임워크 아키텍처, 코드 패턴(R3/MVVM/Handle), 26개 시스템 API |
| `unity-developer` | Unity 엔진 실무 (애니메이션, 물리, 성능 최적화, 셰이더, AI/NavMesh) |
| `ui-workflow` | UI 프로토타입→구현 워크플로우, Prefab 계층 규칙/네이밍 포함 |
| `audio-strategy` | 오디오 전략 (Varco SFX 파이프라인 + MiniMax BGM 포함) |
| `scene-structure` | 씬 플로우, 전환 규칙, 영속 매니저 |
| `asset-management` | Addressable 경로, 서브모듈, Git LFS |
| `testing-strategy` | NUnit 테스트 규칙, R3 오버로드 회피, async 교착 방지 |
| `unity-so-builder` | SO/프리팹 에디터 스크립트 패턴 |
| `auto-playtest` | Coplay MCP 자동 플레이테스트 |
| `ai-3d-pipeline` | AI 3D 모델 생성 파이프라인 (Meshy/Tripo/Coplay MCP) |
| `multiplayer-netcode` | 멀티플레이어 네트워킹 (예측/보간/래그 보상/동기화) |
| `unity-dots` | DOTS/ECS 패턴 (Job System, Burst Compiler) |
| `publishing-guide` | 배포/Steam 통합/콘솔 인증/컨트롤러 지원 |

### 작업 영역별 필수 스킬 매핑

| 작업 영역 | 필수 스킬 | 설명 |
|-----------|-----------|------|
| 새 Manager/System 추가 | `netcode-framework` | Init Chain, MonoSingleton, ServiceType Registry, R3/MVVM |
| EventBus/Command/Pool 등 | `netcode-framework` | 26개 시스템 API (references/systems-api.md) |
| R3 Observable/MVVM 코드 | `netcode-framework` | R3 규칙, ViewModel/View 패턴 (references/code-patterns.md) |
| UI 생성/수정 | `ui-workflow` | 네이밍(Popup_, btn_, t_, img_), 구조 패턴(_ani/_z_back), 프로토타입→Unity 변환. 계층 규칙은 references/hierarchy-rules.md |
| Popup/Toast UI | `ui-workflow`, `netcode-framework` | IPopup 인터페이스, UIManager API |
| 씬 구조/전환 | `scene-structure` | 씬 플로우, 영속 매니저, SceneSystem Strategy |
| 오디오/사운드/SFX 생성 | `audio-strategy` | BGM/SFX/UI 사운드 리소스 확보/관리/Varco API 파이프라인 |
| 3D 모델/에셋 생성 | `ai-3d-pipeline` | AI 모델 생성 파이프라인 |
| 자동화 플레이테스트 | `auto-playtest` | Coplay MCP Play→로그 분석→결과 비교 |
| SO/프리팹 설정 | `unity-so-builder` | 에디터 스크립트 SO 생성, 프리팹 필드 할당 |
| 물리/AI/NavMesh/최적화 | `unity-developer` | 애니메이션, 물리, 성능, 셰이더, AI. 심화 성능은 references/performance-deep-dive.md |
| 멀티플레이어 네트워킹 | `multiplayer-netcode` | 서버 권위, 예측/보간/리콘실리에이션, 래그 보상 |
| DOTS/ECS 구현 | `unity-dots` | Entity Component System, Job System, Burst Compiler |
| 에셋 관리/커밋 | `asset-management` | Addressable 경로, 서브모듈, Git LFS |
| 테스트 작성 | `testing-strategy` | Edit Mode NUnit, R3 오버로드 회피, async 교착 방지 |
| 게임 배포/스토어 | `publishing-guide` | Steam 통합, 콘솔 인증, 컨트롤러, 빌드 체크리스트 |

---

## 렌더 파이프라인 규칙 (URP)

- 프로젝트는 **Universal Render Pipeline (URP)** 사용
- 머티리얼 셰이더는 반드시 `Universal Render Pipeline/Lit` 또는 URP 호환 셰이더만 사용
- Built-in 셰이더 (`Standard`, `Synty/Generic_Basic` 등) 사용 금지 → 분홍색 렌더링 원인
- 외부 에셋 임포트 후 머티리얼 셰이더 확인 필수 → `ConvertToURPShader.cs` 에디터 스크립트로 일괄 변환
- ShaderGraph 사용 시 URP 파이프라인 대상으로 생성

---

## 금지 사항

- `Observable.FromAsync`에 실제 await 없이 사용 금지 → `Observable.Return` 사용
- `.GetAwaiter().GetResult()` 직접 호출 금지 (Unity 메인 스레드 교착) → `Task.Run()` 래핑
- Manager에서 `new()` 직접 생성 금지 → MonoSingleton `.instance` 사용
- `Object.Destroy()` Edit Mode 테스트에서 사용 금지 → `Object.DestroyImmediate()` 사용
- asmdef 파일 생성 금지 → Assembly-CSharp 단일 어셈블리 유지
