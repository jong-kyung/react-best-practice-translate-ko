---
title: localStorage 데이터 버전 관리 및 최소화
impact: MEDIUM
impactDescription: 스키마 충돌 방지, 저장 크기 감소
tags: client, localStorage, storage, versioning, data-minimization
---

## localStorage 데이터 버전 관리 및 최소화

키에 버전 접두사를 추가하고 필요한 필드만 저장한다. 스키마 충돌과 민감 데이터의 우발적 저장을 방지한다.

**잘못된 예:**

```typescript
// No version, stores everything, no error handling
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')
```

**올바른 예:**

```typescript
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // Throws in incognito/private browsing, quota exceeded, or disabled
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// Migration from v1 to v2
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}
```

**서버 응답에서 최소 필드만 저장:**

```typescript
// User object has 20+ fields, only store what UI needs
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}
```

**항상 try-catch로 감싼다:** `getItem()`과 `setItem()`은 시크릿/프라이빗 브라우징(Safari, Firefox), 용량 초과, 비활성화 상태에서 예외를 던진다.

**효과:** 버전 관리를 통한 스키마 진화, 저장 크기 감소, 토큰/PII/내부 플래그 저장 방지.
