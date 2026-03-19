---
name: audio-strategy
description: "사운드/오디오 리소스 확보, 구현, 관리 시 참조. BGM, SFX, UI 사운드, 보이스 전략 포함. Varco AI API 연동 가이드 포함."
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

## Varco AI Sound API

> **상세 가이드는 `varco-sfx-pipeline` 스킬 참조.**
> SFX 생성, 트리밍, 변형, Unity 할당 전체 파이프라인이 별도 스킬로 분리됨.
> BGM 생성은 지원하지 않음 — 효과음/환경음 전용.

### 인증
```
Header: OPENAPI_KEY: {api-key}
Base URL: https://openapi.ai.nc.com
```

### API 목록

| API | 엔드포인트 | 용도 | 크레딧 |
|-----|-----------|------|:------:|
| **TextToSound** | `POST /sound/varco/v1/api/text2sound` | 텍스트 → SFX 생성 | 40/호출 |
| **Variation** | `POST /sound/varco/v1/api/variation` | 기존 SFX 변형 생성 | 50/호출 |
| **MonoToStereo** | `POST /sound/varco/v1/api/mono2stereo` | 모노 → 스테레오 | 10/호출 |
| **Looping** | `POST /sound/varco/v1/api/looping` | 무한 루프 변환 | 50/호출 |
| **Conversion** | `POST /sound/varco/v1/api/conversion` | 몬스터 보이스 변환 | 25/호출 |
| **Enhance** | `POST /sound/varco/v1/api/enhance` | 노이즈 제거 | 25/호출 |

---

### TextToSound (핵심 API)

텍스트 프롬프트로 SFX 생성. **출력은 항상 10초** → ffmpeg 트리밍 필수.

#### 스펙
| 항목 | 값 |
|------|-----|
| 입력 | 텍스트 최대 200자 (한/영/일 지원) |
| 출력 | 44.1kHz, 16bit WAV, **항상 10초** |
| num_sample | 1~3 (기본 1) |

#### curl 예시
```bash
curl -X POST "https://openapi.ai.nc.com/sound/varco/v1/api/text2sound" \
  -H "Content-Type: application/json" \
  -H "OPENAPI_KEY: {api-key}" \
  -d '{"prompt":"metallic sword slash impact, sharp quick hit","num_sample":1}'
```

#### 응답 (base64 WAV)
```json
[{"audio": "UklGRi...base64..."}]
```

#### Node.js 생성 + 저장 코드
```javascript
const https = require('https');
const fs = require('fs');

function genSFX(prompt, outFile) {
  return new Promise((resolve) => {
    const data = JSON.stringify({prompt, num_sample: 1});
    const req = https.request({
      hostname: 'openapi.ai.nc.com',
      path: '/sound/varco/v1/api/text2sound',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'OPENAPI_KEY': process.env.VARCO_API_KEY,
        'Content-Length': Buffer.byteLength(data)
      }
    }, res => {
      let body = '';
      res.on('data', d => body += d);
      res.on('end', () => {
        const arr = JSON.parse(body);
        if (Array.isArray(arr) && arr.length > 0) {
          const buf = Buffer.from(arr[0].audio, 'base64');
          fs.writeFileSync(outFile, buf);
          console.log(`${outFile}: ${buf.length} bytes`);
        }
        resolve();
      });
    });
    req.write(data);
    req.end();
  });
}
```

#### ffmpeg 트리밍 (필수!)
Varco 출력은 항상 10초. 용도에 맞게 트리밍 + fade out 적용:
```bash
# SFX 타격음 (0.5초)
ffmpeg -y -i input.wav -t 0.5 -af "afade=t=out:st=0.3:d=0.2" output.wav

# SFX 기계음 (1.5초)
ffmpeg -y -i input.wav -t 1.5 -af "afade=t=out:st=1.0:d=0.5" output.wav

# 환경음 (5초)
ffmpeg -y -i input.wav -t 5.0 -af "afade=t=out:st=4.0:d=1.0" output.wav
```

