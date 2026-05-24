# Skills 制作规范

本仓库用于维护可复用的 Agent Skills。AI 在本仓库中创建或修改 skill 时，必须遵循本文件，目标是产出同时兼容 OpenCode、OpenAI Codex、Claude Code 以及其他遵循 Agent Skills 开放标准的客户端的 skill。

详细背景资料保存在 `docs/面向AI的Skill制作操作手册.md` 和 `docs/agent-skills-overview.md`。本文件是执行规范，不是资料汇总。

---

## 1. 核心原则

- 以 Agent Skills 开放标准作为最低公共兼容层。
- 默认只使用 OpenCode、Claude 都能安全消费的目录结构和 frontmatter。
- 只有用户明确要求某个平台专有能力时，才使用平台扩展字段或动态命令注入。
- Skill 必须封装一个连贯、可复用的工作单元，不要把多个不相关能力塞进同一个 skill。
- `SKILL.md` 只放每次执行都需要的核心指令；长文档、模板、脚本按需放到辅助目录。
- 写项目特定、领域特定、容易出错的知识；不要解释 agent 已经知道的通用概念。
- 优先给清晰默认流程，而不是罗列等价选项。
- 每个 skill 都要有可执行的工作流、关键约束、必要的输出格式或验证方式。

---

## 2. 仓库结构

本仓库的 canonical skill 位置是：

```text
.well-known/skills/<skill-name>/SKILL.md
```

可选辅助目录：

```text
.well-known/skills/<skill-name>/
├── SKILL.md
├── references/   # 可选：长参考资料、API 说明、规则细节
├── assets/       # 可选：模板、样例、静态资源
└── scripts/      # 可选：可执行脚本或验证器
```

索引文件：

```text
.well-known/skills/index.json
```

不要在没有用户要求的情况下创建 `.opencode/skills/`、`.claude/skills/` 或 `.agents/skills/` 镜像目录。本仓库通过 `.well-known/skills/` 维护远程发布源。

---

## 3. 跨客户端兼容策略

同一个 skill 要能被不同客户端使用，应遵循以下约束：

| 客户端      | 推荐本地路径                                                                                           | 本仓库支持方式                                |
| ----------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| OpenCode    | `.opencode/skills/<name>/SKILL.md`、`.agents/skills/<name>/SKILL.md`、`.claude/skills/<name>/SKILL.md` | 支持 `.well-known/skills/index.json` 远程发现 |
| Claude Code | `.claude/skills/<name>/SKILL.md`                                                                       | 使用标准 `SKILL.md` 结构，必要时用户自行复制  |

默认兼容层：

- 必须使用 `SKILL.md` 文件名。
- 必须使用 YAML frontmatter + Markdown 正文。
- 必须使用标准字段：`name`、`description`。
- 可使用标准可选字段：`license`、`compatibility`、`metadata`。
- 不默认使用 Claude 专有字段，如 `when_to_use`、`argument-hint`、`arguments`、`context`、`agent`、`hooks`、`model`、`effort`。
- 不默认使用 `allowed-tools`，因为不同客户端的解释和权限模型不同。
- 可以使用 `` !`command` `` 动态上下文注入。

如果用户明确要求某个平台专用能力，必须在 `compatibility` 或正文中说明依赖和降级行为。

---

## 4. Skill 命名规则

`skill-name` 必须满足：

```text
^[a-z0-9]+(-[a-z0-9]+)*$
```

要求：

- 只能使用小写字母、数字、单个连字符。
- 不能以连字符开头或结尾。
- 不能出现连续连字符。
- 长度不超过 64 个字符。
- `SKILL.md` frontmatter 中的 `name` 必须与父目录名完全一致。
- 名称应描述能力，不要使用泛泛名称，如 `helper`、`tools`、`workflow`。

示例：

```text
english-to-chinese
git-commit
weekly-report
api-review
database-migration
```

---

## 5. Frontmatter 规范

最小模板：

```markdown
---
name: skill-name
description: 一句话说明这个 skill 做什么，以及用户在什么场景下应该触发它。
license: MIT
---
```

允许字段：

