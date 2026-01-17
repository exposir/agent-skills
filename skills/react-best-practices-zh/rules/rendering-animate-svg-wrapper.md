---
title: 动画化 SVG 包装器而不是 SVG 元素
impact: LOW
impactDescription: 启用硬件加速
tags: rendering, svg, css, animation, performance
---

## 动画化 SVG 包装器而不是 SVG 元素

许多浏览器对 SVG 元素上的 CSS3 动画没有硬件加速。将 SVG 包裹在 `<div>` 中并动画化包装器。

**错误示例（直接动画化 SVG - 无硬件加速）：**

```tsx
function LoadingSpinner() {
  return (
    <svg className="animate-spin" width="24" height="24" viewBox="0 0 24 24">
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  );
}
```

**正确示例（动画化包装器 div - 硬件加速）：**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg width="24" height="24" viewBox="0 0 24 24">
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  );
}
```

这适用于所有 CSS 变换和过渡（`transform`, `opacity`, `translate`, `scale`, `rotate`）。包装器 div 允许浏览器使用 GPU 加速以获得更流畅的动画。
