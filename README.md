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

## 源代码表示

源代码是以UTF-8编码的Unicode文本。文本没有规范化，所以一个带有重音的代码点与由重音和字母组合构成的同一个字符不同；它们被视为两个代码点。为了简单起见，本文档将使用“字符”一词来指代源文本中的Unicode代码点。

每个代码点都是不同的；例如，大写字母和小写字母是不同的字符。

实现限制：为了与其他工具兼容，编译器可能不允许在源文本中使用NUL字符（U+0000）。

实现限制：为了与其他工具兼容，如果UTF-8编码的字节顺序标记（U+FEFF）是源文本中的第一个Unicode代码点，编译器可能会忽略它。在源代码的其他地方可能不允许使用字节顺序标记。

### 字符
以下术语用于表示特定的Unicode字符类别：
```
newline        = /* Unicode代码点U+000A */ .
unicode_char   = /* 除换行符之外的任意Unicode代码点 */ .
unicode_letter = /* 被分类为"Letter"的Unicode代码点 */ .
unicode_digit  = /* 被分类为"Number, decimal digit"的Unicode代码点 */ .
```
在Unicode标准8.0中，第4.5节"General Category"定义了一组字符类别。Go将所有在任何Letter类别Lu、Ll、Lt、Lm或Lo中的字符视为Unicode字母，将Number类别Nd中的字符视为Unicode数字。

字母和数字
下划线字符_（U+005F）被视为小写字母。
```
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```