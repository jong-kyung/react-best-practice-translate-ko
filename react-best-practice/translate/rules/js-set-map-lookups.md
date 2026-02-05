---
title: O(1) 조회에 Set/Map 사용
impact: LOW-MEDIUM
impactDescription: O(n)에서 O(1)
tags: javascript, set, map, data-structures, performance
---

## O(1) 조회에 Set/Map 사용

반복적인 포함 여부 검사를 위해 배열을 Set/Map으로 변환한다.

**잘못된 예(조회마다 O(n)):**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

**올바른 예(조회마다 O(1)):**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```
