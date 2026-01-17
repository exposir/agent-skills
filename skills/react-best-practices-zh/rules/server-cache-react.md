---
title: 使用 React.cache() 进行按请求去重
impact: MEDIUM
impactDescription: 请求内去重
tags: server, cache, react-cache, deduplication
---

## 使用 React.cache() 进行按请求去重

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

在单个请求中，对 `getCurrentUser()` 的多次调用仅执行一次查询。

**避免将内联对象作为参数：**

`React.cache()` 使用浅相等 (`Object.is`) 来确定缓存命中。内联对象每次调用都会创建新引用，从而阻止缓存命中。

**错误示例（总是缓存未命中）：**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } });
});

// 每次调用都会创建新对象，从未命中缓存
getUser({ uid: 1 });
getUser({ uid: 1 }); // 缓存未命中，再次运行查询
```

**正确示例（缓存命中）：**

```typescript
const getUser = cache(async (uid: number) => {
  return await db.user.findUnique({ where: { id: uid } });
});

// 基本参数类型使用值相等
getUser(1);
getUser(1); // 缓存命中，返回缓存结果
```

如果你必须传递对象，请传递相同的引用：

```typescript
const params = { uid: 1 };
getUser(params); // 查询运行
getUser(params); // 缓存命中（相同引用）
```

**Next.js 特定说明：**

在 Next.js 中，`fetch` API 会自动扩展请求记忆功能。具有相同 URL 和选项的请求在单个请求中会自动去重，因此你不需要为 `fetch` 调用使用 `React.cache()`。但是，`React.cache()` 对于其他异步任务仍然至关重要：

- 数据库查询（Prisma, Drizzle 等）
- 繁重的计算
- 身份验证检查
- 文件系统操作
- 任何非 fetch 异步工作

使用 `React.cache()` 在组件树中对这些操作进行去重。

参考：[React.cache documentation](https://react.dev/reference/react/cache)
