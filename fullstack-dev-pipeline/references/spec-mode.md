# Spec 模式

## 概述

阶段6 PMO 不再产出一份扁平的 `task_breakdown.md`，而是将每个任务拆成独立的 **spec 文件**。每份 spec 是 developer 完成该任务所需的**全部信息**——自包含、无外部依赖阅读。

## Spec 文件模板

每份 spec 基于 `~/.claude/templates/spec_template.md` 创建，包含以下结构：

```markdown
# Spec-XXX: <任务名称>

## 元信息
- **spec_id**: spec-XXX
- **负责人**: developer
- **预估工时**: Xh
- **依赖**: [spec-XXX, spec-YYY] 或 none
- **状态**: pending

## 需求描述
<简要描述这个任务要交付什么功能，1-3 句话>

## 上下文
<从架构文档中提取的、本任务涉及的关键设计决策>
- 涉及模块：<module_name>
- 数据表：<table_name>
- 关键约束：<如：接口响应 < 200ms>

## 接口契约
### 输入
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|

### 输出
| 字段 | 类型 | 说明 |
|------|------|------|

### API 端点（如适用）
`METHOD /api/v1/xxx`

## 涉及文件
- **新增**: `src/xxx/yyy.ts`
- **修改**: `src/xxx/zzz.ts`

## 验收条件
- [ ] 条件1
- [ ] 条件2
- [ ] 条件3

## QA 测试要点
<QA 关注的边界场景和测试方向>
```

## Spec 工作流

### 阶段6：PMO 生成 Spec

PMO 委派后执行：

1. 读取 `03-arch/` 下全部架构文档，理解模块边界和接口契约。
2. 读取 `05-change-impact/change_impact_analysis.md`，了解改动范围和文件清单。
3. 读取 `~/.claude/templates/spec_template.md`。
4. 按模块拆解任务，生成独立的 spec 文件，放入 `06-pmo/specs/`：
   ```
   06-pmo/
   ├── task_breakdown.md       # 任务总览 + 依赖关系图
   ├── schedule.md             # 排期与里程碑
   └── specs/
       ├── spec-001-<name>.md
       ├── spec-002-<name>.md
       └── ...
   ```
5. 每个 spec 的 ID 按依赖拓扑排序：无依赖的排前面，同一批次的 spec 可并行开发。
6. 每个 spec 的「涉及文件」必须与 `change_impact_analysis.md` 中的改动文件清单一致。
7. `task_breakdown.md` 中维护全局依赖图和 spec 索引表。

### 阶段7：Developer 按 Spec 开发

调度器委派 developer 时：

1. 读取 `06-pmo/task_breakdown.md` 获取 spec 列表和依赖关系。
2. 读取 `05-change-impact/change_impact_analysis.md` 获取改动范围约束。
3. 按依赖顺序领取 spec（优先领取 `依赖 = none` 或依赖项已完成的 spec）。
4. 领取 spec 后：
   - 读取该 spec 文件，获取完整上下文。
   - 将 spec 状态更新为 `in_progress`。
   - 按 spec 的「接口契约」和「涉及文件」指南实施编码。
   - **改动范围约束**：只能修改 `change_impact_analysis.md` 中列出的文件。如需修改未列出的文件，必须更新该文档的"例外改动"章节并说明原因。
   - **编码完成后必须执行自测验证**（详见下方「自测验证流程」）。
   - 自测通过后将 spec 状态更新为 `done`，在 spec 文件中记录自测结果，在 commit message 中引用 spec ID。
5. 重复步骤 3-4，直到全部 spec 完成。

### 自测验证流程（强制执行）

每个 spec 编码完成后，developer 必须按以下步骤执行自测，通过后才能标记为 `done`：

**步骤1：编译验证**
```
[ ] 执行完整编译（如 npm run build、mvn compile、go build 等）
[ ] 如有外部依赖导致无法完整编译，至少完成：
    - 语法检查（如 eslint、tsc --noEmit、javac 等）
    - 类型检查
    - 导入路径验证
[ ] 记录所有编译错误和警告
```

**步骤2：功能验证**
```
[ ] 执行基本冒烟测试（手动或自动化）
[ ] 验证核心功能符合 spec 的「验收条件」
[ ] 测试边界场景和异常处理
```

**步骤3：记录自测结果**

在 spec 文件末尾添加「自测记录」章节：
```markdown
## 自测记录
- **编译结果**: 通过 / 部分通过（说明原因）
- **功能验证**: 已验证的验收条件列表
- **发现问题**: 无 / 已修复的问题列表
- **自测时间**: YYYY-MM-DD HH:mm
```

**步骤4：问题处理**

如果自测发现问题：
- 修复问题
- 重新执行自测
- 更新自测记录
- 直到通过后才能标记 spec 为 `done`

### QA 并行

QA 在阶段7启动时同步读取所有 spec 文件，基于每份 spec 的「QA 测试要点」和「验收条件」编写 `test_cases.md`。Developer 完成任意 spec 后，QA 即可对该 spec 对应模块执行测试。

### 进度追踪

调度器通过扫描 `06-pmo/specs/` 下 spec 文件的 `状态` 字段来追踪进度，替代原 `milestone_tracker.md` 的手动更新：

```
进度 = done 的 spec 数 / 总 spec 数
```

每完成一个 spec，调度器输出：「spec-003 完成 (3/8)，当前进度 37.5%」。
