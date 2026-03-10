---
name: ui-workflow
description: "UI 프로토타이핑 및 Unity UI 구현 시 참조. JSX/HTML 프로토타입 → Unity UI 변환 워크플로우."
---

# UI 프로토타이핑 워크플로우

> **UI 계층 구조, 네이밍 컨벤션, 접두사 규칙은 CLAUDE.md 'UI Prefab 계층 규칙' 섹션 참조.**
> 이 스킬은 프로토타입에서 Unity UI 구현까지의 워크플로우만 다룬다.

---

## 전체 흐름

```
1단계: 레퍼런스 분석
  → 2단계: JSX/HTML 프로토타입
  → 3단계: Unity UI 스펙 변환
  → 4단계: Coplay MCP로 Unity 구현
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
- 실제 게임 해상도 비율 적용 (`art-style-guide`의 타겟 해상도 참조)
- Unity UI 레이아웃 반영 (앵커 기반, LayoutGroup)
- 인터랙션 포함 (버튼 클릭, 탭 전환, 패널 열기/닫기)
- `art-style-guide`의 색상 팔레트 적용

### 저장 위치
```
docs/02-design/features/{FeatureName}/
├── ui-prototype.jsx    ← JSX 프로토타입
├── ui-spec.md          ← Unity 변환 스펙
└── ...
```

---

## 3단계: Unity UI 스펙 변환

프로토타입 확정 후, 각 UI 요소를 Unity 스펙으로 변환:

| JSX/HTML | Unity 대응 |
|----------|-----------|
| `<div>` (flex container) | VerticalLayoutGroup / HorizontalLayoutGroup |
| `<div>` (grid) | GridLayoutGroup |
| `<img>` | Image 컴포넌트 |
| `<p>` / `<span>` | TextMeshProUGUI |
| `<button>` | Button 컴포넌트 |
| `<input>` | TMP_InputField |
| `<select>` | TMP_Dropdown |
| `overflow: scroll` | ScrollRect + Viewport + Content |

### 앵커 매핑
| CSS 위치 | Unity Anchor |
|----------|-------------|
| `top: 0; left: 0` | Top-Left (0,1)-(0,1) |
| `margin: auto` | Center (0.5,0.5)-(0.5,0.5) |
| `width: 100%; height: 100%` | Stretch (0,0)-(1,1) |
| `position: fixed; bottom: 0` | Bottom-Stretch (0,0)-(1,0) |

---

## 4단계: Coplay MCP로 Unity 구현

### 사용 도구
| 도구 | 용도 |
|------|------|
| `create_ui_element` | UI 오브젝트 생성 (Canvas, Panel, Button, Text 등) |
| `set_ui_layout` | LayoutGroup 설정 (Vertical, Horizontal, Grid) |
| `set_ui_text` | TextMeshPro 텍스트 설정 |
| `set_rect_transform` | RectTransform 위치/크기/앵커 설정 |
| `set_property` | 컴포넌트 프로퍼티 설정 |
| `create_panel_settings_asset` | UI Toolkit Panel Settings |
| `capture_ui_canvas` | UI Canvas 스크린샷 캡처 (검증용) |

### 구현 순서
1. Canvas 생성 → `create_ui_element` (type: Canvas)
2. 패널 구조 생성 → 상위 레이아웃부터 하위로
3. 레이아웃 설정 → `set_ui_layout`
4. 텍스트/이미지 배치 → `create_ui_element` + `set_ui_text`
5. RectTransform 조정 → `set_rect_transform`
6. 캡처 검증 → `capture_ui_canvas`로 결과 확인

---

## MVVM 바인딩

UI 구현 후 CLAUDE.md의 MVVM 패턴으로 데이터 바인딩:

```
ViewModel (GameSessionBaseViewModel 상속)
  ├── ReactiveProperty<T> 상태 정의
  └── Initialize()에서 서비스 구독

View (GameSessionBaseView<TViewModel> 상속)
  ├── [SerializeField] UI 컴포넌트 참조
  └── OnViewModelBound()에서 ViewModel 구독 → UI 갱신
```

CLAUDE.md의 MVVM 섹션에 상세 코드 패턴 있음.

---

## 체크리스트

- [ ] CLAUDE.md의 UI 접두사 규칙 적용 (`Popup_`, `btn_`, `t_`, `img_` 등)
- [ ] `_ani` + `_z_back` 래핑 패턴 적용
- [ ] 상태 전환은 `SetActive` 패턴 (`obj_default`/`obj_locked`/`obj_selected`)
- [ ] MVVM View/ViewModel 바인딩 완료
- [ ] `capture_ui_canvas`로 최종 결과 검증
