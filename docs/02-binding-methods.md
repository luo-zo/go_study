# 第二阶段：理解 ShouldBind 系列

状态：已讲解，并正确判断“ShouldBind 遇到表单 Content-Type 时会按表单方式选择解析器”。

## 选择方法的思路

先判断数据在哪里，再判断实际格式；不要仅根据业务名称选择方法。

| 请求形式 | 数据位置或格式 | 方法 | 常用标签 |
|---|---|---|---|
| Query | URL 的 ? 后 | ShouldBindQuery | form |
| POST URL 编码表单 | application/x-www-form-urlencoded | ShouldBind | form |
| POST multipart 表单 | multipart/form-data，带 boundary | ShouldBind | form |
| JSON | application/json | ShouldBindJSON | json |
| XML | application/xml | ShouldBindXML | xml |

## 自动选择与明确指定

```text
ShouldBind
→ 检查请求方法
  → GET：选择表单绑定方式，读取 Query
  → 其他方法：根据 Content-Type 选择绑定器
    → JSON 类型：JSON
    → XML 类型：XML
    → multipart 类型：multipart 表单
    → 其他情况：默认表单方式
```

ShouldBindQuery 明确只读取 Query；ShouldBindJSON 和 ShouldBindXML 明确选择对应解析器。表单绑定在某些请求中也会涉及 Query，因此不要把 ShouldBind 理解成“永远只读 Body”。

## 同一业务的不同表达

JSON：属性名和值，数字与字符串有类型区别。

```json
{"shop_id": 101, "content": "很好吃", "score": 5}
```

XML：用标签组织内容。

```xml
<comment><shop_id>101</shop_id><content>很好吃</content><score>5</score></comment>
```

Form：普通字段主要作为文本键值对提交，绑定时再转换成 Go 类型；multipart 还可以上传文件。

## 易错场景

Body 实际是 JSON，却把 Content-Type 写为表单类型并调用 ShouldBind。Gin 可能无法得到预期字段，甚至没有返回错误；结构体仍可能保留零值。

因此：

1. 请求头应描述实际 Body 格式。
2. 明确 JSON 接口可以使用 ShouldBindJSON 表达接口意图。
3. 绑定成功不等于业务数据有效，后续仍需校验。

## 练习

为“搜索商户”“发布 JSON 评论”“提交注册表单”分别说出数据位置、格式、绑定方法和结构体标签。

参考：[Gin ShouldBind](https://pkg.go.dev/github.com/gin-gonic/gin#Context.ShouldBind)。

返回：[学习目录](../README.md)
