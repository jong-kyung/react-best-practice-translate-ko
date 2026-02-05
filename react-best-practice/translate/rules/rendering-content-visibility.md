---
title: 긴 목록에 CSS content-visibility
impact: HIGH
impactDescription: 더 빠른 초기 렌더
tags: rendering, css, content-visibility, long-lists
---

## 긴 목록에 CSS content-visibility

오프스크린 렌더링을 지연시키려면 `content-visibility: auto`를 적용한다.

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**예시:**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

메시지 1000개라면 브라우저가 화면 밖 약 990개 항목의 레이아웃/페인트를 건너뛰어 초기 렌더가 10배 빨라진다.
