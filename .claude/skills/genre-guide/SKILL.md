---
name: genre-guide
description: "게임 장르별 필요한 System 서브모듈 조합 가이드. 새 프로젝트 시작 시 장르를 말하면 자동 트리거."
---

# Genre System Guide

## 트리거
장르, 로그라이크, 뱀서, 타워디펜스, FPS, RPG, 핵슬, 카드게임, 서바이벌, 새 게임, 게임 만들기, genre, roguelike, tower defense, survivor, 플랫포머, 퍼즐, 레이싱, 리듬, MOBA, 배틀로얄

---

## 시스템 목록

| 시스템 | 역할 | 상태 |
|--------|------|:----:|
| **Combat** | 데미지, 투사체, 상태이상, IDamageable | ✅ 검증됨 |
| **Spawn** | 웨이브/라운드 적 생성, ISpawnable | ✅ 검증됨 |
| **Drop** | 가중치 랜덤 드롭 테이블, 보상 | ✅ 검증됨 |
| **Stat** | 모디파이어 스택, 시간 제한 자동 만료 | ✅ 검증됨 |
| **Inventory** | 슬롯, 장비 착탈, 소비템, 무게/용량 | 📋 미구현 |
| **Quest** | 퀘스트 상태, 조건, 진행, 보상 | 📋 미구현 |
| **Dialogue** | 대화 트리, 선택지, 조건 분기 | 📋 미구현 |
| **Deck** | 덱/핸드/묘지, 카드 드로우, 비용 | 📋 미구현 |
| **Turn** | 턴 순서, 이니셔티브, 행동력 | 📋 미구현 |
| **Procedural** | 방/맵 절차 생성, 시드, 연결 | 📋 미구현 |
| **Crafting** | 레시피, 재료, 조합, 워크벤치 | 📋 미구현 |
| **Stealth** | 감지, 시야각, 경보 단계, 은신 | 📋 미구현 |
| **Building** | 건설, 배치, 블루프린트, 스냅 | 📋 미구현 |
| **Formation** | 유닛 그룹, 대형, 선택/명령 | 📋 미구현 |
| **FogOfWar** | 시야, 탐사, 미탐사 영역 | 📋 미구현 |
| **Vehicle** | 차량 물리, 조작, 기어, 드리프트 | 📋 미구현 |
| **Combo** | 입력 시퀀스, 콤보 체인, 타이밍 | 📋 미구현 |
| **Economy** | 통화, 상점, 거래, 가격 | 📋 미구현 |
| **Scoring** | 점수, 등급, 리더보드, 콤보 배율 | 📋 미구현 |
| **Rhythm** | 비트 동기화, 판정, 노트 패턴 | 📋 미구현 |
| **Physics2D** | 2D 물리, 플랫폼, 벽 슬라이드, 코요테 타임 | 📋 미구현 |

✅ = 검증 완료 (서브모듈 존재) / 📋 = 설계만 (프로토 검증 후 구현)

---

## 장르별 필수 시스템

### 액션/전투 계열

**타워디펜스**
| Combat | Spawn | Drop | Stat | Building | Economy |
|:------:|:-----:|:----:|:----:|:--------:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 |

**뱀서라이크 (Vampire Survivors류)**
| Combat | Spawn | Drop | Stat | Scoring |
|:------:|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

**로그라이크 액션**
| Combat | Spawn | Drop | Stat | Procedural |
|:------:|:-----:|:----:|:----:|:----------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

**핵앤슬래시 / ARPG (디아블로류)**
| Combat | Drop | Stat | Inventory | Quest | Combo |
|:------:|:----:|:----:|:---------:|:-----:|:-----:|
| ✅ | ✅ | ✅ | 📋 | 📋 | 📋 |

**벨트스크롤 액션 (Streets of Rage류)**
| Combat | Spawn | Drop | Stat | Combo | Scoring |
|:------:|:-----:|:----:|:----:|:-----:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 |

**아레나 서바이벌**
| Combat | Spawn | Drop | Stat | Scoring |
|:------:|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

**보스러시**
| Combat | Spawn | Drop | Stat | Scoring |
|:------:|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

**불릿헬 / 탄막 슈팅**
| Combat | Spawn | Stat | Scoring |
|:------:|:-----:|:----:|:-------:|
| ✅ | ✅ | ✅ | 📋 |

---

### 슈터 계열

**FPS (COD, CS류)**
| Combat | Stat | Inventory | Scoring |
|:------:|:----:|:---------:|:-------:|
| ✅ | ✅ | 📋 | 📋 |

**TPS (Gears류)**
| Combat | Stat | Inventory | Stealth |
|:------:|:----:|:---------:|:-------:|
| ✅ | ✅ | 📋 | 📋 |

**탑다운 슈터 (Hotline Miami류)**
| Combat | Spawn | Drop | Scoring | Combo |
|:------:|:-----:|:----:|:-------:|:-----:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**루터 슈터 (Borderlands류)**
| Combat | Spawn | Drop | Stat | Inventory | Quest |
|:------:|:-----:|:----:|:----:|:---------:|:-----:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 |

