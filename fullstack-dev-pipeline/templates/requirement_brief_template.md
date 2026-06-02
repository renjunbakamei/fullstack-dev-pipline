# 需求简述：{{PROJECT_NAME}}

## 1. 功能概述
{{FUNCTION_OVERVIEW}}

## 2. 核心功能点
{% for feature in core_features %}
- **{{feature.name}}**：{{feature.description}}
{% endfor %}

## 3. 技术约束
| 约束项 | 要求 |
|--------|------|
| 技术栈偏好 | {{TECH_STACK}} |
| 性能要求 | {{PERF_REQ}} |
| 安全要求 | {{SEC_REQ}} |
| 兼容性要求 | {{COMPAT_REQ}} |

## 4. 简化声明
- 复杂度评估：**简单**
- 简化原因：{{SIMPLIFY_REASON}}
- 已跳过的阶段：阶段1（需求分析）、阶段2（需求评审）
- 进入阶段：阶段3（架构设计）
