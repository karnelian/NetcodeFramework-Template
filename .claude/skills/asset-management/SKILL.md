---
name: asset-management
description: "Addressable 경로, 서브모듈 커밋 규칙, Git LFS, 폴더 구조. 에셋 관리/커밋 시 참조."
---

# 에셋 관리 규칙

---

## Addressable 경로 규칙

모든 Manager 프리팹과 Settings SO:
```
Assets/NetcodeFramework/Addressables/Manager/{ManagerName}/{ManagerName}.prefab
Assets/NetcodeFramework/Addressables/Manager/{ManagerName}/{ManagerName}Settings.asset
```

게임별 사운드:
```
Assets/NetcodeFramework/Addressables/Manager/AudioManager/Sound/
├── BGM/
├── SFX/
│   ├── Combat/
│   ├── Environment/
│   └── Character/
├── UI/
└── Ambient/
```

게임별 에셋: `Assets/_Game/{GameName}/` 하위에 배치.

---

## 서브모듈 규칙

`Assets/NetcodeFramework/`는 서브모듈 (`karnelian/NetcodeFramework-Core`).

### 수정 시 커밋 순서
1. 서브모듈 내부에서 변경
2. 서브모듈 디렉토리에서 커밋 (`cd Assets/NetcodeFramework && git commit`)
3. 루트 레포에서 서브모듈 ref 업데이트 커밋

### 주의사항
- 서브모듈 코드 수정 시 반드시 서브모듈 커밋 필요
- 게임별 코드는 `_Game/` 하위에 작성 (서브모듈 밖)
- `ResourceLoadManager.instance.CreateObject(address, parent)`: Addressable + 풀링 자동 적용

---

## Git LFS (.gitattributes)

루트 레포 `.gitattributes`에 LFS 규칙 설정됨:
- Unity 에셋: `*.unity`, `*.asset`, `*.prefab`, `*.controller`, `*.anim`, `*.mixer`, `*.mat` 등
- 오디오: `*.wav`, `*.mp3`, `*.ogg`
- 이미지: `*.png`, `*.jpg`, `*.psd`, `*.tga`, `*.tif`, `*.exr`
- 3D 모델: `*.fbx`, `*.obj`, `*.blend`, `*.glb`, `*.gltf`
- 폰트: `*.ttf`, `*.otf`
- 네이티브: `*.dll`, `*.so`, `*.dylib`
- 비디오: `*.mp4`
- 패키지: `*.unitypackage`

서브모듈에도 별도 `.gitattributes` 있음 (8개 규칙).

---

## 폴더 구조

```
Assets/
├── NetcodeFramework/              ← 서브모듈 (프레임워크 코드)
│   ├── Shared/                    ← 기반 클래스
│   ├── Core/                      ← GameSession, MVVM
│   ├── Plugins/                   ← 26개 시스템
│   ├── Addressables/              ← 프리팹 + SO
│   └── ...
└── _Game/                         ← 게임별 코드/에셋
    └── {GameName}/
        ├── Scripts/
        ├── Prefabs/
        ├── Models/
        ├── Materials/
        ├── UI/
        └── Audio/
```

### 에셋 배치 규칙
- 프레임워크 공통: `Assets/NetcodeFramework/Addressables/` 하위
- 게임별: `Assets/_Game/{GameName}/` 하위
- AI 생성 3D 모델: `Assets/_Game/{GameName}/Models/`
- Resources 폴더 최소화 → Addressable 우선
- 사용하지 않는 에셋 정기 정리
