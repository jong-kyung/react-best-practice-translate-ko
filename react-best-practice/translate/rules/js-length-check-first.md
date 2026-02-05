---
title: 배열 비교 전에 길이 먼저 확인
impact: MEDIUM-HIGH
impactDescription: 길이가 다를 때 비싼 연산 회피
tags: javascript, arrays, performance, optimization, comparison
---

## 배열 비교 전에 길이 먼저 확인

정렬, 딥 이퀄, 직렬화처럼 비용이 큰 비교를 할 때는 먼저 길이를 확인한다. 길이가 다르면 두 배열은 같을 수 없다.

실무에서는 이 비교가 핫 경로(이벤트 핸들러, 렌더 루프)에서 실행될 때 특히 효과가 크다.

**잘못된 예(항상 비싼 비교 수행):**

```typescript
function hasChanges(current: string[], original: string[]) {
  // Always sorts and joins, even when lengths differ
  return current.sort().join() !== original.sort().join()
}
```

`current.length`가 5이고 `original.length`가 100이어도 O(n log n) 정렬이 두 번 실행된다. 배열을 join하고 문자열을 비교하는 오버헤드도 있다.

**올바른 예(먼저 O(1) 길이 확인):**

```typescript
function hasChanges(current: string[], original: string[]) {
  // Early return if lengths differ
  if (current.length !== original.length) {
    return true
  }
  // Only sort when lengths match
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}
```

이 접근이 더 효율적인 이유:
- 길이가 다를 때 정렬/조인 오버헤드를 피한다
- 조인 문자열에 대한 메모리 사용을 피한다(특히 큰 배열에서 중요)
- 원본 배열을 변이하지 않는다
- 차이를 찾는 즉시 조기 반환한다
