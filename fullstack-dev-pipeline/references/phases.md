# 阶段详细说明

## 阶段 0：接收需求与复杂度判断

调度器自身执行（不委派 SubAgent）。

**步骤：**
1. 确认需求名称（作为 `<feature名称>` 用于目录和文件命名）。
2. **Wiki知识库检索（可选）**：
   - 检查是否存在 `wiki/` 目录（wikillm知识库）
   - 如果存在，使用 `wikillm:query` skill 检索与需求相关的内容：
     - 相关功能的实现方式
     - 类似需求的历史决策
     - 项目架构和技术栈信息
     - 编码规范和最佳实践
   - 将检索结果保存到 `00-context/wiki_context.md`
   - **重要原则**：
     - Wiki检索结果仅作为**参考上下文**，不是强制约束
     - 后续SubAgent应优先参考wiki内容以提升效率、减少token消耗
     - 但如果wiki信息不足或过时，SubAgent必须自己查看代码库
     - 检索失败或无相关内容不影响流程继续
3. 从以下 6 个维度评估需求复杂度：

| 维度 | 简单指标 | 复杂指标 |
|------|---------|---------|
| 功能范围 | 单一功能点或单模块改动 | 多模块/多功能联动 |
| 数据模型 | 无新增表/模型 | 新增数据表或修改核心模型 |
| 外部集成 | 无外部 API/服务调用 | 涉及第三方服务集成 |
| 非功能需求 | 无性能/安全特殊要求 | 有性能指标、安全合规要求 |
| 业务逻辑 | UI 调整或配置变更 | 状态机、复杂计算、权限控制 |
| 用户角色 | 单一角色 | 多角色权限区分 |

**判定逻辑：** 任一项命中「复杂指标」→ 复杂需求；全部命中「简单指标」→ 简单需求；信息不足以判定时默认按复杂需求处理。

4. 若信息不足以评估，向用户提出 3-5 个澄清问题（聚焦上述 6 个维度）。
5. 向用户展示复杂度评估结论（模式 C 交互协议），等待用户确认。

**输出：**
- **Wiki检索结果**（如果执行了检索）：`00-context/wiki_context.md`，包含相关功能、架构信息、最佳实践等参考内容。
- 若为**简单需求**：生成 `requirement_brief.md`（轻量需求说明，基于 `../templates/requirement_brief_template.md`），放入 `01-prd/`。
- 若为**复杂需求**：不做额外产出，进入阶段 1。

**目录初始化：** 确认 feature 名称后立即执行：
```bash
mkdir -p ./project-doc/<feature名称>/{00-context,01-prd,02-cto-review,03-arch,04-cto-tech-review,05-change-impact,06-pmo,07-dev-qa,08-code-review,09-qa,10-qa-report}
```

**状态初始化：** 创建 `./project-doc/<feature名称>/status.json`，写入初始状态（参见 `references/status-spec.md`）。

**分支处理（根据复杂度）：**
- **简单需求**：跳过阶段 1、阶段 2，直接进入阶段 3（架构设计）。更新 `status.json` 中 `status` 为 `requirement_confirmed`。
- **复杂需求**：继续阶段 1。更新 `status.json` 中 `status` 为 `requirement_confirmed`。

**Token消耗记录：**
- 阶段0结束时，调度器必须从API返回的usage信息中提取并记录本阶段的token消耗到 `status.json` 的 `token_usage.stages.stage0`
- 填写字段：`tokens`、`input_tokens`、`output_tokens`、`cache_read_input_tokens`、`cache_creation_input_tokens`、`model`、`recorded_at`
- 更新 `token_usage.breakdown.scheduler_tokens`（阶段0无SubAgent调用，`subagent_tokens` 为0）
- 同时更新 `token_usage.total_tokens` 累计值

---

## 阶段 1：需求分析（PM）— 仅复杂需求

- **触发条件**：仅当 `status.json` 中 `complexity = "complex"` 时执行。简单需求跳过此阶段和阶段 2。
- **动作**：如果用户输入为一个飞书链接，则调用 skill:feishu-docs 提取文档；如果是一段话则委派 `product-manager` SubAgent。
- **输入**：用户原始需求 + 补充信息，或飞书文档内容。
- **输出要求**：必须生成 `prd.md`、`requirements_breakdown.md`、`priority_matrix.md`，放入 `01-prd/` 文件夹。每个文档基于 `../templates/` 下的对应模板创建。
- **验收**：检查文件是否存在且内容完整。若不合格，要求 PM 重新输出。
- **交互模式**：非关键阶段交互协议（模式 B）— 展示摘要，30 秒宽限期后自动推进。
- **状态更新**：更新 `status.json` 中 `status` 为 `prd生成完成`。

