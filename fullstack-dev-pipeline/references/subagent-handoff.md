# SubAgent 委派协议

## Agent 类型选择

部分内置 agent 类型缺少 Write/Edit 权限（`cto`、`qa` 仅有只读权限），无法产出文档文件。因此：

| 阶段 | 角色 | 实际使用 Agent 类型 | 原因 |
|------|------|-------------------|------|
| 1 | PM | `product-manager` | 内置类型有 Write/Edit |
| 2 | CTO 需求评审 | **`general-purpose`** | `cto` 类型无 Write/Edit，需写评审报告 |
| 3 | Architect | `architect` | 内置类型有 Write/Edit |
| 4 | CTO 架构评审 | **`general-purpose`** | `cto` 类型无 Write/Edit，需写评审报告 |
| 5 | Developer 改动点识别 | `developer` | 内置类型有 Write/Edit |
| 6 | PMO | `project-manager` | 内置类型有 Write/Edit |
| 7 | Developer | `developer` | 内置类型有 Write/Edit |
| 7 | QA 测试设计 | **`general-purpose`** | `qa` 类型无 Write/Edit，需写测试用例 |
| 8 | Code Review | **`general-purpose`** | 无内置 `code-reviewer` 类型 |
| 9 | QA 测试执行 | **`general-purpose`** | `qa` 类型无 Write/Edit，需写 bug 报告 |
| 10 | QA + PMO 验收 | **`general-purpose`** | QA 部分需写质量报告 |

使用 `general-purpose` 时，**必须在 prompt 首行注入角色身份**（如 `你是一名资深 CTO/QA 工程师…`），确保 subagent 以正确角色视角工作。

## 核心原则：摘要做底线，文件路径做退路

调度器在阶段0执行wiki检索（如果存在 `wiki/` 目录），然后读取 wiki/架构/PRD，向各 subagent **注入精准摘要**。subagent 在 80% 场景下无需额外读取文件。遇到边界情况时，按「扩展阅读」路径自行补充。

### Wiki检索结果的使用

如果阶段0成功执行了wiki检索并生成了 `00-context/wiki_context.md`，调度器应：

1. **提取相关内容**：从wiki检索结果中提取与当前阶段相关的信息（如相关功能实现、技术决策、编码规范）
2. **注入到上下文摘要**：将提取的内容作为「上下文摘要」的一部分传递给SubAgent
3. **标注为参考性质**：明确告知SubAgent这是参考信息，不是强制约束
4. **保留扩展阅读路径**：将 `00-context/wiki_context.md` 加入「扩展阅读」列表，供SubAgent需要时查阅

**重要原则：**
- Wiki内容优先参考，可以显著提升效率、减少token消耗
- 但不强制使用，如果wiki信息不足、过时或不适用，SubAgent必须自己查看代码库
- Wiki检索失败或无相关内容不影响流程继续

## 委派 Prompt 模板

```
## 任务
<该阶段的具体任务描述>

## 评审深度（本项目复杂度：<简单/复杂>）
<从下方「评审深度对照表」中选取适用条目>
<明确列出本阶段不需要关注的事项>

## 上下文摘要（够用就不读文件）
- <从 wiki检索结果/架构/PRD 提取的、本任务直接相关的关键信息>
- <3-5 条，聚焦接口签名、数据表、约束、决策结论>
- <如有wiki检索结果，优先引用相关功能的实现方式、最佳实践>

## 扩展阅读（需要更多细节时自行 Read，非强制）
- project-doc/<feature>/00-context/wiki_context.md  — Wiki检索结果（如有），包含相关功能、架构信息、最佳实践
- project-doc/<feature>/<路径>  — <什么情况下需要读>

## 输出要求
<产出文件列表、模板路径、验收标准>
```

**关键原则：**
- 「上下文摘要」是底线——调度器确保 subagent 只看这部分就能完成核心工作。
- 「扩展阅读」是退路——只在摘要不够用时才读，避免 subagent 盲猜。
- 摘要内容从 wiki检索结果/project-doc/templates 中**提取**，不原样复制大段文字。
- Wiki检索结果（如有）应优先参考，但不强制使用——SubAgent可以根据实际情况决定是否采纳。
- 模板文件路径放在「输出要求」中，因为必须读。
- 「评审深度」是 scope 边界——告诉 subagent **什么不用做**，防止过度产出。

