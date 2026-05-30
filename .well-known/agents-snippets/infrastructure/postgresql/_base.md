# PostgreSQL / ORM 通用规则

## 数据库操作流程

1. 在 `{schema_dir}` 中定义或修改 Model。
2. 执行 `{generate_cmd}` 生成 Client 代码。
3. 如需同步数据库结构，执行 `{migrate_cmd}`。

## Migration

- 每个 migration 必须有明确的描述性名称。
- migration 必须可回滚；写 down migration 时要验证能正确还原。
- 不要手动修改已执行的 migration；新建 migration 修正。
- 生产环境 migration 执行前必须备份。

## Schema 设计

- 每个表必须有主键。
- 使用索引优化高频查询；不要过度索引。
- 时间字段统一使用 UTC。
- 软删除优先于物理删除，除非有明确的数据保留策略。

## 禁止事项

- 禁止手动修改自动生成的 ORM Client 目录下的任何文件。
- 禁止跳过 schema 直接修改数据库结构。
- 禁止在业务代码中拼接 SQL 字符串；使用 ORM 的 query builder 或参数化查询。
- 禁止在生产环境执行未经验证的 migration。

## 变量

| 变量 | 含义 |
|------|------|
| `{schema_dir}` | ORM schema 文件路径 |
| `{generate_cmd}` | 生成 Client 的命令 |
| `{migrate_cmd}` | 执行迁移的命令 |