#### 트리밍 기준
| SFX 유형 | 권장 길이 | fade out |
|----------|:---------:|:--------:|
| 타격/충돌 | 0.3~0.5초 | 0.2초 |
| 기계/건설 | 1.0~1.5초 | 0.5초 |
| UI 차임 | 0.5~1.0초 | 0.3초 |
| 보상/획득 | 1.5~2.0초 | 0.5초 |
| 폭발 | 1.0~2.0초 | 0.5초 |
| 환경음 | 5.0~10.0초 | 1.0초 |

---

### Variation (SFX 변형)

기존 SFX에서 톤 유지하면서 파형 변형. 반복 재생 시 단조로움 해결.

#### 스펙
| 항목 | 값 |
|------|-----|
| 입력 | base64 인코딩 오디오 (0.5~10초) |
| 출력 | 44.1kHz, 16bit WAV, **항상 10초** |
| strength | 0.0~3.0 (기본 1.0, 클수록 큰 변화) |
| num_sample | 1~5 |

#### 활용 예시
```bash
# 총소리 1개 → 5개 변형 생성 (랜덤 재생용)
curl -X POST "https://openapi.ai.nc.com/sound/varco/v1/api/variation" \
  -H "OPENAPI_KEY: {api-key}" \
  -H "Content-Type: application/json" \
  -d '{"source":"base64...", "num_sample":5, "strength":0.8}'
```

---

### Looping (루프 변환)

앰비언스/배경음을 끊김 없는 루프로 변환.

#### 활용
```bash
# 환경음을 무한 루프로 변환
curl -X POST "https://openapi.ai.nc.com/sound/varco/v1/api/looping" \
  -H "OPENAPI_KEY: {api-key}" \
  -H "Content-Type: application/json" \
  -d '{"source":"base64..."}'
```

---

### Conversion (보이스 변환)

사람 목소리 → 몬스터/크리쳐 보이스 변환.

#### 스펙
| 항목 | 값 |
|------|-----|
| source | 변환할 음성 (0.5~10초) |
| reference | 참조 몬스터 음성 (0.5~10초) |
| ratio | 0.0~2.0 (기본 1.0, 클수록 참조 스타일 강화) |
| enhance | true/false (소스 노이즈 제거) |

---

### 추천 워크플로우

#### SFX 라이브러리 구축
```
1. TextToSound로 기본 SFX 생성 (프롬프트 200자 이내)
2. ffmpeg로 트리밍 (용도별 적절 길이)
3. Variation으로 변형 3~5개 생성 (반복 방지)
4. MonoToStereo로 공간감 추가 (필요 시)
5. Unity 프로젝트에 배치
```

#### 환경음/앰비언스 제작
```
1. TextToSound로 환경음 생성 ("비 내리는 숲", "공장 기계 소음" 등)
2. MonoToStereo로 스테레오 변환
3. Looping으로 무한 루프 처리
4. Unity에서 AudioSource.loop = true로 재생
```

#### 몬스터 보이스 제작
```
1. 직접 녹음 또는 TextToSound로 기본 음성 소스 확보
2. Conversion으로 몬스터 텍스처 적용 (reference 오디오 필요)
3. Variation으로 공격/피격/사망 등 다양한 상황 버전 생성
4. Enhance로 노이즈 제거 (필요 시)
```

---

### 프롬프트 작성 팁

| 좋은 예 | 나쁜 예 |
|---------|---------|
| `metallic sword slash impact, sharp quick hit` | `sword` (너무 짧음) |
| `heavy explosion with debris and echo in cave` | `큰 폭발 소리를 만들어주세요` (미사여구) |
| `축축한 동굴 안에서 물방울이 떨어지며 울리는 소리` | `동굴 소리` (모호함) |
| `mechanical turret construction, hydraulic hiss` | `turret building sound effect for my game` (불필요한 설명) |

**핵심**: 사운드 특성 + 질감 + 환경을 구체적으로. 200자 이내. 한/영 모두 가능.

---

## 무료/저가 리소스 확보처 (BGM용)

Varco AI는 BGM 생성 불가. BGM은 아래에서 확보:

### BGM
- Freesound.org: 무료 CC 라이선스 사운드
- OpenGameArt.org: 게임용 무료 음악/SFX
- Incompetech (Kevin MacLeod): CC BY 무료 BGM
- Unity Asset Store: "Free Music" 검색
- Suno / Udio: AI 음악 생성 (상업 라이선스 확인 필수)

