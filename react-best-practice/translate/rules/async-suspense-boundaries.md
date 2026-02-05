---
title: 전략적 Suspense 경계
impact: HIGH
impactDescription: 초기 페인트 속도 향상
tags: async, suspense, streaming, layout-shift
---

## 전략적 Suspense 경계

JSX를 반환하기 전에 async 컴포넌트에서 데이터를 await 하지 말고, Suspense 경계를 사용해 데이터 로딩 동안 래퍼 UI를 더 빠르게 보여준다.

**잘못된 예(데이터 페칭 때문에 래퍼가 차단):**

```tsx
async function Page() {
  const data = await fetchData() // Blocks entire page
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

중간 섹션만 데이터가 필요한데도 전체 레이아웃이 데이터를 기다린다.

**올바른 예(래퍼는 즉시 표시, 데이터는 스트리밍):**

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
  )
}

async function DataDisplay() {
  const data = await fetchData() // Only blocks this component
  return <div>{data.content}</div>
}
```

Sidebar, Header, Footer는 즉시 렌더링되고 DataDisplay만 데이터를 기다린다.

**대안(컴포넌트 간 프라미스 공유):**

```tsx
function Page() {
  // Start fetch immediately, but don't await
  const dataPromise = fetchData()
  
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
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // Unwraps the promise
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // Reuses the same promise
  return <div>{data.summary}</div>
}
```

두 컴포넌트가 같은 프라미스를 공유하므로 한 번만 fetch 된다. 둘이 함께 기다리는 동안 레이아웃은 즉시 렌더된다.

**이 패턴을 쓰지 말아야 할 때:**

- 레이아웃 결정에 필요한 핵심 데이터(배치에 영향)
- 폴드 상단의 SEO 핵심 콘텐츠
- Suspense 오버헤드가 아까운 작고 빠른 쿼리
- 레이아웃 시프트(로딩 → 콘텐츠 점프)를 피하고 싶을 때

**트레이드오프:** 초기 페인트 속도 vs 잠재적 레이아웃 시프트. UX 우선순위에 따라 선택한다.
