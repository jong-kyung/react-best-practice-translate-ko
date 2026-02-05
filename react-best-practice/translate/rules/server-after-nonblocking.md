---
title: 차단하지 않는 작업에 after() 사용
impact: MEDIUM
impactDescription: 더 빠른 응답 시간
tags: server, async, logging, analytics, side-effects
---

## 차단하지 않는 작업에 after() 사용

응답이 전송된 뒤 실행해야 하는 작업은 Next.js의 `after()`로 스케줄한다. 로깅, 분석, 기타 사이드 이펙트가 응답을 막지 않게 한다.

**잘못된 예(응답을 막음):**

```tsx
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request)
  
  // Logging blocks the response
  const userAgent = request.headers.get('user-agent') || 'unknown'
  await logUserAction({ userAgent })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

**올바른 예(논블로킹):**

```tsx
import { after } from 'next/server'
import { headers, cookies } from 'next/headers'
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request)
  
  // Log after response is sent
  after(async () => {
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionCookie = (await cookies()).get('session-id')?.value || 'anonymous'
    
    logUserAction({ sessionCookie, userAgent })
  })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

응답은 즉시 전송되고 로깅은 백그라운드에서 진행된다.

**일반적인 사용 사례:**

- 분석(Analytics) 추적
- 감사 로그
- 알림 전송
- 캐시 무효화
- 정리 작업

**중요한 참고:**

- 응답이 실패하거나 리다이렉트되더라도 `after()`는 실행된다
- Server Actions, Route Handlers, Server Components에서 동작한다

참고: [https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)
