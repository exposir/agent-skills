# React 最佳实践

**版本 1.0.0**
Vercel Engineering
2026 年 1 月

> **注意：**
> 本文档主要供智能体 (Agents) 和大语言模型 (LLMs) 在维护、
> 生成或重构 Vercel 的 React 和 Next.js 代码库时遵循。人类
> 也可能会发现它很有用，但这里的指南针对 AI 辅助工作流程的自动化
> 和一致性进行了优化。

---

## 摘要

为人工智能智能体和 LLM 设计的 React 和 Next.js 应用程序综合性能优化指南。包含 8 个类别的 40 多条规则，按影响力从关键（消除瀑布流、减少包体积）到增量（高级模式）进行优先级排序。每条规则都包含详细的解释、比较错误与正确实现的真实世界示例，以及具体的性能指标，以指导自动重构和代码生成。

---

## 目录

1. [消除瀑布流](#1-eliminating-waterfalls) — **关键 (CRITICAL)**
   - 1.1 [推迟 Await 直到需要时](#11-defer-await-until-needed)
   - 1.2 [基于依赖的并行化](#12-dependency-based-parallelization)
   - 1.3 [防止 API 路由中的瀑布流链](#13-prevent-waterfall-chains-in-api-routes)
   - 1.4 [对独立操作使用 Promise.all()](#14-promiseall-for-independent-operations)
   - 1.5 [策略性 Suspense 边界](#15-strategic-suspense-boundaries)
2. [包体积优化](#2-bundle-size-optimization) — **关键 (CRITICAL)**
   - 2.1 [避免桶文件导入](#21-avoid-barrel-file-imports)
   - 2.2 [条件模块加载](#22-conditional-module-loading)
   - 2.3 [推迟非关键第三方库](#23-defer-non-critical-third-party-libraries)
   - 2.4 [对大型组件使用动态导入](#24-dynamic-imports-for-heavy-components)
   - 2.5 [基于用户意图预加载](#25-preload-based-on-user-intent)
3. [服务端性能](#3-server-side-performance) — **高 (HIGH)**
   - 3.1 [跨请求 LRU 缓存](#31-cross-request-lru-caching)
   - 3.2 [最小化 RSC 边界的序列化](#32-minimize-serialization-at-rsc-boundaries)
   - 3.3 [通过组件组合并行获取数据](#33-parallel-data-fetching-with-component-composition)
   - 3.4 [使用 React.cache() 进行单次请求去重](#34-per-request-deduplication-with-reactcache)
   - 3.5 [对非阻塞操作使用 after()](#35-use-after-for-non-blocking-operations)
4. [客户端数据获取](#4-client-side-data-fetching) — **中高 (MEDIUM-HIGH)**
   - 4.1 [对全局事件监听器进行去重](#41-deduplicate-global-event-listeners)
   - 4.2 [使用 SWR 进行自动去重](#42-use-swr-for-automatic-deduplication)
5. [重渲染优化](#5-re-render-optimization) — **中 (MEDIUM)**
   - 5.1 [推迟状态读取到使用点](#51-defer-state-reads-to-usage-point)
   - 5.2 [提取到记忆化组件](#52-extract-to-memoized-components)
   - 5.3 [缩小 Effect 依赖范围](#53-narrow-effect-dependencies)
   - 5.4 [订阅派生状态](#54-subscribe-to-derived-state)
   - 5.5 [使用函数式 setState 更新](#55-use-functional-setstate-updates)
   - 5.6 [使用惰性状态初始化](#56-use-lazy-state-initialization)
   - 5.7 [对非紧急更新使用 Transitions](#57-use-transitions-for-non-urgent-updates)
6. [渲染性能](#6-rendering-performance) — **中 (MEDIUM)**
   - 6.1 [动画化 SVG 包装器而不是 SVG 元素](#61-animate-svg-wrapper-instead-of-svg-element)
   - 6.2 [对长列表使用 CSS content-visibility](#62-css-content-visibility-for-long-lists)
   - 6.3 [提升静态 JSX 元素](#63-hoist-static-jsx-elements)
   - 6.4 [优化 SVG 精度](#64-optimize-svg-precision)
   - 6.5 [防止无闪烁的水合不匹配](#65-prevent-hydration-mismatch-without-flickering)
   - 6.6 [使用 Activity 组件进行显示/隐藏](#66-use-activity-component-for-showhide)
   - 6.7 [使用显式条件渲染](#67-use-explicit-conditional-rendering)
7. [JavaScript 性能](#7-javascript-performance) — **中低 (LOW-MEDIUM)**
   - 7.1 [批量处理 DOM CSS 更改](#71-batch-dom-css-changes)
   - 7.2 [为重复查找构建索引 Map](#72-build-index-maps-for-repeated-lookups)
   - 7.3 [在循环中缓存属性访问](#73-cache-property-access-in-loops)
   - 7.4 [缓存重复函数调用](#74-cache-repeated-function-calls)
   - 7.5 [缓存 Storage API 调用](#75-cache-storage-api-calls)
   - 7.6 [合并多个数组迭代](#76-combine-multiple-array-iterations)
   - 7.7 [数组比较前先检查长度](#77-early-length-check-for-array-comparisons)
   - 7.8 [从函数提前返回](#78-early-return-from-functions)
   - 7.9 [提升 RegExp 创建](#79-hoist-regexp-creation)
   - 7.10 [使用循环代替排序查找最小/最大值](#710-use-loop-for-minmax-instead-of-sort)
   - 7.11 [使用 Set/Map 进行 O(1) 查找](#711-use-setmap-for-o1-lookups)
   - 7.12 [使用 toSorted() 代替 sort() 以保持不可变性](#712-use-tosorted-instead-of-sort-for-immutability)
8. [高级模式](#8-advanced-patterns) — **低 (LOW)**
   - 8.1 [将事件处理程序存储在 Refs 中](#81-store-event-handlers-in-refs)
   - 8.2 [使用 useLatest 获得稳定的回调 Refs](#82-uselatest-for-stable-callback-refs)

---

## 1. 消除瀑布流

**影响力：关键 (CRITICAL)**

瀑布流是头号性能杀手。每个顺序的 `await` 都会增加完整的网络延迟。消除它们能产生最大的收益。

### 1.1 推迟 Await 直到需要时

**影响力：高 (避免阻塞未使用的代码路径)**

将 `await` 操作移动到实际使用它们的分支中，以避免阻塞不需要它们的代码路径。

**错误：阻塞了两个分支**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId);

  if (skipProcessing) {
    // 立即返回，但仍然等待了 userData
    return { skipped: true };
  }

  // 只有这个分支使用了 userData
  return processUserData(userData);
}
```

**正确：仅在需要时阻塞**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 立即返回而不等待
    return { skipped: true };
  }

  // 仅在需要时获取
  const userData = await fetchUserData(userId);
  return processUserData(userData);
}
```

**另一个例子：提前返回优化**

```typescript
// 错误：总是获取权限
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId);
  const resource = await getResource(resourceId);

  if (!resource) {
    return { error: "Not found" };
  }

  if (!permissions.canEdit) {
    return { error: "Forbidden" };
  }

  return await updateResourceData(resource, permissions);
}

// 正确：仅在需要时获取
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId);

  if (!resource) {
    return { error: "Not found" };
  }

  const permissions = await fetchPermissions(userId);

  if (!permissions.canEdit) {
    return { error: "Forbidden" };
  }

  return await updateResourceData(resource, permissions);
}
```

当被跳过的分支频繁执行，或者被推迟的操作很昂贵时，这种优化特别有价值。

### 1.2 基于依赖的并行化

**影响力：关键 (2-10 倍提升)**

对于具有部分依赖的操作，使用 `better-all` 来最大化并行性。它会在尽可能早的时刻自动启动每个任务。

**错误：profile 不必要地等待 config**

```typescript
const [user, config] = await Promise.all([fetchUser(), fetchConfig()]);
const profile = await fetchProfile(user.id);
```

**正确：config 和 profile 并行运行**

```typescript
import { all } from "better-all";

const { user, config, profile } = await all({
  async user() {
    return fetchUser();
  },
  async config() {
    return fetchConfig();
  },
  async profile() {
    return fetchProfile((await this.$.user).id);
  },
});
```

参考：[https://github.com/shuding/better-all](https://github.com/shuding/better-all)

### 1.3 防止 API 路由中的瀑布流链

**影响力：关键 (2-10 倍提升)**

在 API 路由和 Server Actions 中，即使尚未 `await` 它们，也要立即启动独立操作。

**错误：config 等待 auth，data 等待两者**

```typescript
export async function GET(request: Request) {
  const session = await auth();
  const config = await fetchConfig();
  const data = await fetchData(session.user.id);
  return Response.json({ data, config });
}
```

**正确：auth 和 config 立即启动**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth();
  const configPromise = fetchConfig();
  const session = await sessionPromise;
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id),
  ]);
  return Response.json({ data, config });
}
```

对于具有更复杂依赖链的操作，使用 `better-all` 自动最大化并行性（见基于依赖的并行化）。

### 1.4 对独立操作使用 Promise.all()

**影响力：关键 (2-10 倍提升)**

当异步操作没有相互依赖关系时，使用 `Promise.all()` 并发执行它们。

**错误：顺序执行，3 次往返**

```typescript
const user = await fetchUser();
const posts = await fetchPosts();
const comments = await fetchComments();
```

**正确：并行执行，1 次往返**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments(),
]);
```

### 1.5 策略性 Suspense 边界

**影响力：高 (更快的首次绘制)**

与其在异步组件中等待数据再返回 JSX，不如使用 Suspense 边界在数据加载时更快地显示包装器 UI。

**错误：包装器被数据获取阻塞**

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

即使只有中间部分需要数据，整个布局也会等待数据。

**正确：包装器立即显示，数据流式传输**

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

**替代方案：跨组件共享 Promise**

```tsx
function Page() {
  // 立即启动获取，但不要 await
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
  const data = use(dataPromise); // 解包 Promise
  return <div>{data.content}</div>;
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise); // 重用同一个 Promise
  return <div>{data.summary}</div>;
}
```

两个组件共享同一个 Promise，因此只发生一次获取。布局立即渲染，而两个组件一起等待。

**何时不使用此模式：**

- 布局决策所需的关键数据（影响定位）

- 首屏的关键 SEO 内容

- 极快的小查询，Suspense 开销不值得

- 当你想避免布局偏移（加载中 -> 内容跳变）时

**权衡：** 更快的首次绘制 vs 潜在的布局偏移。根据你的 UX 优先级进行选择。

---

## 2. 包体积优化

**影响力：关键**

减少初始包体积可改善交互时间 (Time to Interactive) 和最大内容绘制 (Largest Contentful Paint)。

### 2.1 避免桶文件导入

**影响力：关键 (200-800ms 导入成本，构建缓慢)**

直接从源文件导入，而不是从桶文件导入，以避免加载数千个未使用的模块。**桶文件 (Barrel files)** 是重新导出多个模块的入口点（例如，执行 `export * from './module'` 的 `index.js`）。

流行的图标和组件库在其入口文件中可能有 **多达 10,000 个重新导出**。对于许多 React 包，**仅导入它们就需要 200-800ms**，影响开发速度和生产环境冷启动。

**为什么 tree-shaking 没用：** 当库被标记为外部（未打包）时，打包器无法优化它。如果你为了启用 tree-shaking 而打包它，构建由于分析整个模块图会变得非常慢。

**错误：导入整个库**

```tsx
import { Check, X, Menu } from "lucide-react";
// 加载 1,583 个模块，开发环境额外耗时 ~2.8s
// 运行时成本：每次冷启动 200-800ms

import { Button, TextField } from "@mui/material";
// 加载 2,225 个模块，开发环境额外耗时 ~4.2s
```

**正确：仅导入你需要的**

```tsx
import Check from "lucide-react/dist/esm/icons/check";
import X from "lucide-react/dist/esm/icons/x";
import Menu from "lucide-react/dist/esm/icons/menu";
// 仅加载 3 个模块 (~2KB vs ~1MB)

import Button from "@mui/material/Button";
import TextField from "@mui/material/TextField";
// 仅加载你使用的
```

**替代方案：Next.js 13.5+**

```js
// next.config.js - 使用 optimizePackageImports
module.exports = {
  experimental: {
    optimizePackageImports: ["lucide-react", "@mui/material"],
  },
};

// 然后你可以保留符合人体工程学的桶导入：
import { Check, X, Menu } from "lucide-react";
// 在构建时自动转换为直接导入
```

直接导入可提供 15-70% 更快的开发启动速度，28% 更快的构建速度，40% 更快的冷启动速度，以及显著更快的 HMR。

受影响的常见库：`lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`。

参考：[https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)

### 2.2 条件模块加载

**影响力：高 (仅在需要时加载大数据)**

仅在功能激活时加载大数据或模块。

**示例：懒加载动画帧**

```tsx
function AnimationPlayer({
  enabled,
  setEnabled,
}: {
  enabled: boolean;
  setEnabled: (v: boolean) => void;
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

`typeof window !== 'undefined'` 检查防止了在 SSR 时打包此模块，优化了服务端包体积和构建速度。

### 2.3 推迟非关键第三方库

**影响力：中 (水合后加载)**

分析、日志和错误跟踪不会阻止用户交互。在水合后加载它们。

**错误：阻塞初始包**

```tsx
import { Analytics } from "@vercel/analytics/react";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

**正确：水合后加载**

```tsx
import dynamic from "next/dynamic";

const Analytics = dynamic(
  () => import("@vercel/analytics/react").then((m) => m.Analytics),
  { ssr: false }
);

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

### 2.4 对大型组件使用动态导入

**影响力：关键 (直接影响 TTI 和 LCP)**

使用 `next/dynamic` 懒加载初始渲染时不需要的大型组件。

**错误：Monaco 与主块一起打包 ~300KB**

```tsx
import { MonacoEditor } from "./monaco-editor";

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />;
}
```

**正确：Monaco 按需加载**

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

### 2.5 基于用户意图预加载

**影响力：中 (减少感知延迟)**

在需要之前预加载沉重的包，以减少感知延迟。

**示例：悬停/聚焦时预加载**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== "undefined") {
      void import("./monaco-editor");
    }
  };

  return (
    <button onMouseEnter={preload} onFocus={preload} onClick={onClick}>
      Open Editor
    </button>
  );
}
```

**示例：功能标志启用时预加载**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== "undefined") {
      void import("./monaco-editor").then((mod) => mod.init());
    }
  }, [flags.editorEnabled]);

  return (
    <FlagsContext.Provider value={flags}>{children}</FlagsContext.Provider>
  );
}
```

`typeof window !== 'undefined'` 检查防止了在 SSR 时打包预加载模块，优化了服务端包体积和构建速度。

---

## 3. 服务端性能

**影响力：高 (HIGH)**

优化服务端渲染和数据获取可消除服务端瀑布流并减少响应时间。

### 3.1 跨请求 LRU 缓存

**影响力：高 (跨请求缓存)**

`React.cache()` 仅在一个请求内有效。对于跨顺序请求共享的数据（用户点击按钮 A 然后点击按钮 B），使用 LRU 缓存。

**实现：**

```typescript
import { LRUCache } from "lru-cache";

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000, // 5 分钟
});

export async function getUser(id: string) {
  const cached = cache.get(id);
  if (cached) return cached;

  const user = await db.user.findUnique({ where: { id } });
  cache.set(id, user);
  return user;
}

// 请求 1: DB 查询，结果被缓存
// 请求 2: 缓存命中，无 DB 查询
```

当顺序用户操作在几秒钟内命中需要相同数据的多个端点时使用。

**使用 Vercel 的 [Fluid Compute](https://vercel.com/docs/fluid-compute)：** LRU 缓存特别有效，因为多个并发请求可以共享相同的函数实例和缓存。这意味着缓存可以跨请求持久化，而不需要 Redis 等外部存储。

**在传统无服务器架构中：** 每次调用都在隔离环境中运行，因此考虑使用 Redis 进行跨进程缓存。

参考：[https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)

### 3.2 最小化 RSC 边界的序列化

**影响力：高 (减少数据传输大小)**

React Server/Client 边界将所有对象属性序列化为字符串，并将它们嵌入到 HTML 响应和后续的 RSC 请求中。此序列化数据直接影响页面重量和加载时间，所以**大小非常重要**。仅传递客户端实际使用的字段。

**错误：序列化所有 50 个字段**

```tsx
async function Page() {
  const user = await fetchUser(); // 50 个字段
  return <Profile user={user} />;
}

("use client");
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>; // 使用 1 个字段
}
```

**正确：仅序列化 1 个字段**

```tsx
async function Page() {
  const user = await fetchUser();
  return <Profile name={user.name} />;
}

("use client");
function Profile({ name }: { name: string }) {
  return <div>{name}</div>;
}
```

### 3.3 通过组件组合并行获取数据

**影响力：关键 (消除服务端瀑布流)**

React Server Components 在树内顺序执行。通过组合重构以并行化数据获取。

**错误：Sidebar 等待 Page 的获取完成**

```tsx
export default async function Page() {
  const header = await fetchHeader();
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  );
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}
```

**正确：两者同时获取**

```tsx
async function Header() {
  const data = await fetchHeader();
  return <div>{data}</div>;
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  );
}
```

**使用 children prop 的替代方案：**

```tsx
async function Layout({ children }: { children: ReactNode }) {
  const header = await fetchHeader();
  return (
    <div>
      <div>{header}</div>
      {children}
    </div>
  );
}

async function Sidebar() {
  const items = await fetchSidebarItems();
  return <nav>{items.map(renderItem)}</nav>;
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  );
}
```

### 3.4 使用 React.cache() 进行单次请求去重

**影响力：中 (请求内去重)**

使用 `React.cache()` 进行服务端请求去重。身份验证和数据库查询受益最大。

**用法：**

```typescript
import { cache } from "react";

export const getCurrentUser = cache(async () => {
  const session = await auth();
  if (!session?.user?.id) return null;
  return await db.user.findUnique({
    where: { id: session.user.id },
  });
});
```

在单个请求中，多次调用 `getCurrentUser()` 仅执行一次查询。

### 3.5 对非阻塞操作使用 after()

**影响力：中 (更快的响应时间)**

使用 Next.js 的 `after()` 来安排应在响应发送后执行的工作。这可以防止日志记录、分析和其他副作用阻塞响应。

**错误：阻塞响应**

```tsx
import { logUserAction } from "@/app/utils";

export async function POST(request: Request) {
  // 执行变更
  await updateDatabase(request);

  // 日志记录阻塞响应
  const userAgent = request.headers.get("user-agent") || "unknown";
  await logUserAction({ userAgent });

  return new Response(JSON.stringify({ status: "success" }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
}
```

**正确：非阻塞**

```tsx
import { after } from "next/server";
import { headers, cookies } from "next/headers";
import { logUserAction } from "@/app/utils";

export async function POST(request: Request) {
  // 执行变更
  await updateDatabase(request);

  // 响应发送后记录日志
  after(async () => {
    const userAgent = (await headers()).get("user-agent") || "unknown";
    const sessionCookie =
      (await cookies()).get("session-id")?.value || "anonymous";

    logUserAction({ sessionCookie, userAgent });
  });

  return new Response(JSON.stringify({ status: "success" }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
}
```

响应立即发送，而日志记录在后台进行。

**常见用例：**

- 分析跟踪

- 审计日志

- 发送通知

- 缓存失效

- 清理任务

**重要提示：**

- 即使响应失败或重定向，`after()` 也会运行

- 适用于 Server Actions、路由处理程序和 Server Components

参考：[https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)

---

## 4. 客户端数据获取

**影响力：中高 (MEDIUM-HIGH)**

自动去重和高效的数据获取模式减少冗余网络请求。

### 4.1 对全局事件监听器进行去重

**影响力：低 (N 个组件 1 个监听器)**

使用 `useSWRSubscription()` 在组件实例之间共享全局事件监听器。

**错误：N 个实例 = N 个监听器**

```tsx
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback();
      }
    };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  }, [key, callback]);
}
```

当多次使用 `useKeyboardShortcut` hook 时，每个实例都会注册一个新的监听器。

**正确：N 个实例 = 1 个监听器**

```tsx
import useSWRSubscription from "swr/subscription";

// 模块级 Map 跟踪每个按键的回调
const keyCallbacks = new Map<string, Set<() => void>>();

function useKeyboardShortcut(key: string, callback: () => void) {
  //将此回调注册到 Map 中
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set());
    }
    keyCallbacks.get(key)!.add(callback);

    return () => {
      const set = keyCallbacks.get(key);
      if (set) {
        set.delete(callback);
        if (set.size === 0) {
          keyCallbacks.delete(key);
        }
      }
    };
  }, [key, callback]);

  useSWRSubscription("global-keydown", () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach((cb) => cb());
      }
    };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  });
}

function Profile() {
  // 多个快捷键将共享同一个监听器
  useKeyboardShortcut("p", () => {
    /* ... */
  });
  useKeyboardShortcut("k", () => {
    /* ... */
  });
  // ...
}
```

### 4.2 使用 SWR 进行自动去重

**影响力：中高 (自动去重)**

SWR 支持跨组件实例的请求去重、缓存和重新验证。

**错误：无去重，每个实例都获取**

```tsx
function UserList() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch("/api/users")
      .then((r) => r.json())
      .then(setUsers);
  }, []);
}
```

**正确：多个实例共享一个请求**

```tsx
import useSWR from "swr";

function UserList() {
  const { data: users } = useSWR("/api/users", fetcher);
}
```

**对于不可变数据：**

```tsx
import { useImmutableSWR } from "@/lib/swr";

function StaticContent() {
  const { data } = useImmutableSWR("/api/config", fetcher);
}
```

**对于变更：**

```tsx
import { useSWRMutation } from "swr/mutation";

function UpdateButton() {
  const { trigger } = useSWRMutation("/api/user", updateUser);
  return <button onClick={() => trigger()}>Update</button>;
}
```

参考：[https://swr.vercel.app](https://swr.vercel.app)

---

## 5. 重渲染优化

**影响力：中 (MEDIUM)**

减少不必要的重渲染可最大限度地减少浪费的计算并提高 UI 响应能力。

### 5.1 推迟状态读取到使用点

**影响力：中 (避免不必要的订阅)**

如果只在回调中使用动态状态（searchParams, localStorage），请勿订阅它。

**错误：订阅所有 searchParams 更改**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams();

  const handleShare = () => {
    const ref = searchParams.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```

**正确：按需读取，无订阅**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search);
    const ref = params.get("ref");
    shareChat(chatId, { ref });
  };

  return <button onClick={handleShare}>Share</button>;
}
```

### 5.2 提取到记忆化组件

**影响力：中 (启用提前返回)**

将昂贵的工作提取到记忆化组件中，以便在计算前启用提前返回。

**错误：即使正在加载也计算头像**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user);
    return <Avatar id={id} />;
  }, [user]);

  if (loading) return <Skeleton />;
  return <div>{avatar}</div>;
}
```

**正确：加载时跳过计算**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user]);
  return <Avatar id={id} />;
});

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />;
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  );
}
```

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，手动使用 `memo()` 和 `useMemo()` 进行记忆化是不必要的。编译器会自动优化重渲染。

### 5.3 缩小 Effect 依赖范围

**影响力：低 (最大限度地减少 effect 重新运行)**

指定原始依赖项而不是对象，以最大限度地减少 effect 重新运行。

**错误：在任何用户字段更改时重新运行**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user]);
```

**正确：仅在 id 更改时重新运行**

```tsx
useEffect(() => {
  console.log(user.id);
}, [user.id]);
```

**对于派生状态，在 effect 外部计算：**

```tsx
// 错误：在 width=767, 766, 765... 时运行
useEffect(() => {
  if (width < 768) {
    enableMobileMode();
  }
}, [width]);

// 正确：仅在布尔值转换时运行
const isMobile = width < 768;
useEffect(() => {
  if (isMobile) {
    enableMobileMode();
  }
}, [isMobile]);
```

### 5.4 订阅派生状态

**影响力：中 (减少重渲染频率)**

订阅派生的布尔状态而不是连续值，以减少重渲染频率。

**错误：在每个像素变化时重渲染**

```tsx
function Sidebar() {
  const width = useWindowWidth(); // 持续更新
  const isMobile = width < 768;
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```

**正确：仅在布尔值更改时重渲染**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery("(max-width: 767px)");
  return <nav className={isMobile ? "mobile" : "desktop"} />;
}
```

### 5.5 使用函数式 setState 更新

**影响力：中 (防止闭包陷阱和不必要的回调重建)**

当基于当前状态值更新状态时，使用函数式更新形式的 setState，而不是直接引用状态变量。这可以防止闭包陷阱，消除不必要的依赖，并创建稳定的回调引用。

**错误：需要 state 作为依赖**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // 回调必须依赖 items，每次 items 改变都会重建
  const addItems = useCallback(
    (newItems: Item[]) => {
      setItems([...items, ...newItems]);
    },
    [items]
  ); // ❌ items 依赖导致重建

  // 如果忘记依赖，会有闭包陷阱风险
  const removeItem = useCallback((id: string) => {
    setItems(items.filter((item) => item.id !== id));
  }, []); // ❌ 缺少 items 依赖 - 将使用陈旧的 items！

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

第一个回调每次 `items` 改变时都会重建，这可能导致子组件不必要地重渲染。第二个回调有一个闭包陷阱 bug——它总是引用初始的 `items` 值。

**正确：稳定的回调，无闭包陷阱**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems);

  // 稳定的回调，从未重建
  const addItems = useCallback((newItems: Item[]) => {
    setItems((curr) => [...curr, ...newItems]);
  }, []); // ✅ 不需要依赖

  // 总是使用最新状态，无闭包陷阱风险
  const removeItem = useCallback((id: string) => {
    setItems((curr) => curr.filter((item) => item.id !== id));
  }, []); // ✅ 安全且稳定

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />;
}
```

**好处：**

1. **稳定的回调引用** - 状态改变时无需重建回调

2. **无闭包陷阱** - 总是操作最新状态值

3. **更少的依赖** - 简化依赖数组并减少内存泄漏

4. **防止 bug** - 消除 React 闭包 bug 的最常见来源

**何时使用函数式更新：**

- 任何依赖于当前状态值的 setState

- 在 useCallback/useMemo 内部需要状态时

- 引用状态的事件处理程序

- 更新状态的异步操作

**何时直接更新是可以的：**

- 将状态设置为静态值：`setCount(0)`

- 仅从 props/参数设置状态：`setName(newName)`

- 状态不依赖于前一个值

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，编译器可以自动优化某些情况，但仍建议使用函数式更新以确保正确性并防止闭包陷阱 bug。

### 5.6 使用惰性状态初始化

**影响力：中 (每次渲染浪费计算)**

将函数传递给 `useState` 以获取昂贵的初始值。如果不使用函数形式，初始化程序将在每次渲染时运行，即使该值仅使用一次。

**错误：每次渲染都运**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 每次渲染都运行，即使初始化后也是如此
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items));
  const [query, setQuery] = useState("");

  // 当 query 改变时，buildSearchIndex 不必要地再次运行
  return <SearchResults index={searchIndex} query={query} />;
}

function UserProfile() {
  // JSON.parse 每次渲染都运行
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem("settings") || "{}")
  );

  return <SettingsForm settings={settings} onChange={setSettings} />;
}
```

**正确：仅运行一次**

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

当从 localStorage/sessionStorage 计算初始值、构建数据结构（索引、映射）、从 DOM 读取或执行繁重的转换时，使用惰性初始化。

对于简单的原语 (`useState(0)`)、直接引用 (`useState(props.value)`) 或廉价的字面量 (`useState({})`)，函数形式是不必要的。

### 5.7 对非紧急更新使用 Transitions

**影响力：中 (保持 UI 响应能力)**

将频繁、非紧急的状态更新标记为 transitions，以保持 UI 响应能力。

**错误：每次滚动都阻塞 UI**

```tsx
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY);
    window.addEventListener("scroll", handler, { passive: true });
    return () => window.removeEventListener("scroll", handler);
  }, []);
}
```

**正确：非阻塞更新**

```tsx
import { startTransition } from "react";

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY));
    };
    window.addEventListener("scroll", handler, { passive: true });
    return () => window.removeEventListener("scroll", handler);
  }, []);
}
```

---

## 6. 渲染性能

**影响力：中 (MEDIUM)**

优化渲染过程可减少浏览器需要做的工作。

### 6.1 动画化 SVG 包装器而不是 SVG 元素

**影响力：低 (启用硬件加速)**

许多浏览器不支持对 SVG 元素进行 CSS3 动画硬件加速。将 SVG 包装在 `<div>` 中并对包装器进行动画处理。

**错误：直接动画化 SVG - 无硬件加速**

```tsx
function LoadingSpinner() {
  return (
    <svg className="animate-spin" width="24" height="24" viewBox="0 0 24 24">
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  );
}
```

**正确：动画化包装器 div - 硬件加速**

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

### 6.2 对长列表使用 CSS content-visibility

**影响力：高 (更快的首次渲染)**

应用 `content-visibility: auto` 来推迟屏幕外渲染。

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**示例：**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map((msg) => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  );
}
```

对于 1000 条消息，浏览器会跳过 ~990 个屏幕外项目的布局/绘制（首次渲染快 10 倍）。

### 6.3 提升静态 JSX 元素

**影响力：低 (避免重新创建)**

将静态 JSX 提取到组件外部以避免重新创建。

**错误：每次渲染都重新创建元素**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />;
}

