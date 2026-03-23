---
name: publishing-guide
description: "게임 퍼블리싱/배포 가이드. Steam 통합(Achievements, Cloud Saves, Leaderboards, Workshop, Rich Presence), 콘솔 인증, 컨트롤러 지원, 입력 추상화. 게임 출시, Steam 연동, 스토어 등록, 콘솔 포팅, 컨트롤러 매핑 작업 시 참조."
---

# 게임 퍼블리싱 가이드

> **빌드 최적화는 `unity-developer` 스킬 참조.**
> **씬 플로우/전환은 `scene-structure` 스킬 참조.**
> 이 스킬은 스토어 배포, 플랫폼 통합, 컨트롤러 지원을 다룬다.

---

## Steam 통합

### Steamworks.NET 기본 설정

1. Steamworks.NET 패키지 설치 (NuGet 또는 직접 임포트)
2. `steam_appid.txt`에 App ID 기입 (루트 폴더)
3. `SteamManager` 초기화 (프레임워크: Bootstrap 씬에서)

### Steam 기능 체크리스트

| 기능 | 용도 | 연동 시스템 |
|------|------|-------------|
| **Achievements** | 플레이어 목표/달성 | `AchievementSystem` |
| **Cloud Saves** | 크로스 디바이스 진행 | `SaveSystem` |
| **Leaderboards** | 랭킹/경쟁 | `LeaderboardSystem` |
| **Workshop** | 유저 모드 | (별도 구현) |
| **Rich Presence** | 인게임 상태 표시 | `GameSessionManager` |
| **Stats** | 플레이 통계 | `AnalyticsSystem` |

### Achievement 구현 패턴
```csharp
// EventBus 연동
EventBus.instance.Subscribe<EnemyKilledEvent>()
    .Subscribe(e =>
    {
        _killCount++;
        if (_killCount >= 100)
            SteamUserStats.SetAchievement("ACH_100_KILLS");
        SteamUserStats.StoreStats();
    })
    .AddTo(_disposables);
```

### Cloud Save 패턴
```csharp
// SaveSystem과 연동
// Steam Cloud는 파일 기반 — SaveSystem의 로컬 파일을 Steam Cloud 경로에 저장

// 저장
byte[] data = SerializeSaveData();
SteamRemoteStorage.FileWrite("save_slot_1.dat", data, data.Length);

// 로드
int fileSize = SteamRemoteStorage.GetFileSize("save_slot_1.dat");
byte[] buffer = new byte[fileSize];
SteamRemoteStorage.FileRead("save_slot_1.dat", buffer, fileSize);
```

---

## 콘솔 인증 요건

| 플랫폼 | 인증 | 핵심 요구사항 |
|--------|------|---------------|
| PlayStation | TRC (Technical Requirements Checklist) | 트로피, 세이브 아이콘, 에러 핸들링, 서스펜드/리줌 |
| Xbox | XR (Xbox Requirements) | GamerScore 달성, 클라우드 세이브, 접근성 |
| Nintendo Switch | Lotcheck | Joy-Con 분리/합체 대응, 독/포터블 모드 |

### 공통 요구사항
- 언어/지역 설정 대응
- 연결 끊김 시 적절한 에러 메시지
- 저장 중 전원 OFF 대응 (세이브 무결성)
- 플랫폼 고유 UI 가이드라인 준수

---

## 컨트롤러 지원

### 입력 추상화 (필수)

```
액션 기반 매핑 (버튼이 아님):
- "confirm" → A (Xbox), Cross (PS), B (Nintendo)
- "cancel"  → B (Xbox), Circle (PS), A (Nintendo)
- "pause"   → Start (Xbox), Options (PS), + (Nintendo)
```

### Unity Input System 설정
```csharp
// InputAction Asset 기반 — 플랫폼별 바인딩
// 프레임워크: InputSystem의 InputActionAsset 사용

// 액션 정의 (Inspector 또는 코드)
var confirmAction = new InputAction("Confirm", binding: "<Gamepad>/buttonSouth");
confirmAction.AddBinding("<Keyboard>/enter");

confirmAction.performed += ctx => OnConfirm();
confirmAction.Enable();
```

### 햅틱 피드백

| 강도 | 용도 | 예시 |
|------|------|------|
| Light | UI 피드백 | 버튼 선택, 커서 이동 |
| Medium | 충격 | 피격, 착지, 아이템 획득 |
| Heavy | 주요 이벤트 | 보스 등장, 폭발, 사망 |

```csharp
// Unity Input System 햅틱
Gamepad.current?.SetMotorSpeeds(lowFreq, highFreq);

// 일정 시간 후 정지
Observable.Timer(TimeSpan.FromSeconds(duration))
    .Subscribe(_ => Gamepad.current?.SetMotorSpeeds(0, 0))
    .AddTo(_disposables);
```

---

## 빌드 & 배포 체크리스트

### 출시 전 공통

| 항목 | 확인 |
|------|------|
| 모든 디버그 로그 제거 또는 조건부 컴파일 | ☐ |
| Release 빌드 프로파일링 (60 FPS 확인) | ☐ |
| 메모리 누수 없음 확인 | ☐ |
| 모든 씬 Build Settings에 등록 | ☐ |
| Addressable 빌드 완료 | ☐ |
| 크래시 리포팅 SDK 연동 (Sentry, Backtrace 등) | ☐ |

### Steam 출시

| 항목 | 확인 |
|------|------|
| Steamworks App ID 설정 | ☐ |
| Achievement 목록 등록 + 테스트 | ☐ |
| Cloud Save 동기화 테스트 | ☐ |
| 스토어 페이지 (스크린샷, 트레일러, 설명) | ☐ |
| Depot/Branch 설정 (기본, 베타) | ☐ |
| 리뷰 제출 전 빌드 업로드 | ☐ |

### 모바일 (참고)

| 플랫폼 | 주요 확인 |
|--------|-----------|
| iOS | Privacy Labels, 스크린샷 (전 기기), 계정 삭제 기능 |
| Android | Target API Level, 데이터 안전 양식, 64-bit 지원 |

---

## 엔진별 참고

### Unity 6 배포 특이사항
- IL2CPP 백엔드 필수 (콘솔, 성능)
- Managed Stripping Level: High (빌드 크기 절감)
- Addressables Remote Catalog: CDN 배포 시
- `DEVELOPMENT_BUILD` 매크로 제거 확인

### 프레임워크 연동
- `PlatformManager.SelectedPlatform` → 빌드 시 플랫폼 자동 감지
- Steam 빌드: `2_2_Steam` MainMenu 씬 사용
- 조건부 컴파일: `#if UNITY_STANDALONE_WIN`, `#if ENABLE_STEAM` 등
