# Skills 仓库 — Agent Skills 完整参考

> 本文件汇总了 Agent Skills 开放标准、OpenCode 实现、Claude Code 扩展功能的完整技术资料。

---

## 1. 信息来源

| 来源                        | URL                                                             | 说明                                                 |
| --------------------------- | --------------------------------------------------------------- | ---------------------------------------------------- |
| **Agent Skills 开放标准**   | https://agentskills.io                                          | Anthropic 发起的跨产品开放标准，被 30+ AI 客户端支持 |
| **Agent Skills 规范**       | https://agentskills.io/specification                            | SKILL.md 格式、frontmatter 字段、目录结构的完整定义  |
| **Agent Skills 最佳实践**   | https://agentskills.io/skill-creation/best-practices            | 如何写好 skill 的指导                                |
| **Agent Skills 快速入门**   | https://agentskills.io/skill-creation/quickstart                | 5 分钟创建第一个 skill                               |
| **Claude Code Skills 文档** | https://code.claude.com/docs/en/skills                          | Claude Code 的完整 skills 文档（最详细）             |
| **OpenCode Skills 文档**    | https://opencode.ai/docs/skills/                                | OpenCode 的 skills 文档                              |
| **OpenCode Skills 源码**    | `packages/opencode/src/skill/`                                  | discovery.ts（远程拉取）、index.ts（注册发现）       |
| **GitHub 示例仓库**         | https://github.com/anthropics/skills                            | Anthropic 官方示例 skills                            |
| **skills-ref 校验工具**     | https://github.com/agentskills/agentskills/tree/main/skills-ref | 命令行校验工具                                       |

---

## 2. Agent Skills 开放标准（agentskills.io）

### 2.1 目录结构

```
skill-name/
├── SKILL.md          # 必须：元数据 + 指令
├── scripts/          # 可选：可执行脚本
├── references/       # 可选：参考文档
├── assets/           # 可选：模板、资源
└── ...               # 任意额外文件
```

### 2.2 SKILL.md 格式

SKILL.md 由 YAML frontmatter + Markdown 正文组成：

```markdown
---
name: skill-name
description: 描述这个 skill 做什么、何时使用它。
license: MIT
compatibility: Designed for Claude Code (or similar products)
metadata:
  author: example-org
  version: "1.0"
allowed-tools: Bash(git:*) Bash(jq:*) Read
---

## 指令正文

这里是 agent 激活 skill 后读取的完整指令...
```

### 2.3 Frontmatter 字段（开放标准规范）

| 字段            | 必填 | 约束                                                                               | 说明                                       |
| --------------- | ---- | ---------------------------------------------------------------------------------- | ------------------------------------------ |
| `name`          | 是   | 1-64 字符，小写字母+数字+连字符，不以 `-` 开头/结尾，无连续 `--`，必须匹配父目录名 | skill 标识符                               |
| `description`   | 是   | 1-1024 字符                                                                        | 描述功能和触发时机，agent 据此决定是否激活 |
| `license`       | 否   | 短文本                                                                             | 许可证名称或引用                           |
| `compatibility` | 否   | 1-500 字符                                                                         | 环境要求（目标产品、系统包、网络等）       |
| `metadata`      | 否   | string→string 键值对                                                               | 任意扩展元数据                             |
| `allowed-tools` | 否   | 空格分隔字符串                                                                     | 预授权工具列表（实验性）                   |

### 2.4 name 字段校验规则

正则：`^[a-z0-9]+(-[a-z0-9]+)*$`

- 有效：`pdf-processing`、`data-analysis`、`code-review`
- 无效：`PDF-Processing`（大写）、`-pdf`（以连字符开头）、`pdf--processing`（连续连字符）

### 2.5 渐进式加载（Progressive Disclosure）

1. **发现**（~100 tokens）：启动时只加载 `name` + `description`
2. **激活**（建议 <5000 tokens）：任务匹配时加载完整 SKILL.md 正文
3. **执行**（按需）：按需加载 scripts/、references/、assets/ 中的文件

