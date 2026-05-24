> ## 文档索引
>
> 获取完整文档索引：[https://agentskills.io/llms.txt](https://agentskills.io/llms.txt)
>
> 在深入探索之前，使用此文件发现所有可用页面。

## 从真实专业知识出发

创建 skill 时的一个常见陷阱是让 LLM 在不提供领域特定上下文的情况下生成 skill——仅依赖 LLM 的通用训练知识。结果往往是模糊、泛化的流程（"妥善处理错误"、"遵循认证最佳实践"），而非使 skill 真正有价值的特定 API 模式、边缘情况和项目约定。有效的 skill 建立在真实专业知识之上。关键在于将领域特定的上下文注入创建过程。

### 从实际任务中提取

在与 agent 的对话中完成一个真实任务，沿途提供上下文、纠正和偏好。然后将可复用的模式提取为 skill。注意以下几点：

- **有效的步骤**——导致成功的操作序列
- **你做出的纠正**——你引导 agent 方法的地方（例如，"使用库 X 而不是 Y"、"检查边缘情况 Z"）
- **输入/输出格式**——数据输入和输出时的样子
- **你提供的上下文**——agent 本来不知道的项目特定事实、约定或约束

### 从现有项目文档中综合

当你拥有大量现有知识时，可以将其输入 LLM 并要求它综合出一个 skill。从团队实际事件报告和运维手册综合出的数据管道 skill，会比从泛化的"数据工程最佳实践"文章综合出的效果更好，因为它捕获了_你的_模式、故障模式和恢复流程。关键是项目特定的材料，而非泛化的参考资料。好的来源材料包括：

- 内部文档、运维手册和风格指南
- API 规范、模式定义和配置文件
- 代码评审评论和问题跟踪器（捕获反复出现的问题和评审者的期望）
- 版本控制历史，尤其是补丁和修复（通过实际变更揭示模式）
- 真实故障案例及其解决方案

## 通过实际执行来优化

skill 的初稿通常需要优化。在真实任务中运行 skill，然后将结果——所有结果，不仅仅是失败的——反馈到创建过程中。问问：什么触发了误报？遗漏了什么？什么可以删减？即使只做一轮"执行-修订"也能显著提升质量，复杂领域往往需要多轮迭代。

更结构化的迭代方法，包括测试用例、断言和评分，请参阅[评估 skill 输出质量](https://agentskills.io/skill-creation/evaluating-skills)。

## 合理使用上下文

skill 激活后，其完整的 `SKILL.md` 正文会连同对话历史、系统上下文和其他活跃的 skills 一起加载到 agent 的上下文窗口中。你的 skill 中的每个 token 都要与该窗口中的其他内容竞争 agent 的注意力。

### 添加 agent 缺少的，省略它已知道的

专注于 agent _没有_你的 skill 就_不会知道_的内容：项目特定的约定、领域特定的流程、不明显的边缘情况，以及要使用的特定工具或 API。你不需要解释 PDF 是什么、HTTP 如何工作、或者数据库迁移是做什么的。

````
<!-- 太冗长 — agent 已经知道 PDF 是什么 -->
## 提取 PDF 文本

PDF（便携式文档格式）文件是一种常见的文件格式，包含
文本、图像和其他内容。要提取 PDF 中的文本，你需要
使用一个库。推荐使用 pdfplumber，因为它能很好地处理大多数情况。

<!-- 更好 — 直接跳到 agent 自己不知道的内容 -->
## 提取 PDF 文本

使用 pdfplumber 进行文本提取。对于扫描文档，回退到
pdf2image 配合 pytesseract。

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

对每一部分内容问自己："没有这条指令，agent 会出错吗？"如果答案是否，就删掉它。如果不确定，就测试一下。如果 agent 没有 skill 也能很好地完成整个任务，那这个 skill 可能没有增加价值。关于如何系统性地测试这一点，请参阅[评估 skill 输出质量](https://agentskills.io/skill-creation/evaluating-skills)。

### 设计连贯的单元

决定 skill 应该覆盖什么，就像决定函数应该做什么：你希望它封装一个连贯的工作单元，能与其他 skill 良好组合。范围过窄的 skill 会迫使单个任务加载多个 skill，增加开销和指令冲突的风险。范围过广的 skill 则难以精确激活。查询数据库并格式化结果的 skill 可能是一个连贯的单元，而同时覆盖数据库管理的 skill 则可能试图做太多事情。

### 追求适度详细

过于全面的 skill 可能弊大于利——agent 难以提取相关内容，可能被不适用的指令引导到无效的路径上。简洁的分步指导加上一个可运行的示例，往往胜过详尽的文档。当你发现自己在覆盖每个边缘情况时，考虑其中大多数是否由 agent 自行判断会更好。

### 用渐进式加载组织大型 skill

[规范](https://agentskills.io/specification#progressive-disclosure)建议将 `SKILL.md` 保持在 500 行和 5,000 token 以内——仅包含 agent 每次运行所需的核心指令。当 skill 确实需要更多内容时，将详细的参考资料移到 `references/` 或类似目录中的单独文件。关键是告诉 agent_何时_加载每个文件。"如果 API 返回非 200 状态码，读取 `references/api-errors.md`"比泛泛的"详见 references/"更有用。这让 agent 按需加载上下文而非一次性加载，这正是[渐进式加载](https://agentskills.io/specification#progressive-disclosure)的设计方式。

## 校准控制力度

并非 skill 的每个部分都需要相同程度的指令精确度。将指令的具体程度与任务的脆弱程度相匹配。

### 将具体程度与脆弱程度相匹配

**给予 agent 自由度**，当多种方法都有效且任务可以容忍变化时。对于灵活的指令，解释_为什么_可能比刚性指令更有效——理解指令背后目的的 agent 能做出更好的上下文相关决策。代码评审 skill 可以描述要关注什么，而无需规定确切步骤：

```
## 代码评审流程

1. 检查所有数据库查询是否存在 SQL 注入（使用参数化查询）
2. 验证每个端点上的认证检查
3. 查找并发代码路径中的竞态条件
4. 确认错误消息不会泄露内部细节
```

**使用精确指令**，当操作脆弱、一致性重要、或必须遵循特定顺序时：

````
## 数据库迁移

严格运行此序列：

```bash
python scripts/migrate.py --verify --backup
```

不要修改命令或添加额外的标志。
````

大多数 skill 是混合的。独立校准每个部分。

### 提供默认选项，而非菜单

当多个工具或方法都可以使用时，选择一个默认值并简要提及其他选项，而不是将它们作为等同的选项呈现。

````
<!-- 选项太多 -->
你可以使用 pypdf、pdfplumber、PyMuPDF 或 pdf2image...

<!-- 清晰的默认值加备选方案 -->
使用 pdfplumber 进行文本提取：

```python
import pdfplumber
```

对于需要 OCR 的扫描 PDF，改用 pdf2image 配合 pytesseract。
````

### 偏好过程而非声明

skill 应该教 agent_如何处理_一类问题，而不是针对特定实例_产出什么_。比较：

```
<!-- 特定答案 — 仅对这个确切任务有用 -->
将 `orders` 表与 `customers` 表通过 `customer_id` 连接，
筛选 `region = 'EMEA'`，并对 `amount` 列求和。

<!-- 可复用方法 — 适用于任何分析查询 -->
1. 从 `references/schema.yaml` 读取模式以找到相关表
2. 使用 `_id` 外键约定连接表
3. 将用户请求中的筛选条件应用为 WHERE 子句
4. 按需聚合数值列并格式化为 markdown 表格
```

这并不意味着 skill 不能包含具体细节——输出格式模板（参见[输出格式模板](https://agentskills.io/skill-creation/best-practices#templates-for-output-format)）、诸如"绝不输出 PII"的约束、以及特定工具的指令都是有价值的。关键在于_方法_应该可泛化，即使个别细节是具体的。

## 有效指令的模式

以下是构建 skill 内容的可复用技巧。并非每个 skill 都需要所有技巧——选择适合你任务的那些。

### 陷阱（Gotchas）部分

许多 skill 中最高价值的内容是一系列陷阱——违反合理假设的环境特定事实。这些不是泛化的建议（"妥善处理错误"），而是对 agent 在未被告知时会犯的错误的具体纠正：

```
## 陷阱

- `users` 表使用软删除。查询必须包含
  `WHERE deleted_at IS NULL`，否则结果将包含已停用的账户。
- 用户 ID 在数据库中是 `user_id`，在认证服务中是 `uid`，
  在计费 API 中是 `accountId`。三者指的是同一个值。
- `/health` 端点只要 Web 服务器在运行就返回 200，
  即使数据库连接已断开。使用 `/ready` 检查完整的
  服务健康状态。
```

将陷阱保留在 `SKILL.md` 中，agent 会在遇到情况之前读取它们。如果你告诉 agent 何时加载，单独的参考文件也可以，但对于不明显的问题，agent 可能无法识别触发条件。

### 输出格式模板

当你需要 agent 以特定格式产出时，提供一个模板。这比用文字描述格式更可靠，因为 agent 擅长匹配具体结构。短模板可以直接放在 `SKILL.md` 中；对于较长模板，或仅在特定情况下需要的模板，存储在 `assets/` 中并从 `SKILL.md` 引用，这样只在需要时才加载。

````
## 报告结构

使用此模板，根据具体分析需要调整各部分：

```markdown
# [分析标题]

## 摘要
[关键发现的单段概述]

## 关键发现
- 发现 1 及支持数据
- 发现 2 及支持数据

## 建议
1. 具体的可执行建议
2. 具体的可执行建议
```
````

### 多步骤工作流的清单

显式清单帮助 agent 跟踪进度并避免跳过步骤，特别是当步骤之间有依赖关系或验证关卡时。

```
## 表单处理工作流

进度：
- [ ] 步骤 1：分析表单（运行 `scripts/analyze_form.py`）
- [ ] 步骤 2：创建字段映射（编辑 `fields.json`）
- [ ] 步骤 3：验证映射（运行 `scripts/validate_fields.py`）
- [ ] 步骤 4：填写表单（运行 `scripts/fill_form.py`）
- [ ] 步骤 5：验证输出（运行 `scripts/verify_output.py`）
```

### 验证循环

指示 agent 在继续之前验证自己的工作。模式是：执行工作，运行验证器（脚本、参考清单或自检），修复问题，重复直到验证通过。

```
## 编辑工作流

1. 进行编辑
2. 运行验证：`python scripts/validate.py output/`
3. 如果验证失败：
   - 查看错误消息
   - 修复问题
   - 再次运行验证
4. 仅在验证通过时继续
```

参考文档也可以充当"验证器"——指示 agent 在最终确定之前对照参考检查其工作。

### 计划-验证-执行

对于批量或破坏性操作，让 agent 先以结构化格式创建中间计划，对照事实来源验证，然后才执行。

```
## PDF 表单填写

1. 提取表单字段：`python scripts/analyze_form.py input.pdf` → `form_fields.json`
   （列出每个字段名称、类型以及是否必填）
2. 创建 `field_values.json`，将每个字段名称映射到其预期值
3. 验证：`python scripts/validate_fields.py form_fields.json field_values.json`
   （检查每个字段名称是否存在于表单中、类型是否兼容，
   以及必填字段是否未遗漏）
4. 如果验证失败，修改 `field_values.json` 并重新验证
5. 填写表单：`python scripts/fill_form.py input.pdf field_values.json output.pdf`
```

关键要素是第 3 步：一个验证脚本，根据事实来源（`form_fields.json`）检查计划（`field_values.json`）。像"字段 'signature_date' 未找到 — 可用字段：customer_name、order_total、signature_date_signed"这样的错误，为 agent 提供了足够的信息来自我纠正。

### 打包可复用脚本

在[迭代 skill](https://agentskills.io/skill-creation/evaluating-skills) 时，比较 agent 在各测试用例中的执行轨迹。如果你发现 agent 每次运行都独立地重新发明相同的逻辑——构建图表、解析特定格式、验证输出——那就是一个信号，说明应该编写一个经过测试的脚本并打包到 `scripts/` 中。更多关于设计和打包脚本的内容，请参阅[在 skill 中使用脚本](https://agentskills.io/skill-creation/using-scripts)。

## 下一步

当你有了一个可用的 skill 后，两篇指南可以帮助你进一步完善它：

- **[评估 skill 输出质量](https://agentskills.io/skill-creation/evaluating-skills)**——设置测试用例、评分结果并系统性地迭代。
- **[优化 skill 描述](https://agentskills.io/skill-creation/optimizing-descriptions)**——测试和改进 skill 的 `description` 字段，使其在正确的提示词上触发。
