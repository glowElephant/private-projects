# VTS Visualizer

3D 디지털 트윈 에디터/뷰어 시스템. 원전 시설 등 산업 설비의 실시간 데이터를 3D에 통합, 예측 유지보수를 지원합니다.

> 기존 12개월 소요 규모의 시스템을 AI 활용하여 **1개월 내 핵심기능 완료**

## 시스템 구조

```
Editor (Unity Editor 커스텀)
  → .vproj (ZIP: AssetBundle + manifest.json + attachments/)
    → Viewer.exe (독립 실행 파일)
```

## 에디터 구조 (3-View 시스템)

| View | 설명 |
|------|------|
| **Tree View** | 모델 계층 구조, 검색/필터링, 데이터 상태 뱃지 (M/T/D/E/A) |
| **3D View** | 카메라 조작 (Orbit/Pan/Dolly/Fly), Perspective/Isometric, 씬 프리셋 6종, 3점 조명 |
| **Editor View** | 6단계 워크플로우 — Model → Data → Effects → Viewers → Network → Export |

## 시각 효과 컨트롤러 (9종)

| 효과 | 설명 |
|------|------|
| Outline | 외곽선 효과, 5가지 모드 |
| X-Ray | Fresnel 기반 반투명 |
| Alarm | 경고 점멸, 맥동 발광 |
| Cross Section | 단면 절단, 최대 6개 평면 |
| Disassembly | 분해/조립 애니메이션, 방향 6종, 속도 0.5x~3x |
| Ghost Effect | 잔상 효과, 이동 궤적 |
| Wireframe | 와이어프레임 표시 |
| Animation | FBX 애니메이션 재생 |
| Transform | 회전/이동/크기 애니메이션 |

## 노드 기반 애니메이션 플로우

- **Event 노드**: onClick, onHover, onDelay(n초), onTagValue(서버 값 수신)
- **Action 노드**: Rotation, Position, Scale, Color, Clip(FBX 재생)
- 병렬 Fork 분기, 1회성(One-time) 옵션, 다중 플로우 지원

## 데이터 바인딩

- **MetaData**: 오브젝트별 Key-Value 속성 (JSON 일괄 임포트)
- **Tag Binding**: 서버 태그 값 연동 (값에 따라 색상/상태 변경)
- **Linked Documents**: PDF, Video, Image, Excel, JSON/CSV, Web URL

## 서버 연동

| 프로토콜 | 용도 |
|---------|------|
| REST API | 태그 목록 조회 |
| Socket.IO | 실시간 Push |
| LiveKit WebRTC | 웹 스트리밍 Presenter |
| Named Pipe | 프로젝트 교체 |

## 기술 스택

`Unity 6` `URP 17.3` `C#` `HLSL Shader` `UI Toolkit` `Socket.IO` `WebRTC` `REST API`

**외부 플러그인**: Paroxe PDFRenderer · Vuplex WebView · ExcelDataReader

---

> 🔒 이 저장소는 private 프로젝트의 README 공개 버전입니다.
