---
title: 조건부 모듈 로딩
impact: HIGH
impactDescription: 필요할 때만 큰 데이터 로드
tags: bundle, conditional-loading, lazy-loading
---

## 조건부 모듈 로딩

기능이 활성화될 때만 큰 데이터나 모듈을 로드한다.

**예시(애니메이션 프레임 lazy-load):**

```tsx
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}
```

`typeof window !== 'undefined'` 체크는 SSR에서 이 모듈이 번들에 포함되지 않게 해 서버 번들 크기와 빌드 속도를 최적화한다.
