# React 베스트 프랙티스

**버전 1.0.0**  
Vercel Engineering  
2026년 1월

> **참고:**  
> 이 문서는 Vercel에서 React 및 Next.js 코드베이스를 유지보수,  
> 생성, 리팩터링할 때 에이전트와 LLM이 따르도록 작성되었다.  
> 사람에게도 유용할 수 있지만, 여기의 가이드는 AI 보조  
> 워크플로에서 자동화와 일관성을 위해 최적화되어 있다.

---

## 요약

AI 에이전트와 LLM을 위해 설계된 React 및 Next.js 애플리케이션의 종합 성능 최적화 가이드다. 영향도 기준으로 치명적(워터폴 제거, 번들 크기 감소)에서 점진적(고급 패턴)까지 우선순위를 매긴 8개 카테고리의 40+ 규칙을 포함한다. 각 규칙에는 잘못된 구현과 올바른 구현을 비교하는 실전 예시와, 자동 리팩터링 및 코드 생성을 안내하는 구체적 영향도 지표가 포함된다.

---

## 목차

1. [워터폴 제거](#1-eliminating-waterfalls) — **CRITICAL**
   - 1.1 [필요할 때까지 await 지연](#11-defer-await-until-needed)
   - 1.2 [의존성 기반 병렬화](#12-dependency-based-parallelization)
   - 1.3 [API 라우트에서 워터폴 체인 방지](#13-prevent-waterfall-chains-in-api-routes)
   - 1.4 [독립적인 작업에 Promise.all()](#14-promiseall-for-independent-operations)
   - 1.5 [전략적 Suspense 경계](#15-strategic-suspense-boundaries)
2. [번들 크기 최적화](#2-bundle-size-optimization) — **CRITICAL**
   - 2.1 [배럴 파일 임포트 피하기](#21-avoid-barrel-file-imports)
   - 2.2 [조건부 모듈 로딩](#22-conditional-module-loading)
   - 2.3 [중요하지 않은 서드파티 라이브러리 지연 로드](#23-defer-non-critical-third-party-libraries)
   - 2.4 [무거운 컴포넌트는 동적 임포트](#24-dynamic-imports-for-heavy-components)
   - 2.5 [사용자 의도 기반 프리로드](#25-preload-based-on-user-intent)
3. [서버 측 성능](#3-server-side-performance) — **HIGH**
   - 3.1 [요청 간 LRU 캐싱](#31-cross-request-lru-caching)
   - 3.2 [RSC 경계의 직렬화 최소화](#32-minimize-serialization-at-rsc-boundaries)
   - 3.3 [컴포넌트 조합으로 데이터 병렬 페칭](#33-parallel-data-fetching-with-component-composition)
   - 3.4 [React.cache()로 요청 내 중복 제거](#34-per-request-deduplication-with-reactcache)
   - 3.5 [차단하지 않는 작업에 after() 사용](#35-use-after-for-non-blocking-operations)
4. [클라이언트 측 데이터 페칭](#4-client-side-data-fetching) — **MEDIUM-HIGH**
   - 4.1 [전역 이벤트 리스너 중복 제거](#41-deduplicate-global-event-listeners)
   - 4.2 [스크롤 성능을 위해 passive 이벤트 리스너 사용](#42-use-passive-event-listeners-for-scrolling-performance)
   - 4.3 [SWR로 자동 중복 제거](#43-use-swr-for-automatic-deduplication)
   - 4.4 [localStorage 데이터 버전 관리 및 최소화](#44-version-and-minimize-localstorage-data)
5. [리렌더 최적화](#5-re-render-optimization) — **MEDIUM**
   - 5.1 [사용 지점에서 상태 읽기 지연](#51-defer-state-reads-to-usage-point)
   - 5.2 [메모이즈 컴포넌트로 추출](#52-extract-to-memoized-components)
   - 5.3 [이펙트 의존성 좁히기](#53-narrow-effect-dependencies)
   - 5.4 [파생 상태에 구독](#54-subscribe-to-derived-state)
   - 5.5 [함수형 setState 업데이트 사용](#55-use-functional-setstate-updates)
   - 5.6 [지연 상태 초기화 사용](#56-use-lazy-state-initialization)
   - 5.7 [긴급하지 않은 업데이트에 Transition 사용](#57-use-transitions-for-non-urgent-updates)
6. [렌더링 성능](#6-rendering-performance) — **MEDIUM**
   - 6.1 [SVG 요소 대신 SVG 래퍼를 애니메이션](#61-animate-svg-wrapper-instead-of-svg-element)
   - 6.2 [긴 목록에 CSS content-visibility](#62-css-content-visibility-for-long-lists)
   - 6.3 [정적 JSX 요소 끌어올리기](#63-hoist-static-jsx-elements)
   - 6.4 [SVG 정밀도 최적화](#64-optimize-svg-precision)
   - 6.5 [깜빡임 없이 하이드레이션 불일치 방지](#65-prevent-hydration-mismatch-without-flickering)
   - 6.6 [표시/숨김에 Activity 컴포넌트 사용](#66-use-activity-component-for-showhide)
   - 6.7 [명시적 조건부 렌더링 사용](#67-use-explicit-conditional-rendering)
7. [자바스크립트 성능](#7-javascript-performance) — **LOW-MEDIUM**
   - 7.1 [DOM CSS 변경 배치](#71-batch-dom-css-changes)
   - 7.2 [반복 조회용 인덱스 맵 만들기](#72-build-index-maps-for-repeated-lookups)
   - 7.3 [루프에서 프로퍼티 접근 캐시](#73-cache-property-access-in-loops)
   - 7.4 [반복 함수 호출 캐시](#74-cache-repeated-function-calls)
   - 7.5 [Storage API 호출 캐시](#75-cache-storage-api-calls)
   - 7.6 [배열 반복을 하나로 합치기](#76-combine-multiple-array-iterations)
   - 7.7 [배열 비교 전에 길이 먼저 확인](#77-early-length-check-for-array-comparisons)
   - 7.8 [함수에서 조기 반환](#78-early-return-from-functions)
   - 7.9 [RegExp 생성 끌어올리기](#79-hoist-regexp-creation)
   - 7.10 [정렬 대신 루프로 최소/최대 찾기](#710-use-loop-for-minmax-instead-of-sort)
   - 7.11 [O(1) 조회에 Set/Map 사용](#711-use-setmap-for-o1-lookups)
   - 7.12 [불변성을 위해 sort() 대신 toSorted()](#712-use-tosorted-instead-of-sort-for-immutability)
8. [고급 패턴](#8-advanced-patterns) — **LOW**
   - 8.1 [이벤트 핸들러를 ref에 저장](#81-store-event-handlers-in-refs)
   - 8.2 [안정적인 콜백 ref를 위한 useLatest](#82-uselatest-for-stable-callback-refs)

---

<a id="1-eliminating-waterfalls"></a>

## 1. 워터폴 제거

**영향도: CRITICAL**

워터폴은 성능을 가장 크게 저하시키는 원인이다. 연속된 await마다 전체 네트워크 지연이 추가된다. 이를 제거하면 가장 큰 성능 향상을 얻는다.

<a id="11-defer-await-until-needed"></a>

### 1.1 필요할 때까지 await 지연

**영향도: HIGH (사용되지 않는 코드 경로의 차단 방지)**

필요한 분기 안으로 `await` 작업을 옮겨, 필요하지 않은 코드 경로가 막히지 않도록 한다.

**잘못된 예: 양쪽 분기 모두 차단**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId);

  if (skipProcessing) {
    // Returns immediately but still waited for userData
    return { skipped: true };
  }

  // Only this branch uses userData
  return processUserData(userData);
}
```

**올바른 예: 필요할 때만 차단**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // Returns immediately without waiting
    return { skipped: true };
  }

  // Fetch only when needed
  const userData = await fetchUserData(userId);
  return processUserData(userData);
}
```

**다른 예시: 조기 반환 최적화**

```typescript
// Incorrect: always fetches permissions
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId);
  const resource = await getResource(resourceId);

  if (!resource) {
    return { error: "Not found" };
  }

  if (!permissions.canEdit) {
    return { error: "Forbidden" };
  }

  return await updateResourceData(resource, permissions);
}

// Correct: fetches only when needed
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId);

  if (!resource) {
    return { error: "Not found" };
  }

  const permissions = await fetchPermissions(userId);

  if (!permissions.canEdit) {
    return { error: "Forbidden" };
  }

  return await updateResourceData(resource, permissions);
}
```

이 최적화는 건너뛰는 분기가 자주 발생하거나, 지연한 작업이 비용이 큰 경우 특히 효과적이다.

<a id="12-dependency-based-parallelization"></a>

### 1.2 의존성 기반 병렬화

**영향도: CRITICAL (2-10배 개선)**

부분 의존성이 있는 작업에는 `better-all`을 사용해 병렬성을 최대화한다. 각 작업을 가능한 가장 이른 시점에 자동으로 시작한다.

**잘못된 예: 프로필이 불필요하게 설정을 기다림**

```typescript
const [user, config] = await Promise.all([fetchUser(), fetchConfig()]);
const profile = await fetchProfile(user.id);
```

**올바른 예: 설정과 프로필을 병렬 실행**

```typescript
import { all } from "better-all";

const { user, config, profile } = await all({
  async user() {
    return fetchUser();
  },
  async config() {
    return fetchConfig();
  },
  async profile() {
    return fetchProfile((await this.$.user).id);
  },
});
```

참고: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)

<a id="13-prevent-waterfall-chains-in-api-routes"></a>

### 1.3 API 라우트에서 워터폴 체인 방지

**영향도: CRITICAL (2-10배 개선)**

API 라우트와 Server Action에서는 서로 독립적인 작업을 즉시 시작하고, 아직 await 하지 않더라도 바로 실행되게 한다.

**잘못된 예: 설정이 인증을 기다리고, 데이터가 둘 다 기다림**

```typescript
export async function GET(request: Request) {
  const session = await auth();
  const config = await fetchConfig();
  const data = await fetchData(session.user.id);
  return Response.json({ data, config });
}
```

**올바른 예: 인증과 설정을 즉시 시작**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth();
  const configPromise = fetchConfig();
  const session = await sessionPromise;
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id),
  ]);
  return Response.json({ data, config });
}
```

더 복잡한 의존성 체인이 있다면 `better-all`을 사용해 병렬성을 자동으로 극대화한다(의존성 기반 병렬화 참고).

<a id="14-promiseall-for-independent-operations"></a>

### 1.4 독립적인 작업에 Promise.all()

**영향도: CRITICAL (2-10배 개선)**

비동기 작업 사이에 의존성이 없다면 `Promise.all()`로 동시에 실행한다.

**잘못된 예: 순차 실행, 왕복 3회**

```typescript
const user = await fetchUser();
const posts = await fetchPosts();
const comments = await fetchComments();
```

**올바른 예: 병렬 실행, 왕복 1회**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments(),
]);
```

<a id="15-strategic-suspense-boundaries"></a>

### 1.5 전략적 Suspense 경계

**영향도: HIGH (초기 페인트 속도 향상)**

JSX를 반환하기 전에 async 컴포넌트에서 데이터를 await 하지 말고, Suspense 경계를 사용해 데이터 로딩 동안 래퍼 UI를 더 빠르게 보여준다.

**잘못된 예: 데이터 페칭 때문에 래퍼가 차단**

```tsx
async function Page() {
  const data = await fetchData(); // Blocks entire page

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  );
}
```

중간 섹션만 데이터가 필요한데도 전체 레이아웃이 데이터를 기다린다.

**올바른 예: 래퍼는 즉시 표시, 데이터는 스트리밍**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  );
}

async function DataDisplay() {
  const data = await fetchData(); // Only blocks this component
  return <div>{data.content}</div>;
}
```

Sidebar, Header, Footer는 즉시 렌더링되고 DataDisplay만 데이터를 기다린다.

**대안: 컴포넌트 간 프라미스 공유**

```tsx
function Page() {
  // Start fetch immediately, but don't await
  const dataPromise = fetchData();

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  );
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise); // Unwraps the promise
  return <div>{data.content}</div>;
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise); // Reuses the same promise
  return <div>{data.summary}</div>;
}
```

두 컴포넌트가 같은 프라미스를 공유하므로 한 번만 fetch 된다. 둘이 함께 기다리는 동안 레이아웃은 즉시 렌더된다.

**이 패턴을 쓰지 말아야 할 때:**

- 레이아웃 결정에 필요한 핵심 데이터(배치에 영향)

- 폴드 상단의 SEO 핵심 콘텐츠

- Suspense 오버헤드가 아까운 작고 빠른 쿼리

- 레이아웃 시프트(로딩 → 콘텐츠 점프)를 피하고 싶을 때

**트레이드오프:** 초기 페인트 속도 vs 잠재적 레이아웃 시프트. UX 우선순위에 따라 선택한다.

---

<a id="2-bundle-size-optimization"></a>

## 2. 번들 크기 최적화

**영향도: CRITICAL**

초기 번들 크기를 줄이면 TTI(Time to Interactive)와 LCP(Largest Contentful Paint)가 개선된다.

<a id="21-avoid-barrel-file-imports"></a>

### 2.1 배럴 파일 임포트 피하기

**영향도: CRITICAL (200-800ms 임포트 비용, 느린 빌드)**

배럴 파일 대신 소스 파일에서 직접 임포트해 사용하지 않는 수천 개 모듈 로딩을 피한다. **배럴 파일**은 여러 모듈을 다시 내보내는 엔트리포인트다(예: `index.js`가 `export * from './module'`을 수행).

인기 아이콘/컴포넌트 라이브러리는 엔트리 파일에 **최대 10,000개의 재-export**가 있을 수 있다. 많은 React 패키지는 **임포트만 하는 데 200-800ms**가 걸려 개발 속도와 프로덕션 콜드 스타트에 영향을 준다.

**트리 쉐이킹이 도움이 안 되는 이유:** 라이브러리가 external(번들 미포함)로 표시되면 번들러가 최적화할 수 없다. 트리 쉐이킹을 위해 번들에 포함하면 전체 모듈 그래프를 분석하느라 빌드가 크게 느려진다.

**잘못된 예: 라이브러리 전체 임포트**

```tsx
import { Check, X, Menu } from "lucide-react";
// Loads 1,583 modules, takes ~2.8s extra in dev
// Runtime cost: 200-800ms on every cold start

import { Button, TextField } from "@mui/material";
// Loads 2,225 modules, takes ~4.2s extra in dev
```

**올바른 예: 필요한 것만 임포트**

```tsx
import Check from "lucide-react/dist/esm/icons/check";
import X from "lucide-react/dist/esm/icons/x";
import Menu from "lucide-react/dist/esm/icons/menu";
// Loads only 3 modules (~2KB vs ~1MB)

import Button from "@mui/material/Button";
import TextField from "@mui/material/TextField";
// Loads only what you use
```

**대안: Next.js 13.5+**

```js
// next.config.js - use optimizePackageImports
module.exports = {
  experimental: {
    optimizePackageImports: ["lucide-react", "@mui/material"],
  },
};

// Then you can keep the ergonomic barrel imports:
import { Check, X, Menu } from "lucide-react";
// Automatically transformed to direct imports at build time
```

직접 임포트는 개발 부팅을 15-70% 빠르게, 빌드를 28% 빠르게, 콜드 스타트를 40% 빠르게 하며 HMR도 크게 빨라진다.

자주 영향을 받는 라이브러리: `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`.

참고: [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)

<a id="22-conditional-module-loading"></a>

### 2.2 조건부 모듈 로딩

**영향도: HIGH (필요할 때만 큰 데이터 로드)**

기능이 활성화될 때만 큰 데이터나 모듈을 로드한다.

**예시: 애니메이션 프레임 lazy-load**

```tsx
function AnimationPlayer({
  enabled,
  setEnabled,
}: {
  enabled: boolean;
  setEnabled: React.Dispatch<React.SetStateAction<boolean>>;
}) {
  const [frames, setFrames] = useState<Frame[] | null>(null);

  useEffect(() => {
    if (enabled && !frames && typeof window !== "undefined") {
      import("./animation-frames.js")
        .then((mod) => setFrames(mod.frames))
        .catch(() => setEnabled(false));
    }
  }, [enabled, frames, setEnabled]);

  if (!frames) return <Skeleton />;
  return <Canvas frames={frames} />;
}
```

`typeof window !== 'undefined'` 체크는 SSR에서 이 모듈이 번들에 포함되지 않게 해 서버 번들 크기와 빌드 속도를 최적화한다.

<a id="23-defer-non-critical-third-party-libraries"></a>

### 2.3 중요하지 않은 서드파티 라이브러리 지연 로드

**영향도: MEDIUM (하이드레이션 이후 로드)**

분석, 로깅, 에러 추적은 사용자 상호작용을 막지 않는다. 하이드레이션 이후에 로드한다.

**잘못된 예: 초기 번들을 막음**

```tsx
import { Analytics } from "@vercel/analytics/react";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

**올바른 예: 하이드레이션 이후 로드**

```tsx
import dynamic from "next/dynamic";

const Analytics = dynamic(
  () => import("@vercel/analytics/react").then((m) => m.Analytics),
  { ssr: false },
);

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

<a id="24-dynamic-imports-for-heavy-components"></a>

### 2.4 무거운 컴포넌트는 동적 임포트

**영향도: CRITICAL (TTI와 LCP에 직접 영향)**

초기 렌더에 필요하지 않은 큰 컴포넌트는 `next/dynamic`으로 지연 로드한다.

**잘못된 예: Monaco가 메인 청크에 번들링되어 ~300KB**

```tsx
import { MonacoEditor } from "./monaco-editor";

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />;
}
```

**올바른 예: Monaco를 필요할 때 로드**

```tsx
import dynamic from "next/dynamic";

const MonacoEditor = dynamic(
  () => import("./monaco-editor").then((m) => m.MonacoEditor),
  { ssr: false },
);

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />;
}
```

<a id="25-preload-based-on-user-intent"></a>

### 2.5 사용자 의도 기반 프리로드

**영향도: MEDIUM (체감 지연 감소)**

필요하기 전에 무거운 번들을 프리로드해 체감 지연을 줄인다.

**예시: 호버/포커스 시 프리로드**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== "undefined") {
      void import("./monaco-editor");
    }
  };

  return (
    <button onMouseEnter={preload} onFocus={preload} onClick={onClick}>
      Open Editor
    </button>
  );
}
```

**예시: 기능 플래그가 켜졌을 때 프리로드**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== "undefined") {
      void import("./monaco-editor").then((mod) => mod.init());
    }
  }, [flags.editorEnabled]);

  return (
    <FlagsContext.Provider value={flags}>{children}</FlagsContext.Provider>
  );
}
```

`typeof window !== 'undefined'` 체크는 SSR에서 프리로드 모듈이 번들에 포함되지 않게 해 서버 번들 크기와 빌드 속도를 최적화한다.

---

<a id="3-server-side-performance"></a>

## 3. 서버 측 성능

**영향도: HIGH**

서버 렌더링과 데이터 페칭을 최적화하면 서버 측 워터폴을 제거하고 응답 시간을 줄일 수 있다.

<a id="31-cross-request-lru-caching"></a>

### 3.1 요청 간 LRU 캐싱

**영향도: HIGH (요청 간 캐시)**

`React.cache()`는 하나의 요청 안에서만 동작한다. 연속 요청 간 공유되는 데이터(사용자가 A 버튼을 누르고 B 버튼을 누르는 경우 등)에는 LRU 캐시를 사용한다.

**구현:**

```typescript
import { LRUCache } from "lru-cache";

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000, // 5 minutes
});

export async function getUser(id: string) {
  const cached = cache.get(id);
  if (cached) return cached;

  const user = await db.user.findUnique({ where: { id } });
  cache.set(id, user);
  return user;
}

// Request 1: DB query, result cached
// Request 2: cache hit, no DB query
```

사용자가 연속적으로 여러 엔드포인트를 호출하면서 같은 데이터를 수 초 내에 필요로 할 때 유용하다.

**Vercel의 [Fluid Compute](https://vercel.com/docs/fluid-compute)에서는:** 여러 동시 요청이 동일한 함수 인스턴스와 캐시를 공유할 수 있어 LRU 캐싱이 특히 효과적이다. 외부 저장소(예: Redis) 없이도 요청 간 캐시가 유지된다.

**전통적 서버리스에서는:** 각 호출이 격리되어 있으므로 요청 간 캐싱이 필요하다면 Redis를 고려한다.

참고: [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)

<a id="32-minimize-serialization-at-rsc-boundaries"></a>

### 3.2 RSC 경계의 직렬화 최소화

**영향도: HIGH (데이터 전송 크기 감소)**

React Server/Client 경계는 모든 객체 프로퍼티를 문자열로 직렬화해 HTML 응답과 이후 RSC 요청에 포함한다. 이 직렬화된 데이터는 페이지 무게와 로드 시간에 직접 영향을 주므로 **크기가 매우 중요**하다. 클라이언트가 실제로 사용하는 필드만 전달한다.

**잘못된 예: 필드 50개 전부 직렬화**

```tsx
async function Page() {
  const user = await fetchUser(); // 50 fields
  return <Profile user={user} />;
}

("use client");
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>; // uses 1 field
}
```

**올바른 예: 필드 1개만 직렬화**

```tsx
async function Page() {
  const user = await fetchUser();
  return <Profile name={user.name} />;
}

("use client");
function Profile({ name }: { name: string }) {
  return <div>{name}</div>;
}
```

<a id="33-parallel-data-fetching-with-component-composition"></a>

### 3.3 컴포넌트 조합으로 데이터 병렬 페칭

**영향도: CRITICAL (서버 워터폴 제거)**

React Server Components는 트리 안에서 순차적으로 실행된다. 컴포지션으로 재구성해 데이터 페칭을 병렬화한다.

**잘못된 예: Sidebar가 Page의 fetch를 기다림**

```tsx
export default async function Page() {
  const header = await fetchHeader();
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  );
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}
```

**올바른 예: 둘 다 동시에 fetch**

```tsx
async function Header() {
  const data = await fetchHeader();
  return <div>{data}</div>;
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  );
}
```

**children prop을 사용하는 대안:**

```tsx
async function Header() {
  const data = await fetchHeader();
  return <div>{data}</div>;
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}

function Layout({ children }: { children: ReactNode }) {
  return (
    <div>
      <Header />
      {children}
    </div>
  );
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  );
}
```

<a id="34-per-request-deduplication-with-reactcache"></a>

### 3.4 React.cache()로 요청 내 중복 제거

**영향도: MEDIUM (요청 내 중복 제거)**

서버 사이드 요청 중복 제거에는 `React.cache()`를 사용한다. 인증과 데이터베이스 쿼리에 특히 효과적이다.

**사용법:**

```typescript
import { cache } from "react";

export const getCurrentUser = cache(async () => {
  const session = await auth();
  if (!session?.user?.id) return null;
  return await db.user.findUnique({
    where: { id: session.user.id },
  });
});
```

하나의 요청 안에서 `getCurrentUser()`를 여러 번 호출해도 쿼리는 한 번만 실행된다.

**인라인 객체 인자는 피한다:**

`React.cache()`는 얕은 비교(`Object.is`)로 캐시 히트를 판단한다. 인라인 객체는 호출마다 새 참조를 만들어 캐시가 맞지 않는다.

**잘못된 예: 항상 캐시 미스**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } });
});

// Each call creates new object, never hits cache
getUser({ uid: 1 });
getUser({ uid: 1 }); // Cache miss, runs query again
```

**올바른 예: 캐시 히트**

```typescript
const params = { uid: 1 };
getUser(params); // Query runs
getUser(params); // Cache hit (same reference)
```

객체를 꼭 써야 한다면 같은 참조를 전달한다:

**Next.js 전용 참고:**

Next.js에서는 `fetch` API가 요청 메모이제이션으로 확장되어 동일한 URL과 옵션의 요청은 하나의 요청 안에서 자동으로 중복 제거된다. 따라서 `React.cache()`가 `fetch`에는 필요 없다. 하지만 다음 작업에는 여전히 `React.cache()`가 중요하다:

- 데이터베이스 쿼리(Prisma, Drizzle 등)

- 무거운 연산

- 인증 체크

- 파일 시스템 작업

- `fetch`가 아닌 모든 비동기 작업

컴포넌트 트리 전반에서 이런 작업을 중복 제거하려면 `React.cache()`를 사용한다.

참고: [https://react.dev/reference/react/cache](https://react.dev/reference/react/cache)

<a id="35-use-after-for-non-blocking-operations"></a>

### 3.5 차단하지 않는 작업에 after() 사용

**영향도: MEDIUM (더 빠른 응답 시간)**

응답이 전송된 뒤 실행해야 하는 작업은 Next.js의 `after()`로 스케줄한다. 로깅, 분석, 기타 사이드 이펙트가 응답을 막지 않게 한다.

**잘못된 예: 응답을 막음**

```tsx
import { logUserAction } from "@/app/utils";

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request);

  // Logging blocks the response
  const userAgent = request.headers.get("user-agent") || "unknown";
  await logUserAction({ userAgent });

  return new Response(JSON.stringify({ status: "success" }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
}
```

**올바른 예: 논블로킹**

```tsx
import { after } from "next/server";
import { headers, cookies } from "next/headers";
import { logUserAction } from "@/app/utils";

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request);

  // Log after response is sent
  after(async () => {
    const userAgent = (await headers()).get("user-agent") || "unknown";
    const sessionCookie =
      (await cookies()).get("session-id")?.value || "anonymous";

    logUserAction({ sessionCookie, userAgent });
  });

  return new Response(JSON.stringify({ status: "success" }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
}
```

응답은 즉시 전송되고 로깅은 백그라운드에서 진행된다.

**일반적인 사용 사례:**

- 분석(Analytics) 추적

- 감사 로그

- 알림 전송

- 캐시 무효화

- 정리 작업

**중요한 참고:**

- 응답이 실패하거나 리다이렉트되더라도 `after()`는 실행된다

- Server Actions, Route Handlers, Server Components에서 동작한다

참고: [https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)

---

<a id="4-client-side-data-fetching"></a>

## 4. 클라이언트 측 데이터 페칭

**영향도: MEDIUM-HIGH**

자동 중복 제거와 효율적인 데이터 페칭 패턴은 중복 네트워크 요청을 줄인다.

<a id="41-deduplicate-global-event-listeners"></a>

### 4.1 전역 이벤트 리스너 중복 제거

**영향도: LOW (N 컴포넌트에 리스너 1개)**

컴포넌트 인스턴스 간 전역 이벤트 리스너를 공유하려면 `useSWRSubscription()`을 사용한다.

**잘못된 예: N 인스턴스 = N 리스너**

```tsx
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback();
      }
    };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  }, [key, callback]);
}
```

`useKeyboardShortcut` 훅을 여러 번 사용하면 각 인스턴스가 새 리스너를 등록한다.

**올바른 예: N 인스턴스 = 리스너 1개**

```tsx
import useSWRSubscription from "swr/subscription";

// Module-level Map to track callbacks per key
const keyCallbacks = new Map<string, Set<() => void>>();

function useKeyboardShortcut(key: string, callback: () => void) {
  // Register this callback in the Map
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set());
    }
    keyCallbacks.get(key)!.add(callback);

    return () => {
      const set = keyCallbacks.get(key);
      if (set) {
        set.delete(callback);
        if (set.size === 0) {
          keyCallbacks.delete(key);
        }
      }
    };
  }, [key, callback]);

  useSWRSubscription("global-keydown", () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach((cb) => cb());
      }
    };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  });
}

function Profile() {
  // Multiple shortcuts will share the same listener
  useKeyboardShortcut("p", () => {
    /* ... */
  });
  useKeyboardShortcut("k", () => {
    /* ... */
  });
  // ...
}
```

<a id="42-use-passive-event-listeners-for-scrolling-performance"></a>

### 4.2 스크롤 성능을 위해 passive 이벤트 리스너 사용

**영향도: MEDIUM (이벤트 리스너로 인한 스크롤 지연 제거)**

touch와 wheel 이벤트 리스너에 `{ passive: true }`를 추가해 즉시 스크롤이 가능하도록 한다. 브라우저는 기본적으로 리스너가 `preventDefault()`를 호출하는지 확인하기 위해 완료를 기다리며, 이 때문에 스크롤이 지연될 수 있다.

**잘못된 예:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX);
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY);

  document.addEventListener("touchstart", handleTouch);
  document.addEventListener("wheel", handleWheel);

  return () => {
    document.removeEventListener("touchstart", handleTouch);
    document.removeEventListener("wheel", handleWheel);
  };
}, []);
```

**올바른 예:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX);
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY);

  document.addEventListener("touchstart", handleTouch, { passive: true });
  document.addEventListener("wheel", handleWheel, { passive: true });

  return () => {
    document.removeEventListener("touchstart", handleTouch);
    document.removeEventListener("wheel", handleWheel);
  };
}, []);
```

**passive 사용:** 트래킹/분석, 로깅, `preventDefault()`를 호출하지 않는 모든 리스너.

**passive 미사용:** 커스텀 스와이프 제스처, 커스텀 줌 컨트롤처럼 `preventDefault()`가 필요한 리스너.

<a id="43-use-swr-for-automatic-deduplication"></a>

### 4.3 SWR로 자동 중복 제거

**영향도: MEDIUM-HIGH (자동 중복 제거)**

SWR은 컴포넌트 인스턴스 간 요청 중복 제거, 캐싱, 재검증을 제공한다.

**잘못된 예: 중복 제거 없음, 각 인스턴스가 개별 fetch**

```tsx
function UserList() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch("/api/users")
      .then((r) => r.json())
      .then(setUsers);
  }, []);
}
```

**올바른 예: 여러 인스턴스가 한 요청 공유**

```tsx
import useSWR from "swr";

function UserList() {
  const { data: users } = useSWR("/api/users", fetcher);
}
```

**불변 데이터용:**

```tsx
import { useImmutableSWR } from "@/lib/swr";

function StaticContent() {
  const { data } = useImmutableSWR("/api/config", fetcher);
}
```

**변경 작업용:**

```tsx
import { useSWRMutation } from "swr/mutation";

function UpdateButton() {
  const { trigger } = useSWRMutation("/api/user", updateUser);
  return <button onClick={() => trigger()}>Update</button>;
}
```

참고: [https://swr.vercel.app](https://swr.vercel.app)

<a id="44-version-and-minimize-localstorage-data"></a>

### 4.4 localStorage 데이터 버전 관리 및 최소화

**영향도: MEDIUM (스키마 충돌 방지, 저장 크기 감소)**

키에 버전 접두사를 추가하고 필요한 필드만 저장한다. 스키마 충돌과 민감 데이터의 우발적 저장을 방지한다.

**잘못된 예:**

```typescript
// No version, stores everything, no error handling
localStorage.setItem("userConfig", JSON.stringify(fullUserObject));
const data = localStorage.getItem("userConfig");
```

**올바른 예:**

```typescript
const VERSION = "v2";

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config));
  } catch {
    // Throws in incognito/private browsing, quota exceeded, or disabled
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`);
    return data ? JSON.parse(data) : null;
  } catch {
    return null;
  }
}

// Migration from v1 to v2
function migrate() {
  try {
    const v1 = localStorage.getItem("userConfig:v1");
    if (v1) {
      const old = JSON.parse(v1);
      saveConfig({
        theme: old.darkMode ? "dark" : "light",
        language: old.lang,
      });
      localStorage.removeItem("userConfig:v1");
    }
  } catch {}
}
```

**서버 응답에서 최소 필드만 저장:**

```typescript
// User object has 20+ fields, only store what UI needs
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem(
      "prefs:v1",
      JSON.stringify({
        theme: user.preferences.theme,
        notifications: user.preferences.notifications,
      }),
    );
  } catch {}
}
```

**항상 try-catch로 감싼다:** `getItem()`과 `setItem()`은 시크릿/프라이빗 브라우징(Safari, Firefox), 용량 초과, 비활성화 상태에서 예외를 던진다.

**효과:** 버전 관리를 통한 스키마 진화, 저장 크기 감소, 토큰/PII/내부 플래그 저장 방지.

---

<a id="5-re-render-optimization"></a>

## 5. 리렌더 최적화

**영향도: MEDIUM**

불필요한 리렌더를 줄이면 낭비되는 계산을 최소화하고 UI 반응성이 좋아진다.

<a id="51-defer-state-reads-to-usage-point"></a>

### 5.1 사용 지점에서 상태 읽기 지연

**영향도: MEDIUM (불필요한 구독 방지)**

콜백에서만 읽는 동적 상태(searchParams, localStorage)에 구독하지 않는다.

**잘못된 예: 모든 searchParams 변경에 구독**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams();

  const handleShare = () => {
    const ref = searchParams.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```

**올바른 예: 필요할 때 읽기, 구독 없음**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search);
    const ref = params.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```

<a id="52-extract-to-memoized-components"></a>

### 5.2 메모이즈 컴포넌트로 추출

**영향도: MEDIUM (조기 반환 가능)**

비싼 작업을 메모이즈된 컴포넌트로 추출해 계산 전에 조기 반환할 수 있게 한다.

**잘못된 예: 로딩 중에도 avatar 계산**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user);
    return <Avatar id={id} />;
  }, [user]);

  if (loading) return <Skeleton />;
  return <div>{avatar}</div>;
}
```

**올바른 예: 로딩 중 계산 생략**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user]);
  return <Avatar id={id} />;
});

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />;
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  );
}
```

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있으면 `memo()`와 `useMemo()`로 수동 메모이즈할 필요가 없다. 컴파일러가 리렌더를 자동 최적화한다.

<a id="53-narrow-effect-dependencies"></a>

### 5.3 이펙트 의존성 좁히기

**영향도: LOW (이펙트 재실행 최소화)**

이펙트 재실행을 최소화하려면 객체 대신 원시값 의존성을 지정한다.

**잘못된 예: 사용자 필드 변경마다 재실행**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user]);
```

**올바른 예: id가 바뀔 때만 재실행**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user.id]);
```

**파생 상태는 effect 밖에서 계산:**

```tsx
// Incorrect: runs on width=767, 766, 765...
useEffect(() => {
  if (width < 768) {
    enableMobileMode();
  }
}, [width]);

// Correct: runs only on boolean transition
const isMobile = width < 768;
useEffect(() => {
  if (isMobile) {
    enableMobileMode();
  }
}, [isMobile]);
```

<a id="54-subscribe-to-derived-state"></a>

### 5.4 파생 상태에 구독

**영향도: MEDIUM (리렌더 빈도 감소)**

연속적인 값 대신 파생된 불리언 상태에 구독해 리렌더 빈도를 줄인다.

**잘못된 예: 픽셀 변화마다 리렌더**

```tsx
function Sidebar() {
  const width = useWindowWidth(); // updates continuously
  const isMobile = width < 768;
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```

**올바른 예: 불리언이 바뀔 때만 리렌더**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery("(max-width: 767px)");
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```

<a id="55-use-functional-setstate-updates"></a>

### 5.5 함수형 setState 업데이트 사용

**영향도: MEDIUM (stale closure 및 불필요한 콜백 재생성 방지)**

현재 상태 값을 기반으로 상태를 업데이트할 때는 상태 변수를 직접 참조하지 말고 setState의 함수형 업데이트를 사용한다. 이렇게 하면 stale closure를 막고, 불필요한 의존성을 제거하며, 안정적인 콜백 참조를 만든다.

**잘못된 예: 상태 의존성 필요**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // Callback must depend on items, recreated on every items change
  const addItems = useCallback(
    (newItems: Item[]) => {
      setItems([...items, ...newItems]);
    },
    [items],
  ); // ❌ items dependency causes recreations

  // Risk of stale closure if dependency is forgotten
  const removeItem = useCallback((id: string) => {
    setItems(items.filter((item) => item.id !== id));
  }, []); // ❌ Missing items dependency - will use stale items!

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

첫 번째 콜백은 `items`가 바뀔 때마다 다시 만들어져 자식 컴포넌트를 불필요하게 리렌더링할 수 있다. 두 번째 콜백은 stale closure 버그가 있어 항상 초기 `items` 값을 참조한다.

**올바른 예: 안정적 콜백, stale closure 없음**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // Stable callback, never recreated
  const addItems = useCallback((newItems: Item[]) => {
    setItems((curr) => [...curr, ...newItems]);
  }, []); // ✅ No dependencies needed

  // Always uses latest state, no stale closure risk
  const removeItem = useCallback((id: string) => {
    setItems((curr) => curr.filter((item) => item.id !== id));
  }, []); // ✅ Safe and stable

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

**장점:**

1. **안정적인 콜백 참조** - 상태가 바뀌어도 콜백을 다시 만들 필요가 없다

2. **stale closure 없음** - 항상 최신 상태 값으로 동작한다

3. **의존성 감소** - 의존성 배열이 단순해지고 메모리 누수가 줄어든다

4. **버그 예방** - React 클로저 버그의 가장 흔한 원인을 제거한다

**함수형 업데이트를 사용할 때:**

- 현재 상태 값에 의존하는 모든 setState

- 상태가 필요한 useCallback/useMemo 내부

- 상태를 참조하는 이벤트 핸들러

- 상태를 갱신하는 비동기 작업

**직접 업데이트가 괜찮은 경우:**

- 고정 값으로 설정: `setCount(0)`

- props/인자만으로 설정: `setName(newName)`

- 이전 값에 의존하지 않는 경우

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있으면 일부 케이스를 자동 최적화하지만, 정확성과 stale closure 방지를 위해 함수형 업데이트를 권장한다.

<a id="56-use-lazy-state-initialization"></a>

### 5.6 지연 상태 초기화 사용

**영향도: MEDIUM (매 렌더마다 낭비되는 계산)**

비싼 초기값은 `useState`에 함수를 전달한다. 함수형을 쓰지 않으면 초기화 로직이 값이 한 번만 쓰여도 매 렌더마다 실행된다.

**잘못된 예: 매 렌더마다 실행**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() runs on EVERY render, even after initialization
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items));
  const [query, setQuery] = useState("");

  // When query changes, buildSearchIndex runs again unnecessarily
  return <SearchResults index={searchIndex} query={query} />;
}

function UserProfile() {
  // JSON.parse runs on every render
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem("settings") || "{}"),
  );

  return <SettingsForm settings={settings} onChange={setSettings} />;
}
```

**올바른 예: 한 번만 실행**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() runs ONLY on initial render
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items));
  const [query, setQuery] = useState("");

  return <SearchResults index={searchIndex} query={query} />;
}

function UserProfile() {
  // JSON.parse runs only on initial render
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem("settings");
    return stored ? JSON.parse(stored) : {};
  });

  return <SettingsForm settings={settings} onChange={setSettings} />;
}
```

localStorage/sessionStorage에서 초기값을 읽거나, 데이터 구조(인덱스, 맵)를 만들거나, DOM을 읽거나, 무거운 변환을 수행할 때 지연 초기화를 사용한다.

단순 원시값(`useState(0)`), 직접 참조(`useState(props.value)`), 값싼 리터럴(`useState({})`)에는 함수형이 필요 없다.

<a id="57-use-transitions-for-non-urgent-updates"></a>

### 5.7 긴급하지 않은 업데이트에 Transition 사용

**영향도: MEDIUM (UI 반응성 유지)**

자주 발생하지만 긴급하지 않은 상태 업데이트는 transition으로 표시해 UI 반응성을 유지한다.

**잘못된 예: 스크롤마다 UI가 막힘**

```tsx
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY);
    window.addEventListener("scroll", handler, { passive: true });
    return () => window.removeEventListener("scroll", handler);
  }, []);
}
```

**올바른 예: 논블로킹 업데이트**

```tsx
import { startTransition } from "react";

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY));
    };
    window.addEventListener("scroll", handler, { passive: true });
    return () => window.removeEventListener("scroll", handler);
  }, []);
}
```

---

<a id="6-rendering-performance"></a>

## 6. 렌더링 성능

**영향도: MEDIUM**

렌더링 과정을 최적화하면 브라우저가 수행해야 하는 작업이 줄어든다.

<a id="61-animate-svg-wrapper-instead-of-svg-element"></a>

### 6.1 SVG 요소 대신 SVG 래퍼를 애니메이션

**영향도: LOW (하드웨어 가속 사용)**

많은 브라우저는 SVG 요소에 대한 CSS3 애니메이션을 하드웨어 가속하지 않는다. SVG를 `<div>`로 감싸고 래퍼에 애니메이션을 적용한다.

**잘못된 예: SVG 직접 애니메이션 - 하드웨어 가속 없음**

```tsx
function LoadingSpinner() {
  return (
    <svg className="animate-spin" width="24" height="24" viewBox="0 0 24 24">
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  );
}
```

**올바른 예: 래퍼 div 애니메이션 - 하드웨어 가속**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg width="24" height="24" viewBox="0 0 24 24">
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  );
}
```

모든 CSS transform/transition(`transform`, `opacity`, `translate`, `scale`, `rotate`)에 적용된다. 래퍼 div가 GPU 가속을 사용해 더 부드러운 애니메이션을 만든다.

<a id="62-css-content-visibility-for-long-lists"></a>

### 6.2 긴 목록에 CSS content-visibility

**영향도: HIGH (더 빠른 초기 렌더)**

오프스크린 렌더링을 지연시키려면 `content-visibility: auto`를 적용한다.

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**예시:**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map((msg) => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  );
}
```

메시지 1000개라면 브라우저가 화면 밖 약 990개 항목의 레이아웃/페인트를 건너뛰어 초기 렌더가 10배 빨라진다.

<a id="63-hoist-static-jsx-elements"></a>

### 6.3 정적 JSX 요소 끌어올리기

**영향도: LOW (재생성 방지)**

정적 JSX를 컴포넌트 밖으로 빼서 재생성을 피한다.

**잘못된 예: 매 렌더마다 요소 재생성**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />;
}

function Container() {
  return <div>{loading && <LoadingSkeleton />}</div>;
}
```

**올바른 예: 같은 요소 재사용**

```tsx
const loadingSkeleton = <div className="animate-pulse h-20 bg-gray-200" />;

function Container() {
  return <div>{loading && loadingSkeleton}</div>;
}
```

큰 정적 SVG 노드처럼 매 렌더마다 재생성 비용이 큰 요소에 특히 유용하다.

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있으면 컴파일러가 정적 JSX를 자동으로 끌어올리고 리렌더를 최적화하므로 수동 끌어올리기가 필요 없다.

<a id="64-optimize-svg-precision"></a>

### 6.4 SVG 정밀도 최적화

**영향도: LOW (파일 크기 감소)**

SVG 좌표 정밀도를 낮춰 파일 크기를 줄인다. 최적 정밀도는 viewBox 크기에 따라 달라지지만, 일반적으로 정밀도를 낮추는 것을 고려한다.

**잘못된 예: 과도한 정밀도**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

**올바른 예: 소수점 1자리**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />
```

**SVGO로 자동화:**

```bash
npx svgo --precision=1 --multipass icon.svg
```

<a id="65-prevent-hydration-mismatch-without-flickering"></a>

### 6.5 깜빡임 없이 하이드레이션 불일치 방지

**영향도: MEDIUM (시각적 깜빡임과 하이드레이션 오류 방지)**

클라이언트 스토리지(localStorage, cookies)에 의존하는 콘텐츠를 렌더링할 때, SSR 깨짐과 하이드레이션 이후 깜빡임을 모두 피하려면 React가 하이드레이션하기 전에 DOM을 업데이트하는 동기 스크립트를 주입한다.

**잘못된 예: SSR 깨짐**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage is not available on server - throws error
  const theme = localStorage.getItem("theme") || "light";

  return <div className={theme}>{children}</div>;
}
```

서버 렌더링은 `localStorage`가 undefined라서 실패한다.

**잘못된 예: 시각적 깜빡임**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState("light");

  useEffect(() => {
    // Runs after hydration - causes visible flash
    const stored = localStorage.getItem("theme");
    if (stored) {
      setTheme(stored);
    }
  }, []);

  return <div className={theme}>{children}</div>;
}
```

컴포넌트는 기본값(`light`)으로 먼저 렌더링되고, 하이드레이션 이후 업데이트되어 잘못된 콘텐츠가 깜빡인다.

**올바른 예: 깜빡임 없음, 하이드레이션 불일치 없음**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">{children}</div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  );
}
```

인라인 스크립트가 요소가 표시되기 전에 동기적으로 실행되어 DOM에 올바른 값이 설정된다. 깜빡임도 하이드레이션 불일치도 없다.

이 패턴은 테마 토글, 사용자 선호 설정, 인증 상태처럼 클라이언트 전용 데이터가 기본값 깜빡임 없이 즉시 렌더링되어야 하는 경우에 특히 유용하다.

<a id="66-use-activity-component-for-showhide"></a>

### 6.6 표시/숨김에 Activity 컴포넌트 사용

**영향도: MEDIUM (상태/DOM 보존)**

자주 가시성이 토글되는 비용 큰 컴포넌트의 상태/DOM을 유지하려면 React의 `<Activity>`를 사용한다.

**사용법:**

```tsx
import { Activity } from "react";

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? "visible" : "hidden"}>
      <ExpensiveMenu />
    </Activity>
  );
}
```

비싼 리렌더와 상태 손실을 피한다.

<a id="67-use-explicit-conditional-rendering"></a>

### 6.7 명시적 조건부 렌더링 사용

**영향도: LOW (0 또는 NaN 렌더링 방지)**

조건이 `0`, `NaN` 같은 falsy 값이 될 수 있다면 `&&` 대신 명시적인 삼항 연산자(`? :`)를 사용한다.

**잘못된 예: count가 0일 때 "0" 렌더링**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count && <span className="badge">{count}</span>}</div>;
}

// When count = 0, renders: <div>0</div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```

**올바른 예: count가 0일 때 아무것도 렌더링하지 않음**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count > 0 ? <span className="badge">{count}</span> : null}</div>;
}

// When count = 0, renders: <div></div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```

---

<a id="7-javascript-performance"></a>

## 7. 자바스크립트 성능

**영향도: LOW-MEDIUM**

핫 경로에서의 마이크로 최적화는 의미 있는 개선으로 이어질 수 있다.

<a id="71-batch-dom-css-changes"></a>

### 7.1 DOM CSS 변경 배치

**영향도: MEDIUM (리플로우/리페인트 감소)**

스타일을 한 프로퍼티씩 바꾸지 않는다. 브라우저 리플로우를 최소화하려면 클래스 또는 `cssText`를 통해 여러 CSS 변경을 함께 묶는다.

**잘못된 예: 리플로우 여러 번 발생**

```typescript
function updateElementStyles(element: HTMLElement) {
  // Each line triggers a reflow
  element.style.width = "100px";
  element.style.height = "200px";
  element.style.backgroundColor = "blue";
  element.style.border = "1px solid black";
}
```

**올바른 예: 클래스 추가 - 리플로우 1회**

```typescript
// CSS file
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}

// JavaScript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
}
```

**올바른 예: cssText 변경 - 리플로우 1회**

```typescript
function updateElementStyles(element: HTMLElement) {
  element.style.cssText = `
    width: 100px;
    height: 200px;
    background-color: blue;
    border: 1px solid black;
  `;
}
```

**React 예시:**

```tsx
// Incorrect: changing styles one by one
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = "100px";
      ref.current.style.height = "200px";
      ref.current.style.backgroundColor = "blue";
    }
  }, [isHighlighted]);

  return <div ref={ref}>Content</div>;
}

// Correct: toggle class
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return <div className={isHighlighted ? "highlighted-box" : ""}>Content</div>;
}
```

가능하면 인라인 스타일보다 CSS 클래스를 선호한다. 클래스는 브라우저에서 캐시되며 관심사 분리를 개선한다.

<a id="72-build-index-maps-for-repeated-lookups"></a>

### 7.2 반복 조회용 인덱스 맵 만들기

**영향도: LOW-MEDIUM (1M ops → 2K ops)**

같은 키로 여러 번 `.find()`를 호출하는 경우 Map을 사용한다.

**잘못된 예(O(n) per lookup):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map((order) => ({
    ...order,
    user: users.find((u) => u.id === order.userId),
  }));
}
```

**올바른 예(O(1) per lookup):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map((u) => [u.id, u]));

  return orders.map((order) => ({
    ...order,
    user: userById.get(order.userId),
  }));
}
```

