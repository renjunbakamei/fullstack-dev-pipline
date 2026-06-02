# 代码审查报告：{{PROJECT_NAME}}

| 审查维度 | 评分（1-10） | 问题描述 | 改进建议 | 严重程度 |
|----------|-------------|----------|----------|----------|
| 设计一致性 | {{DESIGN_SCORE}} | {{DESIGN_ISSUES}} | {{DESIGN_SUGGESTIONS}} | {{DESIGN_SEVERITY}} |
| 代码质量 | {{QUALITY_SCORE}} | {{QUALITY_ISSUES}} | {{QUALITY_SUGGESTIONS}} | {{QUALITY_SEVERITY}} |
| 安全性 | {{SEC_SCORE}} | {{SEC_ISSUES}} | {{SEC_SUGGESTIONS}} | {{SEC_SEVERITY}} |
| 性能 | {{PERF_SCORE}} | {{PERF_ISSUES}} | {{PERF_SUGGESTIONS}} | {{PERF_SEVERITY}} |
| 测试覆盖 | {{TEST_SCORE}} | {{TEST_ISSUES}} | {{TEST_SUGGESTIONS}} | {{TEST_SEVERITY}} |
| 规范遵循 | {{STYLE_SCORE}} | {{STYLE_ISSUES}} | {{STYLE_SUGGESTIONS}} | {{STYLE_SEVERITY}} |

## 审查摘要
- 审查范围：{{REVIEW_SCOPE}}
- 代码行数：{{LOC}}
- 文件数量：{{FILE_COUNT}}

## 严重问题清单
| 编号 | 维度 | 问题描述 | 涉及文件 | 建议修复方案 |
|------|------|----------|----------|-------------|
{% for issue in critical_issues %}
| {{issue.id}} | {{issue.dimension}} | {{issue.description}} | {{issue.file}} | {{issue.fix}} |
{% endfor %}

## 总体结论
- 是否通过：{{FINAL_VERDICT}}（通过/需修改/拒绝）
- 必须修复项：{{MUST_FIX_LIST}}
- 建议优化项：{{OPTIONAL_IMPROVEMENTS}}

## 下一步
{{NEXT_ACTIONS}}