建议 SKILL.md 正文保持在 **500 行以内**，详细参考资料放到单独文件中。

### 2.6 校验工具

```bash
skills-ref validate ./my-skill
```

---

## 3. OpenCode 实现

### 3.1 Skill 存放位置

OpenCode 搜索以下路径的 `*/SKILL.md`：

| 位置                | 路径                                        | 作用域   |
| ------------------- | ------------------------------------------- | -------- |
| 项目配置            | `.opencode/skills/<name>/SKILL.md`          | 当前项目 |
| 全局配置            | `~/.config/opencode/skills/<name>/SKILL.md` | 所有项目 |
| Claude 兼容（项目） | `.claude/skills/<name>/SKILL.md`            | 当前项目 |
| Claude 兼容（全局） | `~/.claude/skills/<name>/SKILL.md`          | 所有项目 |
| Agent 兼容（项目）  | `.agents/skills/<name>/SKILL.md`            | 当前项目 |
| Agent 兼容（全局）  | `~/.agents/skills/<name>/SKILL.md`          | 所有项目 |

优先级：企业 > 全局 > 项目。同名 skill 以高优先级为准。

### 3.2 远程 Skills 发现（index.json）

OpenCode 启动时通过 `skills.urls` 配置拉取远程 skills。

**配置方式**（`opencode.json` / `opencode.jsonc`）：

```json
{
  "skills": {
    "urls": ["https://518luck.github.io/skills/.well-known/skills/"]
  }
}
```

**index.json 格式**（源码：`discovery.ts`）：

```json
{
  "skills": [
    {
      "name": "skill-name",
      "files": ["SKILL.md", "references/guide.md"]
    }
  ]
}
```

| 字段             | 说明                                                            |
| ---------------- | --------------------------------------------------------------- |
| `skills`         | skill 列表（数组）                                              |
| `skills[].name`  | skill 目录名，用于拼接下载 URL                                  |
| `skills[].files` | 需要下载的文件列表，**必须包含 `SKILL.md`**（否则该条目被跳过） |

**下载逻辑**：

- 请求 `<base-url>/index.json` 解析
- 过滤掉 `files` 中不含 `SKILL.md` 的条目
- 对每个 skill，下载 `<base-url>/<name>/<file>` 到本地缓存 `~/.cache/opencode/skills/<name>/`
- 缓存存在则跳过下载

### 3.3 OpenCode Frontmatter 字段

OpenCode 识别的 frontmatter 字段（比开放标准更严格）：

| 字段            | 必填 | 说明                 |
| --------------- | ---- | -------------------- |
| `name`          | 是   | 必须匹配目录名       |
| `description`   | 是   | 1-1024 字符          |
| `license`       | 否   | 许可证               |
| `compatibility` | 否   | 环境兼容性           |
| `metadata`      | 否   | string→string 键值对 |

未知字段被忽略。

### 3.4 权限控制

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

| 值      | 行为             |
| ------- | ---------------- |
| `allow` | 直接加载         |
| `deny`  | 隐藏，拒绝访问   |
| `ask`   | 需用户确认后加载 |

支持通配符：`internal-*` 匹配 `internal-docs`、`internal-tools` 等。

### 3.5 禁用 skill 工具

```json
{
  "agent": {
    "plan": {
      "tools": { "skill": false }
    }
  }
}
```

### 3.6 动态上下文注入

OpenCode 支持在 SKILL.md 中使用 `` !`command` `` 语法进行动态上下文注入，但**仅在通过斜杠命令调用时生效**。

**行为差异**：

| 调用方式               | `` !`command` `` 是否执行 | 说明                                       |
| ---------------------- | ------------------------- | ------------------------------------------ |
| `/skill-name` 斜杠命令 | 是                        | skill 注册为 command，内容经过扩展管道处理 |
| `skill` 工具自动加载   | 否                        | 内容原样传递，不做 shell 扩展              |