function Container() {
  return <div>{loading && <LoadingSkeleton />}</div>;
}
```

**正确：重用同一个元素**

```tsx
const loadingSkeleton = <div className="animate-pulse h-20 bg-gray-200" />;

function Container() {
  return <div>{loading && loadingSkeleton}</div>;
}
```

这对于大型且静态的 SVG 节点特别有用，因为在每次渲染时重新创建它们的代价可能很高。

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，编译器会自动提升静态 JSX 元素并优化组件重渲染，使手动提升变得不必要。

### 6.4 优化 SVG 精度

**影响力：低 (减少文件大小)**

降低 SVG 坐标精度以减小文件大小。最佳精度取决于 viewBox 大小，但通常应考虑降低精度。

**错误：精度过高**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

**正确：1 位小数**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />
```

**使用 SVGO 自动化：**

```bash
npx svgo --precision=1 --multipass icon.svg
```

### 6.5 防止无闪烁的水合不匹配

**影响力：中 (避免视觉闪烁和水合错误)**

当渲染依赖于客户端存储（localStorage, cookies）的内容时，通过注入一个在 React 水合之前更新 DOM 的同步脚本，来避免 SSR 中断和水合后的闪烁。

**错误：破坏 SSR**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage 在服务端不可用 - 抛出错误
  const theme = localStorage.getItem("theme") || "light";

  return <div className={theme}>{children}</div>;
}
```

由于 `localStorage` 未定义，服务端渲染将失败。

**错误：视觉闪烁**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState("light");

  useEffect(() => {
    // 水合后运行 - 导致可见的闪烁
    const stored = localStorage.getItem("theme");
    if (stored) {
      setTheme(stored);
    }
  }, []);

  return <div className={theme}>{children}</div>;
}
```

