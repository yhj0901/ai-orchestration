# AI Orchestration

AI 에이전트 오케스트레이션 기반 소프트웨어 개발 파이프라인

## 개요

Claude Code를 오케스트레이터로 활용하여, 기획 → 설계 → 구현 → QA → 문서화까지 전체 개발 흐름을 에이전트 기반으로 자동화합니다.

## 파이프라인

```
/기획시작 → GATE 1 승인 → /설계시작 → GATE 2 승인 → /백엔드시작 + /프론트시작 → /QA시작 → /문서시작
```

각 단계는 GATE 승인을 통해 다음 단계로 진행됩니다.

## 에이전트 구성 (`agents/`)

| 에이전트 | 파일 | 역할 |
|---------|------|------|
| 기획 | `planning.md` | 5개 서브 에이전트 토론 → 기획문서 생성 |
| 설계 | `design.md` | 6개 서브 에이전트 토론 → 설계문서(OpenAPI, DB, 보안) 생성 |
| 백엔드 | `backend.md` | FastAPI + Clean Architecture 구현 |
| 프론트엔드 | `frontend.md` | Next.js App Router + Clean Architecture 구현 |
| QA | `qa.md` | 통합 테스트 + 보안 테스트 + 시나리오 테스트 |
| 문서 | `docs.md` | 기술문서 + 사용자가이드 + 릴리즈노트 작성 |

## 커맨드 (`.claude/commands/`)

Claude Code에서 `/` 슬래시 커맨드로 실행합니다.

| 커맨드 | 설명 |
|--------|------|
| `/기획시작` | Jira 이슈 연동 + 기획 에이전트 실행 |
| `/설계시작` | GATE 1 승인 후 설계 에이전트 실행 |
| `/백엔드시작` | GATE 2 승인 후 백엔드 구현 시작 |
| `/프론트시작` | GATE 2 승인 후 프론트엔드 구현 시작 |
| `/QA시작` | 구현 완료 후 테스트 실행 |
| `/문서시작` | QA 완료 후 문서 자동 생성 |

## 기술 스택

- **아키텍처**: Clean Architecture
- **프론트엔드**: Next.js (App Router)
- **백엔드**: FastAPI
- **API 문서**: OpenAPI 3.1

## 산출물 구조

```
docs/
├── 기획/          기획문서
├── 설계/          architecture.md, openapi.yaml, db-schema.md, security-design.md
├── 테스트/        버그리포트, 시나리오테스트, 보안테스트
├── 기술문서/      개발자용 기술 문서
├── 사용자가이드/   사용자용 가이드
└── 릴리즈노트/    버전별 릴리즈 노트
```
