# status.json 规范

## 文件位置
`./project-doc/<feature名称>/status.json`

## 状态流转顺序（单向不可逆）

```
init → requirement_confirmed → [prd生成完成 → prd评审通过] →
技术方案生成完成 → 技术方案评审通过 → 改动点识别完成 → 任务拆解完成 →
开发启动已确认 → 开发完成 → code_review通过 → qa测试完成 → project_complete
```

注：`[...]` 内状态仅复杂需求（`complexity = "complex"`）经过。简单需求从 `requirement_confirmed` 直接进入 `技术方案生成完成`。

## 初始化（阶段 0 确认 feature 名称后立即创建）

```json
{
  "feature_name": "<feature名称>",
  "complexity": "simple|complex",
  "status": "init",
  "status_history": [],
  "iteration_tracker": {
    "stage1_iterations": 0,
    "stage2_iterations": 0,
    "stage3_iterations": 0,
    "stage4_iterations": 0,
    "stage5_iterations": 0,
    "stage8_iterations": 0
  },
  "token_usage": {
    "total_tokens": 0,
    "breakdown": {
      "scheduler_tokens": 0,
      "subagent_tokens": 0
    },
    "stages": {
      "stage0": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "需求接收与复杂度判断", "recorded_at": ""},
      "stage1": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "需求分析(PM)", "recorded_at": ""},
      "stage2": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "需求评审(CTO)", "recorded_at": ""},
      "stage3": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "架构设计(Architect)", "recorded_at": ""},
      "stage4": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "架构评审(CTO)", "recorded_at": ""},
      "stage5": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "改动点识别(Developer)", "recorded_at": ""},
      "stage6": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "任务拆解(PMO)", "recorded_at": ""},
      "stage7": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "开发与测试设计", "recorded_at": ""},
      "stage8": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "代码审查", "recorded_at": ""},
      "stage9": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "测试与缺陷修复", "recorded_at": ""},
      "stage10": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "质量验收与项目总结", "recorded_at": ""}
    }
  },
  "feedback_log": [],
  "created_at": "<ISO 时间戳>",
  "updated_at": "<ISO 时间戳>"
}
```

## 更新规则

- `complexity` 字段在阶段 0 用户确认后填入（`"simple"` 或 `"complex"`），后续不可变更。
- `iteration_tracker` 记录各可迭代阶段的修订次数，每次回传修改意见时对应字段 +1。
- `token_usage` 记录每个阶段的token消耗，支持中断后恢复查看：
  - `breakdown` 按角色汇总：
    - `scheduler_tokens`：调度器自身（阶段0、读取文件、生成摘要等）消耗的token累计值
    - `subagent_tokens`：所有SubAgent调用消耗的token累计值
  - 每个 `stages.stageN` 包含以下字段：
    - `tokens`：该阶段总token消耗（scheduler + subagent）
    - `input_tokens`：输入token数（来自API返回的 `usage.input_tokens`）
    - `output_tokens`：输出token数（来自API返回的 `usage.output_tokens`）
    - `cache_read_input_tokens`：缓存命中节省的token数
    - `cache_creation_input_tokens`：缓存写入消耗的token数
    - `subagent_tokens`：该阶段SubAgent调用的token消耗（0表示无SubAgent调用）
    - `model`：使用的模型标识（如 `sonnet`、`opus`、`haiku`）
    - `description`：阶段描述
    - `recorded_at`：记录时间戳（ISO格式）
  - 调度器在每个阶段结束时，从API返回的usage信息中提取上述值填入
  - 同时更新 `total_tokens` 和 `breakdown.scheduler_tokens` / `breakdown.subagent_tokens`
  - 原则：每个阶段结束时**立即更新**，确保流程中断后仍可查看已完成阶段的消耗
- 每次状态变更时，将旧状态追加到 `status_history` 数组（含时间戳和来源阶段），再更新 `status` 和 `updated_at`。
- `feedback_log` 记录用户在各阶段提出的修改意见。每次收到反馈时追加一条记录，包含阶段、时间、意见摘要和是否已处理。

## 更新示例

```json
{
  "feature_name": "medical-appointment",
  "complexity": "complex",
  "status": "prd评审通过",
  "status_history": [
    {"status": "init", "updated_at": "2026-05-28T10:00:00+08:00", "stage": "阶段0", "trigger": "初始化"},
    {"status": "requirement_confirmed", "updated_at": "2026-05-28T10:05:00+08:00", "stage": "阶段0", "trigger": "复杂度评估-复杂"},
    {"status": "prd生成完成", "updated_at": "2026-05-28T10:35:00+08:00", "stage": "阶段1", "trigger": "PM产出完成"},
    {"status": "prd评审通过", "updated_at": "2026-05-28T10:50:00+08:00", "stage": "阶段2", "trigger": "CTO评审通过,用户确认"}
  ],
  "iteration_tracker": {
    "stage1_iterations": 0,
    "stage2_iterations": 1,
    "stage3_iterations": 0,
    "stage4_iterations": 0,
    "stage5_iterations": 0,
    "stage8_iterations": 0
  },
  "token_usage": {
    "total_tokens": 45230,
    "breakdown": {
      "scheduler_tokens": 3500,
      "subagent_tokens": 41730
    },
    "stages": {
      "stage0": {"tokens": 3500, "input_tokens": 12000, "output_tokens": 800, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "sonnet", "description": "需求接收与复杂度判断", "recorded_at": "2026-05-28T10:05:00+08:00"},
      "stage1": {"tokens": 18200, "input_tokens": 45000, "output_tokens": 3200, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 18200, "model": "sonnet", "description": "需求分析(PM)", "recorded_at": "2026-05-28T10:35:00+08:00"},
      "stage2": {"tokens": 23530, "input_tokens": 58000, "output_tokens": 4100, "cache_read_input_tokens": 12000, "cache_creation_input_tokens": 0, "subagent_tokens": 23530, "model": "opus", "description": "需求评审(CTO)", "recorded_at": "2026-05-28T10:50:00+08:00"},
      "stage3": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "架构设计(Architect)", "recorded_at": ""},
      "stage4": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "架构评审(CTO)", "recorded_at": ""},
      "stage5": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "改动点识别(Developer)", "recorded_at": ""},
      "stage6": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "任务拆解(PMO)", "recorded_at": ""},
      "stage7": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "开发与测试设计", "recorded_at": ""},
      "stage8": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "代码审查", "recorded_at": ""},
      "stage9": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "测试与缺陷修复", "recorded_at": ""},
      "stage10": {"tokens": 0, "input_tokens": 0, "output_tokens": 0, "cache_read_input_tokens": 0, "cache_creation_input_tokens": 0, "subagent_tokens": 0, "model": "", "description": "质量验收与项目总结", "recorded_at": ""}
    }
  },
  "feedback_log": [
    {"stage": "阶段2", "time": "2026-05-28T10:45:00+08:00", "feedback": "需补充非功能需求章节", "resolved": true}
  ],
  "created_at": "2026-05-28T10:00:00+08:00",
  "updated_at": "2026-05-28T10:50:00+08:00"
}
```
