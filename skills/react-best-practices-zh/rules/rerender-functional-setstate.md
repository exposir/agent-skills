---
title: 使用函数式 setState 更新
impact: MEDIUM
impactDescription: 防止陈旧闭包和不必要的回调重建
tags: react, hooks, useState, useCallback, callbacks, closures
---

## 使用函数式 setState 更新

当根据当前状态值更新状态时，使用 setState 的函数式更新形式，而不是直接引用状态变量。这可以防止陈旧闭包，消除不必要的依赖，并创建稳定的回调引用。

**错误示例（需要 state 作为依赖）：**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // 回调必须依赖 items，每次 items 改变时都会重新创建
  const addItems = useCallback(
    (newItems: Item[]) => {
      setItems([...items, ...newItems]);
    },
    [items]
  ); // ❌ items 依赖会导致重新创建

  // 如果忘记依赖，就有陈旧闭包的风险
  const removeItem = useCallback((id: string) => {
    setItems(items.filter((item) => item.id !== id));
  }, []); // ❌ 缺少 items 依赖 - 将使用陈旧的 items！

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

第一个回调每次 `items` 改变时都会重新创建，这会导致子组件不必要的重新渲染。第二个回调有一个陈旧闭包 bug——它总是引用初始的 `items` 值。

**正确示例（稳定的回调，无陈旧闭包）：**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // 稳定的回调，从未重新创建
  const addItems = useCallback((newItems: Item[]) => {
    setItems((curr) => [...curr, ...newItems]);
  }, []); // ✅ 不需要依赖

  // 总是使用最新状态，无陈旧闭包风险
  const removeItem = useCallback((id: string) => {
    setItems((curr) => curr.filter((item) => item.id !== id));
  }, []); // ✅ 安全且稳定

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

**好处：**

1. **稳定的回调引用** - 状态改变时不需要重新创建回调
2. **无陈旧闭包** - 总是操作最新的状态值
3. **更少的依赖** - 简化依赖数组并减少内存泄漏
4. **防止 bug** - 消除 React 闭包 bug 的最常见来源

**何时使用函数式更新：**

- 任何依赖于当前状态值的 setState
- 在 useCallback/useMemo 内部需要状态时
- 引用状态的事件处理程序
- 更新状态的异步操作

**何时直接更新是可以的：**

- 将状态设置为静态值：`setCount(0)`
- 仅从 props/arguments 设置状态：`setName(newName)`
- 状态不依赖于先前的值

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，编译器可以自动优化某些情况，但为了正确性和防止陈旧闭包 bug，仍然建议使用函数式更新。
