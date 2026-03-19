---
name: ui-workflow
description: "UI 프로토타이핑 및 Unity UI 구현 시 참조. JSX/HTML 프로토타입 → Unity UI 변환 워크플로우. UI Prefab 계층 규칙(GameObject 네이밍, 구조 패턴 9종), UI 세팅 규격, 플레이스홀더 규칙 포함. UI를 만들거나 배치하려는 요청, HTML/JSX 목업을 Unity로 변환, UI 스펙 문서 작성, 팝업/HUD/인벤토리 등 UI 구현, UI 에셋 배치, 플레이스홀더 UI 작업, UI 프리팹 네이밍/계층 구조 작업에 이 스킬을 사용한다."
---

# UI 프로토타이핑 & 구현 워크플로우

> **에셋 경로/폴더 구조는 `asset-management` 스킬 참조.**
> **MVVM 패턴 상세는 `netcode-framework` 스킬 (references/code-patterns.md) 참조.**

## 참조 파일

| 파일 | 내용 | 참조 시점 |
|------|------|-----------|
| `references/hierarchy-rules.md` | GameObject 네이밍 접두사, 구조 패턴 9종 (_ani+_z_back, 팝업 루트, 슬롯 상태 전환, 버튼, FX, 동적 리스트, 영역 분할, 스킬 슬롯, 다중 상태) | UI 프리팹 생성/수정, 네이밍 확인 시 **반드시** 참조 |
| `references/ui-settings.md` | Canvas 기본 설정, RectTransform 앵커 패턴, LayoutGroup 패턴, 컴포넌트 기본값 | UI 스펙 변환, RectTransform 설정 시 |
| `references/ui-spec-template.md` | ui-spec.md 출력 형식 템플릿 | UI 스펙 문서 작성 시 |
| `references/placeholder-rules.md` | 에셋 부재 시 플레이스홀더 색상/규칙 | 에셋 없이 UI 구현할 때 |

---

## 전체 흐름

```
1단계: 레퍼런스 분석
  → 2단계: JSX/HTML 프로토타입 생성
  → 3단계: Unity UI 스펙 변환 (ui-spec.md 생성)
  → 4단계: 에셋 매칭 (catalog.md 참조)
  → 5단계: Coplay MCP로 Unity 구현
  → 6단계: 검증 & 피드백 반복
```

---

## 1단계: 레퍼런스 분석

게임 장르에 맞는 레퍼런스 게임의 UI 패턴 분석:
- HUD 구성 요소 (HP, 스태미나, 미니맵 등)
- 메뉴/인벤토리 구조
- 팝업/다이얼로그 패턴
- 화면 전환 흐름

---

## 2단계: JSX/HTML 프로토타입 생성

### 작성 규칙
- 실제 게임 해상도 비율 적용
- Unity UI 레이아웃을 의식하여 구성 (앵커 기반, LayoutGroup 대응)
- 인터랙션 포함 (버튼 클릭, 탭 전환, 패널 열기/닫기)

### 저장 위치
```
docs/02-design/features/{FeatureName}/
├── ui-prototype.jsx    ← JSX 프로토타입
├── ui-spec.md          ← Unity 변환 스펙 (3단계에서 생성)
└── ...
```

---

## 3단계: Unity UI 스펙 변환

프로토타입이 확정되면 **ui-spec.md 스펙 문서**를 생성한다.

### 반드시 참조할 문서
- **references/ui-settings.md** — Canvas, 앵커 패턴, LayoutGroup, 컴포넌트 기본값
- **references/hierarchy-rules.md** — GameObject 네이밍, 계층 구조 패턴

### 스펙 문서에 포함할 내용

각 UI 요소마다:

1. **GameObject 이름** — `references/hierarchy-rules.md`의 접두사 규칙 적용
   - 팝업 루트: `Popup_{Name}`
   - 버튼: `btn_{name}`
   - 텍스트: `t_{name}` 또는 `Txt_{Name}`
   - 이미지: `img_{name}` 또는 `Img_{Name}`
   - 레이아웃 컨테이너: `_layout_{name}`
   - 상태 오브젝트: `obj_{state}`

2. **계층 구조** — `references/hierarchy-rules.md`의 패턴 적용
   - `_ani` + `_z_back` 래핑 (애니메이션 가능 요소)
   - 상태 전환은 `obj_default` / `obj_locked` / `obj_selected` (SetActive)
   - 시각 요소와 클릭 영역 분리 (`_ani/_z_back` + `btn` 별도)

3. **RectTransform 설정** — `references/ui-settings.md`의 앵커 패턴 적용

4. **컴포넌트 및 설정값** — `references/ui-settings.md`의 기본값 적용

5. **에셋 지정** — catalog.md에서 매칭하거나 플레이스홀더 표기

6. **해상도 대응** — 고정 크기(px) vs 비율 기반(stretch) 구분

### 스펙 문서 형식

**references/ui-spec-template.md에 전체 템플릿이 있다.** 반드시 이 형식을 따를 것.

### JSX/HTML → Unity 대응 참조