## 阶段 2：需求评审（CTO）— 仅复杂需求

- **触发条件**：仅当 `status.json` 中 `complexity = "complex"` 时执行。
- **动作**：委派 `general-purpose` SubAgent（注入 CTO 角色身份）对阶段 1 的输出进行评审。
- **输入**：PM 产出的所有文档（`01-prd/` 下全部文件）。
- **输出要求**：生成 `review_report_demand.md`，基于 `../templates/review_report_demand_template.md`，并给出结论（通过/需修改/拒绝）。放入 `02-cto-review/`。
- **分支处理**：
  - 若「通过」→ 采用**关键阶段交互协议（模式 A）**，暂停并等待用户确认。用户确认后更新 `status.json` 中 `status` 为 `prd评审通过`，进入阶段 3。
  - 若「需修改」→ 将修改意见回传给 PM，重复阶段 1（最多迭代 3 次）。
  - 若「拒绝」→ 终止流程，向用户解释原因并建议调整需求。

## 阶段 3：架构设计（Architect）

- **触发条件**：始终执行。
  - 复杂需求：在阶段 2 评审通过后进入。
  - 简单需求：从阶段 0 直接进入。
- **动作**：委派 `architect` SubAgent。
- **输入**：需求文档（复杂需求为 PRD，简单需求为 `requirement_brief.md`）。
- **输出要求（根据复杂度）**：
  - **简单需求**：生成 `tech_design.md`（1份合并文档，包含系统架构、模块设计、数据库设计、API设计），放入 `03-arch/`。
  - **复杂需求**：生成 `system_architecture.md`、`module_design.md`、`database_design.md`、`api_design.md`（4份独立文档），放入 `03-arch/`。每个文档基于 `../templates/` 下的对应模板创建。
- **验收**：确认所有设计文档覆盖需求中的核心功能。
- **交互模式**：非关键阶段交互协议（模式 B）— 展示摘要，30 秒宽限期后自动推进。
- **状态更新**：验收通过后更新 `status.json` 中 `status` 为 `技术方案生成完成`。
- **重要**：此阶段的产出是后续所有编码工作的前置条件。阶段 4 评审通过后，技术方案将被锁定，不得在无评审的情况下直接修改。

## 阶段 4：架构评审（CTO）

- **触发条件**：始终执行。
- **动作**：委派 `general-purpose` SubAgent（注入 CTO 角色身份）对架构方案进行评审。
- **输入**：`03-arch/` 下全部设计文档。
- **输出要求（根据复杂度）**：
  - **简单需求**：生成 `review_report_tech.md`（1份），放入 `04-cto-tech-review/`。
  - **复杂需求**：生成 `review_report_tech.md`（基于 `../templates/review_report_tech_template.md`）和 `risk_assessment.md`（基于 `../templates/risk_assessment_template.md`）（2份），放入 `04-cto-tech-review/`。
- **分支处理**：
  - 若「通过」→ 采用**关键阶段交互协议（模式 A）**，暂停并等待用户确认。用户确认后更新 `status.json` 中 `status` 为 `技术方案评审通过`，进入阶段 5。
  - 若「需修改」→ 将修改意见回传给架构师，重复阶段 3（最多迭代 3 次）。
  - 若「拒绝」→ 终止流程，向用户汇报风险原因。

## 阶段 5：改动点识别（Developer）

- **触发条件**：始终执行。
- **动作**：委派 `developer` SubAgent。
- **输入**：`03-arch/` 下的架构设计文档 + 需求文档 + 现有代码库（如适用）。
- **输出要求**：生成 `change_impact_analysis.md`（基于 `../templates/change_impact_analysis_template.md`），放入 `05-change-impact/`，包含：
  1. **改动文件清单**（新增/修改/删除文件列表，含改动原因、关联需求、预估行数）
  2. **改动边界声明**（允许改动的范围、禁止改动的范围、例外情况处理）
  3. **依赖影响分析**（上游依赖、下游影响、风险点）
  4. **改动统计**（文件数量、代码行数、涉及模块）
