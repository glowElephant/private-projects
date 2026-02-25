# GameCI Hub

4개 Unity 프로젝트를 하나로 통합한 CI/CD 파이프라인.

> **빌드 시간 70% 단축** 달성

## 주요 기능

- **4개 Unity 프로젝트 통합** 빌드 파이프라인
- Android APK/AAR, iOS XCFramework **자동 빌드 & 배포**
- Self-hosted Runner 구성 (Linux + macOS ARM64)
- Docker 컨테이너 기반 빌드 환경 표준화
- Slack 알림, 자동 릴리즈

## 아키텍처

```
GitHub Actions Workflow
  ├─ Self-hosted Runner (Linux)
  │    └─ GameCI Docker Container
  │         ├─ Android APK/AAB 빌드
  │         ├─ Android AAR (React 임베딩) 빌드
  │         └─ Library 빌드
  ├─ Self-hosted Runner (macOS ARM64)
  │    └─ iOS XCFramework 빌드
  └─ Release & Slack 알림
```

## 핵심 설계

- 빌드환경은 일정하게 유지, 내부 저장소를 교체해가며 빌드
- GitHub App으로 리눅스 러너가 저장소 접근권한 부여
- Game CI Docker = Unity 버전별 도커 이미지

## 기술 스택

`GitHub Actions` `GameCI` `Docker` `Gradle` `Shell Script`

---

> 🔒 이 저장소는 private 프로젝트의 README 공개 버전입니다.
