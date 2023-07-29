# Go编程语言规范
### 2022年12月15日版本

## 介绍

这是Go编程语言的参考手册。没有泛型的pre-Go1.18版本可以在此处找到。有关更多信息和其他文档，请查看golang.org。

Go是一种通用语言，专为系统编程而设计。它是强类型的、垃圾收集的，并对并发编程有明确的支持。程序是由包构建的，其属性允许有效管理依赖关系。

语法紧凑，易于解析，允许集成开发环境等自动工具进行轻松分析。

## 符号记法
语法是使用扩展巴科斯-诺尔形式（EBNF）的变体指定的：

```
Syntax = { Production } .
Production = production_name "=" [ Expression ] "." .
Expression = Term { "|" Term } .
Term = Factor { Factor } .
Factor = production_name | token [ "…" token ] | Group | Option | Repetition .
Group = "(" Expression ")" .
Option = "[" Expression "]" .
Repetition = "{" Expression "}" .
```

产生式是从术语和以下运算符构建的表达式，优先级逐渐提高：

```
| 交替
() 组合
[] 选项（0或1次）
{} 重复（0至n次）
```

小写的产生式名称用于识别词汇（终端）记号。非终端以驼峰形式表示。词汇记号用双引号""或反引号``包围。

形式a … b表示从a到b的字符集作为替代。水平省略号…也在规范的其他地方用于非正式地表示各种未进一步指定的枚举或代码片段。字符…（而不是三个字符...）并非Go语言的记号。