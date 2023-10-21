# 结构体类型

结构体是一系列命名元素的序列，称为字段，每个字段都有一个名称和类型。字段名可以显式指定（IdentifierList）或隐式指定（EmbeddedField）。在结构体内，非空白字段名必须是唯一的。

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName [ TypeArgs ] .
Tag           = string_lit .
```

// 一个空的结构体。
```go 
struct {}
```

// 一个有6个字段的结构体。
```go 
struct {
	x, y int
	u float32
	_ float32  // 填充
	A *[]int
	F func()
}
```

声明了类型但没有显式字段名的字段称为嵌入式字段。嵌入字段必须指定为类型名 T 或指向非接口类型名的指针 *T，而 T 本身不能是指针类型。未限定的类型名充当字段名。

```go
// 一个有四个嵌入字段的结构体，类型为 T1, *T2, P.T3 和 *P.T4
struct {
	T1        // 字段名是 T1
	*T2       // 字段名是 T2
	P.T3      // 字段名是 T3
	*P.T4     // 字段名是 T4
	x, y int  // 字段名是 x 和 y
}
```

以下声明是非法的，因为字段名在结构体类型中必须是唯一的：

```go
struct {
	T     // 与嵌入字段 *T 和 *P.T 冲突
	*T    // 与嵌入字段 T 和 *P.T 冲突
	*P.T  // 与嵌入字段 T 和 *T 冲突
}
```

如果 x.f 是一个合法的选择器，它指代那个字段或方法 f，那么结构体 x 中嵌入字段的一个字段或方法 f 被称为提升的。

提升的字段表现得就像结构体的普通字段，除了它们不能被用作结构体的复合字面量中的字段名。

给定结构体类型 S 和命名类型 T，提升的方法包含在结构体的方法集中，如下所述：

如果 S 包含嵌入字段 T，那么 S 和 *S 的方法集都包括接收者 T 的提升方法。*S 的方法集还包括接收者 *T 的提升方法。
如果 S 包含嵌入字段 *T，那么 S 和 *S 的方法集都包括接收者 T 或 *T 的提升方法。
字段声明之后可以跟一个可选的字符串文字标签，它成为相应字段声明中所有字段的属性。空标签字符串等同于缺失的标签。标签通过反射接口可见，并参与结构体的类型标识，但其他方面会被忽略。

```go
struct {
	x, y float64 ""  // 一个空的标签字符串就像一个缺失的标签
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// 对应于 TimeStamp 协议缓冲区的结构体。
// 标签字符串定义了协议缓冲区的字段编号；
// 它们遵循 reflect 包概述的约定。
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```

结构体类型 T 不得包含类型 T 的字段，或者包含 T 作为组件的类型的字段，直接或间接地，如果那些包含类型仅仅是数组或结构体类型。

```go
// 无效的结构体类型
type (
	T1 struct{ T1 }            // T1 包含一个 T1 的字段
	T2 struct{ f [10]T2 }      // T2 包含 T2 作为数组的组件
	T3 struct{ T4 }            // T3 包含 T3 作为数组中结构体 T4 的组件
	T4 struct{ f [10]T3 }      // T4 包含 T4 作为数组中结构体 T3 的组件
)

// 有效的结构体类型
type (
	T5 struct{ f *T5 }         // T5 包含 T5 作为指针的组件
	T6 struct{ f func() T6 }   // T6 包含 T6 作为函数类型的组件
	T7 struct{ f [10][]T7 }    // T7 包含 T7 作为数组中切片的组件
)
```