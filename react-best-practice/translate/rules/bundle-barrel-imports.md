---
title: 배럴 파일 임포트 피하기
impact: CRITICAL
impactDescription: 200-800ms import 비용, 느린 빌드
tags: bundle, imports, tree-shaking, barrel-files, performance
---

## 배럴 파일 임포트 피하기

배럴 파일 대신 소스 파일에서 직접 임포트해 사용하지 않는 수천 개 모듈 로딩을 피한다. **배럴 파일**은 여러 모듈을 다시 내보내는 엔트리포인트다(예: `export * from './module'`을 하는 `index.js`).

인기 아이콘/컴포넌트 라이브러리는 엔트리 파일에 **최대 10,000개의 재-export**가 있을 수 있다. 많은 React 패키지는 **임포트만 하는 데 200-800ms**가 걸려 개발 속도와 프로덕션 콜드 스타트에 영향을 준다.

**트리 쉐이킹이 도움이 안 되는 이유:** 라이브러리가 external(번들 미포함)로 표시되면 번들러가 최적화할 수 없다. 트리 쉐이킹을 위해 번들에 포함하면 전체 모듈 그래프를 분석하느라 빌드가 크게 느려진다.

**잘못된 예(라이브러리 전체 임포트):**

```tsx
import { Check, X, Menu } from 'lucide-react'
// Loads 1,583 modules, takes ~2.8s extra in dev
// Runtime cost: 200-800ms on every cold start

import { Button, TextField } from '@mui/material'
// Loads 2,225 modules, takes ~4.2s extra in dev
```

**올바른 예(필요한 것만 임포트):**

```tsx
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// Loads only 3 modules (~2KB vs ~1MB)

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// Loads only what you use
```

**대안(Next.js 13.5+):**

```js
// next.config.js - use optimizePackageImports
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// Then you can keep the ergonomic barrel imports:
import { Check, X, Menu } from 'lucide-react'
// Automatically transformed to direct imports at build time
```

직접 임포트는 개발 부팅을 15-70% 빠르게, 빌드를 28% 빠르게, 콜드 스타트를 40% 빠르게 하며 HMR도 크게 빨라진다.

자주 영향을 받는 라이브러리: `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`.

참고: [How we optimized package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