组件首先使用默认值 (`light`) 渲染，然后在水合后更新，导致不正确内容的可见闪烁。

**正确：无闪烁，无水合不匹配**

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

内联脚本在显示元素之前同步执行，确保 DOM 已经具有正确的值。无闪烁，无水合不匹配。

此模式对于主题切换、用户偏好、身份验证状态以及任何应立即渲染而不闪烁默认值的仅客户端数据特别有用。

### 6.6 使用 Activity 组件进行显示/隐藏

**影响力：中 (保留状态/DOM)**

使用 React 的 `<Activity>` 来为频繁切换可见性的昂贵组件保留状态/DOM。

**用法：**

```tsx
import { Activity } from "react";

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? "visible" : "hidden"}>
      <ExpensiveMenu />
    </Activity>
  );
}
```

避免昂贵的重渲染和状态丢失。

### 6.7 使用显式条件渲染

**影响力：低 (防止渲染 0 或 NaN)**

当条件可能是 `0`、`NaN` 或其他会渲染的假值时，使用显式的三元运算符 (`? :`) 而不是 `&&` 进行条件渲染。

**错误：当 count 为 0 时渲染 "0"**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count && <span className="badge">{count}</span>}</div>;
}

// 当 count = 0, 渲染: <div>0</div>
// 当 count = 5, 渲染: <div><span class="badge">5</span></div>
```

**正确：当 count 为 0 时不渲染任何内容**

```tsx
function Badge({ count }: { count: number }) {
  return <div>{count > 0 ? <span className="badge">{count}</span> : null}</div>;
}