맵을 한 번만 만들면(O(n)) 이후 모든 조회는 O(1)이다.

1000개 주문 × 1000명 사용자: 1M ops → 2K ops.

<a id="73-cache-property-access-in-loops"></a>

### 7.3 루프에서 프로퍼티 접근 캐시

**영향도: LOW-MEDIUM (조회 감소)**

핫 경로에서 객체 프로퍼티 조회를 캐시한다.

**잘못된 예: 3회 조회 × N 반복**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value);
}
```

**올바른 예: 총 1회 조회**

```typescript
const value = obj.config.settings.value;
const len = arr.length;
for (let i = 0; i < len; i++) {
  process(value);
}
```

<a id="74-cache-repeated-function-calls"></a>

### 7.4 반복 함수 호출 캐시

**영향도: MEDIUM (중복 계산 방지)**

렌더 중 같은 입력으로 함수가 반복 호출되면 모듈 레벨 Map으로 결과를 캐시한다.

**잘못된 예: 중복 계산**

```typescript
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // slugify() called 100+ times for same project names
        const slug = slugify(project.name)

        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**올바른 예: 캐시된 결과**

```typescript
// Module-level cache
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // Computed only once per unique project name
        const slug = cachedSlugify(project.name)

        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**단일 값 함수용 더 간단한 패턴:**

```typescript
let isLoggedInCache: boolean | null = null;

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache;
  }

  isLoggedInCache = document.cookie.includes("auth=");
  return isLoggedInCache;
}

