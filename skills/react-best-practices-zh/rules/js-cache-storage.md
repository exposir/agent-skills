---
title: 缓存 Storage API 调用
impact: LOW-MEDIUM
impactDescription: 减少昂贵的 I/O
tags: javascript, localStorage, storage, caching, performance
---

## 缓存 Storage API 调用

`localStorage`, `sessionStorage`, 和 `document.cookie` 是同步且昂贵的。在内存中缓存读取。

**错误示例（每次调用都读取存储）：**

```typescript
function getTheme() {
  return localStorage.getItem("theme") ?? "light";
}
// 调用 10 次 = 10 次存储读取
```

**正确示例（Map 缓存）：**

```typescript
const storageCache = new Map<string, string | null>();

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key));
  }
  return storageCache.get(key);
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value);
  storageCache.set(key, value); // 保持缓存同步
}
```

使用 Map（而不是 hook），这样它就可以在任何地方工作：工具函数、事件处理程序，而不仅仅是 React 组件。

**Cookie 缓存：**

```typescript
let cookieCache: Record<string, string> | null = null;

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split("; ").map((c) => c.split("="))
    );
  }
  return cookieCache[name];
}
```

**重要（在外部更改时失效）：**

如果存储可以从外部更改（另一个标签页，服务器设置的 cookie），请使缓存失效：

```typescript
window.addEventListener("storage", (e) => {
  if (e.key) storageCache.delete(e.key);
});

document.addEventListener("visibilitychange", () => {
  if (document.visibilityState === "visible") {
    storageCache.clear();
  }
});
```
