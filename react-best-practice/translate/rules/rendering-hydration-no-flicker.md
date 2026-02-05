---
title: 깜빡임 없이 하이드레이션 불일치 방지
impact: MEDIUM
impactDescription: 시각적 깜빡임과 하이드레이션 오류 방지
tags: rendering, ssr, hydration, localStorage, flicker
---

## 깜빡임 없이 하이드레이션 불일치 방지

클라이언트 스토리지(localStorage, cookies)에 의존하는 콘텐츠를 렌더링할 때, SSR 깨짐과 하이드레이션 이후 깜빡임을 모두 피하려면 React가 하이드레이션하기 전에 DOM을 업데이트하는 동기 스크립트를 주입한다.

**잘못된 예(SSR 깨짐):**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage is not available on server - throws error
  const theme = localStorage.getItem('theme') || 'light'
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

서버 렌더링은 `localStorage`가 undefined라서 실패한다.

**잘못된 예(시각적 깜빡임):**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light')
  
  useEffect(() => {
    // Runs after hydration - causes visible flash
    const stored = localStorage.getItem('theme')
    if (stored) {
      setTheme(stored)
    }
  }, [])
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

컴포넌트는 기본값(`light`)으로 먼저 렌더링되고, 하이드레이션 이후 업데이트되어 잘못된 콘텐츠가 깜빡인다.

**올바른 예(깜빡임 없음, 하이드레이션 불일치 없음):**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">
        {children}
      </div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}
```

인라인 스크립트가 요소가 표시되기 전에 동기적으로 실행되어 DOM에 올바른 값이 설정된다. 깜빡임도 하이드레이션 불일치도 없다.

이 패턴은 테마 토글, 사용자 선호 설정, 인증 상태처럼 클라이언트 전용 데이터가 기본값 깜빡임 없이 즉시 렌더링되어야 하는 경우에 특히 유용하다.