// Clear cache when auth changes
function onAuthChange() {
  isLoggedInCache = null;
}
```

유틸리티, 이벤트 핸들러 등 어디서나 동작하도록 훅이 아닌 Map을 사용한다.

참고: [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

<a id="75-cache-storage-api-calls"></a>

### 7.5 Storage API 호출 캐시

**영향도: LOW-MEDIUM (비싼 I/O 감소)**

`localStorage`, `sessionStorage`, `document.cookie`는 동기적이고 비용이 크다. 읽기를 메모리에 캐시한다.

**잘못된 예: 매 호출마다 스토리지 읽음**

```typescript
function getTheme() {
  return localStorage.getItem("theme") ?? "light";
}
// Called 10 times = 10 storage reads
```

**올바른 예: Map 캐시**

```typescript
const storageCache = new Map<string, string | null>();

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key));
  }
  return storageCache.get(key);
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value);
  storageCache.set(key, value); // keep cache in sync
}
```

유틸리티, 이벤트 핸들러 등 어디서나 동작하도록 훅이 아닌 Map을 사용한다.

**쿠키 캐싱:**

```typescript
let cookieCache: Record<string, string> | null = null;

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split("; ").map((c) => c.split("=")),
    );
  }
  return cookieCache[name];
}
```

**중요: 외부 변경 시 무효화**

```typescript
window.addEventListener("storage", (e) => {
  if (e.key) storageCache.delete(e.key);
});

