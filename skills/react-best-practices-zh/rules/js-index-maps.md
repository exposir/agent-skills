---
title: 为重复查找构建索引 Map
impact: LOW-MEDIUM
impactDescription: 1M ops 降至 2K ops
tags: javascript, map, indexing, optimization, performance
---

## 为重复查找构建索引 Map

按相同键进行的多次 `.find()` 调用应使用 Map。

**错误示例（每次查找 O(n)）：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map((order) => ({
    ...order,
    user: users.find((u) => u.id === order.userId),
  }));
}
```

**正确示例（每次查找 O(1)）：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map((u) => [u.id, u]));

  return orders.map((order) => ({
    ...order,
    user: userById.get(order.userId),
  }));
}
```

构建 Map 一次 (O(n))，然后所有查找都是 O(1)。
对于 1000 个订单 × 1000 个用户：1M ops → 2K ops。
