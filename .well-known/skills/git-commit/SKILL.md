---
name: git-commit
description: 使用中文约定式提交信息的 Git 提交与推送
---

# Git 提交

按照项目的提交信息约定，暂存、提交并推送变更。

## 提交信息格式

```
<type>(<scope>): <subject>
```

- **subject** 必须使用中文，用通俗易懂的日常语言从用户角度描述业务变更
- 不要使用笼统的信息 — 要具体说明改了什么以及为什么改

### 允许的类型

| 类型     | 含义                                   |
| -------- | -------------------------------------- |
| feat     | 新功能                                 |
| fix      | 问题修复                               |
| docs     | 仅文档变更                             |
| style    | 格式调整（空格、分号等），无逻辑变更   |
| refactor | 代码重构，既不是修复也不是新增功能     |
| perf     | 性能优化                               |
| test     | 添加或更新测试                         |
| chore    | 杂项                                   |
| build    | 构建系统变更（webpack、gulp、npm 等）  |
| ci       | CI/CD 配置变更（chart、Dockerfile 等） |
| revert   | 回滚之前的提交                         |

### 规则

1. 每个提交只能包含**同一类型**的变更 — 不要混合类型
2. 每个提交最多处理 **3 个**问题
3. 如果变更涉及多种类型，请拆分为多个提交

## 工作流程

1. 运行 `git status --short` 和 `git diff` 了解所有变更
2. 按类型分组变更；如果存在多种类型，规划多个提交
3. 对每组变更：
   - `git add` 相关文件
   - 按上述格式撰写提交信息
4. 推送所有提交

## 上下文

### GIT DIFF

!`git diff`

### GIT DIFF --cached

!`git diff --cached`

### GIT STATUS

!`git status --short`
