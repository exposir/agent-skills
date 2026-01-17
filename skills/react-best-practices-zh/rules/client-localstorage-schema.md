---
title: 对 localStorage 数据进行版本控制和最小化
impact: MEDIUM
impactDescription: 防止 schema 冲突，减少存储大小
tags: client, localStorage, storage, versioning, data-minimization
---

## 对 localStorage 数据进行版本控制和最小化

为键添加版本前缀并仅存储所需的字段。防止 schema 冲突和敏感数据的意外存储。

**错误示例：**

```typescript
// 没版本，存储所有内容，无错误处理
localStorage.setItem("userConfig", JSON.stringify(fullUserObject));
const data = localStorage.getItem("userConfig");
```

**正确示例：**

```typescript
const VERSION = "v2";

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config));
  } catch {
    // 在隐身/私人浏览、配额超限或禁用时抛出
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`);
    return data ? JSON.parse(data) : null;
  } catch {
    return null;
  }
}

// 从 v1 迁移到 v2
function migrate() {
  try {
    const v1 = localStorage.getItem("userConfig:v1");
    if (v1) {
      const old = JSON.parse(v1);
      saveConfig({
        theme: old.darkMode ? "dark" : "light",
        language: old.lang,
      });
      localStorage.removeItem("userConfig:v1");
    }
  } catch {}
}
```

**存储服务端响应中的最小字段：**

```typescript
// User 对象有 20+ 个字段，只存储 UI 需要的
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem(
      "prefs:v1",
      JSON.stringify({
        theme: user.preferences.theme,
        notifications: user.preferences.notifications,
      })
    );
  } catch {}
}
```

**始终包裹在 try-catch 中：** `getItem()` 和 `setItem()` 在隐身/私人浏览（Safari, Firefox）、配额超限或被禁用时会抛出异常。

**好处：** 通过版本控制实现 Schema 演进，减少存储大小，防止存储 token/PII/内部标志。
