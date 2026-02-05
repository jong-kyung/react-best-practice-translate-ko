---
name: vercel-react-best-practices
description: Vercel Engineering의 React 및 Next.js 성능 최적화 가이드라인. React/Next.js 코드를 작성, 리뷰, 리팩터링할 때 최적 성능 패턴을 보장하기 위해 사용하는 스킬이다. React 컴포넌트, Next.js 페이지, 데이터 페칭, 번들 최적화, 성능 개선 작업에서 트리거된다.
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# Vercel React Best Practice

Vercel에서 유지 관리하는 React 및 Next.js 애플리케이션 성능 최적화 종합 가이드다. 영향도 우선순위에 따라 8개 카테고리, 45개 규칙으로 구성되어 자동 리팩터링과 코드 생성에 참고할 수 있다.

## 적용 시점

다음 상황에서 이 가이드를 참고한다:

- 새로운 React 컴포넌트 또는 Next.js 페이지 작성
- 데이터 페칭 구현(클라이언트/서버)
- 성능 이슈 코드 리뷰
- 기존 React/Next.js 코드 리팩터링
- 번들 크기 또는 로드 시간 최적화

## 우선순위별 규칙 카테고리

| 우선순위 | 카테고리                  | 영향도      | 접두사       |
| -------- | ------------------------- | ----------- | ------------ |
| 1        | 워터폴 제거               | CRITICAL    | `async-`     |
| 2        | 번들 크기 최적화          | CRITICAL    | `bundle-`    |
| 3        | 서버 측 성능              | HIGH        | `server-`    |
| 4        | 클라이언트 측 데이터 페칭 | MEDIUM-HIGH | `client-`    |
| 5        | 리렌더 최적화             | MEDIUM      | `rerender-`  |
| 6        | 렌더링 성능               | MEDIUM      | `rendering-` |
| 7        | 자바스크립트 성능         | LOW-MEDIUM  | `js-`        |
| 8        | 고급 패턴                 | LOW         | `advanced-`  |

## 빠른 참고

### 1. 워터폴 제거 (CRITICAL)

- `async-defer-await` - 필요한 분기 안으로 await 이동
- `async-parallel` - 독립 작업에 Promise.all() 사용
- `async-dependencies` - 부분 의존성에는 better-all 사용
- `async-api-routes` - API 라우트에서 프라미스는 빨리 시작하고 늦게 await
- `async-suspense-boundaries` - Suspense로 콘텐츠 스트리밍

### 2. 번들 크기 최적화 (CRITICAL)

- `bundle-barrel-imports` - 배럴 파일 피하고 직접 임포트
- `bundle-dynamic-imports` - 무거운 컴포넌트에 next/dynamic 사용
- `bundle-defer-third-party` - 분석/로깅은 하이드레이션 이후 로드
- `bundle-conditional` - 기능 활성화 시에만 모듈 로드
- `bundle-preload` - 호버/포커스 시 프리로드로 체감 속도 개선

### 3. 서버 측 성능 (HIGH)

- `server-cache-react` - 요청 내 중복 제거에 React.cache() 사용
- `server-cache-lru` - 요청 간 캐싱에 LRU 캐시 사용
- `server-serialization` - 클라이언트 컴포넌트로 전달하는 데이터 최소화
- `server-parallel-fetching` - 컴포넌트를 재구성해 페칭 병렬화
- `server-after-nonblocking` - 논블로킹 작업에 after() 사용

### 4. 클라이언트 측 데이터 페칭 (MEDIUM-HIGH)

- `client-swr-dedup` - SWR로 요청 중복 제거
- `client-event-listeners` - 전역 이벤트 리스너 중복 제거

### 5. 리렌더 최적화 (MEDIUM)

- `rerender-defer-reads` - 콜백에서만 쓰는 상태는 구독하지 않기
- `rerender-memo` - 비싼 작업을 메모이즈 컴포넌트로 분리
- `rerender-dependencies` - effect에 원시값 의존성 사용
- `rerender-derived-state` - 연속 값 대신 파생된 불리언에 구독
- `rerender-functional-setstate` - 안정적 콜백을 위해 함수형 setState 사용
- `rerender-lazy-state-init` - 비싼 초기값은 useState에 함수 전달
- `rerender-transitions` - 긴급하지 않은 업데이트에 startTransition 사용

### 6. 렌더링 성능 (MEDIUM)

- `rendering-animate-svg-wrapper` - SVG 요소 대신 div 래퍼 애니메이션
- `rendering-content-visibility` - 긴 목록에 content-visibility 적용
- `rendering-hoist-jsx` - 정적 JSX를 컴포넌트 밖으로 추출
- `rendering-svg-precision` - SVG 좌표 정밀도 줄이기
- `rendering-hydration-no-flicker` - 클라이언트 전용 데이터에 인라인 스크립트 사용
- `rendering-activity` - 표시/숨김에 Activity 컴포넌트 사용
- `rendering-conditional-render` - 조건 렌더링은 && 대신 삼항 사용

### 7. 자바스크립트 성능 (LOW-MEDIUM)

- `js-batch-dom-css` - CSS 변경은 클래스/cssText로 묶기
- `js-index-maps` - 반복 조회는 Map 생성
- `js-cache-property-access` - 루프에서 객체 프로퍼티 캐시
- `js-cache-function-results` - 함수 결과를 모듈 레벨 Map으로 캐시
- `js-cache-storage` - localStorage/sessionStorage 읽기 캐시
- `js-combine-iterations` - 여러 filter/map을 한 루프로 합치기
- `js-length-check-first` - 비싼 비교 전에 배열 길이 확인
- `js-early-exit` - 함수는 조기 반환
- `js-hoist-regexp` - RegExp 생성은 루프 밖으로 끌어올리기
- `js-min-max-loop` - 정렬 대신 루프로 min/max 찾기
- `js-set-map-lookups` - O(1) 조회에 Set/Map 사용
- `js-tosorted-immutable` - 불변성을 위해 toSorted() 사용

### 8. 고급 패턴 (LOW)

- `advanced-event-handler-refs` - 이벤트 핸들러를 ref에 저장
- `advanced-use-latest` - 안정적인 콜백 ref를 위한 useLatest

## 사용 방법

자세한 설명과 코드 예시는 개별 규칙 파일을 확인한다:

```
rules/async-parallel.md
rules/bundle-barrel-imports.md
rules/_sections.md
```

각 규칙 파일에는 다음이 포함된다:

- 왜 중요한지에 대한 간단한 설명
- 잘못된 코드 예시와 설명
- 올바른 코드 예시와 설명
- 추가 맥락과 참고 링크

## 전체 통합 문서

모든 규칙을 확장한 전체 가이드는 `AGENTS.md`를 참고한다.