// 当 count = 0, 渲染: <div></div>
// 当 count = 5, 渲染: <div><span class="badge">5</span></div>
```

---

## 7. JavaScript 性能

**影响力：中低 (LOW-MEDIUM)**

热路径的微优化可以累积成有意义的改进。

### 7.1 批量处理 DOM CSS 更改

**影响力：中 (减少重排/重绘)**

避免一次更改一个属性的样式。通过类或 `cssText` 将多个 CSS 更改组合在一起，以最大限度地减少浏览器重排。

**错误：多次重排**

```typescript
function updateElementStyles(element: HTMLElement) {
  // 每一行触发一次重排
  element.style.width = "100px";
  element.style.height = "200px";
  element.style.backgroundColor = "blue";
  element.style.border = "1px solid black";
}
```

**正确：添加类 - 单次重排**

```typescript
// CSS 文件
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}

// JavaScript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
}
```

**正确：更改 cssText - 单次重排**

```typescript
function updateElementStyles(element: HTMLElement) {
  element.style.cssText = `
    width: 100px;
    height: 200px;
    background-color: blue;
    border: 1px solid black;
  `;
}
```

**React 示例：**

```tsx
// 错误：逐个更改样式
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = "100px";
      ref.current.style.height = "200px";
      ref.current.style.backgroundColor = "blue";
    }
  }, [isHighlighted]);

  return <div ref={ref}>Content</div>;
}

