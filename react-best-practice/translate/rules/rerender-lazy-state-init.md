---
title: 지연 상태 초기화 사용
impact: MEDIUM
impactDescription: 매 렌더마다 낭비되는 계산
tags: react, hooks, useState, performance, initialization
---

## 지연 상태 초기화 사용

비싼 초기값은 `useState`에 함수를 전달한다. 함수형을 쓰지 않으면 초기화 로직이 값이 한 번만 쓰여도 매 렌더마다 실행된다.

**잘못된 예(매 렌더마다 실행):**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() runs on EVERY render, even after initialization
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  // When query changes, buildSearchIndex runs again unnecessarily
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse runs on every render
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

**올바른 예(한 번만 실행):**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() runs ONLY on initial render
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse runs only on initial render
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

localStorage/sessionStorage에서 초기값을 읽거나, 데이터 구조(인덱스, 맵)를 만들거나, DOM을 읽거나, 무거운 변환을 수행할 때 지연 초기화를 사용한다.

단순 원시값(`useState(0)`), 직접 참조(`useState(props.value)`), 값싼 리터럴(`useState({})`)에는 함수형이 필요 없다.
