---
title: 배열 반복을 하나로 합치기
impact: LOW-MEDIUM
impactDescription: 반복 횟수 감소
tags: javascript, arrays, loops, performance
---

## 배열 반복을 하나로 합치기

여러 번의 `.filter()`나 `.map()`은 배열을 여러 번 순회한다. 한 번의 루프로 합친다.

**잘못된 예(3회 반복):**

```typescript
const admins = users.filter(u => u.isAdmin)
const testers = users.filter(u => u.isTester)
const inactive = users.filter(u => !u.isActive)
```

**올바른 예(1회 반복):**

```typescript
const admins: User[] = []
const testers: User[] = []
const inactive: User[] = []

for (const user of users) {
  if (user.isAdmin) admins.push(user)
  if (user.isTester) testers.push(user)
  if (!user.isActive) inactive.push(user)
}
```
