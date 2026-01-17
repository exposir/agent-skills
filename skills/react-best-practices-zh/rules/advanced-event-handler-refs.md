---
title: 将事件处理程序存储在 Refs 中
impact: LOW
impactDescription: 稳定的订阅
tags: advanced, hooks, refs, event-handlers, optimization
---

## 将事件处理程序存储在 Refs 中

在 Effect 中使用回调时，如果不需要在回调变化时重新订阅，请将其存储在 Refs 中。

**错误示例（每次渲染都重新订阅）：**

```tsx
function useWindowEvent(event: string, handler: () => void) {
  useEffect(() => {
    window.addEventListener(event, handler);
    return () => window.removeEventListener(event, handler);
  }, [event, handler]);
}
```

**正确示例（稳定的订阅）：**

```tsx
function useWindowEvent(event: string, handler: () => void) {
  const handlerRef = useRef(handler);
  useEffect(() => {
    handlerRef.current = handler;
  }, [handler]);

  useEffect(() => {
    const listener = () => handlerRef.current();
    window.addEventListener(event, listener);
    return () => window.removeEventListener(event, listener);
  }, [event]);
}
```

**替代方案：如果你使用的是最新的 React，请使用 `useEffectEvent`：**

```tsx
import { useEffectEvent } from "react";

function useWindowEvent(event: string, handler: () => void) {
  const onEvent = useEffectEvent(handler);

  useEffect(() => {
    window.addEventListener(event, onEvent);
    return () => window.removeEventListener(event, onEvent);
  }, [event]);
}
```

`useEffectEvent` 为相同的模式提供了更清晰的 API：它创建一个稳定的函数引用，该引用始终调用处理程序的最新版本。
