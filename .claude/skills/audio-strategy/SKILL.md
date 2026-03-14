---
name: audio-strategy
description: "사운드/오디오 리소스 확보, AI 생성, 구현, 관리 시 참조. VARCO Sound (SFX), MiniMax Music 2.5+ (BGM) 워크플로우 포함."
---

# 사운드/오디오 전략

> AI 오디오 생성 워크플로우는 **references/ai-audio-workflow.md** 참조.

## 오디오 카테고리

| 카테고리 | 설명 | 예시 | AI 생성 도구 |
|----------|------|------|-------------|
| BGM | 배경 음악 | 메인메뉴, 로비, 전투, 보스전 | **MiniMax Music 2.5+** |
| SFX | 효과음 | 타격, 발소리, 환경음, 스킬 | **VARCO Sound** |
| UI Sound | 인터페이스 사운드 | 버튼 클릭, 팝업, 알림, 탭 전환 | **VARCO Sound** |
| Ambient | 환경 사운드 | 바람, 물소리, 숲, 도시 소음 | **VARCO Sound** |
| Voice | 보이스/대사 | NPC 대사, 캐릭터 음성 | ElevenLabs |

---

## AI 오디오 생성 도구

### 1차 선택 (권장)

| 도구 | 용도 | 특징 |
|------|------|------|
| **MiniMax Music 2.5+** | BGM (인스트루멘탈) | API 기반, 14개 구조 태그, 장르별 자동 믹싱, $0.15/곡 |
| **VARCO Sound** | SFX / 환경음 / UI사운드 | 텍스트→사운드, 멀티트랙 출력, 루프 생성, Unity 플러그인 |

### 2차 선택 (보조/대안)

| 도구 | 용도 | 비고 |
|------|------|------|
| ElevenLabs | 보이스/TTS | 고품질 AI 보이스, 감정 표현 |
| Freesound.org | SFX 보충 | 무료 CC 라이선스 |
| Sonniss GDC Bundle | SFX 대량 확보 | 매년 무료 배포 |

### 워크플로우 상세 → references/ai-audio-workflow.md

---

## 기존 프레임워크 연동

AudioSystem은 이미 프레임워크에 구현되어 있음:
- `AudioManager`: MonoSingleton 기반 매니저
- `AudioSystemSettings`: ScriptableObject 설정
- `BGMController` / `SFXController`: 재생 컨트롤러
- `MainAudioMixer.mixer`: Master, BGM, SFX 그룹
- Addressable 경로: `Assets/NetcodeFramework/Addressables/Manager/AudioManager/Sound/`

새로운 오디오 추가 시 기존 시스템 API를 활용할 것.

---

## 오디오 폴더 구조

기존 Addressable 구조 내에 배치:
```
Assets/NetcodeFramework/Addressables/Manager/AudioManager/
├── Sound/
│   ├── BGM/
│   │   ├── Menu/
│   │   ├── Lobby/
│   │   ├── Battle/
│   │   └── Boss/
│   ├── SFX/
│   │   ├── Combat/
│   │   ├── Environment/
│   │   ├── Character/
│   │   └── Interaction/
│   ├── UI/
│   └── Ambient/
└── AudioManagerSettings.asset
```

게임별 사운드는 `_Game/{GameName}/Audio/`에 배치.

---

## 포맷 규칙

| 용도 | 포맷 | Unity 압축 설정 | AI 생성 시 설정 |
|------|------|-----------------|----------------|
| BGM | .ogg | Streaming (메모리 절약) | MiniMax: format mp3 → Unity에서 ogg 변환 |
| SFX (짧은 ~2초) | .wav | Decompress On Load | VARCO: wav 출력 |
| SFX (긴 2초+) | .ogg | Compressed In Memory | VARCO: wav → Unity에서 ogg 변환 |
| UI Sound | .wav | Decompress On Load | VARCO: wav 출력 |

---

## 네이밍 규칙

```
BGM_{Scene}_{Mood}_{Number}.ogg       예: BGM_Battle_Intense_01.ogg
SFX_{Category}_{Name}_{Number}.wav    예: SFX_Combat_Gunshot_01.wav
UI_{Action}_{Number}.wav              예: UI_Click_01.wav
AMB_{Environment}_{Number}.ogg        예: AMB_Forest_Rain_01.ogg
VO_{Character}_{Line}_{Number}.wav    예: VO_NPC_Greeting_01.wav
```

---

## 라이선스 관리

사용하는 모든 오디오 에셋의 라이선스 기록:

| 에셋명 | 출처 | 라이선스 | 크레딧 필요 |
|--------|------|----------|-------------|
| *(사용 시 채움)* | | | |

**AI 생성 오디오 라이선스:**
| 도구 | 상업 이용 | 비고 |
|------|----------|------|
| MiniMax Music 2.5+ | API 이용약관 확인 필요 | 생성물 소유권 정책 확인 |
| VARCO Sound | 베타 기간 무료, 정식 출시 후 확인 | NC AI 이용약관 |