**배틀로얄 (PUBG류)**
| Combat | Drop | Stat | Inventory |
|:------:|:----:|:----:|:---------:|
| ✅ | ✅ | ✅ | 📋 |

**트윈스틱 슈터 (Enter the Gungeon류)**
| Combat | Spawn | Drop | Stat | Procedural |
|:------:|:-----:|:----:|:----:|:----------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

---

### RPG 계열

**턴제 RPG (페르소나, FF류)**
| Combat | Drop | Stat | Turn | Inventory | Quest | Dialogue |
|:------:|:----:|:----:|:----:|:---------:|:-----:|:--------:|
| ✅ | ✅ | ✅ | 📋 | 📋 | 📋 | 📋 |

**전략 RPG / SRPG (FFT, FE류)**
| Combat | Drop | Stat | Turn | Formation |
|:------:|:----:|:----:|:----:|:---------:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**오픈월드 RPG (젤다, 엘든링류)**
| Combat | Drop | Stat | Inventory | Quest | Dialogue | Crafting |
|:------:|:----:|:----:|:---------:|:-----:|:--------:|:--------:|
| ✅ | ✅ | ✅ | 📋 | 📋 | 📋 | 📋 |

**MMO RPG**
| Combat | Spawn | Drop | Stat | Inventory | Quest | Dialogue | Economy | Crafting |
|:------:|:-----:|:----:|:----:|:---------:|:-----:|:--------:|:-------:|:--------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 | 📋 | 📋 | 📋 |

**소울라이크**
| Combat | Drop | Stat | Inventory | Combo |
|:------:|:----:|:----:|:---------:|:-----:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**로그라이크 턴제 (Slay the Spire류)**
| Combat | Drop | Stat | Deck | Turn | Procedural |
|:------:|:----:|:----:|:----:|:----:|:----------:|
| ✅ | ✅ | ✅ | 📋 | 📋 | 📋 |

**아이들 RPG (방치형)**
| Spawn | Drop | Stat | Economy |
|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | 📋 |

---

### 전략 계열

**RTS (스타크래프트류)**
| Combat | Spawn | Stat | Building | Formation | FogOfWar | Economy |
|:------:|:-----:|:----:|:--------:|:---------:|:--------:|:-------:|
| ✅ | ✅ | ✅ | 📋 | 📋 | 📋 | 📋 |

**MOBA (LoL류)**
| Combat | Spawn | Stat | Economy | Scoring |
|:------:|:-----:|:----:|:-------:|:-------:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**오토배틀러 (TFT류)**
| Combat | Spawn | Drop | Stat | Economy |
|:------:|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 |

**4X 전략 (Civilization류)**
| Combat | Stat | Building | FogOfWar | Economy | Turn |
|:------:|:----:|:--------:|:--------:|:-------:|:----:|
| ✅ | ✅ | 📋 | 📋 | 📋 | 📋 |

---

### 카드/보드 계열

**덱빌더 (Slay the Spire류)**
| Combat | Drop | Stat | Deck | Turn |
|:------:|:----:|:----:|:----:|:----:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**TCG (하스스톤류)**
| Combat | Stat | Deck | Turn | Economy |
|:------:|:----:|:----:|:----:|:-------:|
| ✅ | ✅ | 📋 | 📋 | 📋 |

**보드게임 디지털**
| Stat | Turn | Scoring |
|:----:|:----:|:-------:|
| ✅ | 📋 | 📋 |

---

### 플랫포머 계열

**액션 플랫포머 (Hollow Knight류)**
| Combat | Spawn | Drop | Stat | Physics2D | Combo |
|:------:|:-----:|:----:|:----:|:---------:|:-----:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 |

**퍼즐 플랫포머 (Celeste류)**
| Physics2D | Scoring |
|:---------:|:-------:|
| 📋 | 📋 |

**런게임 (Temple Run류)**
| Spawn | Drop | Stat | Scoring |
|:-----:|:----:|:----:|:-------:|
| ✅ | ✅ | ✅ | 📋 |

**메트로이드바니아**
| Combat | Spawn | Drop | Stat | Inventory | Physics2D |
|:------:|:-----:|:----:|:----:|:---------:|:---------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 |

---

### 서바이벌/크래프트 계열

**서바이벌 크래프트 (마크, Valheim류)**
| Combat | Spawn | Drop | Stat | Inventory | Crafting | Building |
|:------:|:-----:|:----:|:----:|:---------:|:--------:|:--------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 | 📋 |

**서바이벌 호러 (RE류)**
| Combat | Drop | Stat | Inventory | Stealth |
|:------:|:----:|:----:|:---------:|:-------:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

**농장 시뮬 (Stardew류)**
| Drop | Stat | Inventory | Crafting | Economy | Dialogue | Quest |
|:----:|:----:|:---------:|:--------:|:-------:|:--------:|:-----:|
| ✅ | ✅ | 📋 | 📋 | 📋 | 📋 | 📋 |