document.addEventListener("visibilitychange", () => {
  if (document.visibilityState === "visible") {
    storageCache.clear();
  }
});
```

스토리지가 외부에서 변경될 수 있다면(다른 탭, 서버가 설정한 쿠키), 캐시를 무효화한다.

<a id="76-combine-multiple-array-iterations"></a>

### 7.6 배열 반복을 하나로 합치기

**영향도: LOW-MEDIUM (반복 횟수 감소)**

여러 번의 `.filter()`나 `.map()`은 배열을 여러 번 순회한다. 한 번의 루프로 합친다.

**잘못된 예: 3회 반복**

```typescript
const admins = users.filter((u) => u.isAdmin);
const testers = users.filter((u) => u.isTester);
const inactive = users.filter((u) => !u.isActive);
```

**올바른 예: 1회 반복**

```typescript
const admins: User[] = [];
const testers: User[] = [];
const inactive: User[] = [];

for (const user of users) {
  if (user.isAdmin) admins.push(user);
  if (user.isTester) testers.push(user);
  if (!user.isActive) inactive.push(user);
}
```

<a id="77-early-length-check-for-array-comparisons"></a>

### 7.7 배열 비교 전에 길이 먼저 확인

**영향도: MEDIUM-HIGH (길이가 다를 때 비싼 연산 회피)**

정렬, 딥 이퀄, 직렬화처럼 비용이 큰 비교를 할 때는 먼저 길이를 확인한다. 길이가 다르면 두 배열은 같을 수 없다.

실무에서는 이 비교가 핫 경로(이벤트 핸들러, 렌더 루프)에서 실행될 때 특히 효과가 크다.

**잘못된 예: 항상 비싼 비교 수행**

```typescript
function hasChanges(current: string[], original: string[]) {
  // Always sorts and joins, even when lengths differ
  return current.sort().join() !== original.sort().join();
}
```

`current.length`가 5이고 `original.length`가 100이어도 O(n log n) 정렬이 두 번 실행된다. 배열을 join하고 문자열을 비교하는 오버헤드도 있다.

**올바른 예(O(1) 길이 확인 먼저):**

```typescript
function hasChanges(current: string[], original: string[]) {
  // Early return if lengths differ
  if (current.length !== original.length) {
    return true;
  }
  // Only sort/join when lengths match
  const currentSorted = current.toSorted();
  const originalSorted = original.toSorted();
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true;
    }
  }
  return false;
}
```

이 접근이 더 효율적인 이유:

- 길이가 다를 때 정렬/조인 오버헤드를 피한다

- 조인 문자열에 대한 메모리 사용을 피한다(특히 큰 배열에서 중요)

- 원본 배열을 변이하지 않는다

- 차이를 찾는 즉시 조기 반환한다

<a id="78-early-return-from-functions"></a>

### 7.8 함수에서 조기 반환

**영향도: LOW-MEDIUM (불필요한 계산 방지)**

결과가 결정되면 조기 반환해 불필요한 처리를 건너뛴다.

**잘못된 예: 답을 찾은 뒤에도 모두 처리**

```typescript
function validateUsers(users: User[]) {
  let hasError = false;
  let errorMessage = "";

  for (const user of users) {
    if (!user.email) {
      hasError = true;
      errorMessage = "Email required";
    }
    if (!user.name) {
      hasError = true;
      errorMessage = "Name required";
    }
    // Continues checking all users even after error found
  }

  return hasError ? { valid: false, error: errorMessage } : { valid: true };
}
```

**올바른 예: 첫 오류에서 즉시 반환**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: "Email required" };
    }
    if (!user.name) {
      return { valid: false, error: "Name required" };
    }
  }

  return { valid: true };
}
```

