---
title: 루프에서 프로퍼티 접근 캐시
impact: LOW-MEDIUM
impactDescription: 조회 감소
tags: javascript, loops, optimization, caching
---

## 루프에서 프로퍼티 접근 캐시

핫 경로에서 객체 프로퍼티 조회를 캐시한다.

**잘못된 예(3회 조회 × N 반복):**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value)
}
```

**올바른 예(총 1회 조회):**

```typescript
const value = obj.config.settings.value
const len = arr.length
for (let i = 0; i < len; i++) {
  process(value)
}
```
