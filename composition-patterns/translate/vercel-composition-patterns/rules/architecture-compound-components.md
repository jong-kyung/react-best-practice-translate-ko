---
title: 컴파운드 컴포넌트 사용
impact: HIGH
impactDescription: prop 드릴링 없이 유연한 컴포지션을 가능하게 함
tags: composition, compound-components, architecture
---

## 컴파운드 컴포넌트 사용

공유 컨텍스트를 가진 컴파운드 컴포넌트로 복잡한 컴포넌트를
구조화하세요. 각 하위 컴포넌트는 props가 아니라 컨텍스트를 통해
공유 상태에 접근합니다. 소비자는 필요한 조각만 조합합니다.

**잘못된 예 (render props가 있는 모놀리식 컴포넌트):**

```tsx
function Composer({
  renderHeader,
  renderFooter,
  renderActions,
  showAttachments,
  showFormatting,
  showEmojis,
}: Props) {
  return (
    <form>
      {renderHeader?.()}
      <Input />
      {showAttachments && <Attachments />}
      {renderFooter ? (
        renderFooter()
      ) : (
        <Footer>
          {showFormatting && <Formatting />}
          {showEmojis && <Emojis />}
          {renderActions?.()}
        </Footer>
      )}
    </form>
  )
}
```

**올바른 예 (공유 컨텍스트를 가진 컴파운드 컴포넌트):**

```tsx
const ComposerContext = createContext<ComposerContextValue | null>(null)

function ComposerProvider({ children, state, actions, meta }: ProviderProps) {
  return (
    <ComposerContext value={{ state, actions, meta }}>
      {children}
    </ComposerContext>
  )
}

function ComposerFrame({ children }: { children: React.ReactNode }) {
  return <form>{children}</form>
}

function ComposerInput() {
  const {
    state,
    actions: { update },
    meta: { inputRef },
  } = use(ComposerContext)
  return (
    <TextInput
      ref={inputRef}
      value={state.input}
      onChangeText={(text) => update((s) => ({ ...s, input: text }))}
    />
  )
}

function ComposerSubmit() {
  const {
    actions: { submit },
  } = use(ComposerContext)
  return <Button onPress={submit}>Send</Button>
}

// 컴파운드 컴포넌트로 내보내기
const Composer = {
  Provider: ComposerProvider,
  Frame: ComposerFrame,
  Input: ComposerInput,
  Submit: ComposerSubmit,
  Header: ComposerHeader,
  Footer: ComposerFooter,
  Attachments: ComposerAttachments,
  Formatting: ComposerFormatting,
  Emojis: ComposerEmojis,
}
```

**사용 예:**

```tsx
<Composer.Provider state={state} actions={actions} meta={meta}>
  <Composer.Frame>
    <Composer.Header />
    <Composer.Input />
    <Composer.Footer>
      <Composer.Formatting />
      <Composer.Submit />
    </Composer.Footer>
  </Composer.Frame>
</Composer.Provider>
```

소비자는 필요한 것만 명시적으로 조합합니다. 숨겨진 조건문이 없습니다.
그리고 state, actions, meta는 상위 프로바이더가 의존성 주입하므로 같은
컴포넌트 구조를 여러 방식으로 사용할 수 있습니다.
