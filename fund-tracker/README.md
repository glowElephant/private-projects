# fund-tracker

커플 & 공금 자산 추적 대시보드. [currency-edge](https://github.com/glowElephant/currency-edge) 환율 자동매매 봇의 슬롯 데이터를 읽기 전용으로 참조하여 출처별 자산 현황을 관리합니다.

> **AI 챗봇 "봄이"** — 자연어로 자산 현황을 질의할 수 있는 요크셔테리어 캐릭터 챗봇

## 주요 기능

- **출처별 자산 추적** — 한아 / 승희 / 생활비 / 여행 (확장 가능)
- **기여도 기반 수익 분배** — 원금 비율대로 수익/손실 자동 계산
- **거래 기록** — 입금/출금 내역 관리 (currency-edge 데이터는 읽기 전용)
- **AI 챗봇 (봄이)** — Claude AI 연동, 자연어로 자산 현황 질의. 대화 캐싱 지원
- **심플 & 러블리 UI** — 파스텔 톤 도넛 차트, 모바일 반응형 SPA

## 작동 원리

1. currency-edge의 6개 슬롯 JSON을 **읽기 전용**으로 합산 → 총 자산
2. 사용자가 기록한 입출금 내역으로 출처별 원금 계산
3. `기여도 = 출처 원금 / 전체 원금`
4. `출처 현재가치 = 총 자산 × 기여도`
5. `수익 = 현재가치 - 원금`

## 구조

```
fund-tracker/
├── server.js          ← Express 서버 + API + Claude 채팅
├── public/
│   └── index.html     ← 대시보드 SPA
├── data/              ← 런타임 생성 (.gitignore)
│   ├── transactions.json
│   └── chat_history.json
└── .env               ← PORT, CURRENCY_EDGE_PATH
```

## 기술 스택

`Node.js` `Express` `Claude AI` `Vanilla HTML/CSS/JS` `JSON 파일 DB`

## 링크

- GitHub: [github.com/glowElephant/fund-tracker](https://github.com/glowElephant/fund-tracker)
