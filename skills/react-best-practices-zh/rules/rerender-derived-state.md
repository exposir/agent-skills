---
title: 订阅派生状态
impact: MEDIUM
impactDescription: 降低重新渲染频率
tags: rerender, derived-state, media-query, optimization
---

## 订阅派生状态

订阅派生的布尔状态而不是连续值，以减少重新渲染频率。

**错误示例（在每个像素更改时重新渲染）：**

```tsx
function Sidebar() {
  const width = useWindowWidth(); // 持续更新
  const isMobile = width < 768;
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```

**正确示例（仅在布尔值更改时重新渲染）：**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery("(max-width: 767px)");
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```
