# 에이전트 스킬

AI 코딩 에이전트를 위한 스킬 모음입니다. 스킬은 에이전트의 역량을 확장하는 패키지된 지침과 스크립트입니다.
이 레포지토리는 [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills/)의 react-best-practices를 한국어로 번역한 내용입니다.

스킬은 [Agent Skills](https://agentskills.io/) 형식을 따릅니다.

## 사용 가능한 스킬

### react-best-practices

Vercel Engineering의 React 및 Next.js 성능 최적화 가이드라인입니다. 영향도에 따라 우선순위를 둔 8개 카테고리의 40개 이상 규칙을 포함합니다.

**다음 상황에서 사용:**

- 새로운 React 컴포넌트 또는 Next.js 페이지 작성
- 데이터 페칭 구현(클라이언트 또는 서버 사이드)
- 성능 문제 코드 리뷰
- 번들 크기 또는 로딩 시간 최적화

**포함된 카테고리:**

- 워터폴 제거 (Critical)
- 번들 크기 최적화 (Critical)
- 서버 사이드 성능 (High)
- 클라이언트 사이드 데이터 페칭 (Medium-High)
- 리렌더 최적화 (Medium)
- 렌더링 성능 (Medium)
- JavaScript 마이크로 최적화 (Low-Medium)

### web-design-guidelines

웹 인터페이스 모범 사례 준수 여부를 확인하기 위해 UI 코드를 리뷰합니다. 접근성, 성능, UX를 포함한 100개 이상의 규칙으로 코드를 점검합니다.

**다음 상황에서 사용:**

- "Review my UI"
- "Check accessibility"
- "Audit design"
- "Review UX"
- "Check my site against best practices"

**포함된 카테고리:**

- 접근성 (aria-labels, 시맨틱 HTML, 키보드 핸들러)
- 포커스 상태 (표시되는 포커스, focus-visible 패턴)
- 폼 (autocomplete, 검증, 오류 처리)
- 애니메이션 (prefers-reduced-motion, 컴포지터 친화적 변환)
- 타이포그래피 (curly quotes, ellipsis, tabular-nums)
- 이미지 (크기 지정, 지연 로딩, alt 텍스트)
- 성능 (가상화, 레이아웃 스래싱, preconnect)
- 내비게이션 & 상태 (URL에 상태 반영, 딥링킹)
- 다크 모드 & 테마 (color-scheme, theme-color 메타)
- 터치 & 인터랙션 (touch-action, tap-highlight)
- 로케일 & i18n (Intl.DateTimeFormat, Intl.NumberFormat)

## 스킬 구조

각 스킬에는 다음이 포함됩니다:

- `SKILL.md` - 에이전트용 지침
- `scripts/` - 자동화를 위한 헬퍼 스크립트(선택)
- `references/` - 보조 문서(선택)

## 라이선스

MIT
