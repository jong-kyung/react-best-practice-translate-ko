---
title: API 라우트에서 워터폴 체인 방지
impact: CRITICAL
impactDescription: 2-10배 개선
tags: api-routes, server-actions, waterfalls, parallelization
---

## API 라우트에서 워터폴 체인 방지

API 라우트와 Server Action에서는 서로 독립적인 작업을 즉시 시작하고, 아직 await 하지 않더라도 바로 실행되게 한다.

**잘못된 예(설정이 인증을 기다리고, 데이터가 둘 다 기다림):**

```typescript
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}
```

**올바른 예(인증과 설정을 즉시 시작):**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id)
  ])
  return Response.json({ data, config })
}
```

더 복잡한 의존성 체인이 있다면 `better-all`을 사용해 병렬성을 자동으로 극대화한다(의존성 기반 병렬화 참고).
