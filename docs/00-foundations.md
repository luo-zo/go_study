# 已有基础：学习起点

状态：用户在学习开始时自述已经学过，后续课程没有从头重复。

## 已学内容索引

| 类别 | 内容 | 大众点评中的用途 |
|---|---|---|
| 项目搭建 | go mod init、go mod tidy、go get、main.go | 创建和管理后端项目依赖 |
| Gin 启动 | gin.Default()、r.Run(":8080") | 启动 HTTP 服务 |
| 路由 | GET、POST、动态路径参数、Query | 商户查询、发布评论 |
| 返回数据 | c.String()、c.JSON()、gin.H、JSONP | 返回文字或结构化结果 |
| 模板 | LoadHTMLGlob、define、template、变量、SetFuncMap、template.FuncMap | 服务端渲染页面 |
| 静态资源 | r.Static() | 提供静态文件 |
| 参数绑定 | ShouldBindQuery、Form 的 ShouldBind、ShouldBindXML、xml 标签 | 读取不同格式的请求数据 |

## 后续反复使用的基础

```go
var comment Comment
err := c.ShouldBindXML(&comment)
```

这里的 &comment 是变量地址。绑定方法通过这个地址写入 comment，而不是返回一个填好的结构体。

Context 是当前请求的处理上下文，可以读取请求、传递当前请求中的数据、写入响应。后续学习中的 c 始终围绕当前请求使用。

gin.Default() 默认带有 Logger 和 Recovery 中间件。后续再增加请求日志时，应理解可能会出现额外日志，而不是认为原先没有日志处理。

## 易混淆点

- c.JSON() 将数据写入响应，不会自动执行 Go 的 return。
- Go 结构体字段首字母大写，使外部包可以访问；JSON/XML 标签负责外部名称映射。
- 上述片段使用了未在本页定义的类型，只用于回顾取地址的含义。

返回：[学习目录](../README.md)
