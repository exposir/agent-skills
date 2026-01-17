---
title: 推迟 Await 直到需要时
impact: HIGH
impactDescription: 避免阻塞未使用的代码路径
tags: async, await, conditional, optimization
---

## 推迟 Await 直到需要时

将 `await` 操作移动到实际使用它们的分支中，以避免阻塞不需要它们的代码路径。

**错误示例（阻塞两个分支）：**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId);

  if (skipProcessing) {
    // 立即返回，但仍然等待了 userData
    return { skipped: true };
  }

  // 只有这个分支使用 userData
  return processUserData(userData);
}
```

**正确示例（仅在需要时阻塞）：**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 立即返回，无需等待
    return { skipped: true };
  }

  // 仅在需要时获取
  const userData = await fetchUserData(userId);
  return processUserData(userData);
}
```

**另一个示例（提前返回优化）：**

```typescript
// 错误示例：总是获取 permissions
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

// 正确示例：仅在需要时获取
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

当被跳过的分支经常被执行，或者被推迟的操作很昂贵时，这种优化特别有价值。
