# Agents Snippets — 片段库规范

本目录维护可复用的 AGENTS.md 规则片段，供 agents-md-writer skill 按需拼装到项目 AGENTS.md 中。

## 片段是什么

片段是一个自包含的 Markdown 块，描述一个特定关注点的 AI 工作规则。片段不是完整的 AGENTS.md，而是拼装的积木。

## 目录结构

```text
agents-snippets/
├── AGENTS.md                    # 本文件：片段库规范
├── infrastructure/              # 基础设施类（Redis、PG、邮件等）
│   ├── redis/
│   │   ├── _base.md             # 语言无关的 Redis 通用规则
│   │   ├── ts-ioredis.md        # TypeScript + ioredis
│   │   ├── go-goredis.md        # Go + go-redis
│   │   └── java-jedis.md        # Java + Jedis
│   ├── postgresql/
│   │   ├── _base.md
│   │   ├── ts-prisma.md
│   │   ├── go-gorm.md
│   │   └── java-mybatis.md
│   └── email/
│       ├── _base.md
│       ├── ts-resend.md
│       └── go-smtp.md
├── architecture/                # 架构模式类（FSD、薄路由层等）
│   ├── fsd/
│   │   ├── _base.md
│   │   └── react.md
│   └── nextjs-thin-route.md
└── style/                       # 代码风格类（按语言分文件）
    ├── typescript.md
    ├── go.md
    ├── naming.md
    └── comment.md
```

## 目录分类约定

| 一级目录          | 内容                       | 二级组织方式                              |
| ----------------- | -------------------------- | ----------------------------------------- |
| `infrastructure/` | 具体服务或中间件的使用规则 | 按服务名建子目录，子目录内按语言+包分文件 |
| `architecture/`   | 架构模式、目录组织规则     | 按模式名建子目录或直接单文件              |
| `style/`          | 代码风格、命名、注释       | 按语言或跨语言关注点分文件                |

## 文件命名格式

### 语言+包文件

格式：`{语言缩写}-{库名}.md`

| 语言缩写 | 含义                    |
| -------- | ----------------------- |
| `ts`     | TypeScript / JavaScript |
| `go`     | Go                      |
| `java`   | Java / Kotlin           |
| `py`     | Python                  |
| `rs`     | Rust                    |

示例：

- `ts-ioredis.md` — TypeScript 项目使用 ioredis
- `go-goredis.md` — Go 项目使用 go-redis
- `java-jedis.md` — Java 项目使用 Jedis
- `ts-prisma.md` — TypeScript 项目使用 Prisma
- `go-gorm.md` — Go 项目使用 GORM

### `_base.md` 文件

`_base.md` 存放该服务/架构的**语言无关通用规则**。拼装时必须包含。

适用场景：

- 基础设施类（redis、postgresql、email）必须有 `_base.md`
- 架构类如果有跨语言通用的规则，也可以有 `_base.md`
- 风格类通常跟语言绑定，不需要 `_base.md`

### 单文件

当某个关注点不需要按语言拆分时，直接用描述性名称：

- `nextjs-thin-route.md` — Next.js 薄路由层
- `naming.md` — 通用命名规范
- `comment.md` — 注释规范

命名规则：只使用小写字母、数字和中划线，符合 `^[a-z0-9]+(-[a-z0-9]+)*$`。

## 片段写作规则

### 自包含

每个片段必须独立可读，不假设其他片段的存在。如果规则依赖另一个服务的约定，在片段内部写明，而不是假设读者已看过另一个片段。

### 变量占位符

片段中项目特有的路径、命令、包名使用 `{变量名}` 标注。拼装时由 skill 或人替换为实际值。

变量格式：`{snake_case_name}`

常用变量：

| 变量                   | 含义                | 示例默认值                      |
| ---------------------- | ------------------- | ------------------------------- |
| `{schema_dir}`         | ORM schema 文件路径 | `prisma/schema/schema.prisma`   |
| `{generate_cmd}`       | 生成 Client 的命令  | `pnpm run prisma:generate`      |
| `{migrate_cmd}`        | 执行迁移的命令      | `pnpm run prisma:migrate`       |
| `{infrastructure_dir}` | 基础设施适配层目录  | `src/shared/lib/infrastructure` |
| `{pkg_manager}`        | 包管理器            | `pnpm`                          |

### 片段结构模板

```markdown
# {服务/关注点名称}{— 语言 + 包（可选）}

> 继承 `_base.md`（仅语言+包文件需要此行）

## 适配层

- 连接和操作统一通过 `{infrastructure_dir}/{service}/` 下的适配函数访问。
- 使用 `{包的完整导入路径}`。

## 具体约定

- 约定 1
- 约定 2

## 禁止事项

- 禁止 ...

## 变量

| 变量    | 默认值 | 说明 |
| ------- | ------ | ---- |
| `{var}` | 默认   | 说明 |

## 注意事项

- 容易出错的事实。
```

### 写什么 vs 不写什么

写：

- 该服务/架构特有的约束和约定
- 容易做错的项目特定事实
- 命令、路径、包名的精确要求
- 禁止事项和陷阱

不写：

- 通用编程概念（什么是 Redis、什么是 ORM）
- 与其他服务的交叉规则（放项目特有 AGENTS.md）
- 重复 `_base.md` 的内容（语言+包文件只写增量）
- 空泛建议（"遵循最佳实践"）

## 哪些内容适合拆成片段

适合：

- 基础设施使用规则（Redis、PG、邮件、认证、OSS 等）
- 通用架构模式（FSD、薄路由层、整洁架构等）
- 通用代码风格（TypeScript 规范、Go 规范、命名规范等）
- 通用注释和文档规范
- 错误处理模式
- 测试约定

不适合：

- 项目特有的业务逻辑约束
- 特定项目的目录结构描述
- 特定项目的命令（除非是通用工具链的标准命令）
- 团队内部的沟通流程
- 与具体部署环境绑定的配置

## 如何新增片段

1. 确定关注点属于哪个一级分类（infrastructure / architecture / style）。
2. 确定是否需要建子目录（infrastructure 类按服务建子目录，architecture 类按模式建子目录或单文件，style 类按语言分文件）。
3. 确定是否需要 `_base.md`（有跨语言通用规则则需要）。
4. 确定语言+包文件的命名（`{语言缩写}-{库名}.md`）。
5. 按片段结构模板写作。
6. 在片段中使用的变量必须在"变量"表格中列出。
