# COEX Indoor AR Navigation

COEX 실내 공간에서 AR로 길을 안내하는 내비게이션 앱.

> 기존 12개월 이상 소요 예상 프로젝트를 AI 활용하여 **3개월 내 완료**

## 핵심 기능

- **Naver ARCeye VLsdk** 기반 실내 정밀 측위 (VPS) — 카메라 영상만으로 위치 인식
- **다층 건물 실내 내비게이션** — 층간 이동, 경로 탐색, POI 검색
- **Unity as a Library (UaaL)** 아키텍처 — Flutter 앱에 Unity AR 모듈 통합
- **XREAL AR 글래스** 컨텐츠 연동 프로토타입
- iOS/Android 네이티브 브릿지 통신

## 아키텍처

```
Flutter App (Host)
  └─ Unity AR Module (UaaL)
       ├─ Naver VLsdk (VPS 측위)
       ├─ AR Foundation (ARCore/ARKit)
       ├─ VContainer DI
       ├─ UniRx MVVM
       └─ UniTask Async
```

## 기술 스택

`Unity` `AR Foundation` `ARCore` `ARKit` `Naver VLsdk` `Flutter` `VContainer` `UniRx` `UniTask`

## 설계 패턴

| 패턴 | 적용 |
|------|------|
| **VContainer DI** | 의존성 주입으로 클린 아키텍처 구성 |
| **UniRx MVVM** | 리액티브 프로그래밍 기반 UI 바인딩 |
| **UniTask** | 비동기 처리로 AR 렌더링 성능 최적화 |
| **UaaL** | 호스트 앱(Flutter)과 Unity 간 모듈 분리 |

---

> 🔒 이 저장소는 private 프로젝트의 README 공개 버전입니다.
