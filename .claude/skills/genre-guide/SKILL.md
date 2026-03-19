---
name: genre-guide
description: "게임 장르별 필요한 System 서브모듈 조합 가이드. 새 프로젝트 시작 시 장르를 말하면 자동 트리거."
---

# Genre System Guide

## 트리거
장르, 로그라이크, 뱀서, 타워디펜스, FPS, RPG, 핵슬, 카드게임, 서바이벌, 새 게임, 게임 만들기, genre, roguelike, tower defense, survivor, 플랫포머, 퍼즐, 레이싱, 리듬, MOBA, 배틀로얄

## 서브모듈 저장소

| 시스템 | Repo URL | 역할 |
|--------|----------|------|
| **Core** | `NetcodeFramework-Core` | EventBus, Pool, Audio, UI, Scene, Camera, TimeScale |
| **Combat** | `NetcodeFramework-System-Combat` | DamageManager, Projectile, StatusEffect, IDamageable |
| **Spawn** | `NetcodeFramework-System-Spawn` | SpawnManager (Wave/Round), ISpawnable |
| **Drop** | `NetcodeFramework-System-Drop` | DropTableManager, 가중치 랜덤 드롭 |
| **Stat** | `NetcodeFramework-System-Stat` | StatModifier, 시간 제한 자동 만료, int 기반 범용 |

Base URL: `https://github.com/karnelian/NetcodeFramework-{name}.git`
서브모듈 경로: `Assets/GenreFramework/{name}`

---

## 장르별 필수 시스템 매트릭스

### 액션/전투 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 타워디펜스 | ✅ | ✅ | ✅ | ✅ |
| 뱀서라이크 (Vampire Survivors류) | ✅ | ✅ | ✅ | ✅ |
| 로그라이크 액션 | ✅ | ✅ | ✅ | ✅ |
| 핵앤슬래시 / ARPG (디아블로류) | ✅ | - | ✅ | ✅ |
| 벨트스크롤 액션 (Streets of Rage류) | ✅ | ✅ | ✅ | ✅ |
| 아레나 서바이벌 | ✅ | ✅ | ✅ | ✅ |
| 보스러시 | ✅ | ✅ | ✅ | ✅ |
| 불릿헬 / 탄막 슈팅 | ✅ | ✅ | - | ✅ |

### 슈터 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| FPS (COD, CS류) | ✅ | - | - | ✅ |
| TPS (Gears류) | ✅ | - | - | ✅ |
| 탑다운 슈터 (Hotline Miami류) | ✅ | ✅ | ✅ | - |
| 루터 슈터 (Borderlands류) | ✅ | ✅ | ✅ | ✅ |
| 배틀로얄 (PUBG, Fortnite류) | ✅ | - | ✅ | ✅ |
| 트윈스틱 슈터 (Enter the Gungeon류) | ✅ | ✅ | ✅ | ✅ |

### RPG 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 턴제 RPG (페르소나, FF류) | ✅ | - | ✅ | ✅ |
| 전략 RPG / SRPG (FFT, FE류) | ✅ | - | ✅ | ✅ |
| 오픈월드 RPG (젤다, 엘든링류) | ✅ | - | ✅ | ✅ |
| MMO RPG | ✅ | ✅ | ✅ | ✅ |
| 소울라이크 | ✅ | - | ✅ | ✅ |
| 로그라이크 턴제 (Slay the Spire류) | ✅ | - | ✅ | ✅ |
| 아이들 RPG (방치형) | - | ✅ | ✅ | ✅ |

### 전략 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| RTS (스타크래프트, AoE류) | ✅ | ✅ | - | ✅ |
| MOBA (LoL, Dota류) | ✅ | ✅ | - | ✅ |
| 오토배틀러 (TFT류) | ✅ | ✅ | ✅ | ✅ |
| 4X 전략 (Civilization류) | ✅ | - | - | ✅ |

### 카드/보드 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 덱빌더 (Slay the Spire류) | ✅ | - | ✅ | ✅ |
| TCG (하스스톤, MTG류) | ✅ | - | - | ✅ |
| 보드게임 디지털 | - | - | - | ✅ |

### 플랫포머 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 액션 플랫포머 (Hollow Knight류) | ✅ | ✅ | ✅ | ✅ |
| 퍼즐 플랫포머 (Celeste류) | - | - | - | - |
| 런게임 (Temple Run류) | - | ✅ | ✅ | ✅ |
| 메트로이드바니아 | ✅ | ✅ | ✅ | ✅ |

### 서바이벌/크래프트 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 서바이벌 크래프트 (마크, Valheim류) | ✅ | ✅ | ✅ | ✅ |
| 서바이벌 호러 (RE류) | ✅ | - | ✅ | ✅ |
| 농장 시뮬 (Stardew류) | - | - | ✅ | ✅ |
| 베이스 빌더 (Factorio, Rimworld류) | ✅ | ✅ | ✅ | ✅ |

### 시뮬레이션/경영 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 타이쿤 / 경영 시뮬 | - | - | - | ✅ |
| 라이프 시뮬 (심즈류) | - | - | - | ✅ |
| 건설 시뮬 (Cities류) | - | - | - | ✅ |
| 갓게임 (Black & White류) | ✅ | ✅ | - | ✅ |

### 스포츠/레이싱/리듬 계열

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 레이싱 | - | - | - | - |
| 스포츠 (FIFA류) | - | - | - | ✅ |
| 리듬 게임 | - | - | - | - |
| 파티 게임 (Fall Guys류) | - | - | ✅ | ✅ |

### 기타

| 장르 | Combat | Spawn | Drop | Stat |
|------|:------:|:-----:|:----:|:----:|
| 비주얼 노벨 | - | - | - | - |
| 포인트 앤 클릭 어드벤처 | - | - | - | - |
| 샌드박스 (Garry's Mod류) | - | - | - | ✅ |
| 호러 (비전투, Outlast류) | - | - | - | ✅ |

---

## 시스템별 사용 빈도

| 시스템 | 사용 장르 수 | 범용도 |
|--------|:----------:|:-----:|
| **Stat** | 35+ / 40 | ★★★★★ |
| **Combat** | 28 / 40 | ★★★★ |
| **Drop** | 24 / 40 | ★★★★ |
| **Spawn** | 18 / 40 | ★★★ |

---

## 서브모듈 추가 명령어

```bash
# Core (필수)
git submodule add https://github.com/karnelian/NetcodeFramework-Core.git Assets/NetcodeFramework

# 시스템 (장르별 선택)
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Spawn.git Assets/GenreFramework/System-Spawn
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
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

---

## 향후 추가 예정 시스템

새 게임 프로토타입으로 검증 후 추가:

| 시스템 | 검증 게임 | 포함 내용 |
|--------|----------|----------|
| System-Inventory | ARPG 프로토 | 슬롯, 장비, 소비템, 무게 |
| System-Quest | ARPG 프로토 | 퀘스트 상태, 조건, 보상 |
| System-Deck | 카드게임 프로토 | 덱/핸드/묘지, 카드 효과 |
| System-Turn | 턴제 프로토 | 턴 순서, 행동력, 이니셔티브 |
| System-Dialogue | RPG 프로토 | 대화 트리, 선택지, 조건 |
| System-Procedural | 로그라이크 프로토 | 방/맵 생성, 시드, 연결 |

**검증 없이 만들지 않음.** 실제 게임에서 써보고 뽑음.
