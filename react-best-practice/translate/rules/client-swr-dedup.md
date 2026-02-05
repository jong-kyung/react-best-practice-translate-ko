---
title: SWR로 자동 중복 제거
impact: MEDIUM-HIGH
impactDescription: 자동 중복 제거
tags: client, swr, deduplication, data-fetching
---

## SWR로 자동 중복 제거

SWR은 컴포넌트 인스턴스 간 요청 중복 제거, 캐싱, 재검증을 제공한다.

**잘못된 예(중복 제거 없음, 각 인스턴스가 개별 fetch):**

```tsx
function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers)
  }, [])
}
```

**올바른 예(여러 인스턴스가 한 요청 공유):**

```tsx
import useSWR from 'swr'

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher)
}
```

**불변 데이터용:**

```tsx
import { useImmutableSWR } from '@/lib/swr'

function StaticContent() {
  const { data } = useImmutableSWR('/api/config', fetcher)
}
```

**변경 작업용:**

```tsx
import { useSWRMutation } from 'swr/mutation'

function UpdateButton() {
  const { trigger } = useSWRMutation('/api/user', updateUser)
  return <button onClick={() => trigger()}>Update</button>
}
```

참고: [https://swr.vercel.app](https://swr.vercel.app)
