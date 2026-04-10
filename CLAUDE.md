# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

AI 에이전트 오케스트레이션 기반 소프트웨어 개발 파이프라인. Claude Code가 오케스트레이터 역할을 수행하며, GATE 승인 기반으로 기획 → 설계 → 구현 → QA → 문서화 흐름을 관리한다.

## Pipeline

```
/기획시작 → GATE 1 승인 → /설계시작 → GATE 2 승인 → /백엔드시작 + /프론트시작 → /QA시작 → /문서시작
```

- GATE 승인은 사용자가 "기획 승인", "설계 승인" 등으로 입력
- 각 커맨드는 `agents/` 폴더의 해당 에이전트 하네스를 먼저 로드한 뒤 실행
- 에이전트 하네스의 "반드시/금지" 규칙을 항상 준수할 것

## Tech Stack

- Architecture: Clean Architecture (레이어 건너뛰기 금지, 의존성 방향은 항상 Domain 안쪽)
- Frontend: Next.js App Router
- Backend: FastAPI
- API Spec: OpenAPI 3.1 — `docs/설계/openapi.yaml`이 프론트/백엔드 계약서
- DB: SQLAlchemy + Alembic
- Quality: Ruff / mypy --strict / bandit (백엔드), ESLint / Prettier / tsc (프론트)

## Git Convention

- Branch: `feature/{기능}` / `fix/{버그}` / `improve/{기능}` / `change/{기능}`
- Commit: `feat/fix/refactor/style/test/docs/chore/db: {내용}`
- Flow: 품질검사 PASS → 커밋 → 코드 리뷰 요청 → 승인 → develop 머지
