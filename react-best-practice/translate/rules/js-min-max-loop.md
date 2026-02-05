---
title: 정렬 대신 루프로 최소/최대 찾기
impact: LOW
impactDescription: O(n) (O(n log n) 대신)
tags: javascript, arrays, performance, sorting, algorithms
---

## 정렬 대신 루프로 최소/최대 찾기

최소/최대 요소를 찾는 데는 배열 한 번 순회면 충분하다. 정렬은 낭비이며 더 느리다.

**잘못된 예(O(n log n) - 최신값 찾기 위해 정렬):**

```typescript
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}
```

최댓값 하나를 찾기 위해 전체 배열을 정렬한다.

**잘못된 예(O(n log n) - 가장 오래된/최신 찾기 위해 정렬):**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}
```

최소/최대만 필요할 때도 불필요하게 정렬한다.

**올바른 예(O(n) - 단일 루프):**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null
  
  let latest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }
  
  return latest
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }
  
  let oldest = projects[0]
  let newest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }
  
  return { oldest, newest }
}
```

배열을 한 번만 순회하며 복사도 정렬도 하지 않는다.

**대안(작은 배열은 Math.min/Math.max):**

```typescript
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)
```

작은 배열에서는 동작하지만, 스프레드 연산자 한계로 매우 큰 배열에서는 느리거나 에러가 날 수 있다. 최대 배열 길이는 Chrome 143 기준 약 124000, Safari 18 기준 약 638000이며 정확한 수치는 달라질 수 있다 - [the fiddle](https://jsfiddle.net/qw1jabsx/4/) 참고. 안정성을 위해 루프 방식을 사용한다.
