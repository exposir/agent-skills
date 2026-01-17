---
title: 使用 useLatest 实现稳定的回调 Refs
impact: LOW
impactDescription: 防止 effect 重新运行
tags: advanced, hooks, useLatest, refs, optimization
---

## 使用 useLatest 实现稳定的回调 Refs

在回调中访问最新值，而无需将其添加到依赖数组中。防止 effect 重新运行，同时避免闭包过时。

**实现：**

```typescript
function useLatest<T>(value: T) {
  const ref = useRef(value);
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref;
}
```

**错误示例（每次回调变化都会重新运行 Effect）：**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300);
    return () => clearTimeout(timeout);
  }, [query, onSearch]);
}
```

**正确示例（稳定的 Effect，新的回调）：**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");
  const onSearchRef = useLatest(onSearch);

  useEffect(() => {
    const timeout = setTimeout(() => onSearchRef.current(query), 300);
    return () => clearTimeout(timeout);
  }, [query]);
}
```
