# AGENTS.md

此文件为 AI 编码代理（如 Claude Code, Cursor, Copilot 等）在处理本仓库代码时提供指导。

## 仓库概览

这是为 Claude.ai 和 Claude Code 提供的技能集合，用于处理 Vercel 部署。技能是打包好的指令和脚本，用于扩展 Claude 的能力。

## 创建新技能

### 目录结构

```
skills/
  {skill-name}/           # 烤串命名法（kebab-case）目录名
    SKILL.md              # 必须：技能定义
    scripts/              # 必须：可执行脚本
      {script-name}.sh    # Bash 脚本（首选）
  {skill-name}.zip        # 必须：用于分发的打包文件
```

### 命名规范

- **技能目录**：`kebab-case`（例如 `vercel-deploy`, `log-monitor`）
- **SKILL.md**：始终大写，始终使用此确切文件名
- **脚本**：`kebab-case.sh`（例如 `deploy.sh`, `fetch-logs.sh`）
- **Zip 文件**：必须与目录名完全匹配：`{skill-name}.zip`

### SKILL.md 格式

````markdown
---
name: { skill-name }
description:
  { 一句话描述何时使用此技能。包含触发词，如“部署我的应用”、“检查日志”等 }
---

# {技能标题}

{简要描述技能的作用。}

## 工作原理

{编号列表解释技能的工作流程}

## 用法

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```
````

**参数：**

- `arg1` - 描述（默认为 X）

**示例：**
{展示 2-3 个常见用法模式}

## 输出

{展示用户将看到的示例输出}

## 向用户展示结果

{Claude 向用户展示结果时的格式模板}

## 故障排除

{常见问题和解决方案，特别是网络/权限错误}

````

### 上下文效率最佳实践

技能通过按需加载——启动时仅加载技能名称和描述。完整的 `SKILL.md` 仅在代理决定该技能相关时才会加载到上下文中。为了最大程度地减少上下文使用：

- **保持 SKILL.md 在 500 行以内** — 将详细的参考资料放在单独的文件中
- **编写具体的描述** — 帮助代理确切知道何时激活技能
- **使用渐进式披露** — 引用仅在需要时才读取的支持文件
- **优先使用脚本而不是内联代码** — 脚本执行不消耗上下文（仅输出消耗）
- **文件引用仅限一层深度** — 直接从 SKILL.md 链接到支持文件

### 脚本要求

- 使用 `#!/bin/bash` shebang
- 使用 `set -e` 以实现快速失败行为
- 将状态消息写入 stderr：`echo "Message" >&2`
- 将机器可读输出 (JSON) 写入 stdout
- 包含临时文件的清理陷阱 (trap)
- 将脚本路径引用为 `/mnt/skills/user/{skill-name}/scripts/{script}.sh`

### 创建 Zip 包

创建或更新技能后：

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
````

### 最终用户安装

为用户记录这两种安装方法：

**Claude Code:**

```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
将技能添加到项目知识中，或将 SKILL.md 内容粘贴到对话中。

如果技能需要网络访问，指导用户在 `claude.ai/settings/capabilities` 中添加所需的域名。
