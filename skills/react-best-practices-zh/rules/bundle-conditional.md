---
title: 条件模块加载
impact: HIGH
impactDescription: 仅在需要时加载大型数据
tags: bundle, conditional-loading, lazy-loading
---

## 条件模块加载

仅在激活功能时加载大型数据或模块。

**示例（懒加载动画帧）：**

```tsx
function AnimationPlayer({
  enabled,
  setEnabled,
}: {
  enabled: boolean;
  setEnabled: React.Dispatch<React.SetStateAction<boolean>>;
}) {
  const [frames, setFrames] = useState<Frame[] | null>(null);

  useEffect(() => {
    if (enabled && !frames && typeof window !== "undefined") {
      import("./animation-frames.js")
        .then((mod) => setFrames(mod.frames))
        .catch(() => setEnabled(false));
    }
  }, [enabled, frames, setEnabled]);

  if (!frames) return <Skeleton />;
  return <Canvas frames={frames} />;
}
```

`typeof window !== 'undefined'` 检查可防止为 SSR 打包此模块，从而优化服务端包体积和构建速度。
