## /api/groups 鉴权与用法

### 鉴权方式
- 系统使用**静态管理密钥**进行鉴权（环境变量：`AUTH_KEY`）。
- 通过以下任意一种方式在请求中携带密钥即可通过鉴权：
  - `Authorization: Bearer <AUTH_KEY>`
  - `X-Api-Key: <AUTH_KEY>`
  - `X-Goog-Api-Key: <AUTH_KEY>`
  - 查询参数：`?key=<AUTH_KEY>`

后台实现参考：
- 受保护的 API 前缀：`/api`（代码：`internal/router/router.go`）
- 受保护路由组使用中间件：`middleware.Auth`（代码：`internal/middleware/middleware.go`）
- 中间件会从上述 4 种位置提取密钥并与配置的 `AUTH_KEY` 比较

### 端点列表（均需鉴权）
- GET `/api/groups`：获取分组列表（含完整字段）
- GET `/api/groups/list`：获取精简分组列表（仅 `id/name/display_name`）
- POST `/api/groups`：创建分组
- PUT `/api/groups/:id`：更新分组
- DELETE `/api/groups/:id`：删除分组
- GET `/api/groups/:id/stats`：获取分组统计

### 响应格式
- 成功：
```json
{
  "code": 0,
  "message": "Success",
  "data": { /* 端点对应的数据 */ }
}
```
- 鉴权失败（401）：
```json
{
  "code": "UNAUTHORIZED",
  "message": "Authentication failed"
}
```

### 快速开始（curl）
先准备环境变量：
```bash
export BASE_URL="http://localhost:3001"   # 服务地址与端口
export AUTH_KEY="<你的 AUTH_KEY>"        # 与服务端一致
```

#### 1) 列出分组（推荐使用 Bearer Token）
```bash
curl -sS -X GET "$BASE_URL/api/groups" \
  -H "Authorization: Bearer $AUTH_KEY"
```

等价方式（自选其一）：
```bash
# 自定义头：X-Api-Key
curl -sS -X GET "$BASE_URL/api/groups" -H "X-Api-Key: $AUTH_KEY"

# 查询参数
curl -sS -X GET "$BASE_URL/api/groups?key=$AUTH_KEY"
```

成功返回示例（部分字段）：
```json
{
  "code": 0,
  "message": "Success",
  "data": [
    {
      "id": 1,
      "name": "demo",
      "display_name": "Demo",
      "description": "Demo group",
      "channel_type": "openai",
      "test_model": "gpt-4o-mini",
      "sort": 0,
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

#### 2) 创建分组
最小可用示例（字段以后端校验为准）：
```bash
curl -sS -X POST "$BASE_URL/api/groups" \
  -H "Authorization: Bearer $AUTH_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "demo",
    "display_name": "Demo",
    "description": "Demo group",
    "channel_type": "openai",
    "upstreams": [{"url": "https://api.openai.com/v1", "weight": 1}],
    "test_model": "gpt-4o-mini",
    "sort": 0
  }'
```

成功返回示例（部分字段）：
```json
{
  "code": 0,
  "message": "Success",
  "data": {
    "id": 2,
    "name": "demo",
    "endpoint": "http://localhost:3001/proxy/demo",  
    "display_name": "Demo",
    "channel_type": "openai",
    "test_model": "gpt-4o-mini",
    "sort": 0,
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

### 常见问题
- 没带密钥或密钥错误会返回 401：`{"code":"UNAUTHORIZED","message":"Authentication failed"}`。
- `AUTH_KEY` 来自服务端环境变量，确保后端进程已设置该值并重启生效。
- 也支持 `X-Goog-Api-Key` 或查询参数 `?key=`，但推荐使用标准 `Authorization: Bearer`。