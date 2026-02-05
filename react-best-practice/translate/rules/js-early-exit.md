---
title: 함수에서 조기 반환
impact: LOW-MEDIUM
impactDescription: 불필요한 계산 방지
tags: javascript, functions, optimization, early-return
---

## 함수에서 조기 반환

결과가 결정되면 조기 반환해 불필요한 처리를 건너뛴다.

**잘못된 예(답을 찾은 뒤에도 모두 처리):**

```typescript
function validateUsers(users: User[]) {
  let hasError = false
  let errorMessage = ''
  
  for (const user of users) {
    if (!user.email) {
      hasError = true
      errorMessage = 'Email required'
    }
    if (!user.name) {
      hasError = true
      errorMessage = 'Name required'
    }
    // Continues checking all users even after error found
  }
  
  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}
```

**올바른 예(첫 오류에서 즉시 반환):**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: 'Email required' }
    }
    if (!user.name) {
      return { valid: false, error: 'Name required' }
    }
  }

  return { valid: true }
}
```