<a id="79-hoist-regexp-creation"></a>

### 7.9 RegExp 생성 끌어올리기

**영향도: LOW-MEDIUM (재생성 방지)**

렌더 안에서 RegExp를 만들지 말고 모듈 스코프로 끌어올리거나 `useMemo()`로 메모이즈한다.

**잘못된 예: 매 렌더마다 새 RegExp 생성**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**올바른 예: 메모이즈 또는 끌어올리기**

```tsx
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**경고: 전역 정규식은 가변 상태를 가진다**

```typescript
const regex = /foo/g;
regex.test("foo"); // true, lastIndex = 3
regex.test("foo"); // false, lastIndex = 0
```

전역 정규식(`/g`)은 `lastIndex` 상태가 변한다:

<a id="710-use-loop-for-minmax-instead-of-sort"></a>

### 7.10 정렬 대신 루프로 최소/최대 찾기

**영향도: LOW (O(n)으로 O(n log n) 대체)**

최소/최대 요소를 찾는 데는 배열 한 번 순회면 충분하다. 정렬은 낭비이며 더 느리다.

**잘못된 예(O(n log n) - 최신값 찾기 위해 정렬):**

```typescript
interface Project {
  id: string;
  name: string;
  updatedAt: number;
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt);
  return sorted[0];
}
```

최댓값 하나를 찾기 위해 전체 배열을 정렬한다.

**잘못된 예(O(n log n) - 가장 오래된/최신 찾기 위해 정렬):**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt);
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] };
}
```

