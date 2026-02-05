---
title: 의존성 주입을 위한 제네릭 컨텍스트 인터페이스 정의
impact: HIGH
impactDescription: 사용 사례 전반에서 의존성 주입 가능한 상태를 가능하게 함
tags: composition, context, state, typescript, dependency-injection
---

## 의존성 주입을 위한 제네릭 컨텍스트 인터페이스 정의

컴포넌트 컨텍스트를 위한 **제네릭 인터페이스**를 세 가지 부분으로
정의하세요: `state`, `actions`, `meta`. 이 인터페이스는 어떤 프로바이더도
구현할 수 있는 계약이며, 같은 UI 컴포넌트가 완전히 다른 상태
구현과도 동작하게 합니다.

**핵심 원칙:** 상태를 끌어올리고, 내부를 조합하며, 상태를 의존성 주입
가능하게 만든다.

**잘못된 예 (UI가 특정 상태 구현에 결합됨):**

```tsx
function ComposerInput() {
  // 특정 훅에 강하게 결합됨
  const { input, setInput } = useChannelComposerState()
  return <TextInput value={input} onChangeText={setInput} />
}
```

**올바른 예 (제네릭 인터페이스가 의존성 주입을 가능하게 함):**

```tsx
// 어떤 프로바이더도 구현할 수 있는 제네릭 인터페이스 정의
interface ComposerState {
  input: string
  attachments: Attachment[]
  isSubmitting: boolean
}

interface ComposerActions {
  update: (updater: (state: ComposerState) => ComposerState) => void
  submit: () => void
}

interface ComposerMeta {
  inputRef: React.RefObject<TextInput>
}

interface ComposerContextValue {
  state: ComposerState
  actions: ComposerActions
  meta: ComposerMeta
}

const ComposerContext = createContext<ComposerContextValue | null>(null)
```

**UI 컴포넌트는 구현이 아닌 인터페이스를 소비:**

```tsx
function ComposerInput() {
  const {
    state,
    actions: { update },
    meta,
  } = use(ComposerContext)

  // 이 컴포넌트는 인터페이스를 구현하는 어떤 프로바이더와도 동작함
  return (
    <TextInput
      ref={meta.inputRef}
      value={state.input}
      onChangeText={(text) => update((s) => ({ ...s, input: text }))}
    />
  )
}
```

**다른 프로바이더가 같은 인터페이스를 구현:**

```tsx
// 프로바이더 A: 일시적인 폼을 위한 로컬 상태
function ForwardMessageProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState)
  const inputRef = useRef(null)
  const submit = useForwardMessage()

  return (
    <ComposerContext
      value={{
        state,
        actions: { update: setState, submit },
        meta: { inputRef },
      }}
    >
      {children}
    </ComposerContext>
  )
}

// 프로바이더 B: 채널을 위한 전역 동기화 상태
function ChannelProvider({ channelId, children }: Props) {
  const { state, update, submit } = useGlobalChannel(channelId)
  const inputRef = useRef(null)

  return (
    <ComposerContext
      value={{
        state,
        actions: { update, submit },
        meta: { inputRef },
      }}
    >
      {children}
    </ComposerContext>
  )
}
```

**같은 구성 UI가 두 경우 모두에서 동작:**

```tsx
// ForwardMessageProvider(로컬 상태)에서 동작
<ForwardMessageProvider>
  <Composer.Frame>
    <Composer.Input />
    <Composer.Submit />
  </Composer.Frame>
</ForwardMessageProvider>

// ChannelProvider(전역 동기화 상태)에서 동작
<ChannelProvider channelId="abc">
  <Composer.Frame>
    <Composer.Input />
    <Composer.Submit />
  </Composer.Frame>
</ChannelProvider>
```

**컴포넌트 외부의 커스텀 UI도 상태와 액션에 접근 가능:**

프로바이더 경계가 중요합니다. 시각적 중첩 여부가 아닙니다. 공유 상태가
필요한 컴포넌트는 `Composer.Frame` 안에 있을 필요가 없습니다. 프로바이더
안에만 있으면 됩니다.

```tsx
function ForwardMessageDialog() {
  return (
    <ForwardMessageProvider>
      <Dialog>
        {/* 컴포저 UI */}
        <Composer.Frame>
          <Composer.Input placeholder="Add a message, if you'd like." />
          <Composer.Footer>
            <Composer.Formatting />
            <Composer.Emojis />
          </Composer.Footer>
        </Composer.Frame>

        {/* 컴포저 바깥이지만 프로바이더 안의 커스텀 UI */}
        <MessagePreview />

        {/* 다이얼로그 하단 액션 */}
        <DialogActions>
          <CancelButton />
          <ForwardButton />
        </DialogActions>
      </Dialog>
    </ForwardMessageProvider>
  )
}

// 이 버튼은 Composer.Frame 밖에 있지만 컨텍스트로 submit을 호출할 수 있음!
function ForwardButton() {
  const {
    actions: { submit },
  } = use(ComposerContext)
  return <Button onPress={submit}>Forward</Button>
}

// 이 미리보기는 Composer.Frame 밖에 있지만 컴포저의 상태를 읽을 수 있음!
function MessagePreview() {
  const { state } = use(ComposerContext)
  return <Preview message={state.input} attachments={state.attachments} />
}
```

`ForwardButton`과 `MessagePreview`는 시각적으로 컴포저 박스 안에 있지
않지만, 여전히 상태와 액션에 접근할 수 있습니다. 이것이 프로바이더로
상태를 끌어올리는 힘입니다.

UI는 조합하는 재사용 가능한 조각이고, 상태는 프로바이더가 의존성
주입합니다. 프로바이더를 바꾸고, UI는 그대로 유지하세요.
