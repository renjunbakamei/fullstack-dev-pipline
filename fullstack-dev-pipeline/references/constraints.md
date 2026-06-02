# 全局约束

## 流程约束

- 每个阶段必须收到明确的输出文件后才能推进。
- **关键阶段（阶段 2、阶段 4、阶段 8）**：评审通过后必须暂停并等待用户人工确认，采用关键阶段交互协议（模式 A）。
- **非关键阶段**：产出后展示摘要，30 秒宽限期后自动推进，采用非关键阶段交互协议（模式 B）。
- **技术方案门禁**：阶段 7 开发开始前，调度器须验证架构设计文档齐备、`05-change-impact/change_impact_analysis.md` 存在、status 达到「改动点识别完成」且 `06-pmo/specs/` 目录存在。门禁不通过则终止开发任务。
- 任何阶段失败（评审拒绝或迭代超限），立即终止并告知用户失败原因和已有产物。

## 目录结构

所有产物保存在 `./project-doc/<feature名称>/` 下，按阶段分 10 个子文件夹：

| 目录 | 内容 |
|------|------|
| `01-prd/` | PM 产出 / 简单需求的轻量需求说明 |
| `02-cto-review/` | 需求评审产出 |
| `03-arch/` | 架构设计产出 |
| `04-cto-tech-review/` | 架构评审产出 |
| `05-change-impact/` | 改动点识别产出 |
| `06-pmo/` | 任务拆解产出（含 `specs/` 子目录） |
| `07-dev-qa/` | 开发与测试设计产出 |
| `08-code-review/` | 代码审查产出 |
| `09-qa/` | 测试与缺陷修复产出 |
| `10-qa-report/` | 质量验收与总结产出 |

## 模板约束

每个阶段生成的 md 文档必须基于 `~/.claude/templates/` 目录下的对应模板文件（`<文档名>_template.md`）创建。各 SubAgent 在生成文档前必须先读取对应模板，严格遵循模板的章节结构和格式要求。
- **Spec 模板**：阶段6 PMO 生成的每个 spec 文件必须基于 `~/.claude/templates/spec_template.md` 创建（详见 `references/spec-mode.md`）。
- **改动点识别模板**：阶段5 Developer 生成的 `change_impact_analysis.md` 必须基于 `~/.claude/templates/change_impact_analysis_template.md` 创建。

## 改动范围约束

**原则：** Developer 只能修改 `change_impact_analysis.md` 中列出的文件。

**门禁检查：**
- Code Review 时检查 git diff，确认所有改动文件都在清单中
- 如有例外改动，必须在 `change_impact_analysis.md` 中记录

**例外处理：**
- 发现 bug 需要修复 → 记录到"例外改动"章节
- 依赖关系变化 → 记录到"例外改动"章节
- 架构调整 → 需要重新走架构评审流程

## 状态追踪

必须在 `./project-doc/<feature名称>/status.json` 中维护当前 feature 的研发状态。状态节点按顺序流转，不可跳过或回退。每个阶段完成/评审通过后须立即更新 `status.json` 中的 `status` 字段。

## 进度汇报

每完成一个阶段，向用户输出进度摘要（例如：「阶段3完成，架构设计已生成，当前状态：技术方案生成完成」）。