| 字段            | 必填 | 规则                                   |
| --------------- | ---- | -------------------------------------- |
| `name`          | 是   | 必须匹配父目录名，符合命名正则         |
| `description`   | 是   | 1-1024 字符，必须包含能力和触发场景    |
| `license`       | 否   | 建议使用 `MIT`，除非用户指定其他许可证 |
| `compatibility` | 否   | 说明平台、系统依赖、脚本依赖或专有能力 |
| `metadata`      | 否   | string 到 string 的键值对              |

`description` 写法要求：

- 写具体任务和触发条件。
- 包含用户可能说出的关键词。
- 使用第三人称或客观描述。
- 如果 skill 只应在窄场景使用，写明 `Use ONLY when...` 或中文等价表述。
- 不要写 `Helps with code`、`Improves workflow`、`Useful tool` 这类泛泛描述。

好例子：

```yaml
description: 根据最近 7 天的 Git 提交记录生成中文周报表格。当用户要求写周报、生成周报、总结本周工作或汇报工作进度时使用。
```

坏例子：

```yaml
description: 帮助处理项目任务。
```

---

## 6. SKILL.md 正文结构

推荐结构：

````markdown
# Skill 标题

一句话说明此 skill 的目标。

## 使用场景

- 何时使用
- 何时不要使用

## 工作流程

1. 第一步
2. 第二步
3. 第三步

## 规则

- 必须遵守的约束
- 不允许的行为

## 输出格式

```markdown
[可复制的输出模板]
```

## 验证

- 如何检查结果是否正确

## 注意事项

- 具体陷阱或边缘情况
````

正文要求：

- 保持简洁，优先小于 500 行和 5000 tokens。
- 使用明确的步骤、检查清单、模板和示例。
- 写 agent 没有 skill 时容易不知道或容易做错的内容。
- 对脆弱步骤使用精确指令；对灵活步骤说明目标和判断标准。
- 多步骤任务必须写验证循环：执行、验证、修复、再验证。
- 输出有固定格式时，必须给模板。
- 涉及风险、边缘情况、项目约定时，必须添加 `注意事项` 或 `Gotchas`。
- 不要把完整长文档直接塞进 `SKILL.md`。

---

## 7. 渐进式加载

当内容较多时，必须拆分：

- `references/`：长规则、API 文档、错误码、领域资料。
- `assets/`：报告模板、配置模板、示例输入输出。
- `scripts/`：可复用脚本、解析器、验证器。

`SKILL.md` 中引用辅助文件时，必须说明何时读取：

```markdown
如果 API 返回非 200 状态码，读取 `references/api-errors.md` 并按错误码表处理。
```

不要只写：

```markdown
更多信息见 references。
```

---

## 8. 脚本规范

只有在脚本能显著降低重复错误时才添加 `scripts/`。

适合打包脚本的场景：

- agent 每次都会重复实现同一段解析或转换逻辑。
- 有明确输入输出和可验证结果。
- 操作容易出错，脚本比自然语言指令更可靠。
- 需要验证生成物格式或一致性。

脚本要求：

- 脚本必须相对 skill 目录引用。
- 在 `SKILL.md` 中说明运行命令、输入、输出和失败处理。
- 不要要求不存在的全局依赖；如有依赖，写入 `compatibility` 或正文。
- 不要在脚本中执行破坏性操作，除非用户明确要求且 workflow 包含确认和验证步骤。

---

## 9. index.json 规范

每新增、删除或移动 skill，都必须更新：

```text
.well-known/skills/index.json
```

格式：

```json
{
  "skills": [
    {
      "name": "skill-name",
      "files": ["SKILL.md"]
    }
  ]
}
```

如果 skill 包含辅助文件，必须全部列入 `files`：

```json
{
  "name": "skill-name",
  "files": [
    "SKILL.md",
    "references/guide.md",
    "assets/template.md",
    "scripts/validate.sh"
  ]
}
```

规则：

- `files` 必须包含 `SKILL.md`。
- `name` 必须等于 skill 目录名。
- `files` 中路径必须相对于 skill 目录。
- 不要列出不存在的文件。
- 不要遗漏 `references/`、`assets/`、`scripts/` 下会被 skill 引用的文件。

---

## 10. 创建 Skill 的工作流

当用户要求创建新 skill 时，按以下顺序执行：

