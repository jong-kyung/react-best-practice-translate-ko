---
title: DOM CSS 변경 배치
impact: MEDIUM
impactDescription: 리플로우/리페인트 감소
tags: javascript, dom, css, performance, reflow
---

## DOM CSS 변경 배치

스타일 쓰기와 레이아웃 읽기를 섞지 않는다. 스타일 변경 사이에 레이아웃 속성(`offsetWidth`, `getBoundingClientRect()`, `getComputedStyle()`)을 읽으면 브라우저가 동기 리플로우를 강제한다.

**잘못된 예(읽기/쓰기가 섞여 리플로우 발생):**

```typescript
function updateElementStyles(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // Forces reflow
  element.style.height = '200px'
  const height = element.offsetHeight  // Forces another reflow
}
```

**올바른 예(쓰기 배치 후 한 번만 읽기):**

```typescript
function updateElementStyles(element: HTMLElement) {
  // Batch all writes together
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
  
  // Read after all writes are done (single reflow)
  const { width, height } = element.getBoundingClientRect()
}
```

**더 나은 방법: CSS 클래스 사용**

```css
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}
```

```typescript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')

  const { width, height } = element.getBoundingClientRect()
}
```

가능하면 인라인 스타일보다 CSS 클래스를 선호한다. CSS 파일은 브라우저에서 캐시되며, 클래스는 관심사 분리를 개선하고 유지보수가 쉽다.
