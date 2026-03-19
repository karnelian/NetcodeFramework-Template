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

## 금지 사항

- `Observable.FromAsync`에 실제 await 없이 사용 금지 → `Observable.Return` 사용
- `.GetAwaiter().GetResult()` 직접 호출 금지 (Unity 메인 스레드 교착) → `Task.Run()` 래핑
- Manager에서 `new()` 직접 생성 금지 → MonoSingleton `.instance` 사용
- `Object.Destroy()` Edit Mode 테스트에서 사용 금지 → `Object.DestroyImmediate()` 사용
- asmdef 파일 생성 금지 → Assembly-CSharp 단일 어셈블리 유지

---

## 스킬 참조

### 프레임워크 전용 (실제 코드 기반)
- `framework-architecture`: Init Chain, ServiceType Registry, Platform Strategy, 새 시스템 추가 절차
- `framework-patterns`: R3 Observable, MVVM, Handle/Using, Extension Methods, 조건부 컴파일
- `framework-systems-api`: 26개 시스템 API 레퍼런스 (EventBus, Command, Pool, TimeScale 등)
- `scene-structure`: 씬 플로우, SceneSystem Strategy, 영속 매니저 목록
- `hierarchy-rules`: UI Prefab 계층 규칙 9패턴, GameObject 네이밍
- `testing-strategy`: Edit Mode NUnit, R3 오버로드 회피, async 교착 방지
- `asset-management`: Addressable 경로, 서브모듈 커밋 규칙, Git LFS

### 범용 Unity 개발
- `unity-developer`: 애니메이션, 물리, 성능 최적화, 셰이더, AI/NavMesh
- `art-style-guide`: 아트 스타일 일관성 규칙
- `audio-strategy`: 사운드 리소스 확보/관리 전략
- `varco-sfx-pipeline`: Varco AI SFX 생성→트리밍→변형→할당 파이프라인
- `auto-playtest`: 자동화 플레이테스트 실행/분석/밸런스 조정
- `genre-guide`: 장르별 필요한 System 서브모듈 조합 가이드
- `unity-so-builder`: 에디터 스크립트 SO 생성, 프리팹 필드 할당 패턴
- `ai-3d-pipeline`: AI 3D 모델 생성 파이프라인 (Coplay MCP)
- `procedural-generation`: 절차적 생성 가이드
- `ui-workflow`: UI 프로토타입 → Unity 변환 워크플로우
