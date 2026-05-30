# 注释规范

## 中文注释

- 所有新增或重写的函数、组件、类、导出常量和非平凡逻辑块上方，必须添加一行简短中文单行注释。
- 注释说明这段代码的业务目的或设计意图，不要复述语法。
- import、简单类型声明、简单常量、明显的 JSX 结构不强制添加注释。

## 示例

```typescript
// 加载规格详情并处理不存在状态
const loadSpec = async (): Promise<void> => {
  // ...
};

// 渲染规格编辑页面主体
function SpecEditorPage(): React.JSX.Element {
  return <SpecEditor />;
}
```

## 禁止事项

- 禁止注释掉代码来"保留"旧实现；使用版本控制系统。
- 禁止添加复述代码逻辑的注释（如 `// 设置 name 为空字符串`）。
