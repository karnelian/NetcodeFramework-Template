# NetcodeFramework Game Template

Unity 6 멀티플레이어 게임 프로젝트 템플릿.
NetcodeFramework Core 서브모듈 기반으로 24개 시스템이 자동 구성됩니다.

## Quick Start

### 1. 새 프로젝트 생성

GitHub에서 **"Use this template"** → 새 레포 이름 입력 후 생성.

### 2. 로컬 클론 및 LFS 설정

```bash
git clone <your-repo-url>
cd <your-repo>
git lfs install
cp .gitattributes.template .gitattributes
git add .gitattributes
git commit -m "Enable Git LFS rules"
```

> `.gitattributes.template`에 LFS 규칙이 미리 정의되어 있습니다.
> GitHub Template은 LFS를 지원하지 않으므로, 클론 후 수동 활성화가 필요합니다.

### 3. Core 서브모듈 추가

```bash
git submodule add https://github.com/karnelian/NetcodeFramework-Core.git Assets/NetcodeFramework
git commit -m "Add NetcodeFramework Core submodule"
```

### 4. Unity 열기

Unity Hub에서 프로젝트 열기. 자동 셋업이 순서대로 실행됩니다:

1. **DependencyInstaller** — UGS 패키지 선택 설치 (필요한 경우)
2. **FrameworkAssetGenerator** — Manager 프리팹 + Settings SO + 씬 자동 생성

### 5. 개발 시작

- 게임 코드: `Assets/_Game/` 에 작성
- `CLAUDE.md`: 프레임워크 코딩 규칙
- `.claude/skills/`: AI 개발 가이드

## 포함 패키지

| 패키지 | 버전 | 용도 |
|--------|------|------|
| Netcode for GameObjects | 2.7.0 | 멀티플레이어 네트워킹 |
| Addressables | 2.7.6 | 에셋 로드 시스템 |
| Input System | 1.17.0 | 입력 처리 |
| Localization | 1.5.9 | 다국어 지원 |
| R3 (Reactive Extensions) | Git | 반응형 프로그래밍 |
| uGUI | 2.0.0 | UI 시스템 |
| URP | 17.3.0 | 렌더 파이프라인 |
| Test Framework | 1.6.0 | 유닛 테스트 |

## 자동 생성 에셋

Unity 열면 FrameworkAssetGenerator가 자동 생성:

- Manager 프리팹 22개
- Settings SO 10개
- 씬 4개 (Bootstrap, Mock, UGS, Steam)
- Addressable 그룹 및 엔트리

## 프로젝트 구조

```
Assets/
├── NetcodeFramework/          ← Core 서브모듈 (자동 생성된 에셋 포함)
│   ├── Core/                  ← GameSession, MVVM
│   ├── Plugins/               ← 24개 시스템
│   ├── Editor/                ← DependencyInstaller, AssetGenerator
│   ├── Shared/                ← MonoSingleton, PlatformManager
│   ├── Addressables/          ← (자동 생성) Manager 프리팹/Settings
│   └── Scenes/                ← (자동 생성) Bootstrap/Mock/UGS/Steam
└── _Game/                     ← 게임별 코드 (여기에 작성)
```

## 요구사항

- **Unity 6000.0.x** (Unity 6 LTS)
- **Git LFS** — 바이너리 에셋 관리
- **Git** — 서브모듈 지원
