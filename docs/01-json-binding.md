# 第一阶段：JSON 参数绑定

状态：已讲解。已经正确判断“缺少 score 时，新声明的 int 字段保持 0”。

## 为什么需要

用户发布评论，需要传递商户编号、内容和评分。客户端发来的是 JSON 文本，业务代码希望访问 Go 结构体字段。

```json
{
  "shop_id": 101,
  "content": "火锅很好吃",
  "score": 5
}
```

JSON 属性名和字符串使用双引号，数字不加引号，最后一个属性后不加逗号。

## 字段对应与执行流程

```go
type Comment struct {
    ShopID  int    `json:"shop_id"`
    Content string `json:"content"`
    Score   int    `json:"score"`
}

var comment Comment
err := c.ShouldBindJSON(&comment)
```

| JSON 属性 | Go 字段 | Go 类型 |
|---|---|---|
| shop_id | ShopID | int |
| content | Content | string |
| score | Score | int |

```text
客户端把 JSON 放进 Body
→ Gin 读取 Body
→ JSON 解析器按标签匹配字段
→ 向 comment 写入数据
→ 返回错误结果 err
```

json 标签也影响结构体返回成 JSON 时的字段名称。ShouldBindJSON 返回错误，数据通过地址写入变量。之后如果配置了 binding 规则，还会执行校验；第一阶段没有配置这些规则。

## 请求与响应方向

| 方法 | 方向 |
|---|---|
| ShouldBindJSON | 请求 JSON → Go 数据 |
| c.JSON | Go 数据 → 响应 JSON |

Content-Type: application/json 描述请求体格式。ShouldBindJSON 已经明确指定 JSON 解析器；错误的 Content-Type 不会把它切换成表单解析器，也不会把非 JSON 文本变成 JSON。

## 实际讨论过的输入

| score 输入 | 本阶段结果 |
|---|---|
| 5 | 写入整数 5 |
| 缺少该属性 | 新声明的 Score 保持零值 0，不因缺失本身报错 |
| 0 | 写入 0，与缺失时的结果相同 |
| "5" | 普通 int 字段发生类型不匹配，本例没有配置字符串数字转换 |
| "很好" | 类型不匹配 |

“缺少字段保持零值”的前提是这里创建了新变量；更一般地说，未出现的字段保持原有值。不要依赖失败后的部分绑定数据继续处理业务。

## 错误分支

```text
err != nil → 写入 HTTP 400 和错误 JSON → return
err == nil → 使用 comment → 返回成功响应
```

返回“评论数据接收成功”不表示已将评论保存到数据库。

## Postman 复习步骤

在课程中的 POST /comments 接口运行后：

1. 方法选 POST，地址为 http://localhost:8080/comments。
2. Body 选择 raw → JSON。
3. 确认 Content-Type 为 application/json。
4. 发送正常评论，观察 data 中的结构体数据。
5. 删除 score，再将 score 改成字符串，比较结果。

## 练习

实现 POST /shop，接收 name、city、score；绑定失败返回 400，成功返回收到的商户数据。此练习未在对话中验收。

参考：[ShouldBindJSON](https://pkg.go.dev/github.com/gin-gonic/gin#Context.ShouldBindJSON)。

返回：[学习目录](../README.md)
