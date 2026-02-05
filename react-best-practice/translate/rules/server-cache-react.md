---
title: React.cache()로 요청 내 중복 제거
impact: MEDIUM
impactDescription: 요청 내 중복 제거
tags: server, cache, react-cache, deduplication
---

## React.cache()로 요청 내 중복 제거

서버 사이드 요청 중복 제거에는 `React.cache()`를 사용한다. 인증과 데이터베이스 쿼리에 특히 효과적이다.

**사용법:**

```typescript
import { cache } from 'react'

export const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({
    where: { id: session.user.id }
  })
})
```

하나의 요청 안에서 `getCurrentUser()`를 여러 번 호출해도 쿼리는 한 번만 실행된다.

**인라인 객체 인자는 피한다:**

`React.cache()`는 얕은 비교(`Object.is`)로 캐시 히트를 판단한다. 인라인 객체는 호출마다 새 참조를 만들어 캐시가 맞지 않는다.

**잘못된 예(항상 캐시 미스):**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// Each call creates new object, never hits cache
getUser({ uid: 1 })
getUser({ uid: 1 })  // Cache miss, runs query again
```

**올바른 예(캐시 히트):**

```typescript
const getUser = cache(async (uid: number) => {
  return await db.user.findUnique({ where: { id: uid } })
})

// Primitive args use value equality
getUser(1)
getUser(1)  // Cache hit, returns cached result
```

객체를 꼭 써야 한다면 같은 참조를 전달한다:

```typescript
const params = { uid: 1 }
getUser(params)  // Query runs
getUser(params)  // Cache hit (same reference)
```

**Next.js 전용 참고:**

Next.js에서는 `fetch` API가 요청 메모이제이션으로 확장되어 동일한 URL과 옵션의 요청은 하나의 요청 안에서 자동으로 중복 제거된다. 따라서 `fetch`에는 `React.cache()`가 필요 없다. 하지만 다음 작업에는 여전히 `React.cache()`가 중요하다:

- 데이터베이스 쿼리(Prisma, Drizzle 등)
- 무거운 연산
- 인증 체크
- 파일 시스템 작업
- `fetch`가 아닌 모든 비동기 작업

컴포넌트 트리 전반에서 이런 작업을 중복 제거하려면 `React.cache()`를 사용한다.

참고: [React.cache documentation](https://react.dev/reference/react/cache)
