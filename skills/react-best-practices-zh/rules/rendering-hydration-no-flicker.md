---
title: 防止 Hydration 不匹配且无闪烁
impact: MEDIUM
impactDescription: 避免视觉闪烁和 hydration 错误
tags: rendering, ssr, hydration, localStorage, flicker
---

## 防止 Hydration 不匹配且无闪烁

当渲染依赖于客户端存储（localStorage, cookies）的内容时，通过注入一个在 React hydrate 之前更新 DOM 的同步脚本，避免 SSR 中断和 hydrate 后的闪烁。

**错误示例（中断 SSR）：**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage 在服务端不可用 - 抛出错误
  const theme = localStorage.getItem("theme") || "light";

  return <div className={theme}>{children}</div>;
}
```

服务端渲染将失败，因为 `localStorage` 未定义。

**错误示例（视觉闪烁）：**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState("light");

  useEffect(() => {
    // 在 hydration 后运行 - 导致可见的闪烁
    const stored = localStorage.getItem("theme");
    if (stored) {
      setTheme(stored);
    }
  }, []);

  return <div className={theme}>{children}</div>;
}
```

组件首先使用默认值 (`light`) 渲染，然后在 hydration 后更新，导致错误内容的可见闪烁。

**正确示例（无闪烁，无 hydration 不匹配）：**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">{children}</div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  );
}
```

内联脚本在显示元素之前同步执行，确保 DOM 已经具有正确的值。无闪烁，无 hydration 不匹配。

此模式对于主题切换、用户首选项、身份验证状态以及任何应立即渲染而不闪烁默认值的仅限客户端数据特别有用。
