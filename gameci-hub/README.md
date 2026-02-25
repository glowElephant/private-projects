# GameCI Hub

여러 Unity 프로젝트를 중앙에서 빌드하는 GitHub Actions CI/CD 파이프라인.

> **빌드 시간 70% 단축** 달성

## Overview

회사 내 4개 Unity 프로젝트의 빌드·배포를 자동화하는 통합 CI/CD 시스템입니다. 하나의 빌드 환경(Docker)을 유지하면서 내부 저장소만 교체하여 빌드하는 구조로, 환경 불일치 문제를 원천 차단합니다. Self-hosted Runner로 사내 물리 머신(Linux + macOS ARM64)을 활용합니다.

## Workflows

| 워크플로우 | 대상 | 설명 |
|-----------|------|------|
| `manual-build.yml` | 3개 프로젝트 | 통합 빌드 파이프라인 |
| `lwar-build.yml` | 1개 프로젝트 | 타겟/환경 파라미터 분기 전용 |

## Build Types

| 타입 | 설명 | 결과물 |
|------|------|--------|
| `android-apk` | 테스트용 APK | `.apk` 파일 |
| `android-export` | UaaL용 Gradle 프로젝트 | `unityLibrary.zip` (+`UnityDataAssetPack`) |
| `ios` | Xcode → UnityFramework | `frameworks.zip` 또는 `xcframework` |
| `all` | android-export + ios | 전체 빌드 |

## Architecture

```
GitHub Actions Workflow
  │
  ├─ Trigger: push / PR / manual dispatch / schedule
  │
  ├─ Self-hosted Runner (Linux)
  │    └─ GameCI Docker Container (Unity 버전별 이미지)
  │         ├─ 저장소 체크아웃 (GitHub App 토큰으로 private 접근)
  │         ├─ Unity 빌드 (Android APK/AAB/AAR/Library)
  │         ├─ 아티팩트 업로드
  │         └─ Slack 빌드 결과 알림
  │
  ├─ Self-hosted Runner (macOS ARM64)
  │    ├─ Unity 빌드 (iOS XCFramework)
  │    ├─ 아티팩트 업로드
  │    └─ Slack 빌드 결과 알림
  │
  └─ Release Job
       ├─ GitHub Release 자동 생성
       ├─ 빌드 아티팩트 첨부
       └─ Slack 릴리즈 알림
```

### Workflow Flow

```
┌─────────┐
│  setup  │  프로젝트별 설정 (Addressables, Gradle AAR, XCFramework)
└────┬────┘
     │
     ├──────────────────┬──────────────────┐
     ▼                  ▼                  ▼
┌─────────────┐  ┌──────────────┐  ┌───────────┐
│ android-apk │  │android-export│  │    ios    │
│  (Linux)    │  │   (Linux)    │  │   (Mac)   │
└─────────────┘  └──────────────┘  └───────────┘
     │                  │                  │
     └──────────────────┴──────────────────┘
                        │
                   ┌────▼────┐
                   │ notify  │  Slack 알림
                   └─────────┘
```

## Key Design

| 설계 | 설명 |
|------|------|
| **빌드 환경 고정** | Docker 컨테이너로 Unity 버전·SDK 고정. "내 PC에서는 되는데" 문제 원천 차단 |
| **저장소 교체 빌드** | 하나의 빌드 환경에서 프로젝트 저장소만 교체하며 빌드. 환경 중복 제거 |
| **GitHub App 인증** | Runner가 private 저장소에 접근하기 위해 GitHub App 토큰 발급 방식 채택 |
| **플랫폼별 Runner 분리** | Android → Linux Docker (빠름), iOS → macOS ARM64 (Xcode 필수) |
| **프로젝트별 분기** | Addressables, Gradle AAR, XCFramework 등 프로젝트 특성에 따라 빌드 단계 자동 분기 |
| **Slack 알림 통합** | 빌드 시작/성공/실패/릴리즈 전 단계 Slack 채널 알림 |

## Project-Specific Settings

| 설정 | 적용 프로젝트 | 설명 |
|------|-------------|------|
| Addressables | 일부 | UnityDataAssetPack 별도 패키징 |
| Gradle AAR | 일부 | `gradle assembleRelease`로 AAR 생성 |
| XCFramework | 일부 | device + MockFramework로 xcframework 생성 |
| 타겟/환경 분기 | 1개 | 타겟(지역별)·환경(release/dev) 파라미터로 빌드 경로 분기 |

## Build Script

각 프로젝트에 `Assets/Editor/BuildScriptGameCI.cs`를 배치하여 빌드 메서드를 표준화합니다.

```csharp
public static class BuildScriptGameCI
{
    public static void BuildAndroidAPK() { ... }
    public static void BuildAndroidExport() { ... }
    public static void BuildiOS() { ... }
}
```

## Runner Configuration

| Runner | Label | 용도 |
|--------|-------|------|
| Linux | `self-hosted, Linux` | Android APK, Android Export |
| macOS | `self-hosted, macOS, ARM64` | iOS Framework/XCFramework |

## Results

| 지표 | Before | After |
|------|--------|-------|
| 빌드 시간 | 수동 30~40분 | 자동 10~15분 (**70% 단축**) |
| 빌드 환경 | 개인 PC별 상이 | Docker 표준화 |
| 배포 | 수동 업로드 | GitHub Release 자동 |
| 모니터링 | 없음 | Slack 실시간 알림 |

## File Structure

```
gameci-hub/
├── .github/
│   └── workflows/
│       ├── manual-build.yml    # 통합 빌드
│       ├── lwar-build.yml      # 전용 빌드
│       └── test-slack.yml      # Slack 연동 테스트
├── templates/
│   └── BuildScript.cs          # 빌드 스크립트 템플릿
└── README.md
```

## Tech Stack

`GitHub Actions` `GameCI` `Docker` `Gradle` `Shell Script` `GitHub App` `Slack Webhook`

---

> 🔒 이 저장소는 private 프로젝트의 README 공개 버전입니다.
