---
title: 파생 상태에 구독
impact: MEDIUM
impactDescription: 리렌더 빈도 감소
tags: rerender, derived-state, media-query, optimization
---

## 파생 상태에 구독

연속적인 값 대신 파생된 불리언 상태에 구독해 리렌더 빈도를 줄인다.

**잘못된 예(픽셀 변화마다 리렌더):**

```tsx
function Sidebar() {
  const width = useWindowWidth()  // updates continuously
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

**올바른 예(불리언이 바뀔 때만 리렌더):**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```
