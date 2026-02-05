---
title: 의존성 기반 병렬화
impact: CRITICAL
impactDescription: 2-10배 개선
tags: async, parallelization, dependencies, better-all
---

## 의존성 기반 병렬화

부분 의존성이 있는 작업에는 `better-all`을 사용해 병렬성을 최대화한다. 각 작업을 가능한 가장 이른 시점에 자동으로 시작한다.

**잘못된 예(프로필이 불필요하게 설정을 기다림):**

```typescript
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)
```

**올바른 예(설정과 프로필을 병렬 실행):**

```typescript
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})
```

참고: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
