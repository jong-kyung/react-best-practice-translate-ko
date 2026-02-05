---
title: RegExp 생성 끌어올리기
impact: LOW-MEDIUM
impactDescription: 재생성 방지
tags: javascript, regexp, optimization, memoization
---

## RegExp 생성 끌어올리기

렌더 안에서 RegExp를 만들지 말고 모듈 스코프로 끌어올리거나 `useMemo()`로 메모이즈한다.

**잘못된 예(매 렌더마다 새 RegExp 생성):**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**올바른 예(메모이즈 또는 끌어올리기):**

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

**경고(전역 정규식은 가변 상태):**

전역 정규식(`/g`)은 `lastIndex` 상태가 변한다:

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0
```
