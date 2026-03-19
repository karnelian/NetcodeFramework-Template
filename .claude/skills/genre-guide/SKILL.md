---
name: genre-guide
description: "게임 장르별 필요한 System 서브모듈 조합 가이드. 새 프로젝트 시작 시 '로그라이크 만들 거야' 같은 말에 자동 트리거."
---

# Genre System Guide

## 트리거
장르, 로그라이크, 뱀서, 타워디펜스, FPS, RPG, 핵슬, 카드게임, 서바이벌, 새 게임, 게임 만들기, genre, roguelike, tower defense, survivor

## 서브모듈 저장소

| 시스템 | Repo | 파일 수 | 역할 |
|--------|------|:------:|------|
| **Core** | `NetcodeFramework-Core` | 274 | EventBus, Pool, Audio, UI, Scene, Camera, TimeScale |
| **Combat** | `NetcodeFramework-System-Combat` | 12 | DamageManager, Projectile, StatusEffect, IDamageable |
| **Spawn** | `NetcodeFramework-System-Spawn` | 5 | SpawnManager (Wave), ISpawnable |
| **Drop** | `NetcodeFramework-System-Drop` | 2 | DropTableManager, 가중치 랜덤 드롭 |
| **Stat** | `NetcodeFramework-System-Stat` | 2 | StatModifier, 시간 제한 모디파이어 자동 만료 |

---

## 장르별 시스템 조합

### 타워디펜스
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Spawn.git Assets/GenreFramework/System-Spawn
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 터렛 배치, 경로 시스템, 건설 비용

### 로그라이크 / 뱀서라이크
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Spawn.git Assets/GenreFramework/System-Spawn
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 업그레이드 카드 UI, 메타 프로그레션, 절차적 맵

### 핵앤슬래시 / ARPG
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 인벤토리, 장비, 스킬 시스템, 퀘스트

### FPS / TPS 슈터
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 무기 시스템, 탄약, 조준, 리코일

### RTS
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Spawn.git Assets/GenreFramework/System-Spawn
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 유닛 선택, 자원 수집, 건물 건설, 포그 오브 워

### 턴제 RPG
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 턴 시스템, 행동력, 대화, 퀘스트

### 카드게임 / 덱빌더
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 덱/핸드/묘지, 카드 효과, 마나/비용

### 서바이벌 (비전투)
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 허기/갈증/체온, 크래프팅, 건축

### 시뮬레이션 / 타이쿤
```bash
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```
게임에서 추가 구현: 경제 시스템, 건설, NPC AI, 시간 흐름

### 레이싱
```bash
# System 서브모듈 불필요 — Core만으로 충분
```
게임에서 추가 구현: 차량 물리, 트랙, 랩타임, AI 주행

---

## 매트릭스 요약

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 타워디펜스 | ✅ | ✅ | ✅ | ✅ |
| 로그라이크/뱀서 | ✅ | ✅ | ✅ | ✅ |
| 핵슬/ARPG | ✅ | - | ✅ | ✅ |
| FPS/TPS | ✅ | - | - | ✅ |
| RTS | ✅ | ✅ | - | ✅ |
| 턴제 RPG | ✅ | - | ✅ | ✅ |
| 카드/덱빌더 | - | - | - | ✅ |
| 서바이벌 (비전투) | - | - | ✅ | ✅ |
| 시뮬/타이쿤 | - | - | - | ✅ |
| 레이싱 | - | - | - | - |

**Stat은 거의 모든 장르에서 사용.** Combat은 전투 있는 게임 필수.

---

## 새 프로젝트 시작 절차

```
1. NetcodeFramework-Template clone
2. Core 서브모듈 추가 (이미 포함)
3. 장르 확인 → 위 매트릭스에서 필요한 시스템 확인
4. git submodule add 실행
5. 개발 시작
```

---

## 시스템 간 의존성

```
Combat → Core (EventBus, Pool, Audio, Shared)
Spawn  → Core (EventBus, Pool, Shared)
Drop   → Core (Pool)
Stat   → 없음 (UnityEngine만)

시스템 간 교차 참조: 0건
어떤 조합이든 충돌 없이 사용 가능
```
