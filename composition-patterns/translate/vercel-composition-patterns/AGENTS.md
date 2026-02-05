# React 컴포지션 패턴

**버전 1.0.0**  
엔지니어링  
2026년 1월

> **참고:**  
> 이 문서는 주로 에이전트와 LLM이 컴포지션을 사용한 React 코드베이스를  
> 유지보수, 생성, 리팩터링할 때 따르도록 작성되었습니다. 사람에게도  
> 유용할 수 있지만, 여기의 가이드는 AI 보조 워크플로의 자동화와  
> 일관성에 최적화되어 있습니다.

---

## 요약

유연하고 유지보수 가능한 React 컴포넌트를 만들기 위한 컴포지션 패턴입니다.
컴파운드 컴포넌트 사용, 상태 끌어올리기, 내부 구성 결합을 통해 불리언 prop
남발을 피합니다. 이 패턴들은 규모가 커질수록 사람과 AI 에이전트 모두가
코드베이스를 더 쉽게 다룰 수 있게 합니다.

---

## 목차

1. [컴포넌트 아키텍처](#1-component-architecture) — **HIGH**
   - 1.1 [불리언 prop 남발 피하기](#11-avoid-boolean-prop-proliferation)
   - 1.2 [컴파운드 컴포넌트 사용](#12-use-compound-components)
2. [상태 관리](#2-state-management) — **MEDIUM**
   - 2.1 [UI에서 상태 관리 분리](#21-decouple-state-management-from-ui)
   - 2.2 [의존성 주입을 위한 제네릭 컨텍스트 인터페이스 정의](#22-define-generic-context-interfaces-for-dependency-injection)
   - 2.3 [프로바이더 컴포넌트로 상태 끌어올리기](#23-lift-state-into-provider-components)
3. [구현 패턴](#3-implementation-patterns) — **MEDIUM**
   - 3.1 [명시적 컴포넌트 변형 만들기](#31-create-explicit-component-variants)
   - 3.2 [렌더 프롭보다 children 조합을 선호](#32-prefer-composing-children-over-render-props)
4. [React 19 API](#4-react-19-apis) — **MEDIUM**
   - 4.1 [React 19 API 변경](#41-react-19-api-changes)

---

<a id="1-component-architecture"></a>
## 1. 컴포넌트 아키텍처

**영향도: HIGH**

prop 남발을 피하고 유연한 컴포지션을 가능하게 하는 컴포넌트 구조화의
기본 패턴.

<a id="11-avoid-boolean-prop-proliferation"></a>
### 1.1 불리언 prop 남발 피하기

**영향도: CRITICAL (유지보수 불가능한 컴포넌트 변형을 방지)**

컴포넌트 동작을 커스터마이즈하기 위해 `isThread`, `isEditing`,
`isDMThread` 같은 불리언 prop을 추가하지 마세요. 불리언 하나가 가능한
상태를 두 배로 늘려, 유지보수 불가능한 조건부 로직을 만들어냅니다.
대신 컴포지션을 사용하세요.

**잘못된 예: 불리언 prop이 복잡도를 지수적으로 증가시킴**

```tsx
function Composer({
  onSubmit,
  isThread,
  channelId,
  isDMThread,
  dmId,
  isEditing,
  isForwarding,
}: Props) {
  return (
    <form>
      <Header />
      <Input />
      {isDMThread ? (
        <AlsoSendToDMField id={dmId} />
      ) : isThread ? (
        <AlsoSendToChannelField id={channelId} />
      ) : null}
      {isEditing ? (
        <EditActions />
      ) : isForwarding ? (
        <ForwardActions />
      ) : (
        <DefaultActions />
      )}
      <Footer onSubmit={onSubmit} />
    </form>
  )
}
```

**올바른 예: 컴포지션으로 조건문 제거**

```tsx
// 채널 컴포저
function ChannelComposer() {
  return (
    <Composer.Frame>
      <Composer.Header />
      <Composer.Input />
      <Composer.Footer>
        <Composer.Attachments />
        <Composer.Formatting />
        <Composer.Emojis />
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}

// 스레드 컴포저 - "채널에도 보내기" 필드 추가
function ThreadComposer({ channelId }: { channelId: string }) {
  return (
    <Composer.Frame>
      <Composer.Header />
      <Composer.Input />
      <AlsoSendToChannelField id={channelId} />
      <Composer.Footer>
        <Composer.Formatting />
        <Composer.Emojis />
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}

// 편집 컴포저 - 다른 푸터 액션
function EditComposer() {
  return (
    <Composer.Frame>
      <Composer.Input />
      <Composer.Footer>
        <Composer.Formatting />
        <Composer.Emojis />
        <Composer.CancelEdit />
        <Composer.SaveEdit />
      </Composer.Footer>
    </Composer.Frame>
  )
}
```

각 변형은 무엇을 렌더링하는지 명확합니다. 단일 거대한 부모를 공유하지
않고도 내부를 공유할 수 있습니다.

<a id="12-use-compound-components"></a>
### 1.2 컴파운드 컴포넌트 사용

**영향도: HIGH (prop 드릴링 없이 유연한 컴포지션을 가능하게 함)**

공유 컨텍스트를 가진 컴파운드 컴포넌트로 복잡한 컴포넌트를
구조화하세요. 각 하위 컴포넌트는 props가 아니라 컨텍스트를 통해
공유 상태에 접근합니다. 소비자는 필요한 조각만 조합합니다.

**잘못된 예: render props가 있는 모놀리식 컴포넌트**

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

**올바른 예: 공유 컨텍스트를 가진 컴파운드 컴포넌트**

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

---

<a id="2-state-management"></a>
## 2. 상태 관리

**영향도: MEDIUM**

조합된 컴포넌트 전반에서 상태를 끌어올리고 공유 컨텍스트를 관리하기 위한
패턴.

<a id="21-decouple-state-management-from-ui"></a>
### 2.1 UI에서 상태 관리 분리

**영향도: MEDIUM (UI 변경 없이 상태 구현을 교체할 수 있게 함)**

프로바이더 컴포넌트는 상태가 어떻게 관리되는지 아는 유일한 장소여야
합니다. UI 컴포넌트는 컨텍스트 인터페이스만 소비하며, 상태가 useState,
Zustand, 서버 동기화 중 어디에서 오는지 알지 못합니다.

**잘못된 예: UI가 상태 구현에 결합됨**

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

**올바른 예: 프로바이더에서 상태 관리 분리**

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

<a id="22-define-generic-context-interfaces-for-dependency-injection"></a>
### 2.2 의존성 주입을 위한 제네릭 컨텍스트 인터페이스 정의

**영향도: HIGH (사용 사례 전반에서 의존성 주입 가능한 상태를 가능하게 함)**

컴포넌트 컨텍스트를 위한 **제네릭 인터페이스**를 세 가지 부분으로
정의하세요: `state`, `actions`, `meta`. 이 인터페이스는 어떤 프로바이더도
구현할 수 있는 계약이며, 같은 UI 컴포넌트가 완전히 다른 상태
구현과도 동작하게 합니다.

**핵심 원칙:** 상태를 끌어올리고, 내부를 조합하며, 상태를 의존성 주입
가능하게 만든다.

**잘못된 예: UI가 특정 상태 구현에 결합됨**

```tsx
function ComposerInput() {
  // 특정 훅에 강하게 결합됨
  const { input, setInput } = useChannelComposerState()
  return <TextInput value={input} onChangeText={setInput} />
}
```

**올바른 예: 제네릭 인터페이스가 의존성 주입을 가능하게 함**

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

<a id="23-lift-state-into-provider-components"></a>
### 2.3 프로바이더 컴포넌트로 상태 끌어올리기

**영향도: HIGH (컴포넌트 경계 밖에서 상태 공유 가능)**

상태 관리를 전용 프로바이더 컴포넌트로 옮기세요. 이렇게 하면 메인 UI
밖의 형제 컴포넌트가 prop 드릴링이나 어색한 ref 없이도 상태에 접근하고
수정할 수 있습니다.

**잘못된 예: 컴포넌트 내부에 갇힌 상태**

```tsx
function ForwardMessageComposer() {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()

  return (
    <Composer.Frame>
      <Composer.Input />
      <Composer.Footer />
    </Composer.Frame>
  )
}

// 문제: 이 버튼이 컴포저 상태에 어떻게 접근하지?
function ForwardMessageDialog() {
  return (
    <Dialog>
      <ForwardMessageComposer />
      <MessagePreview /> {/* 컴포저 상태 필요 */}
      <DialogActions>
        <CancelButton />
        <ForwardButton /> {/* submit 호출 필요 */}
      </DialogActions>
    </Dialog>
  )
}
```

**잘못된 예: useEffect로 상태를 끌어올려 동기화**

```tsx
function ForwardMessageDialog() {
  const [input, setInput] = useState('')
  return (
    <Dialog>
      <ForwardMessageComposer onInputChange={setInput} />
      <MessagePreview input={input} />
    </Dialog>
  )
}

function ForwardMessageComposer({ onInputChange }) {
  const [state, setState] = useState(initialState)
  useEffect(() => {
    onInputChange(state.input) // 매 변경마다 동기화 😬
  }, [state.input])
}
```

**잘못된 예: submit 시 ref에서 상태 읽기**

```tsx
function ForwardMessageDialog() {
  const stateRef = useRef(null)
  return (
    <Dialog>
      <ForwardMessageComposer stateRef={stateRef} />
      <ForwardButton onPress={() => submit(stateRef.current)} />
    </Dialog>
  )
}
```

**올바른 예: 프로바이더로 상태 끌어올리기**

```tsx
function ForwardMessageProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()
  const inputRef = useRef(null)

  return (
    <Composer.Provider
      state={state}
      actions={{ update: setState, submit: forwardMessage }}
      meta={{ inputRef }}
    >
      {children}
    </Composer.Provider>
  )
}

function ForwardMessageDialog() {
  return (
    <ForwardMessageProvider>
      <Dialog>
        <ForwardMessageComposer />
        <MessagePreview /> {/* 커스텀 컴포넌트도 상태/액션 접근 가능 */}
        <DialogActions>
          <CancelButton />
          <ForwardButton /> {/* 커스텀 컴포넌트도 상태/액션 접근 가능 */}
        </DialogActions>
      </Dialog>
    </ForwardMessageProvider>
  )
}

function ForwardButton() {
  const { actions } = use(Composer.Context)
  return <Button onPress={actions.submit}>Forward</Button>
}
```

ForwardButton은 Composer.Frame 밖에 있지만 같은 프로바이더 안에 있으므로
submit 액션에 접근할 수 있습니다. 일회성 컴포넌트라 하더라도 UI 밖에서
컴포저의 상태와 액션에 접근할 수 있습니다.

**핵심 통찰:** 공유 상태가 필요한 컴포넌트는 서로 시각적으로 중첩될
필요가 없습니다. 같은 프로바이더 안에 있기만 하면 됩니다.

---

<a id="3-implementation-patterns"></a>
## 3. 구현 패턴

**영향도: MEDIUM**

컴파운드 컴포넌트와 컨텍스트 프로바이더를 구현하기 위한 구체적인
기술.

<a id="31-create-explicit-component-variants"></a>
### 3.1 명시적 컴포넌트 변형 만들기

**영향도: MEDIUM (자기설명적인 코드, 숨겨진 조건문 없음)**

불리언 prop이 많은 단일 컴포넌트 대신 명시적인 변형 컴포넌트를
만드세요. 각 변형은 필요한 조각을 조합합니다. 코드는 스스로를
설명합니다.

**잘못된 예: 하나의 컴포넌트, 여러 모드**

```tsx
// 이 컴포넌트는 실제로 무엇을 렌더링할까?
<Composer
  isThread
  isEditing={false}
  channelId='abc'
  showAttachments
  showFormatting={false}
/>
```

**올바른 예: 명시적 변형**

```tsx
// 무엇을 렌더링하는지 즉시 명확
<ThreadComposer channelId="abc" />

// 또는
<EditMessageComposer messageId="xyz" />

// 또는
<ForwardMessageComposer messageId="123" />
```

각 구현은 고유하고 명시적이며 자기 완결적입니다. 하지만 공유 부분을
사용할 수 있습니다.

**구현:**

```tsx
function ThreadComposer({ channelId }: { channelId: string }) {
  return (
    <ThreadProvider channelId={channelId}>
      <Composer.Frame>
        <Composer.Input />
        <AlsoSendToChannelField channelId={channelId} />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.Submit />
        </Composer.Footer>
      </Composer.Frame>
    </ThreadProvider>
  )
}

function EditMessageComposer({ messageId }: { messageId: string }) {
  return (
    <EditMessageProvider messageId={messageId}>
      <Composer.Frame>
        <Composer.Input />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.CancelEdit />
          <Composer.SaveEdit />
        </Composer.Footer>
      </Composer.Frame>
    </EditMessageProvider>
  )
}

function ForwardMessageComposer({ messageId }: { messageId: string }) {
  return (
    <ForwardMessageProvider messageId={messageId}>
      <Composer.Frame>
        <Composer.Input placeholder="Add a message, if you'd like." />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.Mentions />
        </Composer.Footer>
      </Composer.Frame>
    </ForwardMessageProvider>
  )
}
```

각 변형은 다음을 명시합니다:

- 어떤 프로바이더/상태를 사용하는지
- 어떤 UI 요소를 포함하는지
- 어떤 액션이 가능한지

불리언 prop 조합을 고민할 필요가 없습니다. 불가능한 상태도 없습니다.

<a id="32-prefer-composing-children-over-render-props"></a>
### 3.2 렌더 프롭보다 children 조합을 선호

**영향도: MEDIUM (더 깔끔한 컴포지션, 더 나은 가독성)**

`renderX` prop 대신 `children`을 사용해 컴포지션하세요. children은 더
읽기 쉽고 자연스럽게 조합되며 콜백 시그니처를 이해할 필요가
없습니다.

**잘못된 예: 렌더 프롭**

```tsx
function Composer({
  renderHeader,
  renderFooter,
  renderActions,
}: {
  renderHeader?: () => React.ReactNode
  renderFooter?: () => React.ReactNode
  renderActions?: () => React.ReactNode
}) {
  return (
    <form>
      {renderHeader?.()}
      <Input />
      {renderFooter ? renderFooter() : <DefaultFooter />}
      {renderActions?.()}
    </form>
  )
}

// 사용법이 어색하고 유연하지 않음
return (
  <Composer
    renderHeader={() => <CustomHeader />}
    renderFooter={() => (
      <>
        <Formatting />
        <Emojis />
      </>
    )}
    renderActions={() => <SubmitButton />}
  />
)
```

**올바른 예: children을 사용하는 컴파운드 컴포넌트**

```tsx
function ComposerFrame({ children }: { children: React.ReactNode }) {
  return <form>{children}</form>
}

function ComposerFooter({ children }: { children: React.ReactNode }) {
  return <footer className='flex'>{children}</footer>
}

// 사용법이 유연함
return (
  <Composer.Frame>
    <CustomHeader />
    <Composer.Input />
    <Composer.Footer>
      <Composer.Formatting />
      <Composer.Emojis />
      <SubmitButton />
    </Composer.Footer>
  </Composer.Frame>
)
```

**렌더 프롭이 적절한 경우:**

```tsx
// 렌더 프롭은 데이터를 다시 전달해야 할 때 유용함
<List
  data={items}
  renderItem={({ item, index }) => <Item item={item} index={index} />}
/>
```

부모가 자식에게 데이터나 상태를 제공해야 한다면 렌더 프롭을 사용하세요.
정적인 구조를 조합하는 경우에는 children을 사용하세요.

---

<a id="4-react-19-apis"></a>
## 4. React 19 API

**영향도: MEDIUM**

React 19+ 전용. `forwardRef`는 사용하지 말고 `useContext()` 대신 `use()`를
사용하세요.

<a id="41-react-19-api-changes"></a>
### 4.1 React 19 API 변경

**영향도: MEDIUM (더 깔끔한 컴포넌트 정의와 컨텍스트 사용)**

> **⚠️ React 19+ 전용.** React 18 이하를 사용한다면 건너뛰세요.

React 19에서는 `ref`가 일반 prop이 되었고(더 이상 `forwardRef` 래퍼
불필요), `use()`가 `useContext()`를 대체합니다.

**잘못된 예: React 19에서 forwardRef 사용**

```tsx
const ComposerInput = forwardRef<TextInput, Props>((props, ref) => {
  return <TextInput ref={ref} {...props} />
})
```

**올바른 예: ref를 일반 prop으로 사용**

```tsx
function ComposerInput({ ref, ...props }: Props & { ref?: React.Ref<TextInput> }) {
  return <TextInput ref={ref} {...props} />
}
```

**잘못된 예: React 19에서 useContext 사용**

```tsx
const value = useContext(MyContext)
```

**올바른 예: useContext 대신 use 사용**

```tsx
const value = use(MyContext)
```

`use()`는 `useContext()`와 달리 조건부로도 호출할 수 있습니다.

---

## 참고 자료

1. [https://react.dev](https://react.dev)
2. [https://react.dev/learn/passing-data-deeply-with-context](https://react.dev/learn/passing-data-deeply-with-context)
3. [https://react.dev/reference/react/use](https://react.dev/reference/react/use)
