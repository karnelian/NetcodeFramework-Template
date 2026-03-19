---
name: varco-sfx-pipeline
description: "Varco AI Sound API로 SFX 생성, 트리밍, 변형, Unity 할당까지의 전체 파이프라인. 효과음 생성 시 반드시 참조."
---

# Varco AI SFX Pipeline

## 트리거
SFX 생성, 효과음 만들기, Varco, 사운드 생성, sound effect, generate audio

## 인증

```
Base URL: https://openapi.ai.nc.com
Header: OPENAPI_KEY: {api-key}
```

API 키는 사용자에게 확인. 코드에 하드코딩하지 말 것.

---

## API 엔드포인트

| API | Method | Path | 용도 | 크레딧 |
|-----|--------|------|------|:------:|
| TextToSound | POST | `/sound/varco/v1/api/text2sound` | 텍스트→SFX | 40 |
| Variation | POST | `/sound/varco/v1/api/variation` | 기존 SFX 변형 | 50 |
| MonoToStereo | POST | `/sound/varco/v1/api/mono2stereo` | 모노→스테레오 | 10 |
| Looping | POST | `/sound/varco/v1/api/looping` | 무한 루프 | 50 |
| Conversion | POST | `/sound/varco/v1/api/conversion` | 보이스 변환 | 25 |
| Enhance | POST | `/sound/varco/v1/api/enhance` | 노이즈 제거 | 25 |

---

## Step 1: TextToSound (텍스트→SFX 생성)

### 스펙
- 입력: 텍스트 **최대 200자** (한/영/일)
- 출력: **항상 10초**, 44.1kHz 16bit WAV (base64)
- num_sample: 1~3

### Node.js 코드

```javascript
const https = require('https');
const fs = require('fs');

function genSFX(prompt, outFile, apiKey) {
  return new Promise(r => {
    const d = JSON.stringify({prompt, num_sample: 1});
    const req = https.request({
      hostname: 'openapi.ai.nc.com',
      path: '/sound/varco/v1/api/text2sound',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'OPENAPI_KEY': apiKey,
        'Content-Length': Buffer.byteLength(d)
      }
    }, res => {
      let b = '';
      res.on('data', c => b += c);
      res.on('end', () => {
        const arr = JSON.parse(b);
        if (Array.isArray(arr) && arr[0]) {
          const buf = Buffer.from(arr[0].audio, 'base64');
          fs.writeFileSync(outFile, buf);
          console.log(`${outFile}: ${buf.length}B`);
        }
        r();
      });
    });
    req.write(d); req.end();
  });
}
```

### 프롬프트 작성 팁

| 좋은 예 | 나쁜 예 |
|---------|---------|
| `metallic sword slash impact, sharp quick hit` | `sword` |
| `heavy explosion with debris and echo in cave` | `폭발 소리 만들어주세요` |
| `축축한 동굴 안에서 물방울이 떨어지며 울리는 소리` | `동굴 소리` |

핵심: **사운드 특성 + 질감 + 환경**을 구체적으로. 200자 이내.

---

## Step 2: ffmpeg 트리밍 (필수!)

Varco 출력은 **항상 10초**. 용도에 맞게 트리밍 + fade out:

```bash
# 타격음 (0.5초)
ffmpeg -y -i input.wav -t 0.5 -af "afade=t=out:st=0.3:d=0.2" output.wav

# 기계음 (1.5초)
ffmpeg -y -i input.wav -t 1.5 -af "afade=t=out:st=1.0:d=0.5" output.wav

# 환경음 (5초)
ffmpeg -y -i input.wav -t 5.0 -af "afade=t=out:st=4.0:d=1.0" output.wav
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

---

## Step 3: Variation (변형 생성)

반복 재생 단조로움 해결. 원본 1개 → 3~5개 변형.

### 스펙
- 입력: base64 인코딩 오디오 (0.5~10초)
- 출력: **항상 10초** (트리밍 필요)
- strength: 0.0~3.0 (기본 1.0, 0.7~0.8 권장)
- num_sample: 1~5

### Node.js 코드

```javascript
function vary(srcFile, outPrefix, apiKey, strength = 0.8) {
  return new Promise(r => {
    const src = fs.readFileSync(srcFile);
    const d = JSON.stringify({
      source: src.toString('base64'),
      num_sample: 3,
      strength
    });
    const req = https.request({
      hostname: 'openapi.ai.nc.com',
      path: '/sound/varco/v1/api/variation',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'OPENAPI_KEY': apiKey,
        'Content-Length': Buffer.byteLength(d)
      }
    }, res => {
      let b = '';
      res.on('data', c => b += c);
      res.on('end', () => {
        const arr = JSON.parse(b);
        arr.forEach((item, i) => {
          const buf = Buffer.from(item.audio, 'base64');
          const out = `${outPrefix}_v${i+1}.wav`;
          fs.writeFileSync(out, buf);
          console.log(`${out}: ${buf.length}B`);
        });
        r();
      });
    });
    req.write(d); req.end();
  });
}
```

변형 후 **원본과 동일한 길이로 트리밍** 필수.

---

## Step 4: Unity 할당

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
```

### 사용 패턴

```csharp
[SerializeField] private AudioClip _hitClip;
[SerializeField] private AudioClip[] _hitClipVariants;

// 재생 시:
AudioManager.instance.PlaySFX3D(
    SFXUtil.PickRandom(_hitClip, _hitClipVariants), position);
```

### 에디터 할당 스크립트 패턴

```csharp
// unity-so-builder 스킬 참조
var so = new SerializedObject(component);
so.FindProperty("_hitClip").objectReferenceValue = clip;
// variants 배열:
var prop = so.FindProperty("_hitClipVariants");
prop.arraySize = clips.Length;
for (var i = 0; i < clips.Length; i++)
    prop.GetArrayElementAtIndex(i).objectReferenceValue = clips[i];
so.ApplyModifiedProperties();
```

---

## 전체 워크플로우

```
1. TextToSound로 기본 SFX 생성
2. ffmpeg로 트리밍 (용도별 길이)
3. Variation으로 변형 3~5개 생성
4. 변형도 동일 길이로 트리밍
5. Unity SFX 폴더에 배치
6. AssignAudioClips 에디터 스크립트로 프리팹 할당
7. SFXUtil.PickRandom으로 랜덤 재생
```

---

## 주의사항

- **BGM 생성 불가** — Varco는 효과음/환경음 전용. 음악은 Suno/Udio 사용
- **항상 10초 출력** — 트리밍 없이 사용하면 불필요한 무음 포함
- **python 미설치 환경** — Node.js로 base64 디코딩 (python3 없을 수 있음)
- **API 키 보안** — 코드에 하드코딩 금지, 환경변수 또는 사용자에게 확인