- **关键原则**：
  - **手术式改动** — 只改必须改的，不做无关重构
  - **明确边界** — 清晰定义允许和禁止的改动范围
  - **可追溯性** — 每个改动都能追溯到具体需求
  - **保守估算** — 宁可多列不可漏列
- **验收标准**：
  - 所有改动文件都有明确的改动原因
  - 每个改动都能追溯到具体需求
  - 改动边界清晰，禁止范围明确
  - 依赖影响分析完整
  - 改动统计数据准确
- **交互模式**：非关键阶段交互协议（模式 B）— 展示改动统计摘要，30 秒宽限期后自动推进。
- **状态更新**：验收通过后更新 `status.json` 中 `status` 为 `改动点识别完成`。
- **价值**：
  - 明确改动边界，防止范围蔓延
  - 提高 PMO 排期的准确性
  - 为 Developer 提供清晰的改动清单
  - 为 Code Review 提供改动范围基准

## 阶段 6：任务拆解（PMO）— Spec 模式

- **触发条件**：始终执行。
- **动作**：委派 `project-manager` SubAgent。
- **输入**：需求文档 + `03-arch/` 下全部架构设计文档 + `05-change-impact/change_impact_analysis.md`。
- **输出要求**（详见 `references/spec-mode.md`）：
  - `task_breakdown.md` — 任务总览 + 依赖关系图 + spec 索引表。
  - `schedule.md` — 排期与里程碑。
  - `06-pmo/specs/spec-XXX-<name>.md` — 每个任务一份独立 spec，按依赖拓扑排序编号。每份 spec 必须包含：需求描述、上下文、接口契约、涉及文件、验收条件、QA 测试要点。
- **验收**：spec 数量覆盖架构文档中所有模块；每个 spec 的接口契约与 `api_design.md` 一致；依赖关系无循环；spec 中的涉及文件与 `change_impact_analysis.md` 一致。
- **交互模式**：非关键阶段交互协议（模式 B）— 展示摘要（spec 总数、并行批次、预估总工时），30 秒宽限期后自动推进。
- **状态更新**：验收通过后更新 `status.json` 中 `status` 为 `任务拆解完成`。

## 阶段 7：开发与测试设计（并行）— Spec 驱动

- **触发条件**：始终执行。
- **前置门禁（强制执行）**：委派 developer 之前，调度器必须依次执行以下两个门禁：

### 门禁1：技术方案完整性检查

```
技术方案门禁检查（根据复杂度）:

简单需求:
  [ ] 03-arch/ 目录存在
  [ ] tech_design.md 存在且内容完整
  [ ] 05-change-impact/ 目录存在
  [ ] change_impact_analysis.md 存在且内容完整
  [ ] status.json → status = "改动点识别完成"
  [ ] 06-pmo/specs/ 目录存在且至少包含 1 个 spec 文件

复杂需求:
  [ ] 03-arch/ 目录存在
  [ ] system_architecture.md 存在且内容完整
  [ ] module_design.md 存在且内容完整
  [ ] database_design.md 存在且内容完整
  [ ] api_design.md 存在且内容完整
  [ ] 05-change-impact/ 目录存在
  [ ] change_impact_analysis.md 存在且内容完整
  [ ] status.json → status = "改动点识别完成"
  [ ] 06-pmo/specs/ 目录存在且至少包含 1 个 spec 文件
```

任何一项失败则：
```
⛔ 技术方案门禁未通过：{失败原因}
开发任务已暂停。请先确保前置阶段已完成。
当前状态：{status}
```

### 门禁2：人工确认门禁（强制执行）

技术方案门禁通过后，调度器必须采用**关键阶段交互协议（模式 A）**，暂停并等待用户确认是否开始开发。

协议格式：
```
============================================================
[开发启动确认] 即将开始开发与测试设计
============================================================

📋 开发准备就绪：
  - 技术方案：已评审通过并锁定
  - 改动范围：已识别 {X} 个文件（新增 {Y} / 修改 {Z}）
  - 任务拆解：已生成 {N} 个 spec，预估工时 {H} 小时
  - 并行批次：{M} 批，关键路径 {K} 个任务

📄 关键文档：
  - 技术方案：{03-arch/ 文件列表}
  - 改动分析：05-change-impact/change_impact_analysis.md
  - 任务清单：06-pmo/task_breakdown.md
  - Spec 目录：06-pmo/specs/

⚠️  开发启动后将按 spec 驱动编码，技术方案将被锁定。

------------------------------------------------------------
请选择操作：
  ✅ 输入「确认」或「开始开发」— 启动开发阶段
  📋 输入「查看 <文件名>」— 查看具体文档内容
  ✏️ 输入「修改：<具体意见>」— 回到对应阶段修订
  ❌ 输入「取消」— 终止流程
------------------------------------------------------------
```

