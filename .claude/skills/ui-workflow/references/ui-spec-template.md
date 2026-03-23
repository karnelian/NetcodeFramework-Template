# [UI 화면 이름] 스펙
참조 프로토타입: [파일명]
참조 카탈로그: catalog.md
참조 UI 세팅: ui-settings.md

---

## Canvas 설정

ui-settings.md의 Canvas 기본 설정을 따른다. 이 화면에서 다른 설정이 필요하면 여기에 명시.

---

## GameObject 계층 구조

hierarchy-rules의 패턴을 적용하여 작성한다.

```
Popup_{Name}
├── img_back                          ← 전체 배경 (딤)
├── _layout_main
│   ├── _layout_header
│   │   ├── _ani
│   │   │   └── _z_back
│   │   │       ├── img_header_bg
│   │   │       └── t_title
│   │   └── btn_close
│   ├── _layout_content
│   │   └── (콘텐츠 요소들)
│   └── _layout_footer
│       └── btn_confirm
└── Popup_Tooltip (기본 비활성)
```

---

## 상세 스펙

각 요소를 아래 형식으로 작성한다. RectTransform 값은 ui-settings.md의 앵커 패턴을 참조.

### [GameObject 이름]
- **Anchor**: [용도] (Min x,y / Max x,y)
- **Pivot**: (x, y)
- **Size**: W x H (고정) 또는 **Offset**: Top, Bottom, Left, Right (stretch)
- **컴포넌트**:
  - [컴포넌트명] — [설정값, ui-settings.md 기본값과 다르면 명시]
- **에셋**: catalog → [패키지명] → [파일명] 또는 플레이스홀더
- **비고**: [추가 설명]

---

### 작성 예시

#### Popup_Inventory
- **Anchor**: Center (Min 0.5, 0.5 / Max 0.5, 0.5)
- **Pivot**: (0.5, 0.5)
- **Size**: 800 x 600
- **컴포넌트**:
  - CanvasGroup — alpha 1, interactable true
- **비고**: 팝업 열기/닫기 시 CanvasGroup alpha 트윈

#### img_back
- **Anchor**: Stretch (Min 0, 0 / Max 1, 1)
- **Pivot**: (0.5, 0.5)
- **Offset**: 0, 0, 0, 0
- **컴포넌트**:
  - Image — Color #000000, alpha 0.5, Raycast Target true
- **에셋**: 없음 (단색)
- **비고**: 딤 배경, 클릭 시 팝업 닫기

#### _layout_header
- **Anchor**: Top-Stretch (Min 0, 1 / Max 1, 1)
- **Pivot**: (0.5, 1)
- **Size**: Height 60 (고정)
- **Offset**: Left 0, Right 0
- **컴포넌트**:
  - HorizontalLayoutGroup — Padding L16 R16 T0 B0, Spacing 8, Child Alignment Middle Left, Force Expand Width false
- **에셋**: catalog → hud_elements → header_bg.png

#### t_title
- **컴포넌트**:
  - TextMeshProUGUI — Font: (ui-settings.md 기본 폰트), Size 24, Color #FFFFFF, Align Left-Middle
  - LayoutElement — Flexible Width 1

#### btn_close
- **Size**: 40 x 40
- **컴포넌트**:
  - Button — Transition: (ui-settings.md 기본값)
  - Image — 에셋 또는 플레이스홀더
- **에셋**: catalog → common_ui → icon_close.png
- **에셋 없으면**: 플레이스홀더 단색 #FF4444 + 자식 t_close "X"

#### _layout_content (GridLayoutGroup 예시)
- **Anchor**: Stretch
- **Offset**: Top 60, Bottom 50, Left 20, Right 20
- **컴포넌트**:
  - GridLayoutGroup — Cell Size 80x80, Spacing (8,8), Constraint Fixed Column Count 6, Child Alignment Upper Left
  - ContentSizeFitter — Vertical Fit Preferred Size

---

## 상태 전환 (해당하는 경우)

hierarchy-rules의 슬롯 상태 전환 패턴 적용:

```
Slot_{Name}_{ID}
├── _ani
│   └── _z_back
│       ├── obj_default       ← SetActive(true) 기본
│       ├── obj_locked        ← SetActive(false)
│       ├── obj_selected      ← SetActive(false)
│       └── obj_applied       ← SetActive(false) 장착/적용됨
└── _collider_{ID}
```

| 상태 | 활성 오브젝트 | 전환 조건 |
|------|-------------|----------|
| 기본 | obj_default | 초기 상태 |
| 잠금 | obj_locked | [조건 명시] |
| 선택 | obj_selected | [조건 명시] |
| 적용됨 | obj_applied | [조건 명시] (장착/활성 상태) |

---

## 플레이스홀더 목록

에셋이 없어서 대체한 요소를 기록한다. 에셋 확보 후 교체할 때 참조.

| 요소 | 현재 대체 방식 | 필요한 에셋 설명 |
|------|--------------|-----------------|
| btn_close | 단색 #FF4444 + "X" | 닫기 아이콘 (40x40) |
| img_slot_bg | 단색 #2A2A2A | 슬롯 배경 (80x80, 9-slice) |
