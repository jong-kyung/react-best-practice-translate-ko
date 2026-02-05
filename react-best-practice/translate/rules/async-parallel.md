---
title: 독립적인 작업에 Promise.all()
impact: CRITICAL
impactDescription: 2-10배 개선
tags: async, parallelization, promises, waterfalls
---

## 독립적인 작업에 Promise.all()

비동기 작업 사이에 의존성이 없다면 `Promise.all()`로 동시에 실행한다.

**잘못된 예(순차 실행, 왕복 3회):**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()
```

**올바른 예(병렬 실행, 왕복 1회):**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```
