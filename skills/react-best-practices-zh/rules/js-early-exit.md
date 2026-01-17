---
title: 函数的提前返回
impact: LOW-MEDIUM
impactDescription: 避免不必要的计算
tags: javascript, functions, optimization, early-return
---

## 函数的提前返回

当结果确定时提前返回，以跳过不必要的处理。

**错误示例（即使找到答案也会处理所有项目）：**

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
    // 即使发现错误也会继续检查所有用户
  }

  return hasError ? { valid: false, error: errorMessage } : { valid: true };
}
```

**正确示例（第一次出错时立即返回）：**

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
