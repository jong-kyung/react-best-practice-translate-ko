---
title: UI에서 상태 관리 분리
impact: MEDIUM
impactDescription: UI 변경 없이 상태 구현을 교체할 수 있게 함
tags: composition, state, architecture
---

## UI에서 상태 관리 분리

프로바이더 컴포넌트는 상태가 어떻게 관리되는지 아는 유일한 장소여야
합니다. UI 컴포넌트는 컨텍스트 인터페이스만 소비하며, 상태가 useState,
Zustand, 서버 동기화 중 어디에서 오는지 알지 못합니다.

**잘못된 예 (UI가 상태 구현에 결합됨):**

```tsx
function ChannelComposer({ channelId }: { channelId: string }) {
  // UI 컴포넌트가 전역 상태 구현을 알고 있음
  const state = useGlobalChannelState(channelId)
  const { submit, updateInput } = useChannelSync(channelId)

  return (
    <Composer.Frame>
      <Composer.Input
        value={state.input}
        onChange={(text) => sync.updateInput(text)}
      />
      <Composer.Submit onPress={() => sync.submit()} />
    </Composer.Frame>
  )
}
```

**올바른 예 (프로바이더에서 상태 관리 분리):**

```tsx
// 프로바이더가 모든 상태 관리 세부 사항을 처리
function ChannelProvider({
  channelId,
  children,
}: {
  channelId: string
  children: React.ReactNode
}) {
  const { state, update, submit } = useGlobalChannel(channelId)
  const inputRef = useRef(null)

  return (
    <Composer.Provider
      state={state}
      actions={{ update, submit }}
      meta={{ inputRef }}
    >
      {children}
    </Composer.Provider>
  )
}

// UI 컴포넌트는 컨텍스트 인터페이스만 알고 있음
function ChannelComposer() {
  return (
    <Composer.Frame>
      <Composer.Header />
      <Composer.Input />
      <Composer.Footer>
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}

// 사용 예
function Channel({ channelId }: { channelId: string }) {
  return (
    <ChannelProvider channelId={channelId}>
      <ChannelComposer />
    </ChannelProvider>
  )
}
```

**다른 프로바이더, 같은 UI:**

```tsx
// 일시적인 폼을 위한 로컬 상태
function ForwardMessageProvider({ children }) {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()

  return (
    <Composer.Provider
      state={state}
      actions={{ update: setState, submit: forwardMessage }}
    >
      {children}
    </Composer.Provider>
  )
}

// 채널을 위한 전역 동기화 상태
function ChannelProvider({ channelId, children }) {
  const { state, update, submit } = useGlobalChannel(channelId)

  return (
    <Composer.Provider state={state} actions={{ update, submit }}>
      {children}
    </Composer.Provider>
  )
}
```

같은 `Composer.Input` 컴포넌트가 두 프로바이더 모두에서 동작하는 이유는
컨텍스트 인터페이스만 의존하고 구현에 의존하지 않기 때문입니다.
