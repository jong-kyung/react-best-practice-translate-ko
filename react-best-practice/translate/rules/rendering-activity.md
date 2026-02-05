---
title: 표시/숨김에 Activity 컴포넌트 사용
impact: MEDIUM
impactDescription: 상태/DOM 보존
tags: rendering, activity, visibility, state-preservation
---

## 표시/숨김에 Activity 컴포넌트 사용

자주 가시성이 토글되는 비용 큰 컴포넌트의 상태/DOM을 유지하려면 React의 `<Activity>`를 사용한다.

**사용법:**

```tsx
import { Activity } from 'react'

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? 'visible' : 'hidden'}>
      <ExpensiveMenu />
    </Activity>
  )
}
```

비싼 리렌더와 상태 손실을 피한다.
