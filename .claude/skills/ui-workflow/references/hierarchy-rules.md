# UI Prefab 계층 규칙

> 이 문서는 `ui-workflow` 스킬의 reference 파일이다.
> UI 워크플로우 전체 흐름은 `SKILL.md` 참조.

기존 프로젝트에서 추출한 UI 제작 패턴. 새 UI 프리팹 작성 시 반드시 따를 것.

---

## GameObject 네이밍

| 접두사 | 용도 | 예시 |
|--------|------|------|
| `Popup_` | 팝업 루트 | `Popup_HeroSelect`, `Popup_StageSelect` |
| `Slot_` | 반복 슬롯 아이템 | `Slot_HeroSelect_100` |
| `img_` / `Img_` | Image 컴포넌트 | `img_back`, `Img_Boss` |
| `t_` / `Txt_` | Text 컴포넌트 | `t_heroname`, `Txt_Stage_Title` |
| `btn_` | Button 래퍼 | `btn_Apply`, `btn_easy` |
| `_layout_` | 레이아웃 컨테이너 | `_layout_heroinfo`, `_layout_main` |
| `_ani` | 애니메이션 타겟 노드 | DOTween/Animator 대상 |
| `_z_back` / `z_back` | Z-order 배경 레이어 | `_ani` 하위 |
| `_mask` | Mask 컴포넌트 | 이미지 클리핑용 |
| `obj_` / `Obj_` | 상태 전환/논리 그룹 | `obj_default`, `Obj_Screen_FX` |
| `frame_` | 프레임/테두리 | `frame_traits` |
| `FX_` | 이펙트/VFX | `FX_Frame_Glow` |
| `_collider_` | 터치/클릭 영역 | `_collider_100` |
| `Panel_` / `panel_` | 섹션 패널 | `Panel_Upper`, `panel_exp` |
| `Contain_` / `contain_` | 콘텐츠 래퍼 | `Contain_Play_Result`, `contain_Lvl` |
| `_pivot` | 피벗 포인트 래퍼 | `_pivot > _ani > (내용)` |
| `_div` | 구분선 | `_div > img_div` |
| `dropdown_` | TMP_Dropdown | `dropdown_difficulty`, `dropdown_stage` |
| `_layout_sort` | 정렬 버튼 행 | `btn_status + btn_stage + ...` |
| `_layout_esc` | ESC/뒤로가기 | `obj_key + btn_esc` |

---

## 핵심 구조 패턴

### 1. `_ani` + `_z_back` 래핑 (필수)
모든 애니메이션 가능한 요소는 2단 래핑:
```
{요소}
└── _ani              ← 트윈/애니 타겟 (scale, position, alpha)
    └── _z_back       ← Z-order 정렬, 실제 콘텐츠 컨테이너
        └── (내용물)
```

### 2. 팝업 루트 구조
```
Popup_{Name}
├── img_back                      ← 전체 배경 (딤/블러)
├── img_top / img_bottom          ← 상단/하단 장식 (선택)
├── _layout_{섹션}                 ← 논리적 섹션들
│   ├── _layout_{하위}
│   └── ...
├── _layout_{카테고리 그룹}        ← 슬롯 그룹
│   ├── _layout_position          ← 카테고리 헤더
│   └── Slot_{Name}_{ID}          ← 반복 슬롯
└── btn_{Action}                  ← 하단 버튼
```

### 3. 슬롯 상태 전환 패턴
```
Slot_{Name}_{ID}
├── _ani
│   └── _z_back
│       ├── obj_default           ← 기본 상태 (SetActive)
│       │   └── img_slothero_default
│       ├── obj_locked            ← 잠금 상태
│       │   └── img_lockicon
│       └── obj_selected          ← 선택 상태
│           ├── img_guideline
│           └── img_slotselected
└── _collider_{ID}                ← 터치 영역
```

### 4. 버튼 패턴
```
btn_{name}
├── _ani
│   ├── _z_back
│   ├── _layout_expanded          ← 텍스트/아이콘 (선택)
│   │   ├── t_title_small
│   │   └── _layout_icontext
│   └── obj_keybinding            ← 키바인딩 힌트 (선택)
└── btn                           ← 실제 Button 컴포넌트
```

### 5. 상태 FX 패턴
```
Obj_Screen_FX
├── Img_Screen_Crack              ← 상태 이미지들
├── Img_Screen_Normal             ← SetActive로 전환
├── Img_Screen_Hover
└── Img_Screen_Selected
```

### 6. 동적 리스트/캐러셀
```
{Container}View
├── {Item}(Clone)                 ← 동적 생성 아이템
│   ├── FX_Frame_Glow
│   └── _ani > z_back > _mask > (내용)
├── {Item}(Clone)
└── ...
```
피벗 포인트(`leftpivot`, `centerpivot`, `rightpivot`)로 위치 제어.

### 7. 팝업 영역 분할 (SkillInfo 스타일)
대형 정보 팝업은 알파벳 약어로 영역 분할:
```
Popup_{Name}
├── _ani
│   └── background
├── _layout_popup
│   ├── _T                        ← Top: 제목/이름
│   │   └── _ani > _layout_title
│   ├── _div > img_div            ← 구분선
│   ├── _B                        ← Bottom: 메인 콘텐츠
│   │   └── _ani > _layout_skill
│   ├── _C                        ← Center: 중앙 콘텐츠
│   │   └── _ani > _layout_skill
│   └── _R                        ← Right: 사이드 정보
│       └── _ani > _layout_skill_position
```

### 8. 스킬/아이템 슬롯 단위 (반복 블록)
```
_layout_skill_{type}
├── slot_skill                    ← 아이콘 슬롯
│   ├── _z_back
│   ├── img_icon
│   ├── obj_control               ← 조작키 아이콘 (선택)
│   └── obj_keybinding            ← 키바인딩 텍스트 (선택)
│       ├── _z_back
│       └── t_keybinding
└── _layout_description           ← 설명 블록
    ├── t_title
    └── t_description
```

### 9. 다중 상태 슬롯 (스킬 업그레이드 스타일)
```
obj_slot_{name}
├── btn                               ← 클릭 영역
├── _ani
│   ├── _layout_z_back_default        ← 기본 상태 배경 (SetActive)
│   │   └── _z_back + _z_back_inner + _z_back_outline
│   ├── _layout_z_back_selected       ← 선택 상태 배경
│   │   └── _z_back + _z_back_inner + _z_back_outline
│   └── _layout_z_back_maxlevel       ← 만렙 상태 배경
│       └── _z_back + _z_back_inner + _z_back_outline
├── slot_skill
│   └── _ani > _z_back + _z_front_gauge + icon_skill + t_cooltime
├── obj_level                         ← 레벨 인디케이터
│   └── _ani > level_1 / level_2 / level_3 (SetActive)
├── t_name + _div
└── Scroll View > Viewport > Content  ← 스크롤 설명
```
`_z_front_gauge`: 쿨타임/게이지 오버레이 (z_back 위에 렌더링).

---

## UI 제작 규칙 요약

1. **시각 요소와 클릭 영역 분리**: `_ani/_z_back`(시각) + `btn`(클릭) 별도
2. **상태 전환은 SetActive**: `obj_default`/`obj_locked`/`obj_selected` 토글
3. **레이아웃 중첩 허용**: `_layout_` 접두사로 논리적 그룹핑
4. **슬롯 ID는 숫자 접미사**: `Slot_HeroSelect_100`, `_collider_100`
5. **모든 텍스트/이미지에 접두사 필수**: `t_`/`Txt_`, `img_`/`Img_`
