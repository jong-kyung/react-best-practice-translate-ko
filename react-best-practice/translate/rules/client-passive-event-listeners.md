---
title: 스크롤 성능을 위해 passive 이벤트 리스너 사용
impact: MEDIUM
impactDescription: 이벤트 리스너로 인한 스크롤 지연 제거
tags: client, event-listeners, scrolling, performance, touch, wheel
---

## 스크롤 성능을 위해 passive 이벤트 리스너 사용

touch와 wheel 이벤트 리스너에 `{ passive: true }`를 추가해 즉시 스크롤이 가능하도록 한다. 브라우저는 기본적으로 리스너가 `preventDefault()`를 호출하는지 확인하기 위해 완료를 기다리며, 이 때문에 스크롤이 지연될 수 있다.

**잘못된 예:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch)
  document.addEventListener('wheel', handleWheel)
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**올바른 예:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch, { passive: true })
  document.addEventListener('wheel', handleWheel, { passive: true })
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**passive 사용:** 트래킹/분석, 로깅, `preventDefault()`를 호출하지 않는 모든 리스너.

**passive 미사용:** 커스텀 스와이프 제스처, 커스텀 줌 컨트롤처럼 `preventDefault()`가 필요한 리스너.
