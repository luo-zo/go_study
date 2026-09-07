# 补充阶段：Header 与 Cookie

状态：已讲解读取、设置、删除和属性。Session 及 Session 登录练习按用户要求跳过，不能标记为已学。

## Header 的两个方向

```text
客户端 → 请求头 → 服务端
客户端 ← 响应头 ← 服务端
```

| 写法 | 含义 |
|---|---|
| c.GetHeader("X-App-Version") | 读取请求中的版本号 |
| c.Header("X-Service-Name", "dianping") | 设置响应头 |
| c.Set("userID", 101) | 仅在当前 Context 内保存数据 |

请求头缺少时 GetHeader 返回空字符串。设置响应头应在写出响应之前。设置响应头后再 GetHeader，不会自动从响应中读到它。HTTP 头字段名不区分大小写，这与 Context 中 c.Set/c.Get 的键不同。

X-App-Version、X-Service-Name 是示例自定义字段，没有自动业务效果。

Postman 上方请求 Headers 用来填写发送内容；下方响应 Headers 用来观察服务端返回头；响应 Body 用来查看 JSON。

## Cookie 为什么存在

用户选择南宁后，希望后续仍默认查询南宁商户。服务器通过响应通知浏览器保存城市，浏览器在后续符合条件的请求中自动带回。

```http
Set-Cookie: city=nanning; Path=/; Max-Age=86400
```

```text
用户选择城市 → 服务端发 Set-Cookie → 浏览器保存
→ 后续匹配请求发 Cookie: city=nanning
→ 后端读取 city → 按城市查询
```

Cookie 存在客户端，不是 c.Set，也不是数据库。客户端不保存或不发送时，后端不能只凭此前发过 Set-Cookie 就读到它。Cookie 内容来自客户端，不能直接把自报 userID 或 role 当成可信身份。

## Gin 设置、读取、删除

设置片段：

```go
c.SetCookie("city", "nanning", 86400, "/", "", false, true)
```

| 参数位置 | 含义 |
|---|---|
| city | 名称 |
| nanning | 值 |
| 86400 | maxAge，秒 |
| / | Path |
| 空字符串 | 不设置 Domain，限定当前主机 |
| false | 不要求 Secure，仅作为本地 HTTP 学习设置 |
| true | HttpOnly |

读取片段：

```go
city, err := c.Cookie("city")
```

读取当前请求携带的值，Cookie 不存在时返回错误；不能直接访问浏览器存储。

删除片段：

```go
c.SetCookie("city", "", -1, "/", "", false, true)
```

通过响应通知浏览器删除。名称、Path、Domain 要与原 Cookie 对应。设置和删除都不会改写已经收到的当前请求，当前请求里原来的 Cookie 仍可能读到。

## maxAge 易错点

| Gin SetCookie 参数 | 行为 |
|---|---|
| 正数 | 指定多少秒后过期 |
| 0 | 不指定持久期限，会话 Cookie |
| 负数 | 通知立即删除 |

不要把 Gin 的 maxAge=0 与 HTTP 原始头 Max-Age=0 混淆：后者表示立即过期。会话 Cookie 也不等于服务端 Session；浏览器会话恢复可能保留会话 Cookie，不能保证关闭窗口就删除。

## 作用范围和属性

| 属性 | 决定什么 | 边界 |
|---|---|---|
| Domain | 发往哪个主机范围 | 省略时只限设置它的主机；设置父域可覆盖其子域 |
| Path | 哪些路径可携带 | /shops 匹配 /shops 和 /shops/101，不是权限控制 |
| Secure | 要求安全连接发送 | 普通线上环境使用 HTTPS；localhost 有特殊处理 |
| HttpOnly | 阻止网页 JS 直接读取 | 浏览器仍可自动携带，后端仍可读取 |
| SameSite | 跨站情况下的携带规则 | 不等于简单比较完整 URL |

Secure 不表示 Cookie 值在客户端被加密存储。HttpOnly 也不会把 Cookie 变成客户端无法修改的可信业务证明。

| SameSite | 简化理解 |
|---|---|
| Strict | 同站携带；跨站点击链接进入通常也不携带 |
| Lax | 同站携带；跨站顶层安全方法导航，例如点击 GET 链接，可携带；普通跨站 POST 通常不携带 |
| None | 允许跨站，必须配合 Secure；仍受浏览器第三方 Cookie 策略影响 |

## 练习

1. 设置 HttpOnly 后，后端还能否读取登录 Cookie？
2. 删除 Cookie 后，在同一次请求中读取，是否一定消失？
3. Path=/shops 的 Cookie 是否会随 /login 请求发送？
4. 说明 c.Set、响应 Header、Cookie 三者的数据保存位置与生命周期。

## Session 状态说明

仅预告过常见服务端会话流程：浏览器保存会话标识，服务端据此查找会话数据。未系统学习创建、存储、读取、失效或登录实操。后续计划暂不依赖这部分。

参考：[Gin SetCookie](https://pkg.go.dev/github.com/gin-gonic/gin#Context.SetCookie)、[MDN Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)。

返回：[学习目录](../README.md)
