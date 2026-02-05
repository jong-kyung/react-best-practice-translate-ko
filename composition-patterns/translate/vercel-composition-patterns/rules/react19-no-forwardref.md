---
title: React 19 API 변경
impact: MEDIUM
impactDescription: 더 깔끔한 컴포넌트 정의와 컨텍스트 사용
tags: react19, refs, context, hooks
---

## React 19 API 변경

> **⚠️ React 19+ 전용.** React 18 이하를 사용한다면 건너뛰세요.

React 19에서는 `ref`가 일반 prop이 되었고(더 이상 `forwardRef` 래퍼
불필요), `use()`가 `useContext()`를 대체합니다.

**잘못된 예 (React 19에서 forwardRef 사용):**

```tsx
const ComposerInput = forwardRef<TextInput, Props>((props, ref) => {
  return <TextInput ref={ref} {...props} />
})
```

**올바른 예 (ref를 일반 prop으로 사용):**

```tsx
function ComposerInput({ ref, ...props }: Props & { ref?: React.Ref<TextInput> }) {
  return <TextInput ref={ref} {...props} />
}
```

**잘못된 예 (React 19에서 useContext 사용):**

```tsx
const value = useContext(MyContext)
```

**올바른 예 (useContext 대신 use 사용):**

```tsx
const value = use(MyContext)
```

`use()`는 `useContext()`와 달리 조건부로도 호출할 수 있습니다.