## 评审深度对照表

调度器根据阶段 0 的复杂度结论，将对应条目注入 subagent 的 prompt。

### PM（阶段 1）

| 维度 | 简单 | 复杂 |
|------|------|------|
| 需求描述 | 用户故事 + 验收条件，≤ 1 页 | 完整 PRD，含业务流程、角色分析 |
| 技术预设 | 禁止涉及技术方案 | 仅描述技术约束，不设计 |
| 非功能需求 | 不写 | 记录用户提出的性能/安全要求 |
| 边界场景 | 仅正常流程 | 含异常流程、边界条件 |

### CTO 需求评审（阶段 2）

| 维度 | 简单 | 复杂 |
|------|------|------|
| 评审重点 | 需求完整性、一致性 | 需求完整性、一致性、可行性、合规性 |
| 非功能检查 | 跳过 | 检查性能/安全/合规要求是否明确 |
| 容灾/降级 | 不涉及 | 如有外部依赖需评估 |
| 评审结论粒度 | 通过/需修改 | 通过/需修改/拒绝 + 风险清单 |

### Architect（阶段 3）

| 维度 | 简单 | 复杂 |
|------|------|------|
| 架构范围 | 单体架构，单部署单元 | 微服务/模块化，可独立部署 |
| 高可用 | 不设计 | 需考虑 |
| 容灾/降级 | 不设计 | 有外部依赖时需设计 |
| 缓存策略 | 不设计 | 按需设计 |
| 数据量假设 | 单表 < 10 万行 | 需给出分表/分库策略 |
| 安全设计 | 基础鉴权 | 完整权限模型 + 数据保护 |

### CTO 架构评审（阶段 4）

| 维度 | 简单 | 复杂 |
|------|------|------|
| 评审重点 | 接口正确性、数据库设计合理性 | 全面评审：架构选型、扩展性、安全性、性能 |
| 高可用/容灾 | 不检查 | 检查容灾/降级方案 |
| 缓存/队列 | 不检查 | 检查缓存策略、消息队列设计 |
| 安全审计 | 基础鉴权逻辑检查 | 完整安全审计 |
| 性能预估 | 不要求 | 要求 QPS/延迟预估 |

### QA（阶段 6/8）

| 维度 | 简单 | 复杂 |
|------|------|------|
| 测试类型 | 功能测试 | 功能 + 性能 + 安全 + 兼容性 |
| 压测 | 不做 | 需设计压测场景 |
| 异常测试 | 仅正常路径 | 含网络异常、超时、并发冲突 |
| 数据边界 | 不关注 | 需测试大数据量、空数据、脏数据 |

## 各阶段 SubAgent 委派规范

### 阶段 1：product-manager

**上下文摘要：**
- 用户原始需求的核心功能点
- 调度器在阶段 0 评估的复杂度结论及依据
- 涉及的模块/角色/外部集成（如有）

**扩展阅读：** 无（PM 的输入就是用户需求本身）

**输出要求：** 读取 `../templates/prd_template.md`、`requirements_breakdown_template.md`、`priority_matrix_template.md` 后生成对应文件。

---

### 阶段 2：CTO 需求评审

**Agent 类型：** `general-purpose`（首行注入：`你是一名资深技术总监/CTO，负责需求评审。`）

**上下文摘要：**
- PRD 核心功能点（1-2 句）
- 关键业务规则
- 涉及的外部系统/集成

**扩展阅读：**
- `project-doc/<feature>/01-prd/prd.md` — 完整需求细节
- `project-doc/<feature>/01-prd/requirements_breakdown.md` — 需求拆解

**输出要求：** 读取 `../templates/review_report_demand_template.md` 后生成评审报告。

---

### 阶段 3：architect

**上下文摘要：**
- 需求核心功能点
- 复杂度评估结论
- 非功能需求（性能指标、安全要求）如有
- 涉及的用户角色

**扩展阅读：**
- `project-doc/<feature>/01-prd/prd.md` — 完整需求文档

**输出要求：** 读取 `../templates/system_architecture_template.md`、`module_design_template.md`、`database_design_template.md`、`api_design_template.md` 后生成 4 份设计文档。

