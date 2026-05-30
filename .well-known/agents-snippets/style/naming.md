# 命名规范

## 通用

- 使用描述性名称；避免在紧凑循环之外使用单字母名称。
- 新文件名必须使用 `kebab-case`（中划线分隔的小写），除非已有约定要求其他格式。

## 按类型

| 类型 | 命名风格 | 示例 |
|------|----------|------|
| 类、类型、React 组件 | `PascalCase` | `UserService`、`SpecEditorPage` |
| 函数、变量、对象键 | `camelCase` | `getUserById`、`isActive` |
| 文件名 | `kebab-case` | `create-spec-form.tsx` |
| 常量 | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| 环境变量 | `UPPER_SNAKE_CASE` | `DATABASE_URL` |