최소/최대만 필요할 때도 불필요하게 정렬한다.

**올바른 예(O(n) - 단일 루프):**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null;

  let latest = projects[0];

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i];
    }
  }

  return latest;
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null };

  let oldest = projects[0];
  let newest = projects[0];

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i];
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i];
  }

  return { oldest, newest };
}
```

배열을 한 번만 순회하며 복사도 정렬도 하지 않는다.

**대안: 작은 배열은 Math.min/Math.max**

```typescript
const numbers = [5, 2, 8, 1, 9];
const min = Math.min(...numbers);
const max = Math.max(...numbers);
```

작은 배열에서는 동작하지만, 스프레드 연산자 한계로 매우 큰 배열에서는 느리거나 에러가 날 수 있다. 안정성을 위해 루프 방식을 사용한다.

<a id="711-use-setmap-for-o1-lookups"></a>

### 7.11 O(1) 조회에 Set/Map 사용

**영향도: LOW-MEDIUM (O(n)에서 O(1))**

반복적인 포함 여부 검사를 위해 배열을 Set/Map으로 변환한다.

**잘못된 예(O(n) per check):**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

**올바른 예(O(1) per check):**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```

<a id="712-use-tosorted-instead-of-sort-for-immutability"></a>

### 7.12 불변성을 위해 sort() 대신 toSorted()

