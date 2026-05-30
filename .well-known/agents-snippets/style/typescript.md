# TypeScript 代码风格

## 类型

- 所有新代码使用 TypeScript；避免使用 `any`。
- 公共 API 和导出函数优先使用显式返回类型。
- 类型专用导入使用 `import type`。
- 除非局部合理，避免非空断言（`!`）。
- 避免使用类型断言（`as`）；优先通过 zod schema 解析、类型守卫或函数签名约束获得类型安全；仅 `as const` 等惯用写法除外。
- 不可变结构优先使用 `readonly` 和 `as const`。
- 优先使用显式 `import`/`export`，而非 `*`。
- 优先使用变量解构，而非属性访问。
- 绝不使用 `@ts-ignore`、`@ts-expect-error` 抑制类型错误；修复根因。

## React 组件

- 使用函数式组件；显式声明 props 类型。
- React 组件必须使用 `function` 声明。
- 非组件函数应优先使用 `const` 箭头函数；自定义 Hook 视为非组件函数。
- 将 hooks 保持在顶层；避免条件式 hooks。

## React 事件类型

- React 表单提交事件禁止使用已弃用的 `FormEvent` / `FormEventHandler`；`onSubmit` 使用从 `"react"` 导入的 `SubmitEvent<HTMLFormElement>` 或 `SubmitEventHandler<HTMLFormElement>`。

## 控制流

- 优先使用提前返回和正向条件；避免双重否定、德摩根式判断和需要反复脑内取反的表达式。
- 显式处理错误；API 尽可能返回有类型的错误。
- 保持异步逻辑线性；尽量避免嵌套的 `try` 块。

## Lint

- 绝不使用 `eslint-disable` 抑制 lint 错误；修复根因。

## 检查命令

- 类型检查：`{typecheck_cmd}`
- Lint：`{lint_cmd}`
- 不要直接运行底层工具（如 `tsc`）；使用项目脚本。

## 变量

| 变量 | 含义 | 示例默认值 |
|------|------|-----------|
| `{typecheck_cmd}` | 类型检查命令 | `pnpm run typecheck` |
| `{lint_cmd}` | Lint 命令 | `pnpm run lint` |
