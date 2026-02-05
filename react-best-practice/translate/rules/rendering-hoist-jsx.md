---
title: 정적 JSX 요소 끌어올리기
impact: LOW
impactDescription: 재생성 방지
tags: rendering, jsx, static, optimization
---

## 정적 JSX 요소 끌어올리기

정적 JSX를 컴포넌트 밖으로 빼서 재생성을 피한다.

**잘못된 예(매 렌더마다 요소 재생성):**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}
```

**올바른 예(같은 요소 재사용):**

```tsx
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}
```

큰 정적 SVG 노드처럼 매 렌더마다 재생성 비용이 큰 요소에 특히 유용하다.

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있으면 컴파일러가 정적 JSX를 자동으로 끌어올리고 리렌더를 최적화하므로 수동 끌어올리기가 필요 없다.
