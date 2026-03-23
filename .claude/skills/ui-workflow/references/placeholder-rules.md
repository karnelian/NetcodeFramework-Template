# 플레이스홀더 규칙

catalog.md에 매칭되는 에셋이 없을 때 적용하는 대체 규칙이다.
플레이스홀더로 대체한 요소는 ui-spec.md의 플레이스홀더 목록에 반드시 기록한다.

---

## 배경 / 패널

Unity 기본 Sprite 사용, 색상으로 역할 구분:
- 메인 패널 배경: #2A2A2A (alpha 0.9)
- 서브 패널 배경: #3A3A3A (alpha 0.8)
- 모달 오버레이 (img_back): #000000 (alpha 0.5)
- 헤더/푸터: #1A1A1A

---

## 버튼

Unity 기본 Button 스프라이트 사용, 색상으로 타입 구분:
- 주요 (btn_confirm 등): #4A90D9
- 보조 (btn_cancel 등): #555555
- 위험 (btn_close, btn_delete 등): #FF4444
- 비활성: #333333
- TextMeshProUGUI 라벨 추가

---

## 아이콘

단색 Image + 텍스트 라벨로 대체, 색상으로 카테고리 구분:
- 무기: #FF6B6B + "무기"
- 방어구: #4ECDC4 + "방어"
- 소비: #45B7D1 + "소비"
- 재료: #96CEB4 + "재료"
- 기타: #AAAAAA + "기타"
- 크기는 원래 의도한 아이콘 크기와 동일하게

---

## 진행 바 (HP, MP, 경험치 등)

Filled 타입 Image 컴포넌트, Fill Method Horizontal, Fill Origin Left:
- HP: #FF3333
- MP: #3333FF
- 스태미나: #FFCC00
- 경험치: #33FF33
- 기본: #FFFFFF

---

## 슬롯 (인벤토리, 장비 등)

- 배경 (img_slot_bg): 단색 #2A2A2A (alpha 0.8)
- 테두리: Outline 컴포넌트 또는 #444444
- 등급 표시 (테두리 색상):
  - 일반: #AAAAAA
  - 고급: #4A90D9
  - 희귀: #9B59B6
  - 전설: #FFD700

---

## 텍스트

TextMeshProUGUI 사용:
- 한글 폰트가 catalog.md에 있으면 해당 폰트 사용
- 없으면 Unity 기본 폰트 + 사용자에게 한글 폰트 필요 알림
- 색상:
  - 제목 (t_title): #FFFFFF
  - 본문: #CCCCCC
  - 보조 정보: #888888
  - 강조: #FFD700
  - 경고/에러: #FF4444

---

## 공통 규칙

1. 플레이스홀더는 육안으로 "임시"임을 알 수 있어야 한다
2. 색상 코드를 일관되게 사용한다
3. 크기/위치/앵커는 최종 에셋 기준으로 정확히 설정한다 (나중에 에셋만 교체)
4. 모든 플레이스홀더를 ui-spec.md 플레이스홀더 목록에 기록한다
5. hierarchy-rules의 네이밍 규칙은 플레이스홀더에도 동일하게 적용한다