1. 明确 skill 的目标、触发场景、输入、输出和边界。
2. 从用户资料、现有文档、真实任务记录中提取领域特定知识。
3. 选择符合正则的 `skill-name`。
4. 创建 `.well-known/skills/<skill-name>/SKILL.md`。
5. 必要时创建 `references/`、`assets/`、`scripts/`。
6. 更新 `.well-known/skills/index.json`。
7. 检查 frontmatter、命名、索引、引用路径和兼容性。
8. 如果仓库有校验工具，运行校验；否则至少人工检查本文件的清单。

当用户要求修改已有 skill 时：

1. 先阅读现有 `SKILL.md` 和相关辅助文件。
2. 保持原有兼容层，不要无故引入平台专有字段。
3. 最小化修改范围。
4. 如果新增或删除辅助文件，同步更新 `index.json`。
5. 检查描述是否仍然准确触发。

---

## 11. 质量检查清单

提交前必须逐项检查：

- skill 目录名符合 `^[a-z0-9]+(-[a-z0-9]+)*$`。
- `SKILL.md` 文件名大小写正确。
- frontmatter 使用合法 YAML。
- `name` 与父目录名完全一致。
- `description` 具体说明能力和触发场景。
- 没有默认加入 Claude-only 或 OpenCode-only 字段。
- 如果使用 `` !`command` `` 动态注入，已说明手动调用场景和不支持客户端的替代步骤。
- 正文是可执行工作流，而不是泛泛建议。
- 正文没有解释通用常识。
- 长内容已拆到 `references/`、`assets/` 或 `scripts/`。
- 引用的辅助文件真实存在。
- `index.json` 包含新 skill。
- `index.json` 的 `files` 包含 `SKILL.md` 和所有被引用的辅助文件。
- 输出格式有模板，复杂任务有验证步骤。
- 注意事项记录了容易出错的项目特定事实。

---

## 12. 标准模板

新建 skill 时优先从此模板开始：

````markdown
---
name: skill-name
description: 说明这个 skill 做什么，以及用户在什么场景下应该触发它。
license: MIT
---

# Skill 标题

一句话说明此 skill 的目标。

## 使用场景

- 当用户要求 ... 时使用。
- 不要在 ... 时使用。

## 工作流程

1. 收集必要上下文。
2. 按项目约定执行核心步骤。
3. 验证结果是否符合要求。
4. 向用户输出最终结果或变更摘要。

## 规则

- 必须 ...
- 不要 ...

## 输出格式

```markdown
[在这里放固定输出模板]
```

## 验证

- 检查 ...
- 如果验证失败，修复后重新验证。

## 注意事项

- 记录 agent 容易做错的具体事实。
````

`index.json` 条目模板：

```json
{
  "name": "skill-name",
  "files": ["SKILL.md"]
}
```

---

## 13. 平台扩展使用规则

默认不要使用平台扩展。确需使用时遵循以下规则。

Claude Code 扩展字段：

- 只有用户明确要求 Claude 专用功能时才添加。
- 添加后必须在 `compatibility` 中说明，例如 `Requires Claude Code for arguments/frontmatter extensions`。
- 不要让核心 workflow 依赖 Claude-only 字段；尽量提供普通客户端也能读懂的正文说明。

动态上下文注入：

- 本仓库允许在适合手动触发的 skill 中使用 `` !`command` ``。
- OpenCode 仅在斜杠命令调用时展开该语法；如果已通过配置禁止 AI 自动执行 skills，可以将动态注入作为 OpenCode 的推荐使用方式。
- Claude Code 支持类似动态上下文注入，但具体行为以 Claude Code 文档为准。
- 不要让 `` !`command` `` 成为唯一可理解的指令。即使占位符未展开，agent 也应知道需要收集哪些上下文。

allowed-tools：

- 默认不要写。
- 只有用户要求某客户端的预授权工具能力时才写。
- 写入后必须说明兼容性影响。

---

## 14. 禁止事项

- 不要创建没有明确触发场景的 skill。
- 不要把一个 skill 写成通用助手人格或泛化开发指南。
- 不要复制整篇外部文档到 `SKILL.md`。
- 不要默认加入平台专有字段来追求“功能更全”。
- 不要引用不存在的脚本、模板或参考文件。
- 不要在未更新 `index.json` 的情况下新增或删除 skill 文件。
- 不要为了兼容性创建多份内容不同的 `SKILL.md`。
- 不要在没有用户明确要求时执行 git commit、push 或发布操作。
