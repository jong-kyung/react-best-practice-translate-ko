---
title: 반복 조회용 인덱스 맵 만들기
impact: LOW-MEDIUM
impactDescription: 1M ops → 2K ops
tags: javascript, map, indexing, optimization, performance
---

## 반복 조회용 인덱스 맵 만들기

같은 키로 여러 번 `.find()`를 호출하는 경우 Map을 사용한다.

**잘못된 예(조회마다 O(n)):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}
```

**올바른 예(조회마다 O(1)):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}
```

맵을 한 번만 만들면(O(n)) 이후 모든 조회는 O(1)이다.
1000개 주문 × 1000명 사용자: 1M ops → 2K ops.