**영향도: MEDIUM-HIGH (React 상태의 변이 버그 방지)**

`.sort()`는 배열을 제자리에서 변이시켜 React state와 props에서 버그를 유발할 수 있다. 변이 없이 새 정렬 배열을 만드는 `.toSorted()`를 사용한다.

**잘못된 예: 원본 배열 변이**

```typescript
function UserList({ users }: { users: User[] }) {
  // Mutates the users prop array!
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**올바른 예: 새 배열 생성**

```typescript
function UserList({ users }: { users: User[] }) {
  // Creates new sorted array, original unchanged
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**React에서 중요한 이유:**

1. props/state 변이는 React의 불변성 모델을 깨뜨린다 - React는 props와 state를 읽기 전용으로 다루길 기대한다

2. stale closure 버그를 만든다 - 클로저(콜백, effect) 내부에서 배열을 변이하면 예기치 않은 동작이 생길 수 있다

**브라우저 지원: 구형 브라우저 대체**

```typescript
// Fallback for older browsers
const sorted = [...items].sort((a, b) => a.value - b.value);
```

`.toSorted()`는 최신 브라우저(Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+)에서 사용 가능하다. 구형 환경에서는 스프레드 연산자를 사용한다:

**기타 불변 배열 메서드:**

- `.toSorted()` - 불변 정렬

- `.toReversed()` - 불변 뒤집기

- `.toSpliced()` - 불변 스플라이스

- `.with()` - 불변 요소 교체

---

<a id="8-advanced-patterns"></a>

## 8. 고급 패턴

**영향도: LOW**

신중한 구현이 필요한 특정 경우를 위한 고급 패턴이다.

<a id="81-store-event-handlers-in-refs"></a>

### 8.1 이벤트 핸들러를 ref에 저장

**영향도: LOW (안정적인 구독)**

콜백 변경으로 재구독하면 안 되는 effect에서 콜백을 ref에 저장한다.

**잘못된 예: 매 렌더마다 재구독**

```tsx
function useWindowEvent(event: string, handler: () => void) {
  useEffect(() => {
    window.addEventListener(event, handler);
    return () => window.removeEventListener(event, handler);
  }, [event, handler]);
}
```

**올바른 예: 안정적 구독**

```tsx
import { useEffectEvent } from "react";

function useWindowEvent(event: string, handler: () => void) {
  const onEvent = useEffectEvent(handler);

  useEffect(() => {
    window.addEventListener(event, onEvent);
    return () => window.removeEventListener(event, onEvent);
  }, [event]);
}
```

**대안: 최신 React라면 `useEffectEvent` 사용:**

`useEffectEvent`는 같은 패턴을 더 깔끔하게 제공한다. 항상 최신 핸들러를 호출하는 안정적인 함수 참조를 만든다.

<a id="82-uselatest-for-stable-callback-refs"></a>

### 8.2 안정적인 콜백 ref를 위한 useLatest

**영향도: LOW (effect 재실행 방지)**

의존성 배열에 추가하지 않고 콜백에서 최신 값을 읽는다. effect 재실행을 막으면서 stale closure를 피한다.

**구현:**

```typescript
function useLatest<T>(value: T) {
  const ref = useRef(value);
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref;
}
```

**잘못된 예: 콜백 변경마다 effect 재실행**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300);
    return () => clearTimeout(timeout);
  }, [query, onSearch]);
}
```

**올바른 예: 안정적 effect, 최신 콜백**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");
  const onSearchRef = useLatest(onSearch);

  useEffect(() => {
    const timeout = setTimeout(() => onSearchRef.current(query), 300);
    return () => clearTimeout(timeout);
  }, [query]);
}
```

---

<a id="references"></a>

## 참고

1. [https://react.dev](https://react.dev)
2. [https://nextjs.org](https://nextjs.org)
3. [https://swr.vercel.app](https://swr.vercel.app)
4. [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
5. [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
6. [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
7. [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
