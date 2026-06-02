# 项目质量验收报告：{{PROJECT_NAME}}

报告日期：{{DATE}}
验收人：{{APPROVER}}

## 验收范围
{{SCOPE}}

## 质量度量
| 指标 | 目标值 | 实际值 | 是否达标 |
|------|--------|--------|----------|
| 测试通过率 | {{TARGET_PASS_RATE}}% | {{ACTUAL_PASS_RATE}}% | {{PASS_RATE_RESULT}} |
| 缺陷密度 | {{TARGET_DEFECT_DENSITY}} | {{ACTUAL_DEFECT_DENSITY}} | {{DEFECT_DENSITY_RESULT}} |
| 代码覆盖率 | {{TARGET_COVERAGE}}% | {{ACTUAL_COVERAGE}}% | {{COVERAGE_RESULT}} |
| 性能达标率 | {{TARGET_PERFORMANCE}}% | {{ACTUAL_PERFORMANCE}}% | {{PERFORMANCE_RESULT}} |

## 遗留问题
| 问题ID | 描述 | 影响 | 是否可接受 |
|--------|------|------|------------|
{{OPEN_ISSUES_ROWS}}

## 最终结论
- 质量等级：{{QUALITY_LEVEL}}（优/良/合格/不合格）
- 是否可上线：{{RELEASE_READY}}（是/否/有条件）
- 有条件上线条件：{{CONDITIONS_FOR_RELEASE}}

## 审批
- 技术总监：{{CTO_SIGN}} 日期：{{CTO_DATE}}
- 产品经理：{{PM_SIGN}} 日期：{{PM_DATE}}
