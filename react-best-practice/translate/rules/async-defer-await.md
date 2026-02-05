---
title: 필요할 때까지 await 지연
impact: HIGH
impactDescription: 사용되지 않는 코드 경로 차단 방지
tags: async, await, conditional, optimization
---

## 필요할 때까지 await 지연

필요한 분기 안으로 `await` 작업을 옮겨, 필요하지 않은 코드 경로가 막히지 않도록 한다.

**잘못된 예(양쪽 분기 모두 차단):**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // Returns immediately but still waited for userData
    return { skipped: true }
  }
  
  // Only this branch uses userData
  return processUserData(userData)
}
```

**올바른 예(필요할 때만 차단):**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // Returns immediately without waiting
    return { skipped: true }
  }
  
  // Fetch only when needed
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

**다른 예시(조기 반환 최적화):**

```typescript
// Incorrect: always fetches permissions
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

// Correct: fetches only when needed
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  const permissions = await fetchPermissions(userId)
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}
```

이 최적화는 건너뛰는 분기가 자주 발생하거나, 지연한 작업이 비용이 큰 경우 특히 효과적이다.
