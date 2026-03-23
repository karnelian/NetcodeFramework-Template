# AI 오디오 생성 워크플로우

> MiniMax Music 2.5+ (BGM) + VARCO Sound (SFX) 를 활용한 게임 오디오 생성 가이드.

---

## 전체 흐름

```
1단계: 오디오 요구사항 정의 (기획서/GDD 기반)
  → 2단계: BGM 생성 (MiniMax Music 2.5+)
  → 3단계: SFX 생성 (VARCO Sound)
  → 4단계: Unity 임포트 & 설정
  → 5단계: 검증 & 반복
```

---

## 1단계: 오디오 요구사항 정의

기획서/GDD에서 오디오 목록을 추출한다:

```markdown
## 오디오 요구사항

### BGM
| ID | 장면 | 분위기 | 장르 | 길이 | 루프 |
|----|------|--------|------|------|------|
| BGM_01 | 메인 메뉴 | 웅장, 기대감 | 오케스트라 | 3~5분 | O |
| BGM_02 | 전투 | 긴장, 격렬 | 일렉트로닉+오케스트라 | 3~5분 | O |
| BGM_03 | 보스전 | 압도적, 절박 | 헤비메탈+오케스트라 | 3~5분 | O |

### SFX
| ID | 카테고리 | 설명 | 길이 |
|----|----------|------|------|
| SFX_01 | Combat | 총 발사음 (라이플) | ~0.5초 |
| SFX_02 | Combat | 피격음 (육체) | ~0.3초 |
| SFX_03 | Environment | 문 열기 | ~1초 |
| SFX_04 | Character | 발걸음 (콘크리트) | ~0.3초 |

### UI Sound
| ID | 액션 | 설명 |
|----|------|------|
| UI_01 | Click | 버튼 클릭 |
| UI_02 | Hover | 버튼 호버 |
| UI_03 | Popup | 팝업 열림 |
| UI_04 | Close | 팝업 닫힘 |
```

---

## 2단계: BGM 생성 — MiniMax Music 2.5+

### API 사양

| 항목 | 값 |
|------|---|
| Endpoint | `POST https://api.minimax.io/v1/music_generation` |
| 인증 | `Authorization: Bearer {API_KEY}` |
| 모델 | `music-2.5+` (인스트루멘탈 지원) |
| 가격 | $0.15/곡 (최대 5분) |
| 출력 | mp3, wav, pcm (hex 또는 URL) |

### 요청 형식

```json
{
  "model": "music-2.5+",
  "prompt": "Epic orchestral battle music with intense percussion, brass fanfares, and dramatic strings. High energy, fast tempo, suitable for a zombie survival shooter boss fight.",
  "is_instrumental": true,
  "audio_setting": {
    "sample_rate": 44100,
    "bitrate": 256000,
    "format": "mp3"
  },
  "output_format": "url"
}
```

### 구조 태그 (보컬 포함 음악 시)

가사(`lyrics`)에 사용 가능한 14개 구조 태그:

```
[Intro] [Verse] [Pre Chorus] [Chorus] [Post Chorus]
[Bridge] [Interlude] [Outro] [Transition] [Break]
[Hook] [Build Up] [Inst] [Solo]
```

### 인스트루멘탈 BGM 프롬프트 작성법

```
{장르} {분위기} music for {장면}.
{악기 설명}.
{템포/에너지 레벨}.
{추가 지시사항}.
```

**예시:**

| 장면 | 프롬프트 |
|------|---------|
| 메인 메뉴 | `Cinematic orchestral theme with sweeping strings, gentle piano, and building brass. Hopeful and epic mood. Medium tempo. Suitable for a game main menu.` |
| 전투 | `Intense electronic-orchestral hybrid battle music. Heavy percussion, distorted synths, urgent brass stabs. Fast tempo, high energy. Loop-friendly.` |
| 보스전 | `Dark aggressive heavy metal with orchestral elements. Pounding double bass drums, shredding guitars, ominous choir. Very fast tempo, maximum intensity.` |
| 로비/평화 | `Calm ambient music with soft piano, acoustic guitar, and gentle pads. Peaceful and relaxed. Slow tempo. Suitable for a game lobby or safe zone.` |
| 호러 | `Eerie dark ambient with unsettling drones, distant whispers, and sparse dissonant piano notes. Creepy atmosphere. Very slow, minimal.` |

### 응답 처리

```python
# output_format: "url" 사용 시
response = requests.post(endpoint, headers=headers, json=payload)
data = response.json()

if data["base_resp"]["status_code"] == 0:
    audio_url = data["data"]["audio"]  # 24시간 유효 URL
    # 다운로드
    audio = requests.get(audio_url)
    with open("BGM_Battle_01.mp3", "wb") as f:
        f.write(audio.content)
```

### 생성 팁

- **루프용 BGM**: 프롬프트에 `"loop-friendly"` 또는 `"seamless loop"` 추가
- **분위기 전환**: 같은 곡의 변형을 프롬프트만 살짝 바꿔서 여러 번 생성
- **길이 제어**: 최대 5분, 게임 BGM은 2~4분이 적당 (루프)
- **여러 후보 생성**: 같은 프롬프트로 3~5개 생성 → 베스트 선택

---

## 3단계: SFX 생성 — VARCO Sound

### 방법 A: VARCO Sound (웹/API)

**웹 UI 사용:**
1. https://sound.varco.ai 접속
2. 텍스트 입력: `"rifle gunshot, single shot, sharp crack, indoor"`
3. 생성 → 멀티트랙으로 분리됨 → 필요한 트랙만 다운로드

