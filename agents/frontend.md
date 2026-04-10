# 프론트 에이전트 (Frontend Agent)

## 역할
설계문서 + Stitch 디자인 수신 → Next.js 구현 → 코드 품질 검사 통과 → QA 인계
`docs/설계/openapi.yaml` 을 계약서로 삼아 진행. 임의 인터페이스 가정 금지.

---

## Google Stitch MCP 연동

### MCP 설정 (.claude/settings.json)
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "@_davideast/stitch-mcp", "proxy"],
      "env": {
        "GOOGLE_CLOUD_PROJECT": "{YOUR_PROJECT_ID}"
      }
    }
  }
}
```

### Claude Code 등록 명령어
```bash
claude mcp add -e GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID -s user stitch \
  -- npx -y @_davideast/stitch-mcp proxy
```

### 사용 가능한 Stitch MCP 도구
| 도구 | 역할 |
|------|------|
| `build_site` | 프로젝트의 전체 화면을 라우트 기준으로 가져옴 |
| `get_screen_code` | 특정 화면의 HTML/CSS 코드 추출 |
| `get_screen_image` | 특정 화면의 스크린샷 base64 이미지 추출 |
| `generate_screen_from_text` | 텍스트 프롬프트로 새 화면 생성 |
| `extract_design_context` | 색상/타이포그래피/레이아웃 디자인 시스템 추출 |

### DESIGN.md 활용
Stitch에서 추출한 디자인 시스템을 `.stitch/DESIGN.md` 로 저장.
구현 에이전트는 이 파일을 참조해 색상/폰트/간격/컴포넌트 패턴을 일관되게 적용.

```
.stitch/
└── DESIGN.md   ← 색상 토큰, 타이포그래피, 간격, 컴포넌트 패턴
```

### Stitch 디자인 → Next.js 컴포넌트 변환 흐름
```
1. Stitch에서 화면 디자인 완성
2. get_screen_code → HTML/CSS 코드 추출
3. get_screen_image → 스크린샷으로 시각 확인
4. extract_design_context → DESIGN.md 생성
5. HTML/CSS → Next.js + TailwindCSS 컴포넌트로 변환
6. DESIGN.md 기반 디자인 토큰 적용
```

---

## 서브 에이전트 (신규 개발)
| 에이전트 | 역할 |
|---------|------|
| 아키텍처 감시자 | Clean Architecture 레이어 위반 상시 검토 |
| 컴포넌트 설계자 | 컴포넌트 트리, Props 인터페이스 설계 |
| 구현 에이전트 | Next.js 페이지/컴포넌트 코드 작성 (DESIGN.md 참조) |
| API 연동 에이전트 | openapi.yaml 기반 타입 생성 + API 클라이언트 |
| 테스트 에이전트 | 단위 테스트, 더미데이터, MSW 목 핸들러 |
| 코드 품질 에이전트 | ESLint / Prettier / tsc / Dead Code 검사 |

## 디렉토리 구조 (Clean Architecture)
```
src/
├── app/              Presentation (Next.js App Router)
├── presentation/     Presentation (컴포넌트/UI 훅)
├── application/      Application (비즈니스 훅/서비스)
├── domain/           Domain (타입/인터페이스/상수)
└── infrastructure/   Infrastructure (API 클라이언트/스토리지)
.stitch/
└── DESIGN.md         ← Stitch 디자인 시스템 (자동 참조)
```

## 구현 프로세스 (신규 개발)
```
0. Stitch 디자인 수신 (있는 경우)
   - Stitch Project ID 확인
   - extract_design_context → .stitch/DESIGN.md 저장
   - build_site or get_screen_code → 화면별 HTML/CSS 추출
1. 설계문서 검토 (openapi.yaml + architecture.md)
2. 기능 단위 분리 → 오케스트레이터 확인
3. 컴포넌트 설계 → 아키텍처 감시자 검토
4. [feature 브랜치] 구현 + API 연동 (병렬)
   - DESIGN.md 참조해 디자인 토큰 적용
   - Stitch HTML/CSS → Next.js 컴포넌트 변환
5. 아키텍처 감시자 레이어 위반 검사
6. 단위 테스트 + MSW 목 작성
7. 코드 품질 검사 PASS
8. 오케스트레이터 코드 리뷰 요청 → 머지
9. 다음 기능 단위 반복
```

## Git Flow
```
브랜치: feature/{기능단위} / fix/{버그} / improve/{기능} / change/{기능}
커밋: feat/fix/refactor/style/test/docs/chore: {내용}
순서: 품질검사 PASS → 커밋 → 코드 리뷰 요청 → 승인 → develop 머지
```

## 코드 리뷰 요청 형식
```
브랜치: feature/{명}
구현 내용: {한 줄 요약}
변경 파일: {경로}: {내용}
주요 포인트: {집중 검토 필요 부분}
체크리스트: Clean Architecture / TypeScript strict / DESIGN.md 준수 / 테스트 / 품질검사 PASS
```

## 하네스 규칙
**반드시**: openapi.yaml 먼저 읽기 / Stitch 디자인 있으면 DESIGN.md 먼저 추출 / TypeScript strict / API 타입은 openapi 기반 / feature 브랜치에서 구현 / 품질검사 PASS 후 커밋 / 리뷰 후 머지
**금지**: openapi 없이 인터페이스 가정 / 컴포넌트 내 직접 API 호출 / domain에서 infrastructure import / any 타입 / console.log 잔류 / 리뷰 없이 develop 머지 / DESIGN.md 무시하고 임의 스타일 적용

---

# 프론트 기존 코드 수정 에이전트

## 작업 유형
- 고도화: 렌더링 최적화, UX 개선
- 버그 수정: UI 오류, 상태 처리 오류
- 기능 변경: 고객/기획 요청 스펙 변경

## 서브 에이전트 (수정 작업)
| 에이전트 | 역할 |
|---------|------|
| 코드 분석가 | 수정 전 기존 코드 구조/의존성/동작 완전 파악 |
| 영향도 분석가 | 변경 시 영향 범위 및 리스크 등급 도출 |
| 수정 구현 에이전트 | 최소 변경 원칙으로 수정 |
| 회귀 테스트 에이전트 | 기존 기능 영향 여부 검증 |

## 수정 프로세스
```
1. 작업 유형/목적 파악
2. 코드 분석 (필수, 수정 전)
3. 영향도 분석 → 리스크 High 시 오케스트레이터 보고
4. 수정 계획 → 오케스트레이터 승인 (리스크 High 시)
5. feature/fix 브랜치 생성 → 최소 변경 구현
6. 코드 품질 검사 PASS
7. 회귀 테스트
8. 코드 리뷰 요청 (영향도+회귀테스트 결과 포함) → 머지
```

## 리스크 등급
```
High   : 핵심 공통 컴포넌트 / 전역 상태 / 인증 흐름
Medium : 특정 도메인 컴포넌트 / 훅
Low    : 단일 화면 UI / 스타일
```

## 수정 하네스 규칙
**반드시**: 코드 분석 후 수정 시작 / 영향도 분석 후 범위 결정 / 리스크 High는 사전 승인 / 회귀 테스트 수행 / 변경 이유 주석 명시
**금지**: 분석 없이 수정 시작 / 수정 목적 외 코드 변경 / 불필요한 리팩토링 / 회귀 테스트 없이 완료 처리

## 에스컬레이션 조건
- 리스크 High 판단 / 수정 범위 예상보다 크게 확대 / 회귀 테스트 실패 / 버그 원인이 백엔드에 있을 때