| JSX/HTML | Unity 대응 |
|----------|-----------|
| `<div>` (flex column) | VerticalLayoutGroup |
| `<div>` (flex row) | HorizontalLayoutGroup |
| `<div>` (grid) | GridLayoutGroup |
| `<img>` | Image 컴포넌트 |
| `<p>` / `<span>` | TextMeshProUGUI |
| `<button>` | Button 컴포넌트 |
| `<input>` | TMP_InputField |
| `<select>` | TMP_Dropdown |
| `overflow: scroll` | ScrollRect + Viewport + Content |
| `position: absolute` | 독립 앵커 설정 |
| `position: fixed` | Canvas 내 별도 레이어 |

---

## 4단계: 에셋 매칭

### catalog.md가 있는 경우
스펙 문서의 각 요소에 catalog.md의 에셋을 매칭한다.
Sprite의 경우 9-slice 여부, 크기를 확인하고 Image Type을 결정한다.

### catalog.md가 없는 경우
사용자에게 알리고 두 가지 중 선택:
1. 에셋 카탈로그를 먼저 생성한다
2. 전체를 플레이스홀더로 진행한다 (`references/placeholder-rules.md` 참조)

---

## 5단계: Coplay MCP로 Unity 구현

### 구현 순서

1. Canvas 생성 — `references/ui-settings.md`의 Canvas 기본 설정대로
2. 계층 구조 생성 — 스펙의 트리 구조대로, `references/hierarchy-rules.md` 패턴 적용
3. RectTransform 설정 — 스펙의 앵커/피벗/사이즈대로
4. 컴포넌트 추가 — 스펙의 컴포넌트 및 설정값대로
5. 에셋 할당 — catalog.md 경로로 스프라이트/폰트 할당
6. 플레이스홀더 적용 — 에셋 없는 요소에 대체 적용

### 사용 도구

| 도구 | 용도 |
|------|------|
| `manage_gameobject` | GameObject 생성, 계층 배치 |
| `manage_components` | 컴포넌트 추가/설정 |
| `manage_ui` | UI 전용 작업 (Canvas, RectTransform 등) |
| `manage_prefabs` | 프리팹 생성/수정 |
| `manage_asset` | 에셋 참조/할당 |
| `batch_execute` | 3개 이상 작업 시 반드시 사용 (10~100배 빠름) |
| `capture_ui_canvas` | UI Canvas 스크린샷 캡처 (검증용) |

### 성능 규칙

- **batch_execute 필수**: 3개 이상의 MCP 작업은 반드시 묶어서 실행한다
- **@ 참조 활용**: 프롬프트에서 `@PlayerController.cs`처럼 특정 에셋을 직접 참조할 수 있다

---

## 6단계: 검증 & 피드백

### 검증 체크리스트

```
□ 레이아웃이 프로토타입(HTML/JSX)과 시각적으로 유사한가?
□ hierarchy-rules 네이밍이 적용되었는가?
  - Popup_, btn_, t_, img_, _layout_ 등 접두사
  - _ani + _z_back 래핑
  - 상태 전환 obj_ 패턴
□ 해상도 변경 시 깨지지 않는가? (Game 뷰에서 테스트)
□ 앵커가 ui-settings.md의 패턴대로 설정되었는가?
□ LayoutGroup이 의도대로 자식을 배치하는가?
□ 에셋이 catalog.md의 경로와 일치하는가?
□ 플레이스홀더 요소가 명확히 구분되는가?
□ 버튼/토글 등 상호작용 요소가 클릭 가능한가?
□ TextMeshProUGUI에 한글이 정상 표시되는가?
□ capture_ui_canvas로 스크린샷 확인했는가?
```

### 피드백 반영
수정 시 ui-spec.md도 같이 업데이트한다.

---

## 에셋 교체 (플레이스홀더 → 최종 에셋)

1. 사용자에게 교체 목록 확인
2. Coplay MCP로 스프라이트/폰트 일괄 교체
3. ui-spec.md의 플레이스홀더 목록 업데이트
4. catalog.md에 새 에셋 반영

---

## 여러 화면 작업

UI 화면이 여러 개인 경우 화면별로 스펙을 분리한다:

```
docs/02-design/features/
├── Inventory/
│   ├── ui-prototype.jsx
│   └── ui-spec.md
├── HUD/
│   ├── ui-prototype.jsx
│   └── ui-spec.md
└── Crafting/
    ├── ui-prototype.jsx
    └── ui-spec.md
```

catalog.md와 ui-settings.md는 전체 프로젝트에서 공유한다.

---

## MVVM 바인딩

UI 구현 후 프레임워크의 MVVM 패턴으로 데이터 바인딩:

```
ViewModel (GameSessionBaseViewModel 상속)
  ├── ReactiveProperty<T> 상태 정의
  └── Initialize()에서 서비스 구독

View (GameSessionBaseView<TViewModel> 상속)
  ├── [SerializeField] UI 컴포넌트 참조
  └── OnViewModelBound()에서 ViewModel 구독 → UI 갱신
```

상세 코드 패턴은 `netcode-framework` 스킬 (references/code-patterns.md) 참조.
