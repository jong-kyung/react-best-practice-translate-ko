---
title: RSC 경계의 직렬화 최소화
impact: HIGH
impactDescription: 데이터 전송 크기 감소
tags: server, rsc, serialization, props
---

## RSC 경계의 직렬화 최소화

React Server/Client 경계는 모든 객체 프로퍼티를 문자열로 직렬화해 HTML 응답과 이후 RSC 요청에 포함한다. 이 직렬화된 데이터는 페이지 무게와 로드 시간에 직접 영향을 주므로 **크기가 매우 중요**하다. 클라이언트가 실제로 사용하는 필드만 전달한다.

**잘못된 예(필드 50개 전부 직렬화):**

```tsx
async function Page() {
  const user = await fetchUser()  // 50 fields
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // uses 1 field
}
```

**올바른 예(필드 1개만 직렬화):**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
```
