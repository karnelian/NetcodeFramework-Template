---
name: auto-playtest
description: "Unity 자동화 플레이테스트 실행/분석/밸런스 조정 절차. Coplay MCP로 Play→로그 분석→결과 비교. 플레이테스트, 자동화 테스트, 봇 돌려 시 참조."
---

# Unity 자동화 플레이테스트

## 트리거
플레이테스트, 자동화 테스트, 봇 돌려, playtest, auto test, 테스트 돌려

---

## 1. 실행 절차 (Coplay MCP)

### Play 시작

```
mcp__coplay-mcp__play_game
```

### 로그 폴링 (sleep 대신!)

**절대 sleep으로 무작정 기다리지 말 것.** 15초마다 게임 종료 확인:

```
1. sleep 15초
2. get_unity_logs(search_term="게임 종료", limit=1)
3. "게임 종료" 있으면 → break
4. 없으면 → STATUS 로그 확인 후 1로
```

### 결과 확인

```
Read playtest_report.txt
```

### Stop (이미 끝났어도 호출)

```
mcp__coplay-mcp__stop_game
```

---

## 2. 결과 분석

### 비교표 작성

매 플레이테스트마다 이전 결과와 비교:

```markdown
| 항목 | 이전 | 이번 |
|------|:---:|:---:|
| 결과 | PlayerDied | Victory |
| 웨이브 | 4 | 10 |
| 킬 | 100 | 671 |
| 생존 | 141초 | 400초 |
| 터렛 | 3대 | 6대 |
| 코어 | 1990/2000 | 2000/2000 |
```

### 에러 체크 (필수)

플레이테스트 후 반드시:

```
get_unity_logs(show_errors=true, show_warnings=true, show_logs=false, limit=10)
```

| 에러 | 원인 |
|------|------|
| `Failed to create agent` | NavMesh 밖 스폰 |
| `Manager 초기화 MISSING` | 싱글톤 리셋 실패 |
| `NullReferenceException` | 프리팹 연결 누락 |
| `Pool '...' 미등록` | PoolManager 미등록 |

### 알려진 문제 감지

| 증상 | 원인 | 해결 |
|------|------|------|
| 적 0마리인데 Wave 안 끝남 | 적 NavMesh 이탈/끼임 | Wave 타임아웃 확인 |
| 터렛 0대 (스크랩 충분) | Manager 미초기화 | SingletonReset 확인 |
| STATUS 로그 중복 | 봇 2개 인스턴스 | 싱글톤 가드 확인 |

---

## 3. 밸런스 판단 기준

| 항목 | 너무 쉬움 | 적절 | 너무 어려움 |
|------|:--------:|:----:|:---------:|
| 봇 생존 | Victory | Wave 4~6 사망 | Wave 1~2 사망 |
| 건물/터렛 | 6대+ | 2~3대 | 0대 |
| 자원 | 상한 도달 | 50~80% | 항상 0 |
| 능력 사용 | 10회+ | 2~4회 | 0~1회 |

**봇 기준 적절 난이도 = 실제 플레이어 +2~3 웨이브**

---

## 4. 밸런스 조정 사이클

```
1. 밸런스 값 변경 (에디터 스크립트 or SO)
2. 봇 플레이테스트 실행
3. 결과 비교표 작성
4. 목표 미달 → 1로 돌아감
5. 목표 달성 → 커밋
```

한 사이클에 **최소 3회 반복** 권장.