**实现原理**（源码 `packages/opencode/src/session/prompt.ts`）：

1. skill 被注册为 command，其 SKILL.md 内容成为命令模板
2. 调用时，`SHELL_REGEX`（`/!`([^`]+)`/g`）匹配所有 `` !`command` `` 模式
3. 所有匹配的 shell 命令通过 `Process.text()` 并行执行（`nothrow: true`）
4. 命令的 stdout 输出替换原始的 `` !`command` `` 占位符

**示例**：

```markdown
## 上下文

### GIT DIFF

!`git diff`

### GIT STATUS

!`git status --short`
```

### 3.7 内置 Skill

OpenCode 内置了一个 `customize-opencode` skill，当用户编辑 opencode 自身配置时自动激活。

---

## 4. Claude Code 扩展功能

> Claude Code 实现了 Agent Skills 开放标准，并在此基础上增加了许多扩展字段和功能。
> OpenCode 也实现了部分扩展功能（如动态上下文注入、字符串替换），见第 3 节。

### 4.1 额外 Frontmatter 字段

| 字段                       | 说明                                                                        |
| -------------------------- | --------------------------------------------------------------------------- |
| `when_to_use`              | 附加触发上下文（追加到 description，共享 1536 字符上限）                    |
| `argument-hint`            | 自动补全时的参数提示，如 `[issue-number]`                                   |
| `arguments`                | 命名位置参数，用于 `$name` 替换                                             |
| `disable-model-invocation` | `true` = 禁止 Claude 自动加载，仅手动 `/name` 触发                          |
| `user-invocable`           | `false` = 从 `/` 菜单隐藏，仅 Claude 可调用                                 |
| `model`                    | 激活时切换的模型                                                            |
| `effort`                   | 激活时的推理深度：`low`/`medium`/`high`/`xhigh`/`max`                       |
| `context`                  | 设为 `fork` 则在子代理中运行                                                |
| `agent`                    | `context: fork` 时指定子代理类型（`Explore`、`Plan`、`general-purpose` 等） |
| `hooks`                    | 绑定到此 skill 生命周期的钩子                                               |
| `paths`                    | Glob 模式，限制 skill 仅在匹配路径时自动激活                                |
| `shell`                    | `bash`（默认）或 `powershell`                                               |

### 4.2 字符串替换变量

| 变量                   | 说明                         |
| ---------------------- | ---------------------------- |
| `$ARGUMENTS`           | 调用时传入的所有参数         |
| `$ARGUMENTS[N]` / `$N` | 第 N 个参数（0-based）       |
| `$name`                | `arguments` 中声明的命名参数 |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID                  |
| `${CLAUDE_EFFORT}`     | 当前推理深度                 |
| `${CLAUDE_SKILL_DIR}`  | 当前 skill 目录路径          |

### 4.3 动态上下文注入

行内命令（`` !`command` ``）在 skill 内容发送给模型**之前**执行，输出替换占位符。

**OpenCode 也支持此功能**（仅限斜杠命令调用），详见第 3.6 节。

```markdown
## PR 上下文

- PR diff: !`gh pr diff`
- 变更文件: !`gh pr diff --name-only`
```

多行命令使用 fenced code block：

````markdown
```!
node --version
npm --version
git status --short
```
````

### 4.4 skillOverrides 设置

从设置中覆盖 skill 可见性：

```json
{
  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  }
}
```

| 值                      | 对 Claude 可见 | 在 `/` 菜单 |
| ----------------------- | -------------- | ----------- |
| `"on"`                  | 名称+描述      | 是          |
| `"name-only"`           | 仅名称         | 是          |
| `"user-invocable-only"` | 隐藏           | 是          |
| `"off"`                 | 隐藏           | 隐藏        |

### 4.5 skill 内容生命周期

- 激活后内容在会话中持续存在
- 自动压缩时，重新附加最近一次调用的 skill（前 5000 tokens）
- 多个 skill 共享 25000 tokens 的组合预算

### 4.6 Skill 存放位置（Claude Code）

| 级别 | 路径                               |
| ---- | ---------------------------------- |
| 企业 | 通过 managed settings 部署         |
| 个人 | `~/.claude/skills/<name>/SKILL.md` |
| 项目 | `.claude/skills/<name>/SKILL.md`   |
| 插件 | `<plugin>/skills/<name>/SKILL.md`  |

---

## 5. 本仓库（518luck/skills）的实际配置

### 5.1 仓库结构

```
skills/
├── .nojekyll                                    # 空文件，防止 GitHub Pages 用 Jekyll 处理
├── .well-known/
│   └── skills/
│       ├── index.json                           # skill 索引
│       ├── english-to-chinese/
│       │   └── SKILL.md                         # 英译中 skill
│       └── git-commit/
│           └── SKILL.md                         # git commit skill
└── README.md
```

### 5.2 index.json 当前内容

```json
{
  "skills": [
    {
      "name": "english-to-chinese",
      "files": ["SKILL.md"]
    },
    {
      "name": "git-commit",
      "files": ["SKILL.md"]
    }
  ]
}
```

### 5.3 在 OpenCode 中使用

在 `~/.config/opencode/opencode.jsonc` 中添加：

```jsonc
{
  "skills": {
    "urls": ["https://518luck.github.io/skills/.well-known/skills/"],
  },
}
```

重启 OpenCode 或清除缓存 `~/.cache/opencode/skills/` 使远程 skills 生效。

### 5.4 添加新 skill 的步骤

1. 创建 `.well-known/skills/<skill-name>/SKILL.md`
2. 在 `index.json` 的 `skills` 数组中添加条目：

```json
{
  "name": "<skill-name>",
  "files": ["SKILL.md"]
}
```

3. 如有辅助文件，添加到 `files` 数组和对应目录：

```json
{
  "name": "my-skill",
  "files": ["SKILL.md", "references/guide.md"]
}
```

4. `git add` + `git commit` + `git push`
5. 清除本地缓存 `rm -rf ~/.cache/opencode/skills/` 或等缓存自然过期

---

## 6. 兼容性矩阵

本仓库的 skills 兼容所有支持 Agent Skills 开放标准的产品：

| 产品              | skill 路径                                                  | 支持远程 index.json      |
| ----------------- | ----------------------------------------------------------- | ------------------------ |
| OpenCode          | `.opencode/skills/` / `.claude/skills/` / `.agents/skills/` | 是（通过 `skills.urls`） |
| Claude Code       | `.claude/skills/`                                           | 否（本地 only）          |
| VS Code + Copilot | `.agents/skills/`                                           | 否                       |
| Cursor            | `.agents/skills/`                                           | 否                       |
| Gemini CLI        | `.agents/skills/`                                           | 否                       |
| OpenAI Codex      | `.agents/skills/`                                           | 否                       |
| JetBrains Junie   | `.agents/skills/`                                           | 否                       |

> 远程 skills 发现（index.json）目前是 OpenCode 独有功能。
> 其他产品需要将 skills 文件放在本地目录中。

---

## 7. SKILL.md 编写最佳实践

### Do

- description 要具体，包含触发关键词："英译中"→"Translates English text to Chinese. Use when the user asks to translate English to Chinese, or when working with bilingual content."
- 正文保持简洁，步骤化，避免叙述性解释
- 将详细参考资料放到 `references/` 目录，在 SKILL.md 中引用
- 用 Gotchas（陷阱）列表记录容易出错的地方
- 提供输出模板，agent 对具体结构的匹配比文字描述更可靠

### Don't

- 不要写 agent 已经知道的东西（什么是 PDF、HTTP 怎么工作等）
- 不要让 SKILL.md 超过 500 行
- 不要在 description 中写泛泛之词（"Helps with code"）
- 不要在同一个 skill 中塞入过多不相关的功能