**调度器行为：**
- 展示此提示后**阻塞等待**用户输入。
- 收到「确认」或「开始开发」→ 更新 `status.json` 中 `status` 为 `开发启动已确认`，继续执行开发任务。
- 收到「查看 xxx」→ 读取并展示对应文件内容，展示后重新显示确认提示。
- 收到「修改：xxx」→ 根据反馈内容判断需要回到哪个阶段（架构设计/改动点识别/任务拆解），将意见传给对应 SubAgent 修订，修订完成后重新执行门禁1和门禁2。
- 收到「取消」→ 记录到 status_history，终止流程。

**两个门禁都通过后方可继续开发。**

- **动作**：同时委派 `developer` 和 `general-purpose`（注入 QA 角色身份）SubAgent。
  - **开发任务（Spec 驱动）**：调度器读取 `task_breakdown.md` 获取依赖图，developer 按依赖顺序逐个领取 spec（流程详见 `references/spec-mode.md`）。每份 spec 是 self-contained 的任务包，developer 读取后即可开始编码，无需来回翻阅架构文档。源代码放入 `07-dev-qa/src/`。
  - **改动范围约束**：Developer 只能修改 `05-change-impact/change_impact_analysis.md` 中列出的文件。如需修改未列出的文件，必须更新该文档的"例外改动"章节并说明原因。
  - **测试设计**：QA 同步读取所有 spec 文件，基于每份 spec 的「验收条件」和「QA 测试要点」编写 `test_cases.md`（基于 `../templates/test_cases_template.md`）。放入 `07-dev-qa/`。
- **编码质量要求（Karpathy Guidelines）**：developer 在编码时必须遵循以下原则：
  1. **简单优先** — 最少代码解决问题，不添加未被要求的功能、抽象或配置
  2. **手术式修改** — 只修改必须修改的代码，匹配现有风格，不重构无关代码
  3. **明确假设** — 如有不确定的地方，在代码注释或自测记录中明确说明
  4. **可验证目标** — 每个 spec 的验收条件必须可验证（测试、编译、运行）
- **并行策略**：无依赖关系的 spec 可分配给多个 developer 并行执行。
- **Developer 自测要求（强制执行）**：每个 spec 完成编码后，developer 必须执行自测验证，通过后才能将 spec 状态标记为 `done`：
  ```
  自测验证清单:
    [ ] 代码编译通过（无语法错误、类型错误）
    [ ] 如有外部依赖导致无法完整编译，至少完成：
        - 语法检查（如 eslint、tsc --noEmit、javac 等）
        - 类型检查
        - 导入路径验证
    [ ] 执行基本冒烟测试（手动或自动化）
    [ ] 验证核心功能符合 spec 的「验收条件」
    [ ] 记录自测结果到 spec 文件的「自测记录」章节
  ```
  如果自测发现问题，developer 必须修复后重新自测，直到通过。
- **进度追踪**：调度器通过扫描 spec 文件中 `状态` 字段追踪进度，每完成一个 spec 输出进度（如「spec-003 done, 3/8」）。
- **交互模式**：非关键阶段交互协议（模式 B）。
- **状态更新**：
  - 全部 spec 状态变为 `done` 且自测验证通过后，更新 `status.json` 中 `status` 为 `开发完成`。

## 阶段 8：代码审查（Code Review）