// 正确：切换类
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return <div className={isHighlighted ? "highlighted-box" : ""}>Content</div>;
}
```

如果可能，优先使用 CSS 类而不是内联样式。类会被浏览器缓存，并提供更好的关注点分离。

### 7.2 为重复查找构建索引 Map

**影响力：中低 (1M 次操作变为 2K 次)**

同一键的多次 `.find()` 调用应使用 Map。

**错误 (每次查找 O(n))：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map((order) => ({
    ...order,
    user: users.find((u) => u.id === order.userId),
  }));
}
```

**正确 (每次查找 O(1))：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map((u) => [u.id, u]));

  return orders.map((order) => ({
    ...order,
    user: userById.get(order.userId),
  }));
}
```

构建一次 Map (O(n))，然后所有查找都是 O(1)。

对于 1000 个订单 × 1000 个用户：1M 次操作 → 2K 次操作。

### 7.3 在循环中缓存属性访问

**影响力：中低 (减少查找)**

在热路径中缓存对象属性查找。

**错误：3 次查找 × N 次迭代**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value);
}
```

**正确：总共 1 次查找**

```typescript
const value = obj.config.settings.value;
const len = arr.length;
for (let i = 0; i < len; i++) {
  process(value);
}
```

### 7.4 缓存重复函数调用

