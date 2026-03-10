---
name: audio-strategy
description: "사운드/오디오 리소스 확보, 구현, 관리 시 참조. BGM, SFX, UI 사운드, 보이스 전략 포함."
---

# 사운드/오디오 전략

## 오디오 카테고리

| 카테고리 | 설명 | 예시 |
|----------|------|------|
| BGM | 배경 음악 | 메인메뉴, 로비, 전투, 보스전 |
| SFX | 효과음 | 타격, 발소리, 환경음, 스킬 |
| UI Sound | 인터페이스 사운드 | 버튼 클릭, 팝업, 알림, 탭 전환 |
| Ambient | 환경 사운드 | 바람, 물소리, 숲, 도시 소음 |
| Voice | 보이스/대사 | NPC 대사, 캐릭터 음성 |

---

## 무료/저가 리소스 확보처

### BGM
- Freesound.org: 무료 CC 라이선스 사운드
- OpenGameArt.org: 게임용 무료 음악/SFX
- Incompetech (Kevin MacLeod): CC BY 무료 BGM
- Unity Asset Store: "Free Music" 검색

### SFX
- Freesound.org: 가장 큰 무료 SFX 라이브러리
- Sonniss GDC Audio Bundle: 매년 무료 배포 (수십 GB)
- BFXR / SFXR: 레트로/픽셀 게임용 SFX 생성기

### AI 오디오 생성
- Suno / Udio: AI 음악 생성 (상업 라이선스 확인 필수)
- ElevenLabs: AI 보이스 생성 (NPC 대사)
- Stable Audio: AI 기반 SFX/BGM 생성

### Coplay MCP 도구
- `generate_sfx`: AI SFX 생성
- `generate_music`: AI BGM 생성
- `generate_tts`: AI TTS 보이스 생성
- `search_tts_voice_id`: TTS 보이스 검색

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
│   ├── SFX/
│   │   ├── Combat/
│   │   ├── Environment/
│   │   └── Character/
│   ├── UI/
│   └── Ambient/
└── AudioManagerSettings.asset
```

게임별 사운드는 `_Game/{GameName}/Audio/`에 배치.

---

## 포맷 규칙

| 용도 | 포맷 | Unity 압축 설정 |
|------|------|-----------------|
| BGM | .ogg | Streaming (메모리 절약) |
| SFX (짧은 ~2초) | .wav | Decompress On Load |
| SFX (긴 2초+) | .ogg | Compressed In Memory |
| UI Sound | .wav | Decompress On Load |

---

## 라이선스 관리

사용하는 모든 오디오 에셋의 라이선스 기록:

| 에셋명 | 출처 | 라이선스 | 크레딧 필요 |
|--------|------|----------|-------------|
| *(사용 시 채움)* | | | |
