---
title: 불변성을 위해 sort() 대신 toSorted()
impact: MEDIUM-HIGH
impactDescription: React 상태의 변이 버그 방지
tags: javascript, arrays, immutability, react, state, mutation
---

## 불변성을 위해 sort() 대신 toSorted()

`.sort()`는 배열을 제자리에서 변이시켜 React state와 props에서 버그를 유발할 수 있다. 변이 없이 새 정렬 배열을 만드는 `.toSorted()`를 사용한다.

**잘못된 예(원본 배열 변이):**

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

**올바른 예(새 배열 생성):**

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

**브라우저 지원(구형 브라우저 대체):**

`.toSorted()`는 최신 브라우저(Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+)에서 사용 가능하다. 구형 환경에서는 스프레드 연산자를 사용한다:

```typescript
// Fallback for older browsers
const sorted = [...items].sort((a, b) => a.value - b.value)
```

**기타 불변 배열 메서드:**

- `.toSorted()` - 불변 정렬
- `.toReversed()` - 불변 뒤집기
- `.toSpliced()` - 불변 스플라이스
- `.with()` - 불변 요소 교체
