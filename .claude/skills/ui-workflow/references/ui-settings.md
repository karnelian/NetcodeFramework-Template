# UI 세팅 규격

> 프로젝트 공통 불변 규격만 정의합니다.
> 크기, 텍스트 사이즈, Spacing 등 **가변적인 수치**는 JSX/HTML 프로토타입에서 결정합니다.

---

## Canvas 기본 설정

| 설정 | 값 |
|------|---|
| Render Mode | **Screen Space - Overlay** (0) |
| UI Scale Mode | **Scale With Screen Size** (1) |
| Reference Resolution | **1920 x 1080** |
| Screen Match Mode | **Match Width Or Height** (0) |
| Match Width Or Height | **0** (Width 기준) |
| Reference Pixels Per Unit | 100 |
| Pixel Perfect | false |
| Sorting Order (일반 UI) | 기본값 또는 **50** |
| Sorting Order (팝업) | **100** |

---

## RectTransform 앵커 패턴

> 구체적 크기(SizeDelta)는 프로토타입에서 결정. 여기서는 **패턴만** 정의.

| 용도 | Anchor Min | Anchor Max | Pivot | 비고 |
|------|-----------|-----------|-------|------|
| 전체 배경 (_z_back) | (0, 0) | (1, 1) | (1, 0) | Stretch, IgnoreLayout: true |
| 딤 배경 (dimmed) | (0, 0) | (1, 1) | (0.5, 0.5) | Stretch |
| 팝업 루트 (popup_*) | (0, 0) | (1, 1) | (0.5, 0.5) | 전체화면 Stretch |
| 중앙 고정 | (0.5, 0.5) | (0.5, 0.5) | (0.5, 0.5) | 고정 크기 |
| 좌측 정렬 | (0, 0.5) | (0, 0.5) | (0, 0.5) | 좌측 기준점 |
| 우하단 고정 | (1, 0) | (1, 0) | (1, 0) | 우하단 기준 |
| 좌하단 고정 | (0, 0) | (0, 0) | (0, 0) | 좌하단 기준 |

---

## 컴포넌트 기본 설정

### Image

| 설정 | 배경 (_z_back) | 아이콘 (img_*) | 딤 배경 (dimmed) | 구분선 (_divider) |
|------|---------------|---------------|-----------------|-----------------|
| Raycast Target | **false** | **false** | **false** | **false** |
| Image Type | **Sliced (1)** | Simple (0) | Simple (0) | Simple (0) |
| Color | (1,1,1,1) | 용도별 | (0,0,0,0) 초기 → 코드에서 0.7 | (1,1,1,**0.2**) |
| PixelsPerUnitMultiplier | **3~4** | 1 | 1 | 1 |
| IgnoreLayout | **true** | false | false | false |

### TextMeshProUGUI

> 폰트 크기, 색상, 정렬은 JSX/HTML 프로토타입 기반으로 결정.

| 설정 | 값 |
|------|---|
| Font (Legacy) | **사용 금지** — TMP 전용 |
| Raycast Target | **항상 false** |
| 폰트 구분 | 주력 / 보조 / 버튼 전용 (프로젝트별 지정) |

### Button

| 설정 | 값 |
|------|---|
| Transition | **None (0)** — ButtonHighlighter로 커스텀 처리 |
| Navigation | **Vertical (3)** — 게임패드 상하 이동 |
| 배경 Image RaycastTarget | **true** (버튼 루트만) |

**ButtonHighlighter 상태별 색상:**
| 상태 | 텍스트 색상 |
|------|-----------|
| Normal | **White** |
| Highlighted | **Yellow** |
| Pressed | **Red** |
| Selected | **Cyan** |

### ScrollRect

| 설정 | 기본값 |
|------|--------|
| Movement Type | Elastic |
| Inertia | true |
| Deceleration Rate | 0.135 |
| Scroll Sensitivity | 1 |

### 프로그레스 바

> Slider 컴포넌트 미사용. Image의 fillAmount 기반.

```
obj_hp (또는 obj_stamina 등)
├── xxx_bar_back        ← 배경 Image
│   ├── front           ← 실제 게이지 (Image, Filled, fillAmount로 제어)
│   └── back            ← 감소 연출용 (선택)
└── t_hp                ← 수치 텍스트 (선택)
```