---

### 阶段 4：CTO 架构评审

**Agent 类型：** `general-purpose`（首行注入：`你是一名资深技术总监/CTO，负责架构方案评审。`）

**上下文摘要：**
- 架构方案的核心决策（技术栈、模块划分、数据库选型）
- 关键接口列表
- 已知风险点

**扩展阅读：**
- `project-doc/<feature>/03-arch/system_architecture.md` — 完整架构
- `project-doc/<feature>/03-arch/module_design.md` — 模块设计
- `project-doc/<feature>/03-arch/database_design.md` — 数据库设计
- `project-doc/<feature>/03-arch/api_design.md` — API 设计

**输出要求：** 读取 `../templates/review_report_tech_template.md` 和 `risk_assessment_template.md` 后生成评审报告。

---

### 阶段 5：Developer 改动点识别

**上下文摘要：**
- 架构设计的核心模块和接口
- 需求涉及的功能点
- 现有代码库结构（如适用）
- 技术栈约束

**扩展阅读：**
- `project-doc/<feature>/03-arch/` — 完整架构设计
- 现有代码库 — 分析改动点（如适用）

**输出要求：** 读取 `../templates/change_impact_analysis_template.md` 后生成 `change_impact_analysis.md`，包含改动文件清单、改动边界、依赖影响分析、改动统计。放入 `05-change-impact/`。

---

### 阶段 6：project-manager

**上下文摘要：**
- 模块列表及各自职责
- 模块间依赖关系（从架构文档提取）
- 改动文件清单及预估行数（从 change_impact_analysis.md 提取）
- 技术栈约束
- 预估总工时范围

**扩展阅读：**
- `project-doc/<feature>/03-arch/` — 完整架构文档（需要接口细节时）
- `project-doc/<feature>/05-change-impact/change_impact_analysis.md` — 改动点分析
- `references/spec-mode.md` — Spec 拆解规范

**输出要求：** 读取 `../templates/spec_template.md`、`task_breakdown_template.md`、`schedule_template.md` 后生成 task_breakdown + schedule + specs/。放入 `06-pmo/`。

---

### 阶段 7：developer

**上下文摘要（每次领取 spec 时注入）：**
- 当前 spec 的需求描述和验收条件
- 涉及的接口契约（入参/出参/端点）
- 涉及的数据表及关键字段
- 依赖的前置 spec 及其产出摘要
- 技术约束（如响应时间、框架限制）
- 改动范围约束（从 change_impact_analysis.md 提取）

**编码质量要求（Karpathy Guidelines）：**
- 简单优先：最少代码解决问题，不添加未被要求的功能、抽象或配置
- 手术式修改：只修改必须修改的代码，匹配现有风格，不重构无关代码
- 明确假设：如有不确定的地方，在代码注释或自测记录中明确说明
- 可验证目标：每个 spec 的验收条件必须可验证（测试、编译、运行）
- 改动范围约束：只能修改 change_impact_analysis.md 中列出的文件，如需例外必须记录

**扩展阅读：**
- `project-doc/<feature>/03-arch/api_design.md` — 完整接口定义
- `project-doc/<feature>/03-arch/database_design.md` — 完整表结构
- `project-doc/<feature>/05-change-impact/change_impact_analysis.md` — 改动范围约束
- `project-doc/<feature>/06-pmo/specs/spec-XXX.md` — 当前 spec 完整内容

**输出要求：** 按 spec 验收条件完成编码，源码放入 `07-dev-qa/src/`。完成后执行自测验证（编译+功能测试），通过后将 spec 状态更新为 `done`。

---

### 阶段 7：QA 测试设计（与 Developer 并行）

**Agent 类型：** `general-purpose`（首行注入：`你是一名资深测试工程师/QA，负责测试用例设计。`）

**上下文摘要：**
- 所有 spec 的需求摘要列表（每个 spec 的 1 句话描述 + 关键验收条件）
- 涉及的用户角色和权限
- 非功能需求（性能指标、安全要求）

**扩展阅读：**
- `project-doc/<feature>/06-pmo/specs/` — 完整 spec（需要测试细节时）
- `project-doc/<feature>/01-prd/prd.md` — 完整需求

