# Agent Skills

AI 编码代理的技能集合。技能是打包好的指令和脚本，用于扩展代理的能力。

技能遵循 [Agent Skills](https://agentskills.io/) 格式。

## 可用技能

### react-best-practices

来自 Vercel 工程团队的 React 和 Next.js 性能优化指南。包含 8 个类别的 40+ 条规则，按影响优先级排序。

**使用场景：**

- 编写新的 React 组件或 Next.js 页面
- 实现数据获取（客户端或服务端）
- 审查代码的性能问题
- 优化包体积或加载时间

**涵盖类别：**

- 消除瀑布流 (Critical)
- 包体积优化 (Critical)
- 服务端性能 (High)
- 客户端数据获取 (Medium-High)
- 重渲染优化 (Medium)
- 渲染性能 (Medium)
- JavaScript 微优化 (Low-Medium)

### web-design-guidelines

审查 UI 代码是否符合 Web 界面最佳实践。审核你的代码是否符合 100+ 条规则，涵盖可访问性、性能和用户体验。

**使用场景：**

- “审查我的 UI”
- “检查可访问性”
- “审核设计”
- “审查 UX”
- “根据最佳实践检查我的网站”

**涵盖类别：**

- 可访问性 (aria-labels, 语义化 HTML, 键盘处理)
- 聚焦状态 (可见聚焦, focus-visible 模式)
- 表单 (自动完成, 验证, 错误处理)
- 动画 (prefers-reduced-motion, 合成器友好的变换)
- 排版 (弯引号, 省略号, tabular-nums)
- 图像 (尺寸, 懒加载, alt 文本)
- 性能 (虚拟化, 布局抖动, preconnect)
- 导航与状态 (URL 反映状态, 深度链接)
- 暗黑模式与主题 (color-scheme, theme-color meta)
- 触摸与交互 (touch-action, tap-highlight)
- 本地化与国际化 (Intl.DateTimeFormat, Intl.NumberFormat)

### vercel-deploy-claimable

将应用程序和网站即时部署到 Vercel。专为 claude.ai 和 Claude Desktop 设计，支持直接从对话中进行部署。部署是“可认领的”——用户可以将所有权转移到他们自己的 Vercel 账户。

**使用场景：**

- “部署我的应用”
- “部署到生产环境”
- “发布上线”
- “部署并给我链接”

**功能：**

- 自动检测 `package.json` 中的 40+ 种框架
- 返回预览 URL（实时站点）和认领 URL（转移所有权）
- 自动处理静态 HTML 项目
- 从上传中排除 `node_modules` 和 `.git`

**工作原理：**

1. 将你的项目打包成 tarball
2. 检测框架（Next.js, Vite, Astro 等）
3. 上传到部署服务
4. 返回预览 URL 和认领 URL

**输出：**

```
Deployment successful!

Preview URL: https://skill-deploy-abc123.vercel.app
Claim URL:   https://vercel.com/claim-deployment?code=...
```

## 安装

```bash
npx add-skill vercel-labs/agent-skills
```

## 用法

安装后，技能将自动可用。当检测到相关任务时，代理将使用它们。

**示例：**

```
Deploy my app
```

```
Review this React component for performance issues
```

```
Help me optimize this Next.js page
```

## 技能结构

每个技能包含：

- `SKILL.md` - 给代理的指令
- `scripts/` - 用于自动化的辅助脚本（可选）
- `references/` - 支持文档（可选）

## 许可证

MIT
