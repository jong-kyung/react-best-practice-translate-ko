---
title: 메모이즈 컴포넌트로 추출
impact: MEDIUM
impactDescription: 조기 반환 가능
tags: rerender, memo, useMemo, optimization
---

## 메모이즈 컴포넌트로 추출

비싼 작업을 메모이즈된 컴포넌트로 추출해 계산 전에 조기 반환할 수 있게 한다.

**잘못된 예(로딩 중에도 avatar 계산):**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}
```

**올바른 예(로딩 중 계산 생략):**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}
```

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있으면 `memo()`와 `useMemo()`로 수동 메모이즈할 필요가 없다. 컴파일러가 리렌더를 자동 최적화한다.
