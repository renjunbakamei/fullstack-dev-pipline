# fullstack-pipeline-optimizer Skill 设计文档

## 概述

这是一个专门用于分析和优化 `fullstack-dev-pipeline` 执行效率的skill。它通过读取历史项目数据，识别token消耗瓶颈、质量问题和流程效率问题，并生成可操作的优化建议。

**核心特点：**
- ✅ 只读分析，不修改任何文件
- ✅ 基于数据驱动的优化建议
- ✅ 17条分析规则，14个优化策略
- ✅ 自动生成详细的优化报告

## 文件结构

```
pipeline-optimizer-design/
├── SKILL.md                              # Skill主文件
├── references/
│   ├── analysis-rules.md                 # 17条分析规则和阈值定义
│   ├── optimization-strategies.md        # 14个优化策略库
│   └── report-template.md                # 报告模板
└── README.md                             # 本文件
```

## 安装方法

1. 将整个 `pipeline-optimizer-design` 目录复制到 `~/.claude/skills/` 下
2. 重命名为 `fullstack-pipeline-optimizer`
3. 重启Claude Code或重新加载skills

```bash
# 在当前目录执行
cp -r pipeline-optimizer-design ~/.claude/skills/fullstack-pipeline-optimizer
```

## 使用方法

### 基本用法

```
@fullstack-pipeline-optimizer 分析 ./project-doc
```

### 指定时间范围

```
@fullstack-pipeline-optimizer 分析 ./project-doc 最近30天
```

### 只分析特定复杂度

```
@fullstack-pipeline-optimizer 分析 ./project-doc 只看复杂需求
```

## 分析维度

### 1. Token消耗分析
- 识别token消耗最高的阶段
- 对比简单需求vs复杂需求
- 发现异常高消耗项目
- 分析消耗趋势

### 2. 质量问题分析
- 统计各阶段迭代次数
- 识别最常返工的阶段
- 分析Code Review拒绝原因
- 统计测试阶段bug类型

### 3. 流程效率分析
- 计算各阶段平均耗时
- 识别流程瓶颈
- 分析并行执行效果
- 评估文档产出必要性

### 4. 文档质量分析
- 评估文档完整性
- 识别经常被修改的文档
- 分析模板适用性

## 分析规则

skill内置17条分析规则，包括：

**Token消耗类（规则1-3）：**
- 单阶段超标检测
- 总消耗超标检测
- 占比失衡检测

**质量问题类（规则4-7）：**
- 高迭代阶段检测
- 迭代趋势恶化检测
- 编译失败率检测
- 重复编译错误检测

**流程效率类（规则8-9）：**
- 阶段耗时过长检测
- 并行度不足检测

**文档质量类（规则10-12）：**
- 文档冗余检测
- 文档频繁修改检测
- 必需文档缺失检测

**趋势分析类（规则13-17）：**
- 成本上升趋势检测
- 质量下降趋势检测
- 效率提升检测
- 复杂度判定准确性检测
- 执行稳定性检测

详见 `references/analysis-rules.md`

## 优化策略

skill提供14个优化策略，分为4类：

**Token消耗优化（T1-T4）：**
- T1: 精简Spec模板
- T2: 优化SubAgent Prompt
- T3: 引入Prompt Caching
- T4: 分级文档策略强化

**质量问题优化（Q1-Q4）：**
- Q1: 强化Developer自测
- Q2: Code Review前置编译
- Q3: 优化架构评审标准
- Q4: 测试用例前置

**流程效率优化（E1-E3）：**
- E1: 增强并行执行
- E2: 简化评审流程
- E3: 自动化重复任务

**文档质量优化（D1-D3）：**
- D1: 模板标准化
- D2: 文档复用机制
- D3: 文档质量门禁

详见 `references/optimization-strategies.md`

## 输出报告

skill会生成 `pipeline_optimization_report.md`，包含：

1. **执行摘要** - 3-5个关键发现和优先行动项
2. **Token消耗分析** - 总体统计、各阶段明细、异常项目
3. **质量问题分析** - 迭代次数、编译失败、bug统计
4. **流程效率分析** - 各阶段耗时、并行效率
5. **文档质量分析** - 文档产出、修改频率
6. **触发规则汇总** - 本次触发的所有规则
7. **优化建议** - 按优先级排序的具体建议
8. **趋势分析** - 历史对比和趋势判断
9. **附录** - 项目清单和原始数据

## 优化建议优先级

**优先级1（立即实施）：**
- 预期效果高、实施难度低、风险低
- 例如：Q1强化自测、Q2前置编译

**优先级2（短期规划）：**
- 预期效果中等、实施难度中等
- 例如：T1精简spec、E2简化评审

**优先级3（长期优化）：**
- 预期效果高但实施难度高
- 例如：T3引入缓存、Q4测试前置

## 使用场景

### 场景1：定期优化
每月运行一次，分析上月所有项目，识别优化机会。

### 场景2：成本控制
当发现token消耗异常增长时，快速定位原因。

### 场景3：质量改进
当Code Review迭代频繁时，分析根本原因。

### 场景4：流程优化
当项目周期过长时，识别瓶颈环节。

## 注意事项

1. **数据要求**：需要至少3-5个已完成项目才能进行有效分析
2. **数据完整性**：确保每个项目的 `status.json` 包含完整的token_usage数据
3. **只读模式**：skill不会修改任何文件，所有建议需要手动实施
4. **建议评估**：实施前请结合实际情况评估风险和可行性

## 与fullstack-dev-pipeline的关系

```
fullstack-dev-pipeline (执行)
         ↓
    生成 status.json
         ↓
fullstack-pipeline-optimizer (分析)
         ↓
    生成优化建议
         ↓
    手动实施改进
         ↓
fullstack-dev-pipeline (改进后执行)
```

## 未来扩展

- [ ] 支持自定义分析规则
- [ ] 支持导出Excel格式报告
- [ ] 支持对比多个时间段的数据
- [ ] 支持自动应用低风险优化（需要用户确认）
- [ ] 集成机器学习预测模型

## 贡献

如果你发现新的优化机会或分析规则，欢迎：
1. 在 `references/analysis-rules.md` 中添加新规则
2. 在 `references/optimization-strategies.md` 中添加新策略
3. 更新 `SKILL.md` 中的分析流程

---

**设计完成时间**：2026-05-29
**设计者**：Claude Opus 4.7
**版本**：v1.0
