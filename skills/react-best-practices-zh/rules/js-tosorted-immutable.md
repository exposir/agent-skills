---
title: 使用 toSorted() 代替 sort() 实现不可变性
impact: MEDIUM-HIGH
impactDescription: 防止 React 状态中的变异 bug
tags: javascript, arrays, immutability, react, state, mutation
---

## 使用 toSorted() 代替 sort() 实现不可变性

`.sort()` 会原地修改数组，这可能会导致 React state 和 props 出现 bug。使用 `.toSorted()` 创建一个新的排序数组而不进行修改。

**错误示例（变异原始数组）：**

```typescript
function UserList({ users }: { users: User[] }) {
  // 变异 users prop 数组！
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  );
  return <div>{sorted.map(renderUser)}</div>;
}
```

**正确示例（创建新数组）：**

```typescript
function UserList({ users }: { users: User[] }) {
  // 创建新排序数组，原始数组未改变
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  );
  return <div>{sorted.map(renderUser)}</div>;
}
```

**为什么这在 React 中很重要：**

1. Props/state 变异破坏了 React 的不可变性模型 - React 期望 props 和 state 被视为只读
2. 导致陈旧闭包 bug - 在闭包（callbacks, effects）中变异数组可能导致意外行为

**浏览器支持（旧浏览器的 fallback）：**

`.toSorted()` 在所有现代浏览器中都可用 (Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+)。对于较旧的环境，请使用展开运算符：

```typescript
// 旧浏览器的 Fallback
const sorted = [...items].sort((a, b) => a.value - b.value);
```

**其他不可变数组方法：**

- `.toSorted()` - 不可变排序
- `.toReversed()` - 不可变反转
- `.toSpliced()` - 不可变拼接
- `.with()` - 不可变元素替换
