# 函数类型

函数类型表示具有相同参数和返回结果类型的所有函数的集合。函数类型的未初始化变量的值是 nil。

函数类型的定义如下：
```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```
这里，“func”关键字表示声明的是一个函数类型，“Signature”则定义了函数的参数和返回值。“Parameters”和“Result”分别表示参数列表和返回结果，它们都是可选的，意味着函数可以没有参数或没有返回值。

在参数或结果列表中，名称（IdentifierList）必须全部存在或全部缺失。如果存在，每个名称代表指定类型的一个项目（参数或结果），并且签名中所有非空白名称必须是唯一的。如果缺失，每个类型代表该类型的一个项目。参数和结果列表总是加上括号的，除非只有一个未命名的结果，它可以写为不带括号的类型。

最终传入的参数在函数签名中可能有一个前缀为“...”的类型。拥有这样参数的函数被称为可变参数函数，并且可能被调用时对该参数传入零个或多个参数。

以下是一些函数类型的例子：
- `func()` 是一个没有参数也没有返回值的函数。
- `func(x int) int` 是一个接受一个整型参数并返回一个整型值的函数。
- `func(a, _ int, z float32) bool` 和 `func(a, b int, z float32) (bool)` 是接受两个整型和一个浮点型参数，并返回一个布尔值的函数。
- `func(prefix string, values ...int)` 是一个接受一个字符串和任意数量整型参数的函数。
- `func(a, b int, z float64, opt ...interface{}) (success bool)` 是一个接受两个整型，一个双精度浮点型，和任意数量的空接口类型参数，并返回一个命名布尔值的函数。
- `func(int, int, float64) (float64, *[]int)` 是一个接受两个整型和一个浮点型参数，并返回一个浮点型和一个整型切片指针的函数。
- `func(n int) func(p *T)` 是一个接受一个整型参数，并返回一个函数的函数。返回的这个函数接受一个类型为 `T` 的指针作为参数。