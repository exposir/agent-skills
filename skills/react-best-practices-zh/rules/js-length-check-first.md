---
title: 数组比较的提前长度检查
impact: MEDIUM-HIGH
impactDescription: 当长度不同时避免昂贵的操作
tags: javascript, arrays, performance, optimization, comparison
---

## 数组比较的提前长度检查

当使用昂贵操作（排序、深度相等、序列化）比较数组时，首先检查长度。如果长度不同，则数组不可能相等。

在实际应用中，当比较在热点路径（事件处理程序、渲染循环）中运行时，这种优化特别有价值。

**错误示例（总是运行昂贵的比较）：**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 总是进行排序和连接，即使长度不同
  return current.sort().join() !== original.sort().join();
}
```

即使 `current.length` 为 5 而 `original.length` 为 100，也会运行两次 O(n log n) 排序。还有连接数组和比较字符串的开销。

**正确示例（首先进行 O(1) 长度检查）：**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 如果长度不同则提前返回
  if (current.length !== original.length) {
    return true;
  }
  // 仅当长度匹配时才进行排序/连接
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
- 它避免了消耗连接字符串的内存（对于大数组尤其重要）
- 它避免了变异原始数组
- 在发现差异时提前返回