### SoundComponent

버튼마다 **2개** 배치:
| SoundType | 용도 |
|-----------|------|
| 3 (Hover) | 마우스 오버 / 게임패드 선택 시 |
| 4 (Click) | 클릭 / 게임패드 Submit 시 |

### DOTween 애니메이션 패턴

| 설정 | 기본값 |
|------|--------|
| isFrom | **1** (From 모드 — endValue에서 시작, 원래 위치로 돌아옴) |
| autoPlay | **0** (코드에서 수동 트리거) |
| DOKill(true) | Destroy 전 반드시 호출 |

---

## 팝업 시스템

### 구조
```
PopupManager (싱글톤, DontDestroyOnLoad)
├── Canvas (ScreenSpaceOverlay, sortingOrder: 100)
│   ├── CanvasScaler (1920x1080, ScaleWithScreenSize, match: 0.5)
│   └── PopupParent (RectTransform, stretch)
│       ├── DimmedBackground (Image, black alpha 0.7, RaycastTarget: off)
│       └── [동적 생성 팝업들 - Stack 관리]
```

### 핵심 클래스

| 클래스 | 역할 |
|--------|------|
| `PopupManager` | 싱글톤. 팝업 생성/스택 관리/딤 배경 제어 |
| `BasePopup` | 모든 팝업의 베이스. closeButton, CanvasGroup |
| `Popup` (static) | 정적 헬퍼. `Popup.Show()`, `Popup.Close()`, `Popup.CloseAll()` |
| `PopupType` (enum) | 팝업 종류 정의. Resources 로드 키 겸용 |

### 팝업 호출
```csharp
Popup.Show(PopupType.SettingsPopup);

Popup.ShowConfirm("제목", "내용",
    onConfirm: () => { Popup.Close(); },
    onCancel: () => { Popup.Close(); },
    "확인", "취소", PopupType.ExitPopup);

Popup.ShowMessage("제목", "내용", "확인", onConfirm: () => { });

Popup.CloseAll();
```

### 팝업 로딩 우선순위
1. Inspector `PopupPrefabMapping` → Dictionary
2. `Resources.Load(popupPath + "/" + PopupType.ToString())`

### 팝업 생명주기
```
Show → Instantiate → Initialize(canClose) → Push Stack → UpdateDimmed
Close → Pop Stack → DOKill(true) → Destroy → UpdateDimmed
```

---

## Input ↔ UI 전환

| 상태 | ActionMap |
|------|----------|
| 인게임 플레이 | `"Player"` |
| UI/메뉴/팝업 열림 | `"UI"` |
| 인터랙티브 로비 | `"InteractiveLobby"` |

---

## 다국어 (Localization)

| 클래스 | 용도 |
|--------|------|
| `LanguageManager` | 언어 관리 (prefab으로 Resources 로드) |
| `LocalizedText` | TMP 텍스트 자동 번역 |
| `LocalizedButton` | 버튼 텍스트 번역 |
| `LocalizedDropdown` | 드롭다운 옵션 번역 |
| `LocalizedInputField` | 입력 필드 placeholder 번역 |
| `GlobalText.GetGlobalText(TextEnum.Key)` | 코드에서 번역 문자열 조회 |

---

## Prefab 네이밍 컨벤션

### Prefab 접두사

| 접두사 | 용도 |
|--------|------|
| `hud_` | 전체 화면 HUD |
| `popup_` | 팝업 오버레이 |
| `panel_` | 부분 패널 |
| `slot_` | 리스트/그리드 셀 |
| `obj_` | 재사용 위젯 |
| `btn_` | 공통 버튼 |
| `toggle_` | 토글 |
| `toast_` | 토스트 알림 |
| `tool_tip` / `obj_tooltip_` | 툴팁 |

### _ani + _z_back 래핑 패턴

UI 요소에 애니메이션을 넣을 때 **콘텐츠와 애니메이션 대상을 분리**하는 구조:

```
popup_shop
├── _z_back                    ← 전체 배경 (딤/배경, 애니메이션 안 걸림)
├── _ani                       ← DOTween 애니메이션 대상 (scale, fade, move 등)
│   ├── _layout_z_back         ← 콘텐츠 영역 배경
│   │   ├── _z_back_outline_T  ← 상단 테두리
│   │   └── _z_back_outline_B  ← 하단 테두리
│   ├── _layout_title
│   ├── _layout_slot
│   └── _layout_esc
```

