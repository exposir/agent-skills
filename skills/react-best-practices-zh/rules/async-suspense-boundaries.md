---
title: 策略性 Suspense 边界
impact: HIGH
impactDescription: 更快的首次绘制
tags: async, suspense, streaming, layout-shift
---

## 策略性 Suspense 边界

不要在异步组件中等待数据再返回 JSX，而是使用 Suspense 边界在数据加载时更快地显示包装器 UI。

**错误示例（包装器被数据获取阻塞）：**

```tsx
async function Page() {
  const data = await fetchData(); // 阻塞整个页面

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  );
}
```

整个布局都在等待数据，即使只有中间部分需要它。

**正确示例（包装器立即显示，数据流式传输）：**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  );
}

async function DataDisplay() {
  const data = await fetchData(); // 仅阻塞此组件
  return <div>{data.content}</div>;
}
```

Sidebar、Header 和 Footer 立即渲染。只有 DataDisplay 等待数据。

**替代方案（跨组件共享 Promise）：**

```tsx
function Page() {
  // 立即开始 fetch，但不 await
  const dataPromise = fetchData();

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  );
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise); // 解包 promise
  return <div>{data.content}</div>;
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise); // 复用同一个 promise
  return <div>{data.summary}</div>;
}
```

两个组件共享同一个 Promise，因此只会发生一次 fetch。布局立即渲染，而两个组件一起等待。

**何时不使用此模式：**

- 布局决策所需的关键数据（影响定位）
- 首屏上方对 SEO 至关重要的内容
- 小型、快速的查询，不值得使用 suspense 开销
- 当你想避免布局偏移时（加载中 → 内容跳动）

**权衡：** 更快的首次绘制 vs 潜在的布局偏移。根据你的 UX 优先级进行选择。
