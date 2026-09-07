# Go Gin 学习笔记：从请求处理到大众点评后端

更新日期：2026-09-07

目标：理解 Gin 请求处理流程，逐步独立开发简化版大众点评后端。本文档根据实际对话整理；“已讲解”不等于“已独立完成项目”。

## 阅读目录与实际进度

| 阶段 | 文档 | 状态 |
|---|---|---|
| 已有基础 | [00 基础回顾](docs/00-foundations.md) | 用户自述已学，简要索引 |
| 第一阶段 | [01 JSON 参数绑定](docs/01-json-binding.md) | 已讲解并完成结果判断 |
| 第二阶段 | [02 ShouldBind 方法选择](docs/02-binding-methods.md) | 已讲解 |
| 第三阶段 | [03 参数校验与错误反馈](docs/03-validation.md) | 已讲解，完成 len 代码修改练习 |
| 第四阶段 | [04 中间件](docs/04-middleware.md) | 已讲解基础流程，完整实操待补 |
| 第五阶段 | [05 路由分组与请求数据](docs/05-routing-and-request.md) | 基础已讲解 |
| 补充阶段 | [06 Header 与 Cookie](docs/06-headers-and-cookies.md) | 已讲解；Session 按用户要求跳过 |
| 第六阶段 | [07 内存商户列表查询](docs/07-restful-shops.md) | 当前：查询流程已讲，完整代码待练习 |

## 下一步

1. 编写并运行内存版 GET /shops。
2. 用 Postman 测试全部查询、城市筛选和空结果。
3. 继续商户详情、新增、更新、删除。

## 后续主线（尚未学习）

| 阶段 | 计划 |
|---|---|
| 第六阶段剩余 | 商户详情、新增、更新、删除，内存数据并发访问处理 |
| 第七阶段 | GORM + MySQL：连接、Create、First、Find、Where、Updates、Delete |
| 第八阶段 | 从 main.go 逐步拆分 Router、Controller、Service、DAO 等 |
| 第九阶段 | 用户注册登录、bcrypt 密码哈希、JWT、Authorization、认证与权限 |
| 第十阶段 | Redis 类型、TTL、缓存、点赞收藏、排行榜与缓存问题 |
| 第十一阶段 | 整合用户、商户、分类、搜索、评论、点赞、收藏、排行榜与认证 |
| 后续进阶 | 消息队列、限流、并发优化、微服务、go-zero |

Session、Session 登录实操为主动跳过项；中间件完整运行练习与权限验证仍待补。前面的注册示例只是接收和校验参数，没有实现真实账户注册。

## 使用说明

- 每篇包含业务目的、执行流程、关键写法、易错点和练习。
- 本版以流程笔记为主。代码块是知识点片段，除明确说明外不能单独运行。
- 不把未实际运行的代码标记为“测试通过”，不把仅讲解的知识标记为“熟练掌握”。
- 后续提交可继续按阶段更新，保留实际练习结果。

## 参考资料

- [Gin API](https://pkg.go.dev/github.com/gin-gonic/gin)
- [validator 校验库](https://pkg.go.dev/github.com/go-playground/validator/v10)
- [Go JSON 编解码](https://pkg.go.dev/encoding/json)
- [MDN Cookie 指南](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)