**왜 분리하는가:**
- `_z_back` (전체 배경)은 고정 — 딤 처리만 담당, 애니메이션 안 걸림
- `_ani`에 DOTween 걸면 **내부 콘텐츠 전체가 한번에** 스케일/페이드/이동됨
- `_ani`는 **중첩 가능** — 부분적으로 다른 애니메이션이 필요할 때

**DOTween 연동:**
- `_ani`에 DOTweenAnimation 컴포넌트 부착
- `isFrom: 1` — 시작 위치(endValue)에서 원래 위치로 복귀하는 방식
- `autoPlay: 0` — 코드에서 `DOTweenAnimation.DOPlay()` 호출로 트리거
- CanvasGroup을 `_ani`에 붙여서 alpha fade 처리

**슬롯 단위 래핑:**
```
Slot_{Name}
├── _ani
│   └── _z_back
│       ├── img_icon
│       └── t_level
└── btn                        ← 클릭 영역은 _ani 밖에 분리
```

→ 버튼 클릭 영역과 시각 요소를 분리하여, 애니메이션 중에도 클릭 판정이 안정적.

### 내부 GameObject 네이밍

| 이름 | 용도 |
|------|------|
| `_z_back` | 배경 이미지 (Sliced, IgnoreLayout, RaycastTarget off) |
| `_z_front_outline` | 전면 아웃라인/테두리 장식 |
| `_ani` | 애니메이션 컨테이너 (DOTween 대상, 중첩 가능) |
| `_mask` | Mask 컴포넌트 영역 (m_ShowMaskGraphic: 0) |
| `_layout_esc` | ESC/닫기 버튼 전용 레이아웃 영역 |
| `_layout_upper` / `_layout_under` | 상하 분할 레이아웃 |
| `obj_hover` | 호버/선택 시 표시되는 시각 효과 (기본 비활성) |
| `obj_default` | 기본 상태 (SetActive) |
| `obj_locked` / `obj_lock` | 잠금 상태 |
| `obj_selected` | 선택 상태 |
| `obj_applied` | 적용됨 상태 (장착/활성) |
| `icon_` | 아이콘 이미지 |
| `front` / `back` | 프로그레스 바 내부 (게이지/배경) |

---

## hierarchy-rules 연동

이 규격으로 만든 UI는 `hierarchy-rules` 스킬의 네이밍 규칙을 따른다:
- 접두사: `Popup_`, `btn_`, `t_`/`Txt_`, `img_`/`Img_`, `_layout_`, `obj_`
- 래핑: `_ani` + `_z_back`
- 상태 전환: `obj_default` / `obj_locked` / `obj_selected`

---

## 법칙 체크리스트

### Prefab 만들 때
- [ ] 접두사 규칙 준수
- [ ] `Prefabs/UI/` 하위에 배치, 기능별 서브폴더
- [ ] 배경: `_z_back` (Sliced, PixelsPerUnit 3~4, IgnoreLayout, RaycastTarget off)
- [ ] 팝업: `BasePopup` 상속 + `PopupType` enum 등록 + CanvasGroup 필수

### Canvas 설정할 때
- [ ] 1920x1080, ScaleWithScreenSize, Match Width (0)
- [ ] 일반 UI sortingOrder: 50, 팝업: 100

### 텍스트 넣을 때
- [ ] TMP 전용 (Legacy Text 금지)
- [ ] RaycastTarget: **항상 off**
- [ ] 폰트/크기/색상은 JSX/HTML 프로토타입에서 결정

### 버튼 만들 때
- [ ] Transition: **None** (ButtonHighlighter로 커스텀)
- [ ] Navigation: **Vertical** (게임패드 지원)

### 코드 작성할 때
- [ ] 팝업: `Popup.Show(PopupType.XXX)` / `Popup.Close()`
- [ ] UI 열기: `GameInputSystem.Instance.SetActionMap("UI")`
- [ ] UI 닫기: 게임 상태에 맞는 ActionMap 복귀
- [ ] 번역: `GlobalText.GetGlobalText(TextEnum.Key)`
- [ ] DOTween 정리: `transform.DOKill(true)` before Destroy
