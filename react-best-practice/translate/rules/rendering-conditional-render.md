---
title: 명시적 조건부 렌더링 사용
impact: LOW
impactDescription: 0 또는 NaN 렌더링 방지
tags: rendering, conditional, jsx, falsy-values
---

## 명시적 조건부 렌더링 사용

조건이 `0`, `NaN` 같은 falsy 값이 될 수 있다면 `&&` 대신 명시적인 삼항 연산자(`? :`)를 사용한다.

**잘못된 예(count가 0일 때 "0" 렌더링):**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count && <span className="badge">{count}</span>}
    </div>
  )
}

// When count = 0, renders: <div>0</div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```

**올바른 예(count가 0일 때 아무것도 렌더링하지 않음):**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? <span className="badge">{count}</span> : null}
    </div>
  )
}

// When count = 0, renders: <div></div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```
