---
title: 最小化 RSC 边界处的序列化
impact: HIGH
impactDescription: 减少数据传输大小
tags: server, rsc, serialization, props
---

## 最小化 RSC 边界处的序列化

React Server/Client 边界将所有对象属性序列化为字符串，并将它们嵌入到 HTML 响应和后续 RSC 请求中。序列化数据直接影响页面重量和加载时间，因此**大小非常重要**。仅传递客户端实际使用的字段。

**错误示例（序列化所有 50 个字段）：**

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

**正确示例（仅序列化 1 个字段）：**

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
