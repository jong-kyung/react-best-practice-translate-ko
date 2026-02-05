---
title: 무거운 컴포넌트는 동적 임포트
impact: CRITICAL
impactDescription: TTI와 LCP에 직접 영향
tags: bundle, dynamic-import, code-splitting, next-dynamic
---

## 무거운 컴포넌트는 동적 임포트

초기 렌더에 필요하지 않은 큰 컴포넌트는 `next/dynamic`으로 지연 로드한다.

**잘못된 예(Monaco가 메인 청크에 번들링되어 ~300KB):**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

**올바른 예(Monaco를 필요할 때 로드):**

```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```
