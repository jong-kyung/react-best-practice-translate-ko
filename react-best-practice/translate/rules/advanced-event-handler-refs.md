---
title: 이벤트 핸들러를 ref에 저장
impact: LOW
impactDescription: 안정적인 구독
tags: advanced, hooks, refs, event-handlers, optimization
---

## 이벤트 핸들러를 ref에 저장

콜백 변경으로 재구독하면 안 되는 effect에서 콜백을 ref에 저장한다.

**잘못된 예(매 렌더마다 재구독):**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

**올바른 예(안정적 구독):**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  const handlerRef = useRef(handler)
  useEffect(() => {
    handlerRef.current = handler
  }, [handler])

  useEffect(() => {
    const listener = (e) => handlerRef.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

**대안: 최신 React라면 `useEffectEvent` 사용:**

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

`useEffectEvent`는 같은 패턴을 더 깔끔하게 제공한다. 항상 최신 핸들러를 호출하는 안정적인 함수 참조를 만든다.
