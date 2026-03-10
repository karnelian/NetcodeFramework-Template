---
name: scene-structure
description: "씬 플로우, 전환 규칙, 영속 매니저 목록. 씬 생성/수정/전환 코드 작성 시 참조."
---

# 씬 구조 규칙

---

## 씬 플로우

```
[1] Bootstrap (영속)
 │  GameSessionManager 생성 → PlatformManager 읽기
 │  → 모든 Manager 우선순위 기반 초기화 (DontDestroyOnLoad)
 │  → 플랫폼별 MainMenu 씬 로드
 │
 ▼
[2] MainMenu (플랫폼별 분기 — Bootstrap에서 결정)
 │  ├── 2_0_Mock          ← 개발/테스트용 (서버 없이 동작)
 │  ├── 2_1_UGS           ← Unity Gaming Services 연동
 │  ├── 2_2_Steam         ← Steam 플랫폼 연동
 │  └── 2_3_Local         ← 로컬/오프라인 모드
 │
 ▼
[3] LobbyBootstrap
 │  로비 진입 전 초기화. 매칭/네트워크 준비.
 │
 ▼
[4] Lobby (로비 형태별 분기)
 │  ├── 4_1_InteractiveLobby  ← 3D 공간 기반
 │  └── 4_2_Lobby             ← UI 기반
 │
 ▼
[5] InGame (스테이지별)
   ├── 5_1_Stage
   ├── 5_2_Stage
   └── ...
```

---

## SceneSystem Strategy 패턴

```csharp
// Initialize 시 모드에 따라 Strategy 자동 선택
_transitionStrategy = _cachedIsMultiplayer
    ? new MultiplayerTransitionStrategy(this)
    : new SingleplayerTransitionStrategy(this);

// 씬 로드 (Strategy가 알아서 처리)
SceneManager.instance.LoadSceneAsObservable("StageName", sceneData);
```

- **SingleplayerTransitionStrategy**: 일반 씬 로드 + LoadingScreen
- **MultiplayerTransitionStrategy**: NetworkManager 동기화 + 호스트/클라이언트 분기

---

## 영속 매니저 (DontDestroyOnLoad)

Bootstrap에서 생성, 앱 종료까지 유지. **모든 MonoSingleton Manager가 영속.**

| Priority | 매니저 | 역할 |
|----------|--------|------|
| 1 | ResourceLoadManager | Addressables 에셋 로딩 |
| 2 | SceneManager | 씬 전환 (Strategy 패턴) |
| 2 | UIManager | 팝업 스택 + 토스트 |
| 2 | AudioManager | BGM/SFX 풀링 |
| 2 | OptionSystem | 설정 관리 |
| 2 | InputManager | 입력 처리 |
| 2 | CameraManager | 카메라 제어 |
| 2 | NotificationManager | 알림 히스토리 |
| 3 | AuthenticationManager | 인증 |
| 3 | SaveManager | 저장/로드 |
| 3 | LobbyManager | 로비 |
| 3 | RelayManager | 네트워크 릴레이 |
| 3 | ChatManager | 채팅 |
| 3 | RemoteConfigManager | 원격 설정 |
| 3 | LeaderboardManager | 리더보드 |
| 3 | AchievementManager | 업적 |
| 3 | EconomyManager | 재화/아이템 |
| 4 | AnalyticsManager | 텔레메트리 |

**씬 전환 시 이 매니저들이 이미 존재한다고 가정. 절대 중복 생성하지 않는다.**

---

## 씬 네이밍 규칙

| 규칙 | 설명 |
|------|------|
| 번호 prefix | 로드 순서: `1_`, `2_`, `3_` ... |
| 소숫점 구분 | 같은 단계 변형: `_0_`, `_1_`, `_2_` |
| PascalCase | 번호 뒤 이름 |
| 역할 명시 | 이름만으로 역할 파악 가능 |

## 씬 폴더 구조

```
Assets/NetcodeFramework/Scenes/
├── 1_Bootstrap.unity
├── MainMenu/
│   ├── 2_0_Mock.unity
│   ├── 2_1_UGS.unity
│   ├── 2_2_Steam.unity
│   └── 2_3_Local.unity
├── 3_LobbyBootstrap.unity
├── Lobby/
│   ├── 4_1_InteractiveLobby.unity
│   └── 4_2_Lobby.unity
└── InGame/
    ├── 5_1_Stage.unity
    └── ...
```

## 씬 생성 시 주의사항

- 같은 단계의 씬은 상호 배타적 (2_1_UGS와 2_2_Steam 동시 로드 안 됨)
- 새 스테이지 추가 시 `5_N_StageName` 형식
- Build Settings 씬 순서 = 번호 체계
- 각 씬에서 영속 매니저 새로 생성 금지
