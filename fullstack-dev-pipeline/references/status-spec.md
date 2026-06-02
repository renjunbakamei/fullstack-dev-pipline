# status.json 规范

## 文件位置
`./project-doc/<feature名称>/status.json`

## 状态流转顺序（单向不可逆）

```
init → requirement_confirmed → [prd生成完成 → prd评审通过] →
技术方案生成完成 → 技术方案评审通过 → 改动点识别完成 → 任务拆解完成 →
开发完成 → 自测用例通过 → code_review通过 → qa测试完成 → project_complete
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
    "stages": {
      "stage0": {"tokens": 0, "description": "需求接收与复杂度判断"},
      "stage1": {"tokens": 0, "description": "需求分析(PM)"},
      "stage2": {"tokens": 0, "description": "需求评审(CTO)"},
      "stage3": {"tokens": 0, "description": "架构设计(Architect)"},
      "stage4": {"tokens": 0, "description": "架构评审(CTO)"},
      "stage5": {"tokens": 0, "description": "改动点识别(Developer)"},
      "stage6": {"tokens": 0, "description": "任务拆解(PMO)"},
      "stage7": {"tokens": 0, "description": "开发与测试设计"},
      "stage8": {"tokens": 0, "description": "代码审查"},
      "stage9": {"tokens": 0, "description": "测试与缺陷修复"},
      "stage10": {"tokens": 0, "description": "质量验收与项目总结"}
    }
  },
  "created_at": "<ISO 时间戳>",
  "updated_at": "<ISO 时间戳>"
}
```

## 更新规则

- `complexity` 字段在阶段 0 用户确认后填入（`"simple"` 或 `"complex"`），后续不可变更。
- `iteration_tracker` 记录各可迭代阶段的修订次数，每次回传修改意见时对应字段 +1。
- `token_usage` 记录每个阶段的token消耗：
  - 调度器在每个阶段结束时，记录该阶段消耗的token数到对应的 `stages.stageN.tokens`
  - 同时更新 `total_tokens` 为所有阶段的累计值
  - Token数据来源：SubAgent返回的 `total_tokens` 字段，或调度器自身的token消耗
- 每次状态变更时，将旧状态追加到 `status_history` 数组（含时间戳和来源阶段），再更新 `status` 和 `updated_at`。

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
    "stage7_iterations": 0
  },
  "token_usage": {
    "total_tokens": 45230,
    "stages": {
      "stage0": {"tokens": 3500, "description": "需求接收与复杂度判断"},
      "stage1": {"tokens": 18200, "description": "需求分析(PM)"},
      "stage2": {"tokens": 23530, "description": "需求评审(CTO)"},
      "stage3": {"tokens": 0, "description": "架构设计(Architect)"},
      "stage4": {"tokens": 0, "description": "架构评审(CTO)"},
      "stage5": {"tokens": 0, "description": "任务拆解(PMO)"},
      "stage6": {"tokens": 0, "description": "开发与测试设计"},
      "stage7": {"tokens": 0, "description": "代码审查"},
      "stage8": {"tokens": 0, "description": "测试与缺陷修复"},
      "stage9": {"tokens": 0, "description": "质量验收与项目总结"}
    }
  },
  "created_at": "2026-05-28T10:00:00+08:00",
  "updated_at": "2026-05-28T10:50:00+08:00"
}
```
