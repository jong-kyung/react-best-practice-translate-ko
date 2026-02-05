---
title: 안정적인 콜백 ref를 위한 useLatest
impact: LOW
impactDescription: effect 재실행 방지
tags: advanced, hooks, useLatest, refs, optimization
---

## 안정적인 콜백 ref를 위한 useLatest

의존성 배열에 추가하지 않고 콜백에서 최신 값을 읽는다. effect 재실행을 막으면서 stale closure를 피한다.

**구현:**

```typescript
function useLatest<T>(value: T) {
  const ref = useRef(value)
  useLayoutEffect(() => {
    ref.current = value
  }, [value])
  return ref
}
```

**잘못된 예(effect가 콜백 변경마다 재실행):**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}
```

**올바른 예(안정적 effect, 최신 콜백):**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchRef = useLatest(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchRef.current(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}
```
