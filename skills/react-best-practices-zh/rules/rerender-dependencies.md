---
title: 缩小 Effect 依赖范围
impact: LOW
impactDescription: 最小化 effect 重新运行
tags: rerender, useEffect, dependencies, optimization
---

## 缩小 Effect 依赖范围

指定基本类型依赖而不是对象，以最大限度地减少 Effect 重新运行。

**错误示例（在任何用户字段更改时从新运行）：**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user]);
```

**正确示例（仅在 id 更改时重新运行）：**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user.id]);
```

**对于派生状态，在 Effect 外部计算：**

```tsx
// 错误示例：运行于 width=767, 766, 765...
useEffect(() => {
  if (width < 768) {
    enableMobileMode();
  }
}, [width]);

// 正确示例：仅在布尔值转换时运行
const isMobile = width < 768;
useEffect(() => {
  if (isMobile) {
    enableMobileMode();
  }
}, [isMobile]);
```
