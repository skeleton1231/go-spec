接口类型
接口类型定义了一个类型集。接口类型的变量可以存储任何在接口的类型集中的类型的值。这样的类型被称为实现了该接口。未初始化的接口类型变量的值为 nil。

```go
InterfaceType  = "interface" "{" { InterfaceElem ";" } "}" .
InterfaceElem  = MethodElem | TypeElem .
MethodElem     = MethodName Signature .
MethodName     = identifier .
TypeElem       = TypeTerm { "|" TypeTerm } .
TypeTerm       = Type | UnderlyingType .
UnderlyingType = "~" Type .
```
接口类型由一系列接口元素指定。一个接口元素可以是一个方法或一个类型元素，其中类型元素是一个或多个类型术语的并集。类型术语可以是单一类型或单一底层类型。

基本接口
在最基本的形式上，接口指定了一个（可能为空的）方法列表。由这样的接口定义的类型集是实现所有这些方法的类型的集合，相应的方法集正好由接口指定的方法组成。完全通过方法列表定义类型集的接口称为基本接口。

```go
// 一个简单的文件接口。
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
```
每个明确指定的方法的名称必须是唯一的，不能是空白的。

```go
interface {
	String() string
	String() string  // 非法：String 不唯一
	_(x int)         // 非法：方法必须有非空白名称
}
```
多种类型可能实现一个接口。例如，如果两种类型 S1 和 S2 有以下方法集：

```go
func (p T) Read(p []byte) (n int, err error)
func (p T) Write(p []byte) (n int, err error)
func (p T) Close() error
```
（其中 T 代表 S1 或 S2），那么 S1 和 S2 都实现了 File 接口，无论 S1 和 S2 有哪些其他方法或共享哪些方法。

一个接口的类型集的成员类型都实现了该接口。任何给定的类型可能实现多个不同的接口。例如，所有类型都实现了空接口，它代表了所有（非接口）类型的集合：

```go
interface{}
```
为方便起见，预声明的类型 any 是空接口的别名。

同样，考虑这个接口规范，它出现在类型声明中，用来定义一个名为 Locker 的接口：

```go
type Locker interface {
	Lock()
	Unlock()
}
```
如果 S1 和 S2 也实现

```go
func (p T) Lock() { … }
func (p T) Unlock() { … }
```
它们就同时实现了 Locker 接口和 File 接口。

嵌入式接口
在稍微更通用的形式中，接口 T 可以使用（可能是合格的）接口类型名称 E 作为接口元素。这被称为在 T 中嵌入接口 E。T 的类型集是 T 的显式声明方法定义的类型集和 T 的嵌入式接口的类型集的交集。换句话说，T 的类型集是实现 T 的所有显式声明方法和 E 的所有方法的所有类型的集合。

```go
type Reader interface {
	Read(p []byte) (n int, err error)
	Close() error
}

type Writer interface {
	Write(p []byte) (n int, err error)
	Close() error
}

// ReadWriter 的方法是 Read、Write 和 Close。
type ReadWriter interface {
	Reader  // 在 ReadWriter 的方法集中包括 Reader 的方法
	Writer  // 在 ReadWriter 的方法集中包括 Writer 的方法
}
```
当嵌入接口时，具有相同名称的方法必须具有相同的签名。

```go
type ReadCloser interface {
	Reader   // 在 ReadCloser 的方法集中包括 Reader 的方法
	Close()  // 非法：Reader.Close 和 Close 的签名不同
}
```
通用接口
在最通用的形式中，接口元素也可以是任意类型术语 T，或者是指定底层类型 T 的形式为 ~T 的术语，或者是术语 t1|t2|…|tn 的并集。与方法规范一起，这些元素使得精确定义接口的类型集成为可能，如下所示：

- 空接口的类型集是所有非接口类型的集合。
- 非空接口的类型集是其接口元素的类型集的交集。
- 方法规范的类型集是其方法集包括该方法的所有非接口类型的集合。
- 非接口类型术语的类型集只包括该类型。
- 形式为 ~T 的术语的类型集是其底层类型为 T 的所有类型的集合。
- 术语 t1|t2|…|tn 的并集的类型集是这些术语的类型集的并集。

“所有非接口类型的集合”的量化不仅指的是手头程序中声明的所有（非接口）类型，还指的是所有可能程序中的所有可能类型，因此是无限的。同样，考虑到实现特定方法的所有非接口类型的集合，这些类型的方法集的交集将恰好包含该方法，即使程序中的所有类型总是将该方法与另一种方法配对。

根据构造，一个接口的类型集永远不包含接口类型。

```go
// 一个仅代表 int 类型的接口。
interface {
	int
}

// 一个代表所有底层类型为 int 的类型的接口。
interface {
	~int
}

// 一个代表所有底层类型为 int 且实现 String 方法的类型的接口。
interface {
	~int
	String() string
}

// 一个代表空类型集的接口：没有类型既是 int 又是 string。
interface {
	int
	string


}
```
上述例子的最后一个接口，其类型集为空，因为没有类型既是 int 又是 string。因此，没有类型实现该接口，尽管每个单独的类型术语（int 和 string）都代表一个非空的类型集。同样，一个由单个方法组成的接口，其签名包含一个接口类型的参数或结果，将代表一个空的类型集，因为没有非接口类型可以实现一个带有接口类型的方法。这些情况都是稀有的，不太可能在实践中发生。