**输出要求：** 读取 `../templates/test_cases_template.md` 后生成 `test_cases.md`。放入 `07-dev-qa/`。

---

### 阶段 8：Code Review

**Agent 类型：** `general-purpose`（首行注入：`你是一名资深代码审查工程师，负责代码质量审查。`）

**上下文摘要：**
- Spec 列表及各自的核心验收条件
- 关键技术决策（从架构文档提取）
- 改动范围约束（从 change_impact_analysis.md 提取）
- 安全/性能要求

**审查重点（包含 Karpathy Guidelines）：**
1. 编译验证：执行完整编译、运行测试、检查类型错误
2. 改动范围检查：对比 git diff 与 change_impact_analysis.md，检查是否有超出范围的改动
3. 代码质量：检查是否存在过度设计、不必要的抽象、过度复杂的实现
4. 手术式修改：检查是否修改了无关代码、是否做了不必要的重构
5. 设计一致性：代码是否遵循架构设计文档
6. 安全性、性能、测试覆盖

**扩展阅读：**
- `project-doc/<feature>/03-arch/` — 完整架构（检查设计一致性时）
- `project-doc/<feature>/05-change-impact/change_impact_analysis.md` — 改动范围基准
- `project-doc/<feature>/06-pmo/specs/` — 完整 spec（检查需求覆盖时）
- `project-doc/<feature>/07-dev-qa/test_cases.md` — 测试用例

**输出要求：** 读取 `../templates/code_review_report_template.md` 后生成审查报告。审查 `07-dev-qa/src/` 下全部源码。报告必须包含编译验证结果、改动范围检查结果和代码质量评分。放入 `08-code-review/`。

---

### 阶段 9：QA 测试执行

**Agent 类型：** `general-purpose`（首行注入：`你是一名资深测试工程师/QA，负责测试执行与缺陷报告。`）

**上下文摘要：**
- 测试用例总数及分类（功能/性能/安全）
- 已知的模块依赖关系
- 出口条件：通过率 ≥ 95%，无致命 bug

**扩展阅读：**
- `project-doc/<feature>/06-pmo/specs/` — spec 验收条件
- `project-doc/<feature>/07-dev-qa/src/` — 源码

**输出要求：** 读取 `../templates/bug_list_template.md` 后执行测试并生成 `bug_list.md`。放入 `09-qa/`。

---

### 阶段 10：QA + PMO 验收与总结

**Agent 类型：**
- QA 部分：`general-purpose`（首行注入：`你是一名资深测试工程师/QA，负责质量验收。`）
- PMO 部分：`project-manager`（内置类型有 Write/Edit）

**上下文摘要：**
- 项目初始目标和复杂度
- 各阶段关键决策摘要
- 测试结果摘要（通过率、bug 统计）

**扩展阅读：**
- `project-doc/<feature>/` — 全部阶段产物（需要详细数据时）

**输出要求：** QA 读取 `../templates/quality_report_template.md` 生成 `quality_report.md`；PMO 生成 `project_summary.md`。放入 `10-qa-report/`。

---

## 调度器执行规则

1. **一次读取，多次分发**：调度器在阶段 0 读取 wiki 和 references，后续各阶段从自己已读取的内容中提取摘要注入给 subagent。
2. **复杂度必须注入每个 subagent**：阶段 0 的复杂度结论（`status.json` 中的 `complexity` 字段）是 scope 边界，必须写入每个 subagent prompt 的「评审深度」section，按对照表选取适用条目。
3. **摘要控制在 5 条以内**：只写该 subagent 直接需要的信息，不写背景知识。
4. **扩展阅读指向具体文件**：不写「读 03-arch/ 下全部文件」，而是列出具体文件名及"什么情况下需要读"。
5. **模板文件必须放在输出要求中**：subagent 生成文档前必须读模板，这是强制步骤。
6. **developer 的上下文摘要要在每次领取 spec 时即时生成**：包含该 spec 的接口契约、数据表、依赖 spec 的产出摘要。
7. **每个阶段结束后立即更新 token 数据**：从 Agent 工具返回结果中提取 usage 信息，写入 `status.json` 对应阶段的 `token_usage.stages.stageN`，确保中断后可恢复查看。多 SubAgent 阶段（如阶段7）汇总所有调用后写入。