**影响力：中 (避免冗余计算)**

当在渲染期间使用相同的输入重复调用相同的函数时，使用模块级 Map 缓存函数结果。

**错误：冗余计算**

```typescript
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map((project) => {
        // slugify() 对相同的项目名称调用了 100+ 次
        const slug = slugify(project.name);

        return <ProjectCard key={project.id} slug={slug} />;
      })}
    </div>
  );
}
```

**正确：缓存结果**

```typescript
// 模块级缓存
const slugifyCache = new Map<string, string>();

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!;
  }
  const result = slugify(text);
  slugifyCache.set(text, result);
  return result;
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map((project) => {
        // 每个唯一的项目名称仅计算一次
        const slug = cachedSlugify(project.name);

        return <ProjectCard key={project.id} slug={slug} />;
      })}
    </div>
  );
}
```

**单值函数的更简单模式：**

```typescript
let isLoggedInCache: boolean | null = null;

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache;
  }

  isLoggedInCache = document.cookie.includes("auth=");
  return isLoggedInCache;
}

// 当 auth 改变时清除缓存
function onAuthChange() {
  isLoggedInCache = null;
}
```

使用 Map（而不是 hook），使其随处可用：实用程序、事件处理程序，而不仅仅是 React 组件。

参考：[https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

### 7.5 缓存 Storage API 调用

**影响力：中低 (减少昂贵的 I/O)**

`localStorage`、`sessionStorage` 和 `document.cookie` 是同步且昂贵的。在内存中缓存读取。

**错误：每次调用都读取存储**

```typescript
function getTheme() {
  return localStorage.getItem("theme") ?? "light";
}
// 调用 10 次 = 10 次存储读取
```

**正确：Map 缓存**

```typescript
const storageCache = new Map<string, string | null>();

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key));
  }
  return storageCache.get(key);
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value);
  storageCache.set(key, value); // 保持缓存同步
}
```

使用 Map（而不是 hook），使其随处可用：实用程序、事件处理程序，而不仅仅是 React 组件。

**Cookie 缓存：**

```typescript
let cookieCache: Record<string, string> | null = null;

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split("; ").map((c) => c.split("="))
    );
  }
  return cookieCache[name];
}
```

**重要：在外部更改时失效**

```typescript
window.addEventListener("storage", (e) => {
  if (e.key) storageCache.delete(e.key);
});

document.addEventListener("visibilitychange", () => {
  if (document.visibilityState === "visible") {
    storageCache.clear();
  }
});
```

如果存储可以在外部更改（另一个标签页，服务器设置的 cookie），使缓存失效：

### 7.6 合并多个数组迭代

**影响力：中低 (减少迭代)**

多个 `.filter()` 或 `.map()` 调用会多次迭代数组。合并为一个循环。

**错误：3 次迭代**

```typescript
const admins = users.filter((u) => u.isAdmin);
const testers = users.filter((u) => u.isTester);
const inactive = users.filter((u) => !u.isActive);
```

**正确：1 次迭代**

```typescript
const admins: User[] = [];
const testers: User[] = [];
const inactive: User[] = [];

for (const user of users) {
  if (user.isAdmin) admins.push(user);
  if (user.isTester) testers.push(user);
  if (!user.isActive) inactive.push(user);
}
```

### 7.7 数组比较前先检查长度

**影响力：中高 (当长度不同时避免昂贵的操作)**

在进行昂贵的数组比较操作（排序、深度相等、序列化）之前，先检查长度。如果长度不同，数组不可能相等。

在现实世界的应用中，当比较运行在热路径（事件处理程序、渲染循环）中时，此优化特别有价值。

**错误：即使长度不同也总是运行昂贵的比较**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 即使长度不同也总是排序和连接
  return current.sort().join() !== original.sort().join();
}
```

即使 `current.length` 为 5 而 `original.length` 为 100，也会运行两次 O(n log n) 排序。还有连接数组和比较字符串的开销。

**正确 (先进行 O(1) 长度检查)：**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 如果长度不同则提前返回
  if (current.length !== original.length) {
    return true;
  }
  // 仅当长度匹配时进行排序/连接
  const currentSorted = current.toSorted();
  const originalSorted = original.toSorted();
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true;
    }
  }
  return false;
}
```