**주요 기능:**
| 기능 | 설명 | 게임 활용 |
|------|------|----------|
| Text → Sound | 텍스트로 SFX 생성 | 모든 효과음 |
| Image → Sound | 이미지 분석 → 사운드 | 씬 스크린샷으로 환경음 생성 |
| Video → Sound | 영상 분석 → 사운드 | 게임플레이 영상에 자동 사운드 배치 |
| Loop Generation | 루프 사운드 생성 | 환경음, 배경 소음 |
| Variation | 유사한 변형 생성 | 발걸음, 타격음 등 랜덤 변형 |
| Multitrack | 트랙 분리 출력 | 개별 레이어 편집 |

**SFX 프롬프트 작성법:**
```
{물체/행위}, {세부사항}, {환경}, {거리감}
```

| 카테고리 | 프롬프트 예시 |
|----------|-------------|
| 총소리 | `rifle gunshot, single shot, sharp crack, indoor concrete room` |
| 발걸음 | `footstep on concrete, military boot, walking pace` |
| 타격 | `melee hit, blunt weapon on flesh, heavy impact` |
| 문 | `heavy metal door opening, creaking, echo in hallway` |
| 폭발 | `explosion, medium distance, debris falling, outdoor` |
| 환경 | `city night ambience, distant traffic, occasional siren` |
| UI 클릭 | `soft UI click, digital, subtle, clean` |
| UI 호버 | `gentle UI hover sound, light whoosh, soft` |

### SFX 변형 생성 전략

같은 소리의 변형이 필요할 때 (발걸음, 타격 등):

```
SFX_Footstep_Concrete_01.wav  ← 기본
SFX_Footstep_Concrete_02.wav  ← VARCO Variation으로 생성
SFX_Footstep_Concrete_03.wav  ← 프롬프트 살짝 변경
SFX_Footstep_Concrete_04.wav
```

→ 게임에서 랜덤 재생하여 반복감 감소

---

## 4단계: Unity 임포트 & 설정

### 파일 배치

```
Assets/.../AudioManager/Sound/
├── BGM/
│   ├── BGM_Menu_Epic_01.ogg          ← MiniMax에서 mp3 → ogg 변환
│   ├── BGM_Battle_Intense_01.ogg
│   └── BGM_Boss_Dark_01.ogg
├── SFX/
│   ├── Combat/
│   │   ├── SFX_Combat_Gunshot_01.wav  ← VARCO Sound
│   │   ├── SFX_Combat_Gunshot_02.wav
│   │   └── SFX_Combat_Hit_01.wav
│   ├── Environment/
│   │   └── SFX_Env_DoorOpen_01.wav
│   └── Character/
│       ├── SFX_Char_Footstep_01.wav
│       ├── SFX_Char_Footstep_02.wav
│       └── SFX_Char_Footstep_03.wav
├── UI/
│   ├── UI_Click_01.wav
│   ├── UI_Hover_01.wav
│   └── UI_Popup_01.wav
└── Ambient/
    └── AMB_City_Night_01.ogg
```

### Unity AudioClip Import 설정

| 용도 | Force To Mono | Load Type | Compression |
|------|--------------|-----------|-------------|
| BGM | false (스테레오) | Streaming | Vorbis, Quality 70% |
| SFX 짧은 (<2초) | true | Decompress On Load | PCM |
| SFX 긴 (>2초) | context별 | Compressed In Memory | Vorbis, Quality 70% |
| UI Sound | true | Decompress On Load | PCM |
| Ambient | false | Streaming | Vorbis, Quality 60% |

### mp3 → ogg 변환

MiniMax 출력이 mp3인 경우 Unity에서 직접 임포트 가능하지만, ogg가 권장:
```bash
# ffmpeg 사용 (있는 경우)
ffmpeg -i BGM_Battle_01.mp3 -c:a libvorbis -q:a 7 BGM_Battle_01.ogg
```
또는 Unity가 자동 변환하므로 mp3 그대로 임포트해도 무방.

---

## 5단계: 검증 & 반복

### 검증 체크리스트

```
□ BGM이 장면 분위기와 일치하는가?
□ BGM 루프 시 끊김이 없는가? (시작/끝 연결)
□ SFX가 시각 효과와 타이밍이 맞는가?
□ SFX 볼륨이 BGM과 균형 잡혀있는가?
□ UI 사운드가 즉각적으로 반응하는가? (지연 없음)
□ 환경음이 루프 시 자연스러운가?
□ 여러 SFX 동시 재생 시 깨지지 않는가?
□ 모바일 타겟 시 메모리 사용량 적절한가?
```

### 반복 생성 기준

- BGM: 3~5개 후보 생성 → 팀 투표 또는 플레이테스트
- SFX: 기본 1개 + Variation 2~3개
- 마음에 안 들면 프롬프트 수정 후 재생성 (MiniMax $0.15/곡이라 부담 적음)

---

## API 키 관리

```
# .env 또는 프로젝트 설정에 보관 (git에 올리지 않음)
MINIMAX_API_KEY=your_key_here

# MiniMax API 키 발급: https://platform.minimax.io → Account → API Keys
# VARCO Sound: https://sound.varco.ai 에서 계정 생성 (베타 기간 무료)
```

---

## 빠른 참조

### MiniMax Music 2.5+ (BGM)

```bash
curl -X POST https://api.minimax.io/v1/music_generation \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-2.5+",
    "prompt": "Epic orchestral battle music, intense, fast tempo",
    "is_instrumental": true,
    "output_format": "url",
    "audio_setting": {
      "sample_rate": 44100,
      "bitrate": 256000,
      "format": "mp3"
    }
  }'
```

