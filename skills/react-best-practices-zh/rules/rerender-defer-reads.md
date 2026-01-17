---
title: 将状态读取推迟到使用点
impact: MEDIUM
impactDescription: 避免不必要的订阅
tags: rerender, searchParams, localStorage, optimization
---

## 将状态读取推迟到使用点

如果只在回调中读取动态状态（searchParams, localStorage），则不要订阅它。

**错误示例（订阅搜索参数的所有更改）：**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams();

  const handleShare = () => {
    const ref = searchParams.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```

**正确示例（按需读取，无订阅）：**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search);
    const ref = params.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```
