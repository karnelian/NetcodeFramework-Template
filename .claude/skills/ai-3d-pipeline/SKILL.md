---
name: ai-3d-pipeline
description: "AI 3D 모델 생성 도구 선택, 워크플로우, Unity 임포트 시 참조. Coplay MCP 도구 체인 포함."
---

# AI 3D 모델 생성 파이프라인

## 도구별 용도

### Meshy (범용 — 첫 번째 선택)
- 용도: 프롭, 환경 오브젝트, 캐릭터 초안
- 강점: PBR 텍스처(Diffuse, Roughness, Metallic, Normal) 포함 내보내기
- 출력: FBX, GLB, OBJ
- 추천: 빠른 프로토타입, 프롭/소품 대량 생성

### Tripo AI (캐릭터 특화)
- 용도: 캐릭터, 몬스터, NPC
- 강점: 모델링 → 텍스처 → 리토폴로지 → 리깅 전체 파이프라인
- 출력: FBX, GLB
- 추천: 바로 애니메이션 붙여야 할 캐릭터

### Rodin / Hyper3D (고퀄리티)
- 용도: 히어로 에셋, 메인 캐릭터, 핵심 환경
- 강점: 초고해상도 4K PBR 텍스처, 포토리얼리스틱
- 추천: 게임의 핵심 비주얼 모델

### 3DAI Studio (비교용)
- 용도: 여러 AI 모델을 한 플랫폼에서 비교
- 추천: 어떤 스타일이 맞는지 테스트

---

## Coplay MCP 워크플로우

### Text-to-3D
```
generate_3d_model_from_text (프롬프트로 3D 생성)
  → generate_3d_model_texture (텍스처 생성/개선)
  → auto_rig_3d_model (자동 리깅)
  → apply_animation_to_rigged_model (애니메이션 적용)
```

### Image-to-3D
```
generate_or_edit_images (2D 컨셉아트 생성)
  → generate_3d_model_from_image (이미지 → 3D 변환)
  → generate_3d_model_texture (텍스처 개선)
  → auto_rig_3d_model (필요 시 리깅)
```

### 애니메이션 연동
- `search_animation_library`: 기존 애니메이션 검색
- `list_model_animation_clips`: 모델에 포함된 클립 확인
- `apply_animation_to_rigged_model`: 리깅된 모델에 애니메이션 적용

---

## 추천 워크플로우

1. **컨셉** → Midjourney/Stable Diffusion으로 2D 컨셉아트 또는 `generate_or_edit_images`
2. **3D 변환** → 컨셉아트를 Image-to-3D로 변환 (Meshy 또는 Tripo)
3. **리깅** → 캐릭터: Tripo 자동 리깅 또는 Mixamo 또는 `auto_rig_3d_model`
4. **후처리** → 필요 시 Blender에서 토폴로지 정리, UV 수정
5. **Unity 임포트** → FBX로 내보내기 → Unity에 임포트

---

## Unity 임포트 설정

### 모델 임포트 체크사항
| 설정 | 값 |
|------|---|
| Scale Factor | 아트 스타일 가이드 기준에 맞춤 |
| Normals | Import (AI 생성 모델의 노멀 유지) |
| Materials | Extract Materials로 별도 관리 |
| Animation Type | Humanoid (캐릭터) / None (프롭) |

### AI 생성 모델 공통 주의사항
- **폴리곤 수**: `list_objects_with_high_polygon_count`로 확인. 기준 초과 시 Blender Decimate 또는 LOD 설정
- **텍스처 해상도**: `art-style-guide` 기준 초과 시 리사이즈
- **머티리얼**: URP에 맞게 셰이더 변환 필요 (`assign_material_to_fbx`, `assign_material`)
- **피벗 포인트**: AI 모델은 피벗이 맞지 않는 경우 많음 → Blender에서 원점 조정

---

## 파일 배치

| 용도 | 경로 |
|------|------|
| 게임별 모델 | `Assets/_Game/{GameName}/Models/` |
| 게임별 텍스처 | `Assets/_Game/{GameName}/Textures/` |
| 게임별 머티리얼 | `Assets/_Game/{GameName}/Materials/` |
| 프레임워크 공유 | `Assets/NetcodeFramework/Addressables/` |

---

## 에셋 평가 기준

AI 생성 모델 채택 시 (`art-style-guide` 체크리스트와 동일):
- [ ] 아트 스타일 가이드와 일관성
- [ ] 폴리곤 수 기준 이내
- [ ] UV 맵 깨끗한가
- [ ] 리깅 품질 (캐릭터)
- [ ] URP 셰이더 호환
