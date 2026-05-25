# Skills

可复用的 Agent Skills 仓库，通过 GitHub Pages 发布，兼容 OpenCode、Claude Code、Cursor、VS Code Copilot、Gemini CLI 等支持 [Agent Skills 开放标准](https://agentskills.io) 的客户端。

## 当前 Skills

| Skill | 说明 | 触发场景 |
|-------|------|----------|
| `english-to-chinese` | 英文文本翻译为简体中文 | "翻译成中文"、"英译中"、"中文版本" |
| `git-commit` | 使用中文约定式提交信息完成 Git 提交 | "提交代码"、"生成提交信息"、"commit"、"push" |
| `weekly-report` | 根据 Git 提交记录生成中文周报表格 | "写周报"、"生成周报"、"总结本周工作" |

## 使用方式

### OpenCode（远程发现）

OpenCode 支持从远程 URL 自动发现和缓存 skills。在 `opencode.json` 中配置：

```json
{
  "skills": {
    "remote": "https://518luck.github.io/skills/.well-known/skills/"
  }
}
```

配置后 OpenCode 会自动下载 `index.json` 并缓存 skills 到本地。更新 skills 后清除缓存：

```bash
rm -rf ~/.cache/opencode/skills/
```

### Claude Code / 其他客户端

将需要的 skill 目录复制到项目本地：

```bash
# Claude Code
cp -r .well-known/skills/<skill-name> .claude/skills/

# 通用
cp -r .well-known/skills/<skill-name> .agents/skills/
```

## 仓库结构

```
.well-known/skills/
├── index.json                    # Skill 注册表（远程发现入口）
├── english-to-chinese/
│   └── SKILL.md
├── git-commit/
│   └── SKILL.md
└── weekly-report/
    └── SKILL.md
```

每个 skill 包含：
- `SKILL.md` — 核心指令文件（YAML frontmatter + Markdown 正文）
- `references/` — 可选，长参考资料
- `assets/` — 可选，模板和静态资源
- `scripts/` — 可选，可执行脚本或验证器

## 创建新 Skill

1. 在 `.well-known/skills/` 下创建以 skill 名称命名的目录（小写字母、数字、连字符）。
2. 编写 `SKILL.md`，包含 YAML frontmatter（`name`、`description`）和 Markdown 正文。
3. 更新 `.well-known/skills/index.json`，添加新 skill 条目。
4. 推送到 `main` 分支，GitHub Pages 自动发布。

详细的 skill 制作规范见 [AGENTS.md](./AGENTS.md)。

## 技术参考

- [AGENTS.md](./AGENTS.md) — Skill 制作执行规范
- [docs/面向AI的Skill制作操作手册.md](./docs/面向AI的Skill制作操作手册.md) — Agent Skills 标准详解与客户端兼容性
- [docs/agent-skills-overview.md](./docs/agent-skills-overview.md) — Skill 编写最佳实践

## 许可证

MIT