这种新方法更有效，因为：

- 当长度不同时，它避免了排序和连接数组的开销

- 它避免了消耗内存用于连接的字符串（对于大型数组尤为重要）

- 它避免了修改原始数组

- 当发现差异时，它会提前返回

### 7.8 从函数提前返回

**影响力：中低 (避免不必要的计算)**

当结果已确定时提前返回，以跳过不必要的处理。

**错误：即使找到答案后仍处理所有项目**

```typescript
function validateUsers(users: User[]) {
  let hasError = false;
  let errorMessage = "";

  for (const user of users) {
    if (!user.email) {
      hasError = true;
      errorMessage = "Email required";
    }
    if (!user.name) {
      hasError = true;
      errorMessage = "Name required";
    }
    // 即使发现错误仍继续检查所有用户
  }

  return hasError ? { valid: false, error: errorMessage } : { valid: true };
}
```

**正确：在第一个错误时立即返回**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: "Email required" };
    }
    if (!user.name) {
      return { valid: false, error: "Name required" };
    }
  }

  return { valid: true };
}
```

### 7.9 提升 RegExp 创建

**影响力：中低 (避免重新创建)**

不要在渲染中创建 RegExp。将所有的 RegExp 提升到模块范围或使用 `useMemo()` 进行记忆化。

**错误：每次渲染都新建 RegExp**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**正确：记忆化或提升**

```tsx
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**警告：全局正则表达式具有可变状态**

