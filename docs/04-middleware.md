# 第四阶段：Gin 中间件

状态：已讲解概念和结果判断；完整运行练习、权限验证案例待补。

## 为什么需要

查询商户、发布评论、收藏商户都可能需要日志与耗时统计。中间件把公共请求处理逻辑集中起来；登录验证还可以决定是否允许进入接口。

```go
func Middleware(c *gin.Context)
```

中间件和路由处理函数操作同一次请求的 Context。

## c.Next：执行后续处理，然后回来

```text
A 中间件前半部分
  B 中间件前半部分
    商户接口
  B 中间件后半部分
A 中间件后半部分
```

以上顺序以 A、B 都调用 Next 为前提。Next 执行当前处理链中剩余的处理函数，完成后返回到当前调用位置之后。

它不是“中止中间件并跳回注册中间件的位置”。它也不是 Go return。

耗时统计流程：记录 start → Next → 计算 time.Since(start) → 写日志。测量的是后续处理等经过时间，不是数据库查询独占时间，也不是客户端网络往返总时间。

## c.Abort 与 return

```go
c.JSON(401, gin.H{"message": "请先登录"})
c.Abort()
return
```

| 操作 | 作用 |
|---|---|
| c.JSON | 写出响应 |
| c.Abort | 阻止尚未执行的后续处理链 |
| return | 退出当前 Go 函数 |

Abort 后当前函数的下一行仍会执行，除非 return。仅 return 而不 Abort，也不能保证 Gin 不执行后面的接口。省略 Next 本身同样不能当作拒绝请求的方法。

Abort 不撤销已经执行的操作；外层已进入的 Next 返回后，外层中间件剩余代码仍可能运行，例如记录拒绝请求的耗时。

## c.Set / c.Get

```go
c.Set("userID", 101)
userID, exists := c.Get("userID")
```

保存和读取当前请求内部的数据。exists 表示键是否存在，不代表读取出的值已经具备业务有效性。键区分大小写，保存 userID 后读取 userid 得到 exists=false。

这不会自动设置 HTTP Header、Cookie，也不会把数据保存到下一次请求。

## 三种作用范围

| 范围 | 注册片段 | 案例 |
|---|---|---|
| 全局 | r.Use(RequestTimer) | 请求耗时 |
| 单路由 | r.POST("/comments", AuthMiddleware, createComment) | 评论需要登录 |
| 路由组 | user.Use(AuthMiddleware) | 一组用户操作需要登录 |

全局和组 Use 应放在相关路由注册之前。上表是注册形式，不是可以独立运行的程序。

```text
全局中间件 → 路由组中间件 → 单路由中间件 → 接口
```

同一个 AuthMiddleware 可以复用到评论和收藏接口。登录、注册应放在不要求已登录的入口，否则未登录用户会被挡在登录接口之前。

## 待补练习

- 运行日志示例，验证 A 前、B 前、接口、B 后、A 后的顺序。
- 拒绝请求后确认接口未执行，但外层日志仍执行。
- 登录和权限验证不同：身份有效不代表有管理员权限。权限案例尚未展开。
- Session 登录实操已按用户要求跳过，未来认证阶段再使用验证后的身份数据。

参考：[Gin Context](https://pkg.go.dev/github.com/gin-gonic/gin#Context)。

返回：[学习目录](../README.md)
