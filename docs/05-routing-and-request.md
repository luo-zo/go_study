# 第五阶段：路由分组与请求数据

状态：分组、请求方法、Path/Query/Body/Header 已讲解。

## 路由分组

```go
shops := r.Group("/shops")
{
    shops.GET("", getShops)
    shops.GET("/:id", getShop)
    shops.POST("", createShop)
    shops.PUT("/:id", updateShop)
    shops.DELETE("/:id", deleteShop)
}
```

上述为注册片段，处理函数需要另行实现。Group 创建公共路径前缀和中间件组织关系；外层花括号只是 Go 代码块，不是 Gin 的分组机制。

```text
/shops + "" → /shops
/shops + /:id → /shops/:id
```

同一路径可使用不同请求方法。GET /shops/101 按方法和路径匹配到详情处理函数，不会执行组内所有接口。

| 请求 | 业务 |
|---|---|
| GET /shops | 查询商户列表 |
| GET /shops/101 | 查询 101 号商户 |
| POST /shops | 新增商户 |
| PUT /shops/101 | 更新该商户完整可编辑信息 |
| DELETE /shops/101 | 删除该商户 |

PUT 的接口约定应明确完整更新语义；后续如果只修改部分字段，可另学 PATCH。本阶段仅建立设计概念。

## 请求数据的位置

| 位置 | 用途 | 案例 |
|---|---|---|
| Path | 标识资源 | /shops/101 |
| Query | 筛选、排序、分页 | ?city=南宁&page=2 |
| Body | 提交新增或修改数据 | 评论内容和评分 |
| Header | 格式说明、凭据等附加信息 | Content-Type: application/json |

Query 不是 GET 专属；同一次请求可以同时使用以上多个位置。

发布评论示意：

```http
POST /shops/101/comments
Content-Type: application/json
```

```json
{"content": "火锅很好吃", "score": 5}
```

Path 回答“给哪家商户”，Body 回答“提交什么评论”，Header 描述 Body 的格式。实际读取顺序由后端代码决定。

## 分组不等于授权

组名 /user 或 /admin 自身没有安全效果；权限来自实际注册的中间件和业务检查。r.Group 不会自动创建数据库查询或保存逻辑。

## 练习

1. 删除 25 号商户：说明方法和路径。
2. 查询南宁第 2 页商户：说明参数应放在哪里。
3. 判断 GET /shops 和 POST /shops 为什么可以同时存在。

返回：[学习目录](../README.md)
