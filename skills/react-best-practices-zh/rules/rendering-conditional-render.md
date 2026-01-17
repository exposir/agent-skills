---
title: 使用显式条件渲染
impact: LOW
impactDescription: 防止渲染 0 或 NaN
tags: rendering, conditional, jsx, falsy-values
---

## 使用显式条件渲染

当条件可能为 `0`、`NaN` 或其他渲染的 falsy 值时，使用显式三元运算符 (`? :`) 代替 `&&` 进行条件渲染。

**错误示例（当 count 为 0 时渲染 "0"）：**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count && <span className="badge">{count}</span>}</div>;
}

// 当 count = 0 时，渲染：<div>0</div>
// 当 count = 5 时，渲染：<div><span class="badge">5</span></div>
```

**正确示例（当 count 为 0 时不渲染任何内容）：**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count > 0 ? <span className="badge">{count}</span> : null}</div>;
}

// 当 count = 0 时，渲染：<div></div>
// 当 count = 5 时，渲染：<div><span class="badge">5</span></div>
```