```typescript
const regex = /foo/g;
regex.test("foo"); // true, lastIndex = 3
regex.test("foo"); // false, lastIndex = 0
```

全局正则表达式 (`/g`) 具有可变的 `lastIndex` 状态：

### 7.10 使用循环代替排序查找最小/最大值

**影响力：低 (O(n) 代替 O(n log n))**

查找最小或最大元素仅需要对数组进行一次遍历。排序是浪费且较慢的。

**错误 (O(n log n) - 排序以查找最新)：**

```typescript
interface Project {
  id: string;
  name: string;
  updatedAt: number;
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt);
  return sorted[0];
}
```

仅仅为了找到最大值而对整个数组进行排序。

**错误 (O(n log n) - 排序以查找最旧和最新)：**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt);
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] };
}
```

当只需要最小/最大值时，仍然不必要地进行排序。

**正确 (O(n) - 单次循环)：**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null;

  let latest = projects[0];

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i];
    }
  }

  return latest;
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null };

  let oldest = projects[0];
  let newest = projects[0];

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i];
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i];
  }

  return { oldest, newest };
}
```

单次遍历数组，无复制，无排序。

**替代方案：用于小数组的 Math.min/Math.max**

```typescript
const numbers = [5, 2, 8, 1, 9];
const min = Math.min(...numbers);
const max = Math.max(...numbers);
```

这适用于小数组，但由于展开运算符的限制，对于非常大的数组可能会较慢。为了可靠性，请使用循环方法。

### 7.11 使用 Set/Map 进行 O(1) 查找

**影响力：中低 (O(n) 变为 O(1))**

将数组转换为 Set/Map 以进行重复的成员资格检查。

**错误 (每次检查 O(n))：**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

**正确 (每次检查 O(1))：**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```

### 7.12 使用 toSorted() 代替 sort() 以保持不可变性

**影响力：中高 (防止 React 状态中的变更 bug)**

`.sort()` 就地改变数组，这可能会导致 React 状态和 props 的 bug。使用 `.toSorted()` 创建一个新的排序数组而不进行变更。

**错误：改变原始数组**

```typescript
function UserList({ users }: { users: User[] }) {
  // 改变了 users prop 数组！
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  );
  return <div>{sorted.map(renderUser)}</div>;
}
```

**正确：创建新数组**

```typescript
function UserList({ users }: { users: User[] }) {
  // 创建新的排序数组，原始数组未改变
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  );
  return <div>{sorted.map(renderUser)}</div>;
}
```

**为什么这对 React 很重要：**

1. Props/state 变更破坏了 React 的不可变性模型 - React 期望 props 和 state 被视为只读

2. 导致闭包陷阱 bug - 在闭包（回调、effects）内改变数组会导致意外行为

**浏览器支持：旧浏览器的回退**

```typescript
// 旧浏览器的回退
const sorted = [...items].sort((a, b) => a.value - b.value);
```

所有现代浏览器（Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+）都支持 `.toSorted()`。对于旧环境，使用展开运算符：

**其他不可变数组方法：**

- `.toSorted()` - 不可变排序

- `.toReversed()` - 不可变反转

- `.toSpliced()` - 不可变拼接

- `.with()` - 不可变元素替换

---

## 8. 高级模式

**影响力：低 (LOW)**

针对需要仔细实现的特定情况的高级模式。

### 8.1 将事件处理程序存储在 Refs 中

**影响力：低 (稳定的订阅)**

当在不应因回调更改而重新订阅的 effect 中使用回调时，将回调存储在 refs 中。

**错误：每次渲染都重新订阅**

```tsx
function useWindowEvent(event: string, handler: () => void) {
  useEffect(() => {
    window.addEventListener(event, handler);
    return () => window.removeEventListener(event, handler);
  }, [event, handler]);
}
```

**正确：稳定的订阅**

```tsx
import { useEffectEvent } from "react";

function useWindowEvent(event: string, handler: () => void) {
  const onEvent = useEffectEvent(handler);

  useEffect(() => {
    window.addEventListener(event, onEvent);
    return () => window.removeEventListener(event, onEvent);
  }, [event]);
}
```

**替代方案：如果你使用的是最新 React，请使用 `useEffectEvent`：**

`useEffectEvent` 为同一种模式提供了更清晰的 API：它创建一个稳定的函数引用，该引用始终调用处理程序的最新版本。

### 8.2 useLatest 获得稳定的回调 Refs

**影响力：低 (防止 effect 重新运行)**

在不将回调添加到依赖数组的情况下访问回调中的最新值。防止 effect 重新运行，同时避免闭包陷阱。

**实现：**

```typescript
function useLatest<T>(value: T) {
  const ref = useRef(value);
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref;
}
```

**错误：每次回调更改时 effect 都会重新运行**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300);
    return () => clearTimeout(timeout);
  }, [query, onSearch]);
}
```

**正确：稳定的 effect，新鲜的回调**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");
  const onSearchRef = useLatest(onSearch);

  useEffect(() => {
    const timeout = setTimeout(() => onSearchRef.current(query), 300);
    return () => clearTimeout(timeout);
  }, [query]);
}
```

---

## 参考资料

1. [https://react.dev](https://react.dev)
2. [https://nextjs.org](https://nextjs.org)
3. [https://swr.vercel.app](https://swr.vercel.app)
4. [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
5. [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
6. [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
7. [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
