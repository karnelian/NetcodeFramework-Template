---
name: multiplayer-netcode
description: "멀티플레이어 네트워킹 가이드. 서버 권위 모델, 클라이언트 예측/보간/리콘실리에이션, 래그 보상, 델타 압축, Interest 관리. 멀티플레이어 구현, 네트워크 동기화, 치트 방지, 네트워크 최적화 시 반드시 참조. NetcodeFramework의 NetworkSystem과 연동."
---

# 멀티플레이어 네트워킹 가이드

> **NetcodeFramework 시스템 API는 `netcode-framework` 스킬 (references/systems-api.md) 참조.**
> **씬 전환 (Multiplayer/Singleplayer Strategy)은 `scene-structure` 스킬 참조.**
> 이 스킬은 네트워크 동기화 알고리즘과 최적화 패턴을 다룬다.

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/networking-patterns.md` | 상태 동기화, 클라이언트 예측, 래그 보상, 메시지 직렬화, Interest 관리, 델타 압축 전체 코드 | 네트워크 시스템 구현 시 |

---

## 아키텍처 선택

```
멀티플레이어 유형?
│
├── 경쟁 / 실시간 (FPS, MOBA)
│   └── 데디케이티드 서버 (서버 권위)
│
├── 협동 / 캐주얼
│   └── 호스트 기반 (한 플레이어가 서버)
│
├── 턴제
│   └── 클라이언트-서버 (단순)
│
└── 대규모 (MMO)
    └── 분산 서버
```

| 아키텍처 | 지연시간 | 비용 | 보안 |
|----------|---------|------|------|
| Dedicated Server | 낮음 | 높음 | 강함 |
| P2P | 가변 | 낮음 | 약함 |
| Host-based | 중간 | 낮음 | 중간 |

---

## 핵심 개념

### 서버 권위 모델

모든 게임 로직은 서버에서 검증. 클라이언트는 입력만 전송.

```csharp
public class NetworkPlayer
{
    public int PlayerId { get; set; }
    public Vector3 Position { get; set; }
    public float Health { get; set; }

    // 서버에서 이동 검증
    public bool TryMove(Vector3 newPosition, float deltaTime)
    {
        float maxDistance = MoveSpeed * deltaTime * 1.1f; // 10% 허용치
        if (Vector3.Distance(Position, newPosition) > maxDistance)
            return false; // 치트 의심 → 거부
        Position = newPosition;
        return true;
    }
}
```

### 동기화 방식

| 방식 | 동기화 대상 | 적합한 경우 |
|------|------------|------------|
| State Sync | 게임 상태 | 단순, 소수 오브젝트 |
| Input Sync | 플레이어 입력 | 액션 게임 |
| Hybrid | 둘 다 | 대부분의 게임 |

---

## 래그 보상 기법

| 기법 | 목적 |
|------|------|
| **Client Prediction** | 로컬 플레이어 즉시 반응 (서버 응답 기다리지 않음) |
| **Interpolation** | 원격 플레이어 부드러운 표시 (과거 상태 보간) |
| **Reconciliation** | 예측 오류 시 서버 상태로 되감기 + 재적용 |
| **Lag Compensation** | 서버에서 시간 되감기 → 히트스캔 판정 |

### 클라이언트 예측 흐름
```
1. 입력 수집 → 시퀀스 번호 부여
2. 서버에 입력 전송
3. 로컬에서 즉시 적용 (예측)
4. 입력을 pendingInputs 큐에 저장
5. 서버 응답 수신 → 확인된 입력 제거
6. 서버 위치에서 시작 → 남은 입력 재적용 (리콘실리에이션)
```

### 보간 흐름
```
1. 서버에서 원격 플레이어 상태 수신 (타임스탬프 포함)
2. 순환 버퍼에 저장
3. 렌더링 시: 현재 시각 - 보간 딜레이(~100ms) 시점의 두 상태 찾기
4. Lerp(위치) + Slerp(회전)으로 보간
```

---

## 네트워크 최적화

### 대역폭 절감

| 기법 | 절감 효과 |
|------|-----------|
| Delta Compression | 변경된 필드만 전송 |
| Quantization | float → ushort 정밀도 축소 |
| Priority | 중요한 데이터 우선 전송 |
| Area of Interest | 근처 엔티티만 동기화 |

### 업데이트 레이트 가이드

| 대상 | 레이트 |
|------|--------|
| 가까운 플레이어 | 60 Hz |
| 중간 거리 | 20 Hz |
| 먼 오브젝트 | 10 Hz |
| 정적 오브젝트 | 변경 시에만 |

### 목표 지표

| 지표 | 목표 |
|------|------|
| Latency | < 100ms |
| Tick Rate | 20–60 Hz (장르별) |
| Packet Size | < 1200 bytes (단편화 방지) |

---

## 보안 체크리스트

| 항목 | 방법 |
|------|------|
| 서버 권위 | 모든 게임 로직 서버에서 검증 |
| 입력 검증 | 클라이언트 입력 범위/속도 체크 |
| Rate Limiting | 플러딩 방지 |
| 암호화 | 민감 데이터 암호화 전송 |
| Anti-cheat | 통계 분석, Sanity Check |

---

## NetcodeFramework 연동

| 프레임워크 시스템 | 멀티플레이어 역할 |
|-------------------|-------------------|
| `SceneSystem` | MultiplayerTransitionStrategy로 씬 전환 |
| `EventBus` | 네트워크 이벤트 발행/구독 |
| `PoolSystem` | 네트워크 오브젝트 풀링 |
| `CommandSystem` | 네트워크 명령 기록/재생 |

```csharp
// 씬 전환 (자동으로 멀티플레이어 Strategy 적용)
SceneManager.instance.LoadSceneAsObservable("StageName", sceneData);
```
