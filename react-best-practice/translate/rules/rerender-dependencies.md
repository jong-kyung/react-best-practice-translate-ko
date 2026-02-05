---
title: 이펙트 의존성 좁히기
impact: LOW
impactDescription: 이펙트 재실행 최소화
tags: rerender, useEffect, dependencies, optimization
---

## 이펙트 의존성 좁히기

이펙트 재실행을 최소화하려면 객체 대신 원시값 의존성을 지정한다.

**잘못된 예(사용자 필드 변경마다 재실행):**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user])
```

**올바른 예(id가 바뀔 때만 재실행):**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user.id])
```

**파생 상태는 effect 밖에서 계산:**

```tsx
// Incorrect: runs on width=767, 766, 765...
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])

// Correct: runs only on boolean transition
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])
```
