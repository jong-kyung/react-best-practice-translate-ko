---
title: 불리언 prop 남발 피하기
impact: CRITICAL
impactDescription: 유지보수 불가능한 컴포넌트 변형을 방지
tags: composition, props, architecture
---

## 불리언 prop 남발 피하기

컴포넌트 동작을 커스터마이즈하기 위해 `isThread`, `isEditing`, `isDMThread`
같은 불리언 prop을 추가하지 마세요. 불리언 하나가 가능한 상태를 두 배로
늘려, 유지보수 불가능한 조건부 로직을 만들어냅니다. 대신 컴포지션을
사용하세요.

**잘못된 예 (불리언 prop이 복잡도를 지수적으로 증가시킴):**

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

**올바른 예 (컴포지션으로 조건문 제거):**

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