**베이스 빌더 (Factorio, Rimworld류)**
| Combat | Spawn | Drop | Stat | Building | Crafting | Economy |
|:------:|:-----:|:----:|:----:|:--------:|:--------:|:-------:|
| ✅ | ✅ | ✅ | ✅ | 📋 | 📋 | 📋 |

---

### 시뮬레이션/경영 계열

**타이쿤 / 경영 시뮬**
| Stat | Building | Economy |
|:----:|:--------:|:-------:|
| ✅ | 📋 | 📋 |

**라이프 시뮬 (심즈류)**
| Stat | Dialogue | Economy | Building |
|:----:|:--------:|:-------:|:--------:|
| ✅ | 📋 | 📋 | 📋 |

**건설 시뮬 (Cities류)**
| Stat | Building | Economy |
|:----:|:--------:|:-------:|
| ✅ | 📋 | 📋 |

**갓게임 (Black & White류)**
| Combat | Spawn | Stat | Building | Economy |
|:------:|:-----:|:----:|:--------:|:-------:|
| ✅ | ✅ | ✅ | 📋 | 📋 |

---

### 스포츠/레이싱/리듬 계열

**레이싱**
| Vehicle | Scoring |
|:-------:|:-------:|
| 📋 | 📋 |

**스포츠 (FIFA류)**
| Stat | Scoring |
|:----:|:-------:|
| ✅ | 📋 |

**리듬 게임**
| Rhythm | Scoring |
|:------:|:-------:|
| 📋 | 📋 |

**파티 게임 (Fall Guys류)**
| Drop | Stat | Scoring |
|:----:|:----:|:-------:|
| ✅ | ✅ | 📋 |

---

### 스텔스/잠입 계열

**잠입 액션 (MGS, Hitman류)**
| Combat | Stat | Inventory | Stealth | Quest |
|:------:|:----:|:---------:|:-------:|:-----:|
| ✅ | ✅ | 📋 | 📋 | 📋 |

**호러 잠입 (Outlast류)**
| Stat | Stealth | Inventory |
|:----:|:-------:|:---------:|
| ✅ | 📋 | 📋 |

---

### 기타

**비주얼 노벨**
| Dialogue |
|:--------:|
| 📋 |

**포인트 앤 클릭 어드벤처**
| Inventory | Dialogue | Quest |
|:---------:|:--------:|:-----:|
| 📋 | 📋 | 📋 |

**샌드박스 (Garry's Mod류)**
| Stat | Building |
|:----:|:--------:|
| ✅ | 📋 |

---

## 시스템별 사용 빈도

| 시스템 | 사용 장르 수 | 상태 |
|--------|:----------:|:----:|
| **Stat** | 35+ | ✅ |
| **Combat** | 28 | ✅ |
| **Drop** | 24 | ✅ |
| **Inventory** | 20 | 📋 |
| **Spawn** | 18 | ✅ |
| **Economy** | 15 | 📋 |
| **Scoring** | 14 | 📋 |
| **Quest** | 9 | 📋 |
| **Building** | 9 | 📋 |
| **Dialogue** | 8 | 📋 |
| **Crafting** | 6 | 📋 |
| **Turn** | 6 | 📋 |
| **Deck** | 4 | 📋 |
| **Stealth** | 4 | 📋 |
| **Combo** | 5 | 📋 |
| **Procedural** | 4 | 📋 |
| **Formation** | 3 | 📋 |
| **FogOfWar** | 3 | 📋 |
| **Physics2D** | 3 | 📋 |
| **Vehicle** | 1 | 📋 |
| **Rhythm** | 1 | 📋 |

---

## 서브모듈 저장소

### ✅ 사용 가능 (검증 완료)

```bash
git submodule add https://github.com/karnelian/NetcodeFramework-Core.git Assets/NetcodeFramework
git submodule add https://github.com/karnelian/NetcodeFramework-System-Combat.git Assets/GenreFramework/System-Combat
git submodule add https://github.com/karnelian/NetcodeFramework-System-Spawn.git Assets/GenreFramework/System-Spawn
git submodule add https://github.com/karnelian/NetcodeFramework-System-Drop.git Assets/GenreFramework/System-Drop
git submodule add https://github.com/karnelian/NetcodeFramework-System-Stat.git Assets/GenreFramework/System-Stat
```

### 📋 미구현 (프로토 검증 후 추가)

검증 우선순위 (사용 빈도 기준):
1. **Inventory** (20개 장르) → ARPG 프로토로 검증
2. **Economy** (15개 장르) → 타이쿤 프로토로 검증
3. **Scoring** (14개 장르) → 아케이드 프로토로 검증
4. **Quest** (9개 장르) → RPG 프로토로 검증
5. **Building** (9개 장르) → 베이스빌더 프로토로 검증
6. **Dialogue** (8개 장르) → RPG 프로토로 검증
7. 나머지 → 해당 장르 프로토 시 검증

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
