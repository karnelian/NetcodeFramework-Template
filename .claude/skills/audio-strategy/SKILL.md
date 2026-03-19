---
name: audio-strategy
description: "게임 오디오 전략 통합 가이드. 오디오 카테고리, BGM/SFX/UI/Ambient/Voice 리소스 확보, Varco AI SFX 파이프라인(생성→트리밍→변형→할당), MiniMax BGM 생성, Unity AudioSystem 연동, 포맷 규칙. 사운드/오디오/효과음/BGM/SFX 생성, Varco, MiniMax, 오디오 구현 시 반드시 참조."
---

# 게임 오디오 전략

> **AudioManager API는 `netcode-framework` 스킬 (references/systems-api.md) 참조.**

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/varco-sfx-pipeline.md` | Varco AI API 전체 스펙, Node.js 코드, ffmpeg 트리밍, Variation, Unity 할당 패턴 | SFX 생성/변형 작업 시 |
| `references/ai-audio-workflow.md` | MiniMax Music BGM 생성, 오디오 요구사항 템플릿, Unity 임포트 설정, 검증 체크리스트 | BGM 생성, 오디오 전체 워크플로우 진행 시 |

---

## 오디오 카테고리

| 카테고리 | 설명 | 예시 |
|----------|------|------|
| BGM | 배경 음악 | 메인메뉴, 로비, 전투, 보스전 |
| SFX | 효과음 | 타격, 발소리, 환경음, 스킬 |
| UI Sound | 인터페이스 사운드 | 버튼 클릭, 팝업, 알림, 탭 전환 |
| Ambient | 환경 사운드 | 바람, 물소리, 숲, 도시 소음 |
| Voice | 보이스/대사 | NPC 대사, 캐릭터 음성 |

---

## 리소스 확보 전략

| 용도 | 도구 | 비용 |
|------|------|------|
| **SFX** | Varco AI Sound API | 크레딧 기반 (40/호출) |
| **BGM** | MiniMax Music 2.5+ | $0.15/곡 |
| **BGM (대안)** | Suno / Udio | 구독 (상업 라이선스 확인) |
| **무료 리소스** | Freesound.org, OpenGameArt, Incompetech | CC 라이선스 |
| **Coplay MCP** | `generate_sfx`, `generate_music`, `generate_tts` | API 키 필요 |

---

## Varco AI SFX — 핵심 요약

> **상세 코드/API 스펙은 `references/varco-sfx-pipeline.md` 참조.**

### 인증
```
Base URL: https://openapi.ai.nc.com
Header: OPENAPI_KEY: {api-key}
```
API 키는 사용자에게 확인. 코드에 하드코딩 금지.

### API 목록

| API | 용도 | 크레딧 |
|-----|------|:------:|
| TextToSound | 텍스트 → SFX 생성 | 40 |
| Variation | 기존 SFX 변형 | 50 |
| MonoToStereo | 모노 → 스테레오 | 10 |
| Looping | 무한 루프 변환 | 50 |
| Conversion | 몬스터 보이스 변환 | 25 |
| Enhance | 노이즈 제거 | 25 |

### 전체 워크플로우
```
1. TextToSound로 기본 SFX 생성 (프롬프트 200자 이내)
2. ffmpeg로 트리밍 (항상 10초 출력 → 용도별 길이로)
3. Variation으로 변형 3~5개 생성 (반복 방지)
4. 변형도 동일 길이로 트리밍
5. Unity SFX 폴더에 배치
6. 에디터 스크립트로 프리팹 할당
7. SFXUtil.PickRandom으로 랜덤 재생
```

### 트리밍 기준표

| SFX 유형 | 권장 길이 | fade out |
|----------|:---------:|:--------:|
| 타격/충돌 | 0.3~0.5초 | 0.2초 |
| 기계/건설 | 1.0~1.5초 | 0.5초 |
| UI 차임 | 0.5~1.0초 | 0.3초 |
| 보상/획득 | 1.5~2.0초 | 0.5초 |
| 폭발 | 1.0~2.0초 | 0.5초 |
| 환경음 | 5.0~10.0초 | 1.0초 |
| 보스 등장/사망 | 2.0~3.0초 | 1.0초 |

### 프롬프트 작성 팁

| 좋은 예 | 나쁜 예 |
|---------|---------|
| `metallic sword slash impact, sharp quick hit` | `sword` (너무 짧음) |
| `heavy explosion with debris and echo in cave` | `큰 폭발 소리를 만들어주세요` (미사여구) |
| `축축한 동굴 안에서 물방울이 떨어지며 울리는 소리` | `동굴 소리` (모호함) |

**핵심**: 사운드 특성 + 질감 + 환경을 구체적으로. 200자 이내. 한/영 모두 가능.

---

## 프레임워크 연동

AudioSystem은 프레임워크에 구현됨:
- `AudioManager`: MonoSingleton 기반 매니저
- `AudioSystemSettings`: ScriptableObject 설정
- `BGMController` / `SFXController`: 재생 컨트롤러
- `MainAudioMixer.mixer`: Master, BGM, SFX 그룹

### 재생 API
```csharp
AudioManager.instance.PlaySFX(clip);
AudioManager.instance.PlaySFX3D(clip, position);
AudioManager.instance.PlayBGM(clip);

// HasInstance 체크 필수
if (AudioManager.HasInstance) AudioManager.instance.PlaySFX(clip);
```

### 랜덤 재생 유틸
```csharp
public static class SFXUtil
{
    public static AudioClip PickRandom(AudioClip main, AudioClip[] variants)
    {
        if (variants == null || variants.Length == 0) return main;
        var index = Random.Range(-1, variants.Length);
        return index < 0 ? main : variants[index];
    }
}

// 사용
AudioManager.instance.PlaySFX3D(
    SFXUtil.PickRandom(_hitClip, _hitClipVariants), position);
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

## 오디오 폴더 구조

```
Assets/NetcodeFramework/Addressables/Manager/AudioManager/Sound/
├── BGM/
├── SFX/
│   ├── Combat/
│   ├── Environment/
│   └── Character/
├── UI/
└── Ambient/
```

게임별: `Assets/_Game/{GameName}/Audio/` 하위에 배치.

---

## 라이선스 관리

| 출처 | 라이선스 | 크레딧 필요 |
|------|----------|-------------|
| Varco AI (NCSOFT) | API 이용약관 | 확인 필요 |
| MiniMax Music | API 이용약관 | 확인 필요 |
| Freesound.org | CC BY / CC0 | 라이선스별 상이 |
| 프로젝트 자체 생성 | - | - |
