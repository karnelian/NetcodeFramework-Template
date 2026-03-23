# Varco AI SFX Pipeline

> 전략 개요/트리밍 기준은 `SKILL.md` 참조.
> 이 문서는 Varco API 상세 스펙과 실행 코드를 다룬다.

---

## 인증

```
Base URL: https://openapi.ai.nc.com
Header: OPENAPI_KEY: {api-key}
```

API 키는 사용자에게 확인. 코드에 하드코딩 금지.

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

## TextToSound (텍스트→SFX 생성)

### 스펙
- 입력: 텍스트 **최대 200자** (한/영/일)
- 출력: **항상 10초**, 44.1kHz 16bit WAV (base64)
- num_sample: 1~3

### curl 예시
```bash
curl -X POST "https://openapi.ai.nc.com/sound/varco/v1/api/text2sound" \
  -H "Content-Type: application/json" \
  -H "OPENAPI_KEY: {api-key}" \
  -d '{"prompt":"metallic sword slash impact, sharp quick hit","num_sample":1}'
```

### 응답
```json
[{"audio": "UklGRi...base64..."}]
```

### Node.js 생성 + 저장
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

---

## ffmpeg 트리밍 (필수!)

Varco 출력은 **항상 10초**. 용도에 맞게 트리밍 + fade out:

```bash
# 타격음 (0.5초)
ffmpeg -y -i input.wav -t 0.5 -af "afade=t=out:st=0.3:d=0.2" output.wav

# 기계음 (1.5초)
ffmpeg -y -i input.wav -t 1.5 -af "afade=t=out:st=1.0:d=0.5" output.wav

# 환경음 (5초)
ffmpeg -y -i input.wav -t 5.0 -af "afade=t=out:st=4.0:d=1.0" output.wav
```

---

## Variation (변형 생성)

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

## Looping (루프 변환)

앰비언스/배경음을 끊김 없는 루프로 변환.

```bash
curl -X POST "https://openapi.ai.nc.com/sound/varco/v1/api/looping" \
  -H "OPENAPI_KEY: {api-key}" \
  -H "Content-Type: application/json" \
  -d '{"source":"base64..."}'
```

---

## Conversion (보이스 변환)

사람 목소리 → 몬스터/크리쳐 보이스 변환.

| 항목 | 값 |
|------|-----|
| source | 변환할 음성 (0.5~10초) |
| reference | 참조 몬스터 음성 (0.5~10초) |
| ratio | 0.0~2.0 (기본 1.0, 클수록 참조 스타일 강화) |
| enhance | true/false (소스 노이즈 제거) |

---

## Unity 할당 패턴

### 에디터 스크립트로 일괄 할당
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

## 주의사항

- **BGM 생성 불가** — Varco는 효과음/환경음 전용. BGM은 MiniMax 사용
- **항상 10초 출력** — 트리밍 없이 사용하면 불필요한 무음 포함
- **Node.js 사용** — python3 미설치 환경 대비
- **API 키 보안** — 환경변수 또는 사용자에게 확인
