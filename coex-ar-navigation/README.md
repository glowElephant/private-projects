# COEX Indoor AR Navigation

COEX 스타필드몰 실내 AR 길찾기 Unity 클라이언트.

> 기존 12개월 이상 소요 예상 프로젝트를 AI 활용하여 **3개월 내 완료**

## Overview

GPS가 작동하지 않는 실내 환경에서 카메라 영상만으로 위치를 인식하고, AR로 3D 경로를 그려 목적지까지 안내하는 내비게이션 앱입니다. Unity as a Library(UaaL) 아키텍처로 네이티브 앱(iOS/Android)에 임베드되는 형태로 동작합니다.

## Features

- **AR 기반 실내 길찾기** — ARCeye VLSDK 기반 Visual Localization 정밀 측위 (VPS)
- **다층 건물 내비게이션** — 층간 이동(엘리베이터/에스컬레이터), 경로 탐색
- **POI 카테고리 시스템** — 대분류/중분류 카테고리 필터링, 검색
- **다국어 지원** — 한국어, 영어
- **멀티 플랫폼** — iOS 및 Android
- **Unity as a Library** — 네이티브 앱에 라이브러리로 통합

## Architecture

```
네이티브 앱 (iOS / Android)
  ├─ 네이티브 UI
  └─ Unity AR Module (UaaL)
       ├─ Naver ARCeye VLsdk (VPS 측위)
       ├─ AR Foundation (ARCore / ARKit)
       ├─ 3D 경로 렌더링
       ├─ VContainer DI
       ├─ UniRx MVVM
       └─ UniTask Async
```

### 씬 구조

```
네이티브 앱 ↔ load (메인/로딩) ↔ main (AR 네비게이션)
```

| 씬 | 역할 |
|---|---|
| `load` | Unity 진입점, POI 선택, 카테고리 시트 |
| `main` | AR 네비게이션, VLSDK 사용 |

## Project Structure

| 폴더 | 설명 |
|------|------|
| `Scripts` | Unity 서비스 로직 (Core, Manager, UI 등) |
| `VLSDK` | ARCeye VLSDK 설정 |
| `Prefabs` | UI 프리팹 |
| `Resources` | 동적 로드 애셋 (로컬라이제이션, 텍스처 등) |
| `Scenes` | 씬 파일 |
| `StreamingAssets` | POI 데이터, 카테고리 매칭 JSON |
| `Plugins` | 네이티브 플러그인 (iOS) |

## Design Patterns

| 패턴 | 적용 | 이유 |
|------|------|------|
| **VContainer DI** | 의존성 주입으로 클린 아키텍처 | AR/측위/내비 모듈 간 결합도 최소화 |
| **UniRx MVVM** | 리액티브 UI 바인딩 | 위치 변화 → 경로 업데이트 → UI 자동 반영 |
| **UniTask** | 비동기 처리 | VPS 측위, 경로 계산 시 AR 렌더링 블로킹 방지 |
| **UaaL** | 네이티브-Unity 모듈 분리 | 독립적 개발·배포, 네이티브 UI + AR 3D 공존 |

## Native Integration

### 네이티브 ↔ Unity 통신

양방향 메시지 패싱으로 네이티브 앱과 Unity 모듈이 통신합니다.

| 방향 | 메시지 예시 |
|------|------------|
| Native → Unity | 언어 설정, 뒤로가기, POI 선택 |
| Unity → Native | Unity Ready 알림, 네비게이션 완료 |

플랫폼별 브릿지 코드와 개발자 가이드 문서를 제공하여 네이티브 개발팀과 협업합니다.

## Build Delivery

| 플랫폼 | 빌드 결과물 |
|--------|------------|
| Android | `unityLibrary` Gradle 모듈 → 네이티브 앱에 모듈로 추가 |
| iOS | `UnityFramework.framework` + `VLSDK.framework` → Xcode 프로젝트에 임베드 |

## Tech Stack

`Unity 2022.3` `AR Foundation 5.x` `ARCore` `ARKit` `Naver ARCeye VLsdk` `URP 14.x` `VContainer` `UniRx` `UniTask` `C#`

## Technical Challenges

| 과제 | 해결 |
|------|------|
| 실내 GPS 불가 | VLsdk VPS로 카메라 기반 측위 |
| 다층 경로 탐색 | 층별 내비 메시 + 층간 이동 포인트 연결 그래프 |
| Native ↔ Unity 통신 | UaaL 네이티브 브릿지, 플랫폼별 메시지 채널 |
| 폴더블 기기 대응 | 화면 접기/펼치기 시 레이아웃 동적 재계산 |
| AR 렌더링 성능 | UniTask 비동기 처리로 메인 스레드 블로킹 방지 |

---

> 🔒 이 저장소는 private 프로젝트의 README 공개 버전입니다.