- **触发条件**：始终执行。
- **动作**：委派 `general-purpose` SubAgent（注入代码审查工程师角色身份）。
- **输入**：`07-dev-qa/` 下全部源码 + `03-arch/` 设计文档 + `05-change-impact/change_impact_analysis.md` + `07-dev-qa/test_cases.md`。
- **输出要求**：生成 `code_review_report.md`，基于 `../templates/code_review_report_template.md`。放入 `08-code-review/`。
- **审查维度**：
  1. **编译验证（强制执行）** — 必须先执行编译验证，通过后才能继续其他审查：
     ```
     编译验证清单:
       [ ] 执行完整编译（如 npm run build、mvn compile、go build 等）
       [ ] 记录所有编译错误和警告
       [ ] 运行已有的单元测试（如 npm test、mvn test 等）
       [ ] 检查类型错误（如 tsc、mypy 等）
       [ ] 验证导入路径和依赖完整性
     ```
     如果编译失败或存在严重警告，必须在审查报告中标注为「不通过」，并详细列出错误信息。
  2. **改动范围检查（新增）** — 检查是否有超出 `change_impact_analysis.md` 范围的改动：
     - 对比 git diff 与改动文件清单
     - 检查是否有未列出的文件被修改
     - 验证例外改动是否已记录并说明原因
  3. **代码质量（Karpathy Guidelines）** — 检查是否遵循编码最佳实践：
     - 是否存在过度设计（未被要求的抽象、配置、功能）
     - 是否做了不必要的重构或修改无关代码
     - 是否有过度复杂的实现（能用50行解决却写了200行）
     - 是否有不可能发生场景的错误处理
     - 修改是否都能追溯到需求（手术式修改原则）
  4. **设计一致性** — 代码是否遵循架构设计文档
  5. **命名与可读性** — 命名规范、函数复杂度、重复代码
  6. **安全性** — 输入校验、权限控制、数据保护
  7. **性能** — 潜在瓶颈（N+1 查询、未缓存、大循环）
  8. **测试覆盖** — 测试用例与代码覆盖的对应关系
  9. **规范遵循** — 编码约定、框架最佳实践
- **审查报告必须包含**：
  - 编译验证结果（通过/失败，包含错误详情）
  - 测试运行结果（通过率、失败用例）
  - 改动范围检查结果（是否有超出范围的改动）
  - 各维度审查结论
  - 代码质量评分（简洁性、必要性）
- **分支处理**：
  - 若「通过」（编译通过 + 改动范围合规 + 其他维度无严重问题）→ 采用**关键阶段交互协议（模式 A）**，暂停并等待用户确认。用户确认后更新 `status.json` 中 `status` 为 `code_review通过`，进入阶段 9。
  - 若「需修改」（编译失败、超出改动范围或存在问题）→ 将修改意见回传给 developer 修订代码，修订后重新审查（最多迭代 3 次，超限则终止并建议人工介入）。
  - 若「拒绝」→ 终止流程，向用户汇报拒绝原因。

## 阶段 9：测试与缺陷修复（迭代）

- **触发条件**：始终执行。
- **动作**：`general-purpose` SubAgent（注入 QA 角色身份）执行测试（基于 `test_cases.md`），输出 `bug_list.md`（基于 `../templates/bug_list_template.md`）。每个 bug 自动指派给对应的 developer。
- **循环**：修复 → 回归 → 直至所有致命/严重 bug 关闭。
- **出口条件**：测试用例通过率 ≥ 95%（可配置），且无致命级 bug。
- **交互模式**：非关键阶段交互协议（模式 B）。
- **状态更新**：出口条件达成后更新 `status.json` 中 `status` 为 `qa测试完成`。

## 阶段 10：质量验收与项目总结

- **触发条件**：始终执行。
- **动作**：委派 `general-purpose` SubAgent（注入 QA 角色身份）输出 `quality_report.md`（基于 `../templates/quality_report_template.md`）。放入 `10-qa-report/`。
- **输出要求（根据复杂度）**：
  - **简单需求**：仅生成 `quality_report.md`（1份）。
  - **复杂需求**：生成 `quality_report.md` + `project_summary.md`（2份，后者由 `project-manager` 输出）。
- **Token消耗统计（调度器生成）**：
  - 调度器读取 `status.json` 中的 `token_usage` 数据
  - 生成 `token_usage_report.md`，包含：
    - 总token消耗
    - 各阶段token消耗明细表（tokens / input / output / cache / subagent 分列）
    - 各阶段占比分析（含饼图数据）
    - 调度器 vs SubAgent 消耗对比
    - 缓存命中率分析（cache_read / total_input）
    - 简单需求 vs 复杂需求的对比（如有历史数据）
    - 优化建议（识别token消耗异常高的阶段，分析缓存优化空间）
  - 放入 `10-qa-report/`
- **交互模式**：非关键阶段交互协议（模式 B）。
- **最终结论**：向用户汇报"✅ 可上线"或"⚠️ 有条件上线（附风险）"，并展示token消耗统计摘要。
- **状态更新**：验收完成，更新 `status.json` 中 `status` 为 `project_complete`。
