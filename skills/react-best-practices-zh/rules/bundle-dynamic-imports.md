---
title: 重型组件的动态导入
impact: CRITICAL
impactDescription: 直接影响 TTI 和 LCP
tags: bundle, dynamic-import, code-splitting, next-dynamic
---

## 重型组件的动态导入

使用 `next/dynamic` 懒加载初始渲染不需要的大型组件。

**错误示例（Monaco 打包在主 Chunk 中 ~300KB）：**

```tsx
import { MonacoEditor } from "./monaco-editor";

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />;
}
```

**正确示例（Monaco 按需加载）：**

```tsx
import dynamic from "next/dynamic";

const MonacoEditor = dynamic(
  () => import("./monaco-editor").then((m) => m.MonacoEditor),
  { ssr: false }
);

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />;
}
```
