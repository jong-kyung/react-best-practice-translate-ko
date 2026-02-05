---
title: 명시적 컴포넌트 변형 만들기
impact: MEDIUM
impactDescription: 자기설명적인 코드, 숨겨진 조건문 없음
tags: composition, variants, architecture
---

## 명시적 컴포넌트 변형 만들기

불리언 prop이 많은 단일 컴포넌트 대신 명시적인 변형 컴포넌트를
만드세요. 각 변형은 필요한 조각을 조합합니다. 코드는 스스로를
설명합니다.

**잘못된 예 (하나의 컴포넌트, 여러 모드):**

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

**올바른 예 (명시적 변형):**

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
