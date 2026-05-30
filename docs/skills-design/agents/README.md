# Agents MD Skills 设计文档

## 背景

最初的 `agents-md-writer` 是一个单一的 Skill，负责所有 AGENTS.md 相关的任务：根目录生成、子目录生成、片段拼装、维护更新、概念转规范、片段回流建议。随着需求增长，这个 Skill 承担了 7 个不同的职责，任务路由变成复杂的大 if-else，难以维护。

同时，项目中积累了大量跨项目可复用的规则（Redis 使用规范、PostgreSQL/Prisma 约定、TypeScript 风格等），需要一个独立的片段库来存储和管理这些可复用知识。

## 设计决策

### 拆分为三个独立 Skill

三个 Skill 平级独立，没有父子关系，各自在 `index.json` 中单独注册，AI 根据 description 各自匹配触发。

| Skill                     | 职责                                               | 触发场景                             |
| ------------------------- | -------------------------------------------------- | ------------------------------------ |
| `agents-md-generator`     | 从零生成 AGENTS.md，支持片段拼装和手动定制两种模式 | 用户要求初始化、生成、创建 AGENTS.md |
| `agents-md-maintainer`    | 维护已有 AGENTS.md，概念转规范，片段回流建议       | 用户要求补充、修正、更新已有规则     |
| `agents-snippets-manager` | 管理片段库本身，新建、更新、校验片段文件           | 用户要求提取规则到片段库、管理片段   |

### 冲突处理

用户通常不会仔细了解 Skill 之间的分工，只会说"帮我生成一个 AGENTS.md"。设计上通过 description 精准区分：

- `agents-md-generator`：目标是项目 AGENTS.md，从零开始（初始化、生成、创建）
- `agents-md-maintainer`：目标是已有 AGENTS.md，修改（补充、修正、更新）。description 中使用 `Use ONLY when the target AGENTS.md already exists`
- `agents-snippets-manager`：目标是片段库，不是项目 AGENTS.md。description 中使用 `Use ONLY when the target is the snippets library`

generator 内部自动判断使用片段拼装还是手动定制模式，用户不需要关心这个决策。

### 片段库设计

片段库位于 `.well-known/agents-snippets/`，不注册在 `index.json` 中，不参与远程 Skill 加载。

目录结构按"服务为主轴"组织：

```
agents-snippets/
├── AGENTS.md                    # 片段库规范文档
├── infrastructure/              # 基础设施类
│   ├── redis/
│   │   ├── _base.md             # 语言无关通用规则
│   │   ├── ts-ioredis.md        # TypeScript + ioredis
│   │   └── go-goredis.md        # Go + go-redis
│   ├── postgresql/
│   │   ├── _base.md
│   │   └── ts-prisma.md
│   └── email/
│       ├── _base.md
│       └── ts-resend.md
├── architecture/                # 架构模式类
│   └── nextjs-thin-route.md
└── style/                       # 代码风格类
    ├── typescript.md
    ├── naming.md
    └── comment.md
```

关键约定：

- `infrastructure/` 按服务名建子目录，每个服务目录内有 `_base.md`（语言无关）和 `{语言缩写}-{库名}.md`（语言+包特定）
- `architecture/` 按模式名建子目录或单文件
- `style/` 按语言或跨语言关注点分文件
- 每个片段自包含，使用 `{变量}` 占位符标注项目特有路径/命令
- 变量在片段末尾的"变量"表格中列出

### 前置判断（generator 内部）

generator 在执行前先判断目标 AGENTS.md 的性质：

- 适合片段拼装的信号：项目刚初始化、用户说"标准化"、项目使用了片段库中已有的基础设施
- 不适合片段拼装的信号：内容高度绑定业务逻辑、项目有独特架构、用户说"按实际情况写"
- 不确定时问用户一个简短问题

这个判断是 generator 内部逻辑，不暴露给用户和其他 Skill。

## 目录结构

### 最终文件结构

```
.well-known/
├── skills/
│   ├── index.json
│   ├── agents-md-generator/           # Skill 1(生成AGENTS.md)
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── root-agents.md         # 根目录手动定制
│   │       ├── nested-agents.md       # 子目录生成 + 父链去重
│   │       └── snippets-assembler.md  # 片段匹配 + 拼装
│   ├── agents-md-maintainer/          # Skill 2(维护已有的AGENTS.md)
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── maintain-agents.md     # 维护 + 概念转规范 + 片段回流
│   └── agents-snippets-manager/       # Skill 3(管理片段库)
│       ├── SKILL.md
│       └── references/
│           └── snippet-writing.md     # 片段写作规范
└── agents-snippets/                   # 片段库（数据，不是 Skill）
    ├── AGENTS.md                      # 片段库规范
    ├── infrastructure/
    ├── architecture/
    └── style/
```

### 旧的 agents-md-writer 处理

删除 `.well-known/skills/agents-md-writer/` 目录，其职责被三个新 Skill 完全替代。

## 演进路线

### 阶段 1：Skill + 文件系统（当前）

- 片段库用 Git 仓库中的 `.md` 文件存储
- Skill 直接读文件系统做匹配和拼装
- 适合片段数量 < 50

### 阶段 2：结构化索引

- 片段库增长到 50+ 时，引入 JSON 索引替代长文档索引
- Skill 读 JSON 索引做精确匹配
- 存储仍然是 Git 文件系统

### 阶段 3：MCP Server

- `agents-snippets-manager` 的核心逻辑迁移到 MCP Server
- MCP Server 暴露 `search_snippets`、`get_snippet`、`validate_snippet` 等工具
- `agents-md-generator` 改为调用 MCP 工具，不再直接读文件
- `agents-md-maintainer` 基本不变
- 存储可以迁移到数据库（SQLite / PostgreSQL），但 Git 文件系统也可以继续用

迁移时只需：

1. 把 snippets-manager 的查询逻辑搬到 MCP Server
2. generator 里"读文件"改为"调 MCP 工具"
3. 片段文件本身零改动

### 存储演进

| 阶段           | 片段数量 | 存储方式                         |
| -------------- | -------- | -------------------------------- |
| 现在           | < 50     | Git 仓库 + 文件系统              |
| 中期           | 50-500   | Git + JSON 索引 或 SQLite        |
| 长期（多人）   | 500+     | PostgreSQL                       |
| 长期（多区域） | 500+     | PostgreSQL + MinIO（大文件模板） |

## 设计讨论记录

### TTL 规则修正

初始版本的 `redis/_base.md` 中 TTL 规则过于绝对："所有 key 必须设置 TTL，不允许永久 key"。

实际上 Redis 有两种角色：

- **缓存层**：必须设 TTL，否则内存只增不减
- **主数据存储**（排行榜、计数器、延迟队列）：可以不设 TTL

已修正为区分两种场景，作为主数据存储时允许不设 TTL，但要求用前缀区分用途。

### 为什么选择"服务为主轴"而非"语言为主轴"

片段库的主访问路径是"项目用了什么服务"，不是"项目用什么语言"。用户思考"我需要 Redis 片段"比"我需要 Go 语言片段"更自然。因此 infrastructure/ 下按服务建子目录，语言+包在文件名中区分。

### 为什么片段库放在 `.well-known/agents-snippets/` 而非独立仓库

前期为了方便迭代和修改，放在同一个仓库。`agents-snippets/` 不注册在 `index.json` 中，不影响远程 Skill 加载。将来如果需要独立，可以直接迁移为独立仓库，Skill 只需要改路径引用。
