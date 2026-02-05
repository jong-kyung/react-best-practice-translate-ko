---
title: 함수형 setState 업데이트 사용
impact: MEDIUM
impactDescription: stale closure 및 불필요한 콜백 재생성 방지
tags: react, hooks, useState, useCallback, callbacks, closures
---

## 함수형 setState 업데이트 사용

현재 상태 값을 기반으로 상태를 업데이트할 때는 상태 변수를 직접 참조하지 말고 setState의 함수형 업데이트를 사용한다. 이렇게 하면 stale closure를 막고, 불필요한 의존성을 제거하며, 안정적인 콜백 참조를 만든다.

**잘못된 예(상태 의존성 필요):**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // Callback must depend on items, recreated on every items change
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])  // ❌ items dependency causes recreations
  
  // Risk of stale closure if dependency is forgotten
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])  // ❌ Missing items dependency - will use stale items!
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

첫 번째 콜백은 `items`가 바뀔 때마다 다시 만들어져 자식 컴포넌트를 불필요하게 리렌더링할 수 있다. 두 번째 콜백은 stale closure 버그가 있어 항상 초기 `items` 값을 참조한다.

**올바른 예(안정적 콜백, stale closure 없음):**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // Stable callback, never recreated
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])  // ✅ No dependencies needed
  
  // Always uses latest state, no stale closure risk
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])  // ✅ Safe and stable
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
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
