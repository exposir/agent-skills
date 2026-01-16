---
name: vercel-react-best-practices-zh
description: Vercel 工程团队提供的 React 和 Next.js 性能优化指南。编写、审查或重构 React/Next.js 代码时应使用此技能，以确保最佳性能模式。适用于涉及 React 组件、Next.js 页面、数据获取、打包优化或性能改进的任务。
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# Vercel React 最佳实践

由 Vercel 维护的 React 和 Next.js 应用程序综合性能优化指南。包含 8 个类别的 45 条规则，按影响力排序，用于指导自动重构和代码生成。

## 何时应用

在以下情况下参考这些指南：

- 编写新的 React 组件或 Next.js 页面
- 实现数据获取（客户端或服务端）
- 审查代码中的性能问题
- 重构现有的 React/Next.js 代码
- 优化包体积或加载时间

## 规则类别按优先级排序

| 优先级 | 类别            | 影响力             | 前缀         |
| ------ | --------------- | ------------------ | ------------ |
| 1      | 消除瀑布流      | 关键 (CRITICAL)    | `async-`     |
| 2      | 包体积优化      | 关键 (CRITICAL)    | `bundle-`    |
| 3      | 服务端性能      | 高 (HIGH)          | `server-`    |
| 4      | 客户端数据获取  | 中高 (MEDIUM-HIGH) | `client-`    |
| 5      | 重渲染优化      | 中 (MEDIUM)        | `rerender-`  |
| 6      | 渲染性能        | 中 (MEDIUM)        | `rendering-` |
| 7      | JavaScript 性能 | 中低 (LOW-MEDIUM)  | `js-`        |
| 8      | 高级模式        | 低 (LOW)           | `advanced-`  |

## 快速参考

### 1. 消除瀑布流 (CRITICAL)

- `async-defer-await` - 将 await 移动到实际使用的分支中
- `async-parallel` - 对独立操作使用 Promise.all()
- `async-dependencies` - 对部分依赖项使用 better-all
- `async-api-routes` - 在 API 路由中尽早启动 promise，并在需要时再 await
- `async-suspense-boundaries` - 使用 Suspense 以此流式传输内容

### 2. 包体积优化 (CRITICAL)

- `bundle-barrel-imports` - 直接导入，避免使用桶文件 (barrel files)
- `bundle-dynamic-imports` - 对大型组件使用 next/dynamic
- `bundle-defer-third-party` - 在水合 (hydration) 后加载分析/日志记录工具
- `bundle-conditional` - 仅在功能激活时加载模块
- `bundle-preload` - 在悬停/聚焦时预加载以提高感知速度

### 3. 服务端性能 (HIGH)

- `server-cache-react` - 使用 React.cache() 进行单次请求内的去重
- `server-cache-lru` - 使用 LRU 缓存进行跨请求缓存
- `server-serialization` - 最小化传递给客户端组件的数据
- `server-parallel-fetching` - 重构组件以并行化数据获取
- `server-after-nonblocking` - 对非阻塞操作使用 after()

### 4. 客户端数据获取 (MEDIUM-HIGH)

- `client-swr-dedup` - 使用 SWR 进行自动请求去重
- `client-event-listeners` - 对全局事件监听器进行去重

### 5. 重渲染优化 (MEDIUM)

- `rerender-defer-reads` - 如果只在回调中使用，不要订阅状态
- `rerender-memo` - 将昂贵的计算提取到记忆化组件中
- `rerender-dependencies` - 在 effect 中使用原始值依赖
- `rerender-derived-state` - 订阅派生的布尔值，而不是原始值
- `rerender-functional-setstate` - 使用函数式 setState 以获得稳定的回调
- `rerender-lazy-state-init` - 将函数传递给 useState 以进行昂贵的初始化
- `rerender-transitions` - 对非紧急更新使用 startTransition

### 6. 渲染性能 (MEDIUM)

- `rendering-animate-svg-wrapper` - 动画化 div 包装器，而不是 SVG 元素
- `rendering-content-visibility` - 对长列表使用 content-visibility
- `rendering-hoist-jsx` - 将静态 JSX 提取到组件外部
- `rendering-svg-precision` - 降低 SVG 坐标精度
- `rendering-hydration-no-flicker` - 对仅客户端数据使用内联脚本
- `rendering-activity` - 使用 Activity 组件进行显示/隐藏
- `rendering-conditional-render` - 使用三元运算符而不是 && 进行条件渲染

### 7. JavaScript 性能 (LOW-MEDIUM)

- `js-batch-dom-css` - 通过类或 cssText 批量处理 CSS 更改
- `js-index-maps` - 为重复查找构建 Map
- `js-cache-property-access` - 在循环中缓存对象属性
- `js-cache-function-results` - 在模块级 Map 中缓存函数结果
- `js-cache-storage` - 缓存 localStorage/sessionStorage 读取
- `js-combine-iterations` - 将多个 filter/map 合并为一个循环
- `js-length-check-first` - 在昂贵的比较之前先检查数组长度
- `js-early-exit` - 从函数中提前返回
- `js-hoist-regexp` - 将 RegExp 创建提升到循环外部
- `js-min-max-loop` - 使用循环查找最小/最大值而不是排序
- `js-set-map-lookups` - 使用 Set/Map 进行 O(1) 查找
- `js-tosorted-immutable` - 使用 toSorted() 保持不可变性

### 8. 高级模式 (LOW)

- `advanced-event-handler-refs` - 将事件处理程序存储在 ref 中
- `advanced-use-latest` - 使用 useLatest 获得稳定的回调 ref

## 如何使用

阅读 `AGENTS.md` 文件了解详细解释和代码示例。

## 完整编译文档

所有规则的完整指南：`AGENTS.md`