### Coplay MCP 도구 (API 키 필요)
- `generate_sfx`: AI SFX 생성
- `generate_music`: AI BGM 생성
- `generate_tts`: AI TTS 보이스 생성

---

## 기존 프레임워크 연동

AudioSystem은 이미 프레임워크에 구현되어 있음:
- `AudioManager`: MonoSingleton 기반 매니저
- `AudioSystemSettings`: ScriptableObject 설정
- `BGMController` / `SFXController`: 재생 컨트롤러
- `MainAudioMixer.mixer`: Master, BGM, SFX 그룹

### 재생 API
```csharp
// SFX 재생
AudioManager.instance.PlaySFX(clip);
AudioManager.instance.PlaySFX3D(clip, position);

// BGM 재생
AudioManager.instance.PlayBGM(clip);
```

### 오디오 할당 에디터 스크립트
```
ScrapMagnet > Audio > Assign Audio Clips
```
기존 SFX를 적 프리팹(hit/death), 플레이어(melee/burst/collect/eject), 터렛(fire)에 일괄 할당.

---

## 오디오 폴더 구조

```
Assets/_Game/Scrap_Magnet/Audio/
├── BGM/                    # 배경 음악 (.ogg)
├── SFX/                    # 효과음 (.wav)
│   ├── sfx_combat_*.wav    # 전투 (melee_hit, burst, turret_fire)
│   ├── sfx_collect_*.wav   # 수집 (magnet, attach, eject)
│   ├── sfx_enemy_*.wav     # 적 (hit, death)
│   ├── sfx_player_*.wav    # 플레이어 (hit)
│   ├── sfx_core_*.wav      # 코어 (hit)
│   ├── sfx_system_*.wav    # 시스템 (wave_start, gameover)
│   ├── sfx_turret_*.wav    # 터렛 (place)
│   ├── sfx_upgrade_*.wav   # 업그레이드 (select)
│   ├── sfx_crystal_*.wav   # 크리스탈 (earn)
│   └── sfx_ui_*.wav        # UI (click)
└── Voice/                  # 보이스 (향후)
```

---

## 포맷 규칙

| 용도 | 포맷 | Unity 압축 설정 |
|------|------|-----------------|
| BGM | .ogg | Streaming (메모리 절약) |
| SFX (짧은 ~2초) | .wav | Decompress On Load |
| SFX (긴 2초+) | .ogg | Compressed In Memory |
| UI Sound | .wav | Decompress On Load |

---

## 현재 SFX 목록 (16개)

| 파일 | 출처 | 길이 | 용도 |
|------|------|:----:|------|
| sfx_combat_burst.wav | 기존 | - | 스크랩 버스트 |
| sfx_combat_turret_fire.wav | 기존 | - | 터렛 발사 |
| sfx_combat_melee_hit.wav | **Varco AI** | 0.5초 | 근접 타격 |
| sfx_collect_magnet.wav | 기존 | - | 자석 수집 |
| sfx_collect_attach.wav | 기존 | - | 스크랩 부착 |
| sfx_collect_eject.wav | 기존 | - | 스크랩 사출 |
| sfx_ui_click.wav | 기존 | - | UI 클릭 |
| sfx_enemy_hit.wav | 기존 | - | 적 피격 |
| sfx_enemy_death.wav | 기존 | - | 적 사망 |
| sfx_player_hit.wav | 기존 | - | 플레이어 피격 |
| sfx_core_hit.wav | 기존 | - | 코어 피격 |
| sfx_system_wave_start.wav | 기존 | - | 웨이브 시작 |
| sfx_system_gameover.wav | 기존 | - | 게임오버 |
| sfx_turret_place.wav | **Varco AI** | 1.5초 | 터렛 배치 |
| sfx_upgrade_select.wav | **Varco AI** | 1.0초 | 업그레이드 선택 |
| sfx_crystal_earn.wav | **Varco AI** | 2.0초 | 크리스탈 획득 |

---

## 라이선스 관리

| 에셋명 | 출처 | 라이선스 | 크레딧 필요 |
|--------|------|----------|-------------|
| sfx_combat_melee_hit 외 3개 | VARCO API Platform (NCSOFT) | API 이용약관 | 확인 필요 |
| 기존 SFX 12개 | 프로젝트 자체 생성 | - | - |
