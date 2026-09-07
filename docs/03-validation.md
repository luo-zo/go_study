# 第三阶段：参数校验与中文错误反馈

状态：已讲解全部计划规则。用户修改了用户名 len=4、密码 len=8，并正确同步修改错误分支。email、oneof 尚未提交完整练习代码。

## 绑定与校验

绑定解决“数据能否读入结构体”，校验解决“字段是否满足配置的要求”。例如 score=8 可以绑定到 int，却不满足 1～5 分的要求。

```text
请求 → 绑定 → binding 校验 → 成功后继续业务
                         → 失败后返回错误并结束处理
```

绑定方法就在接口处理函数内部调用，不应理解成 Controller 完全执行前 Gin 一定自动完成全部绑定。商户是否存在等业务检查也不是 required 能解决的。

## 已学规则

| 规则 | 当前案例中的含义 | 例子 |
|---|---|---|
| required | 字符串不能为 ""，整数不能为 0 | 评论内容、商户编号 |
| min | 数字下限，或字符串最少字符数 | min=1、min=5 |
| max | 数字上限，或字符串最多字符数 | max=5、max=200 |
| len | 字符串长度必须刚好等于指定值 | len=6 验证码 |
| email | 检查邮箱格式 | required,email |
| oneof | 值必须属于指定集合 | oneof=hotpot bbq dessert |

字符串长度规则按 Unicode 码点数计算；普通汉字“好吃”是 2。required 不会自动去掉空格；字符串 "000000" 不是空字符串。

```go
Score int `json:"score" binding:"required,min=1,max=5"`
```

规则间用逗号，oneof 的候选值之间用空格。required 判断零值，不等于检查“客户端是否明确出现过这个属性”。数字的 min=1 本身也会拒绝 0。

## 已完成的 len 修改

```go
type RegisterRequest struct {
    Username string `json:"username" binding:"required,len=4"`
    Password string `json:"password" binding:"required,len=8"`
}
```

这是“用户名刚好 4 个字符、密码刚好 8 个字符”的练习规则，不是推荐的正式密码策略。"小明同学" 和 "12345678" 可以通过这些规则；9 个字符的密码也会失败。

## 其他规则的边界

- len=6 允许 "abcdef"，只保证长度，不保证全为数字，也不保证验证码正确。
- 验证码用字符串能保留前导零，例如 "012345"。
- email 格式通过，不证明邮箱真实存在或属于当前用户。
- oneof=dine_in takeaway delivery 不接受中文“堂食”，也不接受 "DINE_IN"。

## 错误反馈流程

```text
ShouldBindJSON 返回 err
→ 判断是否为 validator.ValidationErrors
  → 是：遍历字段错误，生成中文提示列表
  → 否：处理 JSON 格式、字段类型等问题
→ 返回 HTTP 400
→ return，停止当前业务处理
```

### errors.As：判断并取出错误

```go
var validationErrors validator.ValidationErrors
if errors.As(err, &validationErrors) {
    // 可以遍历校验错误
}
```

第一行准备变量；errors.As 检查 err 及其包装链中是否存在匹配类型。如果找到，就将匹配的错误赋给 validationErrors，并返回 true。& 允许函数写入这个变量。类型名不等于变量名，声明关键字必须是 var，不能写成 ar。

### FieldError：准备返回给客户端的数据

```go
type FieldError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
}
```

```go
item := FieldError{
    Field:   fieldErr.Field(),
    Message: "字段不符合要求",
}
```

fieldErr 是校验库提供的错误，item 是我们整理后的响应项。默认配置下 Field() 返回 Go 字段名，例如 Username；不是自动返回 JSON 名 username。

### 两层 switch

```text
外层 Field() → Username 或 Password：确定字段
内层 Tag() → required、min、len 等：确定失败规则
→ 设置 item.Field 为 JSON 属性名
→ 设置 item.Message 为对应中文提示
→ append 到错误列表
```

Tag() 返回 "min" 而不是 "min=3"。如果改成 len=4，错误分支也要匹配 "len"，提示要写“必须是 4 个字符”。Go switch 默认不向下贯穿，不需要 break。

同一个字段的多个规则按顺序校验，通常只报告该字段第一个失败规则；多个字段可以各有错误。不能理解为自动汇总所有可能失败的规则。

## 响应示例

在原始用户名 min=3、密码 min=8 的版本中，提交 "小明"、"123" 后：

```json
{
  "message": "参数错误",
  "errors": [
    {"field": "username", "message": "用户名至少需要 3 个字符"},
    {"field": "password", "message": "密码至少需要 8 个字符"}
  ]
}
```

中文提示由后端自己映射生成，Gin 不会自动生成这份响应。响应不回显密码。参数通过也不意味着真实注册完成，本阶段没有创建账户。

## 练习

1. len 版本中将 username 改为空，观察是否优先返回 required 提示。
2. 添加 Email，规则为 required,email，并映射中文提示。
3. 为 Category 添加 required,oneof=hotpot bbq dessert，比较合法和非法选项。

参考：[validator 文档](https://pkg.go.dev/github.com/go-playground/validator/v10)。

返回：[学习目录](../README.md)
