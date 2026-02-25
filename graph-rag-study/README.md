# GraphRAG 학습 & 실전 응용 프로젝트

빌더 조쉬의 ["RAG 어렵다고요?"](https://www.youtube.com/watch?v=AISuJHMCWog) 영상을 시작점으로, RAG/GraphRAG를 학습하고 **Claude Code용 하이브리드 RAG 시스템**을 구축한 프로젝트.

## 이 프로젝트가 하는 일

Claude Code에서 질문하면 **과거 세션 이력, 프로젝트 코드, 문서, 지식 그래프**에서 자동으로 관련 정보를 찾아서 답변에 활용합니다.

```
사용자: "OutlineController 연관 클래스가 뭐야?"

→ rag_agent가 자동 판단: "관계 질문이네 → 그래프 우선 검색"
→ 그래프에서 의존성/상속/필드 관계 탐색
→ 벡터 DB에서 관련 코드 보충 검색
→ Reranking으로 결과 재정렬
→ Claude Code가 정리해서 답변
```

## 시스템 구성

```
질문 입력
  │
  ▼
[Self-RAG] 검색 필요한가? ─── 불필요 → 스킵 (비용 절약)
  │ 필요
  ▼
[Agentic RAG] 질문 유형 자동 분류
  ├─ 관계/의존성 질문 → 그래프 우선 검색
  ├─ 과거 이력/에러 → 세션 우선 검색
  ├─ 코드/구현 상세 → 코드 우선 검색
  ├─ 규칙/문서/API → 문서 우선 검색
  └─ 판별 불가 → 하이브리드 전체 검색
  │
  ▼
[Reranking] 벡터 유사도 + 키워드 매칭 + 소스 가중치로 재정렬
  │
  ▼
결과 반환 → Claude Code가 활용
```

## 데이터 소스

| 소스 | 내용 | 규모 |
|------|------|------|
| 세션 이력 | Claude Code 과거 대화 (Windows + WSL) | 362개 세션, 5,010 청크 |
| C# 코드 | Visualizer_Unity 스크립트 | 110개 파일, 1,324 청크 |
| 프로젝트 문서 | API_REFERENCE, DEVLOG, CLAUDE.md 등 | 155 청크 |
| 규칙 파일 | .claude/rules/*.md | 9 청크 |
| 학습 문서 | RAG/GraphRAG 학습 노트 | 66 청크 |
| **지식 그래프** | 클래스 관계, 의존성, 에러→해결법 | **1,202 노드, 1,157 엣지** |

## 기술 스택

| 구성 요소 | 선택 | 이유 |
|---------|------|------|
| 벡터 DB | FAISS (faiss-cpu) | Python 3.14 호환 (Chroma 불가) |
| 그래프 | NetworkX + JSON | Neo4j 없이 로컬에서 충분 |
| 임베딩 | OpenAI text-embedding-3-small | 1536차원, 저렴 |
| 그래프 추출 | OpenAI gpt-4o-mini | 엔티티/관계 자동 추출 |
| 시각화 | pyvis | 인터랙티브 HTML 그래프 뷰어 |
| MCP 서버 | FastMCP (stdio) | Claude Code 도구로 직접 연결 |

---

## 새 PC 세팅 (원커맨드)

```bash
git clone https://github.com/glowElephant/graph-RAG-study.git C:/Git/graph-RAG-study
cd C:/Git/graph-RAG-study

# 대화형 (사람이 직접 실행)
python setup.py

# 비대화형 (Claude Code가 직접 실행)
python setup.py --pc server --key sk-여기에API키
```

`setup.py`가 자동으로:
1. PC 프로필 선택 (hostname 자동 감지, 미등록 PC는 새 프로필 자동 생성)
2. WSL 감지 → 세션 경로 자동 추가
3. 글로벌 Claude Code 설정 연결 (`dotfiles/` → `~/.claude/`)
4. `.env` 파일 생성 (OpenAI API 키)
5. Python 의존성 설치
6. MCP 서버 등록

### 최초 인덱싱 (data/가 비어있을 때만)

이미 data/가 git에 포함되어 있으므로, clone하면 인덱스도 함께 받아집니다.
**새로 인덱싱할 필요가 없습니다.** 바로 사용 가능.

만약 처음부터 다시 빌드하고 싶다면:
```bash
python rag-system/preprocessor.py      # 세션 전처리
python rag-system/indexer.py            # 벡터 인덱싱 (~$0.5~1)
python rag-system/graph_extractor.py    # 그래프 추출 (~$1~3)
```

### 일상 동기화 (증분 + 커밋 + 푸시)

```bash
python sync.py
```

sync.py가 하는 일:
1. `git pull` → 다른 PC의 최신 데이터 받기
2. `dotfiles/` → `~/.claude/` 설정 동기화
3. 증분 업데이트 → 내 세션을 공유 인덱스에 추가
4. `git commit & push` → 다른 PC에 공유

### PC별 설정 (config.json)

`rag-system/config.json`에 PC 프로필이 정의되어 있습니다:

```json
{
  "pc_profiles": {
    "company": {
      "name": "회사 PC",
      "session_dirs": ["~/.claude/projects", "//wsl$/Ubuntu/home/hanah/.claude/projects"],
      "unity_project": "C:/Git/Visualizer_Unity"
    },
    "server": {
      "session_dirs": ["~/.claude/projects"],
      "unity_project": null
    },
    "laptop": {
      "session_dirs": ["~/.claude/projects"],
      "unity_project": null
    }
  }
}
```

- `unity_project`가 null인 PC에서는 Unity 관련 인덱싱이 자동 스킵됩니다
- 새 PC 추가: 미등록 PC에서 `setup.py` 실행하면 자동 생성
- hostname으로 PC 자동 감지 (`--pc` 인자로 직접 지정도 가능)
- WSL이 설치되어 있으면 세션 경로 자동 추가
- `config.local.json`은 각 PC에서 setup.py가 생성 (gitignore)

### 주의사항

- **동시에 2대에서 sync.py 돌리면 안 됨** (FAISS 바이너리 충돌)
- 한 PC에서 sync → push 확인 → 다른 PC에서 sync
- `.env` 파일은 각 PC에서 개별 생성 (API 키, gitignore)
- 심볼릭 링크가 안 되면 (Windows 개발자 모드 꺼짐) 자동으로 복사 대체

### 글로벌 설정 (dotfiles/)

`dotfiles/` 폴더에 3대 PC가 공유하는 Claude Code 설정이 있습니다:
- `CLAUDE.md` — 글로벌 지시사항 (RAG 도구 우선 사용 규칙 포함)
- `settings.json` — Claude Code 설정
- `keybindings.json` — 키바인딩

수정하면 `sync.py`가 자동으로 커밋+푸시하고, 다른 PC에서 sync하면 반영됩니다.

---

---

## MCP 도구 목록

| 도구 | 용도 |
|------|------|
| `rag_agent` | **메인 도구** - 질문 자동 분류 → 최적 전략 선택 → 검색 → Reranking |
| `rag_search` | 벡터 검색 (소스 필터 가능) |
| `rag_search_sessions` | 과거 세션 전용 검색 |
| `rag_search_code` | C# 코드 전용 검색 |
| `rag_search_docs` | 프로젝트 문서 전용 검색 |
| `rag_search_graph` | 지식 그래프 탐색 (관계/의존성) |
| `rag_hybrid_search` | 벡터 + 그래프 동시 검색 |
| `rag_stats` | 벡터 DB 상태 |
| `rag_graph_stats` | 그래프 상태 |

---

## 프로젝트 구조

```
graph-RAG-study/
├── setup.py                        # 새 PC 셋업 (한 번만 실행)
├── sync.py                         # 증분 + 커밋 + 푸시 (주기적 실행)
├── dotfiles/                       # 글로벌 Claude Code 설정 (3대 PC 공유)
│   ├── CLAUDE.md                   # ~/.claude/CLAUDE.md 원본
│   ├── settings.json               # ~/.claude/settings.json 원본
│   └── keybindings.json            # ~/.claude/keybindings.json 원본
├── rag-system/                     # 핵심 코드
│   ├── config.json                 # PC 프로필 + 모델 설정
│   ├── config.local.json           # 현재 PC 식별 (gitignore)
│   ├── preprocessor.py             # 세션 JSONL → 마크다운 전처리
│   ├── indexer.py                  # FAISS 벡터 인덱싱
│   ├── graph_extractor.py          # GraphRAG 엔티티/관계 추출
│   ├── visualize_graph.py          # 지식 그래프 HTML 시각화
│   ├── mcp_server.py               # MCP 서버 (전체 RAG 기능)
│   └── incremental_update.py       # 증분 업데이트
├── data/                           # RAG 데이터 (git, 3대 PC 공유)
│   ├── vector_db/                  # FAISS 인덱스 + 청크
│   ├── knowledge_graph.json        # NetworkX 지식 그래프
│   ├── graph_viewer.html           # 인터랙티브 그래프 시각화
│   ├── processed_sessions/         # 전처리된 세션
│   └── update_state.json           # 증분 업데이트 상태
├── docs/                           # 학습 문서
├── video/                          # 영상 자료
├── .env                            # OpenAI API 키 (gitignore)
├── CLAUDE.md                       # 프로젝트 컨텍스트
└── README.md                       # 이 파일
```

## 구현 로드맵 (전체 완료)

- [x] **Step 1: 벡터 RAG** - FAISS 인덱싱, MCP 서버, 벡터 검색
- [x] **Step 2: GraphRAG** - 엔티티/관계 추출, 그래프 검색, 시각화
- [x] **Step 3: Agentic RAG** - 질문 유형 자동 분류, 전략 자동 선택, 폴백
- [x] **Step 4: Self-RAG + Reranking** - 검색 필요성 판단, 결과 재정렬

## 온톨로지 (그래프 스키마) 현황

현재는 **정식 온톨로지 없이 LLM이 자유 추출**한 상태입니다.

`graph_extractor.py`의 프롬프트에 대략적인 유형만 지시하고, gpt-4o-mini가 알아서 분류했습니다.

### 현재 상태의 문제점

| 문제 | 상세 |
|------|------|
| unknown 노드 279개 (23%) | LLM이 유형을 판단 못한 노드들 |
| 노드 유형 17종으로 분산 | field 332, unknown 279, feature 172, class 157, file 100 등 |
| 프롬프트에 없는 관계 생성 | `implemented_by`, `has_property` 등 LLM 재량으로 만든 것 |
| 엔티티명 중복 가능성 | 같은 대상이 다른 이름으로 등록될 수 있음 (정규화 없음) |

### 개선 방향 (미구현)

1. **노드 유형 고정**: class, field, component, shader, file, error, solution, enum, struct, method
2. **관계 유형 허용 목록**: depends_on, has_field, requires, uses, inherits, controls, solved_by, caused_by, related_to, modified
3. **unknown 금지**: LLM에게 반드시 분류하도록 강제
4. **엔티티명 정규화**: 대소문자 통일, .cs 접미사 처리 등
5. 온톨로지 정의 후 `graph_extractor.py` 프롬프트 수정 → 재추출

온톨로지를 설계하면 그래프 품질이 크게 향상되지만, 현재 상태로도 기본적인 관계 탐색은 정상 작동합니다.

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| Chroma + Python 3.14 비호환 | FAISS(`faiss-cpu`)로 대체 |
| OpenAI Rate Limit (429) | exponential backoff 재시도 로직 내장 |
| `~/.mcp.json` 인식 안 됨 | `claude mcp add --scope user` 사용 |
| MCP 서버 안 뜸 | `claude mcp list`로 Connected 확인, Python 경로 점검 |

## 비용 정리

| 작업 | 비용 | 빈도 |
|------|------|------|
| 초기 벡터 인덱싱 (6,564청크) | ~$0.5~1 | 1회 |
| 초기 그래프 추출 (140개 소스) | ~$1~3 | 1회 |
| 증분 업데이트 (세션 1개) | ~$0.001~0.01 | 수시 |
| 검색 쿼리 1회 | ~$0.00001 | 매 질문 |
| **총 초기 비용** | **~$2~4** | |

## 학습 문서

RAG/GraphRAG 개념부터 이해하고 싶다면:

1. **[RAG 개념 정리](docs/rag-basics.md)** - RAG가 뭔지, 왜 필요한지
2. **[GraphRAG 개념 정리](docs/graphrag-overview.md)** - 지식 그래프 기반 RAG
3. **[RAG vs GraphRAG 비교](docs/comparison.md)** - 언제 무엇을 쓸까
4. **[도구 및 라이브러리](docs/tools-and-libs.md)** - 실전 도구 선택 가이드

원본 영상: 빌더 조쉬 ["RAG 어렵다고요?"](https://www.youtube.com/watch?v=AISuJHMCWog)
