---
title: SVG 요소 대신 SVG 래퍼를 애니메이션
impact: LOW
impactDescription: 하드웨어 가속 사용
tags: rendering, svg, css, animation, performance
---

## SVG 요소 대신 SVG 래퍼를 애니메이션

많은 브라우저는 SVG 요소에 대한 CSS3 애니메이션을 하드웨어 가속하지 않는다. SVG를 `<div>`로 감싸고 래퍼에 애니메이션을 적용한다.

**잘못된 예(SVG 직접 애니메이션 - 하드웨어 가속 없음):**

```tsx
function LoadingSpinner() {
  return (
    <svg 
      className="animate-spin"
      width="24" 
      height="24" 
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}
```

**올바른 예(래퍼 div 애니메이션 - 하드웨어 가속):**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg 
        width="24" 
        height="24" 
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}
```

모든 CSS transform/transition(`transform`, `opacity`, `translate`, `scale`, `rotate`)에 적용된다. 래퍼 div가 GPU 가속을 사용해 더 부드러운 애니메이션을 만든다.
