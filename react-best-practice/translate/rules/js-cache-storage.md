---
title: Storage API 호출 캐시
impact: LOW-MEDIUM
impactDescription: 비싼 I/O 감소
tags: javascript, localStorage, storage, caching, performance
---

## Storage API 호출 캐시

`localStorage`, `sessionStorage`, `document.cookie`는 동기적이고 비용이 크다. 읽기를 메모리에 캐시한다.

**잘못된 예(매 호출마다 스토리지 읽음):**

```typescript
function getTheme() {
  return localStorage.getItem('theme') ?? 'light'
}
// Called 10 times = 10 storage reads
```

**올바른 예(Map 캐시):**

```typescript
const storageCache = new Map<string, string | null>()

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key))
  }
  return storageCache.get(key)
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value)
  storageCache.set(key, value)  // keep cache in sync
}
```

유틸리티, 이벤트 핸들러 등 어디서나 동작하도록 훅이 아닌 Map을 사용한다.

**쿠키 캐싱:**

```typescript
let cookieCache: Record<string, string> | null = null

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split('; ').map(c => c.split('='))
    )
  }
  return cookieCache[name]
}
```

**중요(외부 변경 시 무효화):**

스토리지가 외부에서 변경될 수 있다면(다른 탭, 서버가 설정한 쿠키), 캐시를 무효화한다.

```typescript
window.addEventListener('storage', (e) => {
  if (e.key) storageCache.delete(e.key)
})

document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') {
    storageCache.clear()
  }
})
```
