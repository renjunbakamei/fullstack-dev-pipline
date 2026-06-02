# 接口设计说明书：{{PROJECT_NAME}}

## 接口总览
| 接口路径 | 方法 | 功能 | 认证方式 | 请求示例 | 响应示例 |
|----------|------|------|----------|----------|----------|
{{API_OVERVIEW_ROWS}}

## 详细定义

### {{API_METHOD}} {{API_PATH}}
- **描述**：{{API_DESCRIPTION}}
- **请求头**：
```json
{{REQUEST_HEADERS}}
```
- **请求体**:
```json
{{REQUEST_BODY_EXAMPLE}}
```
- **响应体（成功）**：
```json
{{SUCCESS_RESPONSE_EXAMPLE}}
```
- **错误码**：
| 错误码 | 含义 | 处理建议 |
|--------|------|----------|
{{ERROR_CODE_ROWS}}

## 公共错误码
| 错误码 | 含义 |
|--------|------|

