# React 最佳实践

一个结构化的仓库，用于创建和维护针对 Agent 和 LLM 优化的 React 最佳实践。

## 结构

- `rules/` - 独立的规则文件（每个规则一个文件）
  - `_sections.md` - 章节元数据（标题、影响、描述）
  - `_template.md` - 创建新规则的模板
  - `area-description.md` - 独立的规则文件
- `src/` - 构建脚本和工具
- `metadata.json` - 文档元数据（版本、组织、摘要）
- **`AGENTS.md`** - 编译输出（生成）
- **`test-cases.json`** - 用于 LLM 评估的测试用例（生成）

## 快速开始

1. 安装依赖：

   ```bash
   pnpm install
   ```

2. 从规则构建 AGENTS.md：

   ```bash
   pnpm build
   ```

3. 验证规则文件：

   ```bash
   pnpm validate
   ```

4. 提取测试用例：
   ```bash
   pnpm extract-tests
   ```

## 创建新规则

1. 复制 `rules/_template.md` 到 `rules/area-description.md`
2. 选择合适的区域前缀：
   - `async-` 用于 消除瀑布流 (第 1 章)
   - `bundle-` 用于 包体积优化 (第 2 章)
   - `server-` 用于 服务端性能 (第 3 章)
   - `client-` 用于 客户端数据获取 (第 4 章)
   - `rerender-` 用于 重渲染优化 (第 5 章)
   - `rendering-` 用于 渲染性能 (第 6 章)
   - `js-` 用于 JavaScript 性能 (第 7 章)
   - `advanced-` 用于 高级模式 (第 8 章)
3. 填写 frontmatter 和内容
4. 确保你有清晰的示例和解释
5. 运行 `pnpm build` 以重新生成 AGENTS.md 和 test-cases.json

## 规则文件结构

每个规则文件应遵循此结构：

````markdown
---
title: 在此填写规则标题
impact: MEDIUM
impactDescription: 可选描述
tags: tag1, tag2, tag3
---

## 在此填写规则标题

简要解释规则及其重要性。

**错误示例（描述有什么问题）：**

```typescript
// 糟糕的代码示例
```
````

**正确示例（描述什么是正确的）：**

```typescript
// 优秀的代码示例
```

示例后的可选解释性文本。

参考：[链接](https://example.com)

## 文件命名规范

- 以 `_` 开头的文件是特殊的（从构建中排除）
- 规则文件：`area-description.md` (例如 `async-parallel.md`)
- 章节由文件名前缀自动推断
- 规则在每个部分内按标题按字顺排序
- ID（例如 1.1, 1.2）在构建期间自动生成

## 影响级别

- `CRITICAL` - 最高优先级，主要的性能提升
- `HIGH` - 显著的性能改进
- `MEDIUM-HIGH` - 中高收益
- `MEDIUM` - 中等性能改进
- `LOW-MEDIUM` - 中低收益
- `LOW` - 增量改进

## 脚本

- `pnpm build` - 将规则编译为 AGENTS.md
- `pnpm validate` - 验证所有规则文件
- `pnpm extract-tests` - 提取用于 LLM 评估的测试用例
- `pnpm dev` - 构建并验证

## 贡献

添加或修改规则时：

1. 为你的章节使用正确的文件名前缀
2. 遵循 `_template.md` 结构
3. 包含清晰的 错误/正确 示例和解释
4. 添加适当的标签
5. 运行 `pnpm build` 以重新生成 AGENTS.md 和 test-cases.json
6. 规则按标题自动排序 - 无需管理编号！

## 致谢

最初由 [@shuding](https://x.com/shuding) 在 [Vercel](https://vercel.com) 创建。
