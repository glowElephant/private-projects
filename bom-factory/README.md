# Bom Factory

강아지 인터뷰 영상 자동 생성 GUI 툴

## 기능

1. **이미지 준비**: 강아지 이미지 업로드 및 AI 변형
2. **인터뷰 콘티**: Gemini와 대화하며 콘티 작성
3. **영상 생성**: Veo 3로 8초 클립 생성 (음성 포함)
4. **영상 합치기**: 클립 이어붙여 최종 영상 생성

## 기술 스택

- **GUI**: PyQt6
- **텍스트 생성**: Gemini 2.0 Flash
- **이미지 생성**: Imagen 4
- **영상 생성**: Veo 3

## 설치

```bash
# 의존성 설치
pip install -r requirements.txt

# FFmpeg 설치 필요 (영상 처리용)
# Windows: https://ffmpeg.org/download.html
```

## 실행

```bash
python run.py
```

## 설정

1. `config/api_keys.yaml`에 Google API 키 입력
2. API 키 발급: https://aistudio.google.com
3. 유료 기능 사용 시 Google Cloud 결제 설정 필요

## 예상 비용

| 항목 | 모델 | 비용 |
|------|------|------|
| 텍스트 | Gemini 2.0 Flash | 거의 무료 |
| 이미지 | Imagen 4 | $0.04/장 |
| 영상 | Veo 3 (8초) | $6.00 |
| 영상 | Veo 3 Fast (8초) | $1.20 |

## 폴더 구조

```
Bom_Factory/
├── src/                    # 소스 코드
│   ├── gui/               # GUI 컴포넌트
│   ├── core/              # 핵심 로직
│   ├── api/               # API 클라이언트
│   └── utils/             # 유틸리티
├── projects/              # 프로젝트 데이터
├── config/                # 설정 파일
└── logs/                  # 로그
```

## 라이선스

MIT License
