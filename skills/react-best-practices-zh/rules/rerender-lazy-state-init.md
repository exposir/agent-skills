---
title: 使用惰性状态初始化
impact: MEDIUM
impactDescription: 每次渲染都浪费计算
tags: react, hooks, useState, performance, initialization
---

## 使用惰性状态初始化

将函数传递给 `useState` 以获取昂贵的初始值。如果没有函数形式，即使值仅使用一次，初始化程序也会在每次渲染时运行。

**错误示例（每次渲染都运行）：**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 在每次渲染时运行，即使在初始化之后
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items));
  const [query, setQuery] = useState("");

  // 当 query 改变时，buildSearchIndex再次不必要地运行
  return <SearchResults index={searchIndex} query={query} />;
}

function UserProfile() {
  // JSON.parse 在每次渲染时运行
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem("settings") || "{}")
  );

  return <SettingsForm settings={settings} onChange={setSettings} />;
}
```

**正确示例（仅运行一次）：**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 仅在初始渲染时运行
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items));
  const [query, setQuery] = useState("");

  return <SearchResults index={searchIndex} query={query} />;
}

function UserProfile() {
  // JSON.parse 仅在初始渲染时运行
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem("settings");
    return stored ? JSON.parse(stored) : {};
  });

  return <SettingsForm settings={settings} onChange={setSettings} />;
}
```

当从 localStorage/sessionStorage 计算初始值、构建数据结构（索引、Map）、从 DOM 读取或执行繁重的转换时，请使用惰性初始化。

对于简单的基本类型（`useState(0)`）、直接引用（`useState(props.value)`）或廉价的字面量（`useState({})`），函数形式是不必要的。
