# 数据库设计文档：{{PROJECT_NAME}}

## 1. ER图
```mermaid
{{ER_DIAGRAM_MERMAID}}
```

## 2. 表结构详细定义
{{TABLE_DEFINITIONS}}

### 示例：表 `{{EXAMPLE_TABLE_NAME}}`
| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
{{EXAMPLE_TABLE_COLUMNS}}

## 3. 索引设计
```sql
{{INDEX_DESIGN_SQL}}
```

## 4. 初始化数据脚本
```sql
{{SEED_DATA_SQL}}
```

## 5. 数据迁移策略
{{MIGRATION_STRATEGY}}
