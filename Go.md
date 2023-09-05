https://www.yuque.com/qyuhen/go

# 语法

## 类型

### 基本类型

#### **变量**

关键字 var 用于定义变量。和 C 不同，类型被放在变量名后。

自动初始化为二进制零值（zero value）。
显式提供初始值，可省略类型，由编译器推断。
同一作用域内不能重复定义。
未使用局部变量当做错误。

```go
var x int      // 0
var y = false  // bool
```



可一次定义多个变量，包括用不同初始值定义不同类型。

```go
var (
	x, y int            // 相同类型
	a, s = 100, "abc"   // 不同类型
)
```



**简短模式：（short variable declaration）**

显式初始化。
不能提供类型。
仅限函数内部。

```go
func main() {
	x := 100

	if x > 0 {
		x := "abc"       // 定义同名变量！
		println(&x, x)
	}

	println(&x, x)
}

// 0xc000032760 abc
// 0xc000032758 100
```



简短模式并不总是重新定义，也可能是部分退化的赋值操作。 

**退化赋值** 前提条件：最少有一个新变量被定义，且必须是同一作用域。



```go
func main() {
	x := 100
	println(&x)

	x, y := 200, "abc"  // x 退化赋值，y 是新变量。

	println(&x, x)
	println(y)
}

/*
0xc000032768
0xc000032768 200
abc
*/

```



```go
func main() {
	x := 100
	println(&x)

	{                       // 不同作用域。
		x, y := 200, "abc"  // 全部新定义。不同地址
		println(&x, x)
		println(y)
	}
}

/*
0xc000032768
0xc000032760 200
abc
*/
```



多变量赋值，先计算出全部右值，然后再依次赋值。

```go
func main() {
	x, y := 1, 2
	x, y = y+3, x+2

	println(x, y)  // 5, 3
}
```

#### **常量**

常量表示恒定不变的值，通常是一些字面量。

必须是编译期可确定的值。
可指定类型，或由编译器推断。
不支持字面量类型后缀。（C: 123LL）
可在函数内定义。
未使用常量不会引发编译错误。

```go
const (
	ptrSize = unsafe.Sizeof(uintptr(0))
	strSize = len("hello, world!")
)
```

```go
func main() {
	const x, f = 100, 1.23

	{
		const x = "abc"
		println(x)
	}
}
```



如指定类型，须确保值类型一致，可做显式转换。 右值不能超出常量取值范围，否则引发错误。

```go
const (
	x, y int  = 99, -999
	b    byte = byte(x)

	// n = uint8(y)   
	//     ~~~~~ cannot convert y (constant -999 of type int) to type uint8
)
```



在常量组中如不指定类型和初始化值，则其（类型和初始化表达式）与上一常量相同。

```go
func main() {
	const (
		x uint16 = 120
		y
		s = "abc"
		z
	)

	fmt.Printf("%T, %v\n", y, y)   // uint16, 120
	fmt.Printf("%T, %v\n", z, z)   // string, abc
}
```



#### 枚举

没有明确意义上的枚举类型（enum），借助 iota 自增常量值来实现枚举效果。

```go
const (
  x = iota   // 0
	y          // 1
	z          // 2
)

const (
	_  = iota               // 0
	KB = 1 << (10 * iota)   // 1 << (10 * 1)
	MB                      // 1 << (10 * 2)
	GB                      // 1 << (10 * 3)
)

func main() {
	println(KB, MB)  // 1024 1048576
}
```



自增作用范围为常量组。 可使用多个 iota，各自单独计数，只须列数相同即可。

```go
const (
	_, _ = iota, iota * 10   // 0, 0 * 10
	a, b                     // 1, 1 * 10
	c, d                     // 2, 2 * 10
)
```



如中断自增，须显式恢复，且后续自增值按行序递增。

```go
const (
	a = iota   // 0
	b          // 1
	c = 100    // 100
	d          // 100 （与上一行常量右值表达式相同）
	e = iota   // 4   （恢复 iota 自增，计数包括 c、d 两行）
	f          // 5
)
```



默认数据类型为 int，可显式指定。

```go
const (
	a         = iota   // int, 0
	b float32 = iota   // float32, 1.0
	c         = iota   // int, 2（若没有 iota，则与 b 类型相同）
)
```



用自定义类型实现用途明确的枚举类型。 但不能将取值范围严格限定在预定义的枚举值内。

```go
type color byte

const (
	black color = iota
	red
	blue
)

func test(c color) {
	println(c)
}

func main() {
	test(red)

	const m = 100
	test(m)      // 无类型常量和字面量不超出 color/byte 取值范围即可。

	// const n int = 100
	// test(n)
	//      ~ cannot use n (constant 100 of type int) as type color

	// x := 2
	// test(x)
	//      ~ cannot use x (variable of type int) as type color
}
```



#### 命名

选用有实际含义，易于阅读和理解的字母或单词组合。

以字母或下划线开始。
由字母、数字和下划线组合而成。
区分大小写。

使用驼峰（camel case）拼写格式。
局部变量优先使用短名。
不使用保留关键字。
不与预定义常量、类型、内置函数同名。

空标识符是对编译器的一种建议，提示进行语法检查，但不一定生成对应机器指令。


名为 “_” 的 空标识符（blank identifier）是预设成员，不应重新定义。 可用作表达式左值，表示忽略，但无法作右值返回内容。

```go
func main() {
	x := 100
	_ = x         // 规避变量未使用错误。
}
```



#### 基础类型

```go
==== type ==== | = len = | = default = | ====== comment =============

 bool               1         false
 byte               1           0         uint8

 int,   uint        8           0         x86:4, x64:8
 int8,  uint8       1           0         -128 ~ 127,     0 ~ 255
 int16, uint16      2           0         -32768 ~ 32767, 0 ~ 65535
 int32, uint32      4           0
 int64, uint64      8           0

 float32            4          0.0
 float64            8          0.0

 complex64          8
 complex128        16

 rune               4           0         unicode code point, int32
 uintptr            8           0         uint

 string                        ""         len()
 array                                    len() == cap()
 struct

 function                      nil
 interface                     nil

 map                           nil        make(), len()
 slice                         nil        make(), len(), cap()
 channel                       nil        make(), len(), cap()
```

支持二进制、八进制、十六进制以及科学记数法。

math：定义了各数字类型的取值范围。
strconv：在不同进制（字符串）间转换。



```go
func main() {
	a, b, c := 0b1010, 0o144, 0x64

	fmt.Printf("0b%b, %#o, %#x\n", a, b, c)
	fmt.Println(math.MinInt8, math.MaxInt8)
}

// 0b1010, 0144, 0x64
// -128 127
```



可用下划线作为数字分隔符，以便于阅读。

```go
func main() {
	a := 111_22_3_44
	println(a) // 11122344
}
```



注意，nil 是预定义标识符，不是关键字。

其含义是默认零值（zero value），而非 “空”。
没有类型，不能作为初始值供编译器推断。
不同类型的 nil 值不能比较。

```go
func main() {
	x := nil   // use of untyped nil in assignment
}
```

```go
func main() {
	var m map[string]int = nil
	var c []int = nil

	// fmt.Println(m == c)   // invalid: mismatched types map[string]int and []int
}
```





标准库 sync/atomic 提供了某些基本类型的原子操作版本。

```go
package main

import (
	"sync/atomic"
)

func main() {
	// var x int
	var x atomic.Int64

	go func() {
		x.Add(1)
	}()

	x.Store(x.Load() + 1)

	select{}
}
```



#### 别名

只是语法糖，编译器会将其还原为原始类型。
让通用类型具备更清晰的上下文含义。

```go
byte        // alias for uint8
rune        // alias for int32
```

```go
type X = int   // 别名

func main() {
	var a int = 100
	var b X = a

	fmt.Printf("%T, %v\n", b, b)    // int, 100
  fmt.Println(reflect.TypeOf(b))  // int
}
```

### 引用类型

特指 slice、map、channel 这三种预定义类型。

相比数字、数组等简单类型，引用类型有更复杂的内部结构。除分配内存外，还需初始化一系列属性，诸如指针、长度，甚至哈希分布、数据队列等。

从语言规范看，没有传统意义上的值类型和引用类型。只所以如此表述，除历史原因（早期文档）外，还因为它们与 make 函数的特殊关系，且被编译器和运行时特别对待（优化）。

```
      slice            array
  +-----------+       +-----//-----+
  |   array  -|-----> |  ...  ...  |
  +-----------+       +-----//-----+
  |   len     |
  +-----------+
  |   cap     |
  +-----------+
  
  |<-- new -->|                     
  |<------------- make ----------->|
  
```

以上图 slice 结构为例，由头和底层数组两部分组成。

new：按类型大小分配零值内存，返回指针。
make：转为类型构造函数（或指令），完成内部初始化。



函数 new 只关心类型长度（sizeof），不涉及内部结构和逻辑。

```go
func main() {
	s := *new([]int)             // slice: { ptr, len, cap }
	m := *new(map[string]int)    // map: ptr
	c := *new(chan int)          // chan: ptr

	println(unsafe.Sizeof(s))    // 24
	println(unsafe.Sizeof(m))    // 8
	println(unsafe.Sizeof(c))    // 8
}

/*

(gdb) x/3xg &s  ; 没有底层数组。
0xc000036740:	0x0000000000000000	0x0000000000000000
0xc000036750:	0x0000000000000000

(gdb) x/xg &m   ; 仅指针。
0xc000036708:	0x0000000000000000

(gdb) x/xg &c
0xc000036710:	0x0000000000000000

*/

/*

$ go build -gcflags "-N"
$ go tool objdump -S -s "main\.main" ./test

func main() {

    ; 不涉及内部分配和逻辑处理。

	...

	m := *new(map[string]int)

	MOVQ $0x0, 0x20(SP)	
	MOVQ 0x20(SP), AX	
	MOVQ AX, 0x8(SP)	

	c := *new(chan int)

	MOVQ $0x0, 0x18(SP)	
	MOVQ 0x18(SP), AX	
	MOVQ AX, 0x10(SP)
}
*/
```



虽然 new 的行为很像 malloc，但优先在栈上分配。

```go
func main() {
	p := new(int)
	*p = 100
}

/*

$ go build -gcflags "-N"
$ go tool objdump -S -s "main\.main" ./test

func main() {
	p := new(int)

	MOVQ $0x0, 0(SP)	
	LEAQ 0(SP), AX		
	MOVQ AX, 0x8(SP)	

	*p = 100
	MOVQ $0x64, 0(SP)	
}

*/
```



编译器将 make 翻译成 makeslice、makemap 之类的构造函数调用（或等价指令）。

```go

func main() {
	s := make([]int, 10, 100)
	println(unsafe.Sizeof(s))  // 24
}

/*

(gdb) x/3xg &s  ; 分配底层数组，并初始化 len 和 cap 字段。
0xc000032758:	0x000000c000032438	0x000000000000000a
0xc000032768:	0x0000000000000064

*/
```

```go
func main() {
	s := make([]int, 10000)
	s[0] = 1

	m := make(map[string]int, 10)
	m["a"] = 1
}

/*

$ go build -gcflags "-N"
$ go tool objdump -S -s "main\.main" ./test

func main() {

	s := make([]int, 10000)
		CALL runtime.makeslice(SB)	

	m := make(map[string]int, 10)
		CALL runtime.makemap(SB)

*/
```

### 类型转换

除常量、别名以及未命名类型外，强制要求显式类型转换。

```go
a := 10
b := byte(a)
c := a + int(b)    // 确保类型一致。
```

```go
func main() {
	x := 100
	// var b bool = x  // ~ cannot use x (int) as type bool
	// if x {} 				 // ~ non-boolean condition in if statement
}
```

#### **词法歧义**

如目标类型是指针、单向通道或没有返回值的函数，那么可能会造成语法解析错误。

```go
func main() {
	x := 100
	// p := *int(&x) // ~ cannot convert &x (*int) to type int
	println(p)
}
```

正确做法是用括号，让编译器将 *int 解析为指针类型。

```go
(*int)(p)			// --> 如果没有括号 --> *(int(p))
(<-chan int)(c)		// 			          <-(chan int(c))
(func())(x)			//			          func() x

func()int(x)	    // --> 有返回值的函数类型可省略括号，但依然建议使用。
(func()int)(x)		//     使用括号后，更易阅读。
```

### 自定义类型

关键字 type 基于现有类型（underlying type）定义用户自定义类型。

```go
type A int         // 定义新类型
type B = int       // 别名
```



即使底层类型相同，也非同一类型（区别于别名）。
除运算符外，不继承任何信息（方法等）。
不能隐式转换，不能直接比较。

```go
type X int

func main() {
	var a int = 100
	// var b X = a  // ~ cannot use a (int) as type X

	b := X(a)
	println(b)

	// println(a == b) // ~ invalid operation: mismatched types int and X
}
```



#### 未命名类

与 bool、int  这类有明确标识的类型不同，array、slice、map、channel 等与其元素类型或长度属性相关，被称作未命名类型（unnamed type）。可用 type 提供具体名称，变为命名类型。

```go
[]int             // unnamed type
type A []int      // named type

[2]int, [3]int   // 未命名类型: 长度不同，不是同一类型。
[]int,  []byte   // 未命名类型: 元素类型不同，不是同一类型。
```



**具有相同声明的未命名类型被视作同一类型：**

相同基类型的指针（pointer）。
相同元素类型和长度的数组（array）。
相同元素类型的切片（slice）。
相同键值类型的字典（map）。
相同数据类型及操作方向的通道（channel）。
相同字段序列（字段名、类型、标签、顺序）的结构体（struct）。
相同签名（参数和返回值列表，不包括参数名）的函数（function）。
相同方法集（方法名、方法签名，不包括顺序）的接口（interface）。

```go
func main() {
	var a, b interface {
		test()
	}
	println(a == b)
}
```



**转换规则：**

所属类型相同。
基础类型相同，且其中一个是未命名类型。
数据类型相同，将双向通道赋值给单向通道，且其中一个为未命名类型。
将默认值 nil 赋值给切片、字典、通道、指针、函数或接口。
对象实现了目标接口。

```go
func main() {
  // 同一类型（相同声明的未命名类型）。
	var a [2]int
	var b [2]int = a

  // 基础类型相同，其中一个是未命名类型。
	type array [2]int
	var c array = a

	_, _, _ = a, b, c
}
```

## 语句

语句（statement）执行一到多个动作，表达式（expression）计算并返回结果。 表达式属于语句，而语句未见得是表达式。

**关键字**
仅 25 个 保留关键字（keywords），体现语法规则的简洁性。 保留关键字不能用作常量、变量、函数名以及结构字段等标识符。

```go
break        default       func        interface   select
case         defer         go          map         struct
chan         else          goto        package     switch
const        fallthrough   if          range       type
continue     for           import      return      var
```



**符号**
符号列表。其中，自增（`++`）、自减（`--`）是语句，且没有三元运算符（`?:`）。`^` 即是一元 NOT，也是二元 XOR。而 `~` 为泛型新增。

```go
+      &       +=      &=       &&      ==      !=      (      )
-      |       -=      |=       ||      <       <=      [      ]
*      ^       *=      ^=       <-      >       >=      {      }
/      <<      /=      <<=      ++      =       :=      ,      ;
%      >>      %=      >>=      --      !       ...     .      :
       &^              &^=              ~
```



**二元**
一元优先级最高，二元分五个级别。相同优先级的二元运算符，从左往右依次计算。

```go
highest 	 *     /     %     <<    >>    &    &^
 	         +     -     |     ^ 
 	         ==    !=    <     <=    >     >=
 	         &&
lowest	   ||
```



除位移外，二元操作数类型必须相同。

```go
func main() {
    var a int
    var b int64
    // _ = a + b // ~ invalid operation: mismatched types int and int64
}
```



位移右操作数是整数类型，或可转换的无类型常量。
```go
func main() {
	b := 23
	x := 1 << b  
	println(x)
}
```



位清除（AND NOT, bit clear）和 异或（XOR）不同。 
清除将左右操作数对应二进制位都为 1 的重置为 0（类似位图），以达到一次清除多个标记位的目的。

```
0101 & 0011 = 0001  // AND   与：都为 1
0101 | 0011 = 0111  // OR    或：至少一个 1
0101 ^ 0011 = 0110  // XOR   异或：只有一个 1
      ^0111 = 1000  // NOT   取反 (一元)
```

```go

func main() {
	a := 0b01100101
	b := 0b11010100
	
	x := a ^ b   // 10110001
	c := a &^ b  // 00100001
    
	fmt.Printf("%08b\n", x)
	fmt.Printf("%08b\n", c)
}
```

```go
const (
	read    byte = 1 << iota
	write
	exec
	freeze
)

func main() {
	a := read | write | freeze
	b := read | exec
    
	c := a &^ b 
    
	println(c == write | freeze) // true
	println(c & write == write)  // true
	println(c & read == read)    // fasle
}
```



**初始化**

对复合类型（数组、切片、字典、结构体）变量初始化时，有一些语法限制。

初始化表达式须含类型标签。（数组、字典元素可省略）
左大括号不能另起一行。
多个成员初始值以逗号分隔。
允许多行，但每行须以逗号或右大括号结束。



**作用域**

通常意义上的作用域由大括号构成。作用域构成独立空间，内部所定义成员无法被外部访问。 当然，也有更大范围的作用域，如包（package），它有自己的规则。

作用域限定变量可被谁访问，而生命周期表示对象存活时间（可被不同作用域引用），不同混同。

同级作用域相互隔离。
下层可访问上层作用域成员。
下层作用域遮蔽上层作用域同名成员。

```go
func main() {
	x := 1

	{
		x := "abc"     // 新变量，遮蔽。
		y := 2
		println(x, y)  // abc, 2
	}

	println(x)         // 1
	println(y)         // undefined: y
}
```

### 选择语句

两种分支选择控制语句。

if：适合逻辑控制，分支数量不宜过多。
switch：适合数据匹配，分支内容不宜过长。

#### If

条件表达式值必须是布尔类型，可省略括号，且左大括号不能另起一行。

善用初始化语句，组合调用和返回值处理。
如定义局部变量，其作用域覆盖所有分支。
减少嵌套和分支，让正常逻辑处于相同层次。
如条件表达式过于冗长，可将其重构为函数。

```go
func do(x int) error {
	if x <= 0 {
		return errors.New("x <= 0")
	}
	return nil
}

func main() {
	x := 10
    
  // 局部变量 err 作用域覆盖所有分支。
	if err := do(x); err == nil {
    // 正常逻辑。
		x++
		println(x)
  } else {
    // 错误处理。
		log.Fatalln(err)
	}
}
```

```go
func main() {
	x := 10
	err := do(x)

    // 错误处理是对正常逻辑的补充。
    // 是阅读时可被 “忽略” 的片段。
	if err != nil {
		log.Fatalln(err)
	}
	
	x++
	println(x)
}
```



#### Switch

利用 switch 语句进行多路匹配。

支持初始化语句。
从上到下、从左到右顺序匹配。
全部匹配失败，执行 default。
隐式 break中断。
相邻空 case 不构成多条件匹配。
匹配条件中，不能有重复常量值，变量则按次序匹配。

```go
func main() {
	a, b, c := 1, 2, 3
    
	switch x := 5; x {
	case a, b:              // OR
		println("a | b")
	case c:
		println("c")
	case 4:
		println("d")
	default:
		println(x)
	}
}
```

```go
func main() {
	switch x := 1; x {
	case 0:              // 隐式 break 中断。
	case 1:
	case 2:
		println("b")
	}
}
```

```go
func main() {
	switch x := 1; x {
	case 1:
	case 1, 2:  // duplicate case 1 (constant of type int)
	}
}
```

```go
func main() {
	a, b := 1, 2
	x := a

	switch x {
	case b, a: println("b, a")
	case a   : println("a")
	}
}

// b, a
```



**执行 fallthrough** 

按源码顺序。
不检查后续条件。
必须是当前块（case）结尾，且后面有其他块。
某些时候，省略条件变量以替换分支过多的 if 语句。

```go
func main() {
	switch x := 5; x {
	case 5:
		println("5")
		fallthrough    // 不判断 case 6 匹配条件，直接执行下一个case块
	case 6:
		println("6")
	case 7:
		println("7")
  default:
    println("default")
	}
}

// 5
// 6
```

```go
func main() {
	switch x := 5; x {
	default:
		println("0")
	case 5:
		println("5")

		// 按源码顺序，所以上面的 default 不执行。
		// 除非将 default 挪到下面。

		// fallthrough
		// ~~~~~~~~~~~ cannot fallthrough final case in switch
	}
}
```

```go
func main() {
	switch x := 5; x {
	case 5:
		x += 10

		// if x >= 15 {
		// 	fallthrough   // fallthrough statement out of place
		// }

		if x < 15 { break }
		fallthrough
	case 6:
	}
}
```

```go
func main() {
	switch x := 1; {        // switch x := 5; true { ... }
	case x == -1, x == 1:   // OR, ||
		println("a")
	case x > 1 && x <= 10:
		println("a")
	case x > 10:
		println("b")
	default:
		println("z")
	}
}
```

### 循环语句

仅 for 一种循环语句，常用方式都能支持。

```go
func main() {
	
	for i := 0; i < 3; i++ {
	}

  x := 0
	for x < 10 {
		x++
	}

	for {
		break
	}

}
```



初始化语句仅执行一次。
条件表达式中的函数被多次调用，或被内联优化。
所定义变量，在整个循环周期内重复使用。

```go
func count() int {
	print("count.")
	return 3
}

func main() {

	// 初始化语句仅执行一次。
	for i, c := 0, count(); i < c; i++ {
		println("a", i)
	}
    
	c := 0

	// 条件表达式，多次调用。
	for c < count() {
		println("b", c)
		c++
	}
}
```

#### Range

用 `for...range` 迭代数组、切片、字典等。

```

  type              1st value      2nd value
-----------------+--------------+--------------+-----------------
  string            index          s[index]       unicode, rune
  array, slice      index          v[index]
  map               key            value
  channel           element
```





```go
func main() {
	data := []int{10, 11, 12}

	for i, s := range data {
		println(i, s)
	}
}

/*
0 10
1 11
2 12
*/

// 允许返回单值，或用 _ 忽略。
func main() {
	data := []int{10, 11, 12}

	for i := range data {
		println(i)
	}

	for _, v := range data {
		println(v)
	}

	for range data {
		println("a")
	}
}
```



定义的局部变量会重复使用。

```go
func main() {
	data := []int{ 1, 2, 3 }
    
	for i, s := range data {
		println(&i, &s)
	}
}

/*

0xc000032748 0xc000032740
0xc000032748 0xc000032740
0xc000032748 0xc000032740

*/
```



`range` 会复制目标数据。受直接影响的是数组，改用切片或指针。

```go
func main() {
	data := [...]int{1, 2, 3}

	for i, x := range data {
		if i == 0 {
			data[0] += 100
			data[1] += 200
			data[2] += 300
		}

		println(x, data[i])
	}
}

/*

1 101
2 202
3 303

$ go tool objdump -S -s "main\.main" ./test

func main() {

	data := [...]int{1, 2, 3}

	MOVQ $0x1, 0x20(SP)	
	MOVQ $0x2, 0x28(SP)	
	MOVQ $0x3, 0x30(SP)	

	for i, x := range data {
		
	MOVQ $0x1, 0x38(SP)	
	MOVQ $0x2, 0x40(SP)	
	MOVQ $0x3, 0x48(SP)	

*/
```



如目标表达式是函数调用，仅执行一次。

```go
//go:noinline
func data() []int {
	println("data")
	return []int{10, 20, 30}
}

func main() {
	for _, x := range data() {
		println(x)
	}
}

/*

data
10
20
30

*/
```



### 跳转

#### Goto

讨伐已久的 `goto`，在运行时源码中不算稀客。

虽然某些设计模式可用来消除 goto 语句，但在性能优先的场合，它能发挥积极作用。



使用前，先定义标签。

标签区分大小写。
未使用的标签会引发编译错误。
不能跳转到其他函数，或内层代码块。
不能导致变量定义跳跃。

```go
func main() {

// start:
// label start defined and not used

	for i := 0; i < 3; i++ {
		if i > 1 {
			goto exit
		}
		println(i)
	}
exit:
	println("exit.")
}
```

```go
func test() {
	goto test
test:
}

func main() {
	for{
	loop:
	}

	goto test   // label test not defined
	goto loop   // goto loop jumps into block
}
```

```go
func main() {
	// goto done
	// ~~~~~~~~~ goto done jumps over declaration of v

	v := 0
done:
	println(v)
}
```

#### **中断**

与定点跳转不同，中断结束当前或指定语句块。

`break`：终止 switch/for/select 语句块。
`continue`：仅用于 for，进入下轮循环。

```go
func main() {
	for i := 0; i < 10; i++ {
        
		if i % 2 == 0 {
            continue     // 立即进入下轮循环。(goto next)
		}

		if i > 5 {
			break        // 立即终止整个循环。
		}
        
		println(i)
	}
}

/*
1
3
5
*/
```



配合标签，在多层嵌套中指定目标。

```go
func main() {
    
outer:
	for x := 0; x < 10; x++ {
        
inner:		
		for y := 0; y < 10; y++ {
			if x % 2 == 0 {
				continue outer
			}

			if y > 3 {
				println()
				break inner
			}
            
			print(x, ":", y, " ")
		}
	}
}

/*

1:0 1:1 1:2 1:3 
3:0 3:1 3:2 3:3 
5:0 5:1 5:2 5:3 
7:0 7:1 7:2 7:3 
9:0 9:1 9:2 9:3

*/
```

```go
func main() {
    
start:
	switch 1 {
	case 1:
		println(1)
        
		for {
			break start
		}

		println(2)
	}

	println(3)
}

// 1
// 3
```

## 函数

### 基本概念

函数（function）是结构化编程的最小模块单元。

将复杂算法过程分解成若干较小任务，隐藏细节，使得程序结构更加清晰，易于维护。函数被设计成相对独立，通过接收输入参数完成一段算法指令，输出或存储相关结果。因此，函数还是代码复用和测试的基本单元。

关键字 func 用于定义函数。有些不方便的限制，但也借鉴了动态语言的优点。

无需前置声明。
支持不定长变参。
支持多返回值。
支持命名返回值。
支持匿名函数和闭包。

不支持命名嵌套定义（nested）。
不支持同名函数重载（overload）。
不支持默认参数。

函数是第一类对象。
只能判断是否为 nil，不支持比较操作。

```go
func main() {
    // 不支持命名函数嵌套，得改用匿名。
    func add(x, y int) int {  // syntax error: unexpected add
        return x + y
    }
}
```

```go
func a() {}
func b() {}

func main() {
	println(a == nil)
	// println(a == b) // ~ invalid: func can only be compared to nil
}
```



具备相同签名（参数及返回值列表，不包括参数名）的视作同一类型。

```go
func exec(f func()) {
	f()
}

func main() {
	var f func() = func() { println("hello, world!" )}
	exec(f)
}
```



基于阅读和维护角度，使用命名类型更简洁。

```go
type FormatFunc func(string, ...any) string

// 如不使用命名类型，这个参数签名会长到没法看。
func toStringV2(f FormatFunc, s string, a ...any) string {
    return f(s, a...)
}

func toStringV1(f func(string, ...any) string, s string, a ...any) string {
    return f(s, a...)
}

func main() {
	println(toStringV2(fmt.Sprintf, "%d", 100))
}
```



安全返回局部变量指针。

编译器通过逃逸分析（escape analysis）来决定，是否在堆上分配内存。
但优化后（内联），最终生成的代码未必如此。总之，为了减少垃圾回收压力，编译器竭尽全力在栈分配内存。

```go
func test() *int {
	a := 0x100
	return &a
}

func main() {
	var a *int = test()
	println(a, *a)
}

/*
$ go build -gcflags "-m"
  moved to heap: a
*/
```



不支持尾递归优化。

>**尾递归：**若函数在尾位置调用自身（或是一个尾调用本身的其他函数等等），则称这种情况为**尾递归**。

```go
func factaux (n, ret int) int {
    if n < 2 { return ret }
    return factaux(n - 1, ret * n);
}

func main() {
    println(factaux(3, 1))
}

/*

$ go build
$ go tool objdump -s "main\.factaux" ./test

TEXT main.factaux(SB)
     CALL main.factaux(SB)			
  
*/
```





#### **函数命名规则**

在避免冲突的前提下，函数命名本着 精简短小、望文知意 的原则。

●避免只能通过大小写区分的同名函数。
●避免与内置函数同名，这会导致误用。
●避免使用数字，除非特定专有名词。

函数和方法的命名规则稍有不同。 方法通过选择符调用，且具备状态上下文，可使用更简短的动词命名。



#### **函数参数**

对参数的处理偏向保守。

●按签名顺序传递相同数量和类型的实参。
●不支持有默认值的可选参数。
●不支持命名实参。
●不能忽略 _ 命名的参数。

●参数列表中，相邻同类型参数声明可合并。
●参数可视作函数局部变量。

形参（parameter）是函数定义中的参数，实参（argument）则是函数调用时所传递参数。 形参同函数局部变量，而实参是函数外部对象。

```go
func test(x, y int, s string, _ bool) *int {
	// var x string // ~  x redeclared in this block
	return nil
}

func main() {
	// test(1, 2, "abc")
	//      ~~~~~~~~~~~ not enough arguments in call to test
	//                      have (number, number, string) 
	//                      want (int, int, string, bool)
}
```



不管是指针、引用类型，还是其他类型，参数总是 **值拷贝传递（pass by value）**。区别无非是复制完整目标对象，还是仅复制头部或指针而已。

```go
//go:noline
func test(x *int, s []int) {
	println(*x, len(s))
}

func main() {
	x := 100
	s := []int{1, 2, 3}
	test(&x, s)
}

/*

$ go build
$ go tool objdump -S -s "main\.main" ./test

func main() {

	x := 100
  0x462c94		MOVQ $0x64, 0x20(SP)
  
	s := []int{1, 2, 3}
  0x462ca9		MOVQ $0x1, 0x28(SP)	
  0x462cb2		MOVQ $0x2, 0x30(SP)	
  0x462cbb		MOVQ $0x3, 0x38(SP)	
  
	test(&x, s)
  0x462cc4		LEAQ 0x20(SP), AX	; &x
  0x462cc9		LEAQ 0x28(SP), BX	; s.ptr
  0x462cce		MOVL $0x3, CX		; s.len
  0x462cd3		MOVQ CX, DI		
  0x462cd6		CALL main.test(SB)	
}

*/
```



**何时用指针类型参数：**

●实参对象复制成本过高。
●需要修改目标对象。
●用二级指针实现传出（out）参数。

```go
//go:noinline
func test(p **int) {
	x := 100
	*p = &x
}

func main() {
	var p *int
	test(&p)

	println(*p)
}
```



参数命名为 _，表示忽略。
 比如，实现特定类型函数，忽略掉无用参数，以避免内部污染。

```go
func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    ...
}
```



出现如下情形，建议以复合结构代替多个形参。

●参数过多，不便阅读。
●有默认值的可选参数。
●后续调整，新增或重新排列。

```go
type Option struct {
	addr    string
	port    int
	path    string
	timeout time.Duration
	log     *log.Logger
}

// 创建默认参数。
func newOption() *Option {
	return &Option{
		addr:    "0.0.0.0",
		port:    8080,
		path:    "/var/test",
		timeout: time.Second * 5,
		log:     nil,
	}
}

func server(option *Option) {
	fmt.Println(option)
}

func main() {
	opt := newOption()
	opt.port = 8085     // 修改默认设置。
	server(opt)
}
```



#### **函数变参**

变参本质上就是切片。只能接收一到多个同类型参数，且必须放在列表尾部。

```go
func test(s string, a ...int) {
	fmt.Printf("%T, %v\n", a, a)
}

func main() {
	test("abc", 1, 2, 3, 4)
}

// []int, [1 2 3 4]
```



切片作为变参时，须进行展开操作。

```go
func test(a ...int) {
	fmt.Println(a)
}

func main() {
	a := [3]int{10, 20, 30}
    
    // 转换为切片后展开。
	test(a[:]...) 
}
```



既然变参是切片，那么参数复制的仅是切片自身。正因如此，就有机会修改实参。

```go
//go:noinline
func test(a ...int) {
	for i := range a {
		a[i] += 100
	}
}

func main() {
	a := []int{1, 2, 3}
	test(a...)
	
	println(a[1])
}

/*

$ go build
$ go tool objdump -S -s "main\.main" ./test

func main() {

	a := []int{1, 2, 3}
  0x462c20		MOVQ $0x1, 0x20(SP)	
  0x462c29		MOVQ $0x2, 0x28(SP)	
  0x462c32		MOVQ $0x3, 0x30(SP)	
  
	test(a...)
  0x462c3b		LEAQ 0x20(SP), AX	; a.ptr
  0x462c40		MOVL $0x3, BX		; a.len
  0x462c45		MOVQ BX, CX		
  0x462c48		CALL main.test(SB)

*/
```



#### 参数生命周期

参数和其他局部变量的生命周期，未必能坚持到函数调用结束。 垃圾回收非常积极，对后续不再使用的对象，可能会提前清理。

```go
//go:noinline
func test(x []byte) {
	println(len(x))

    // 模拟垃圾回收触发。
	runtime.SetFinalizer(&x, func(*[]byte){ println("drop!") })
	runtime.GC()

	println("exit.")
    
    // 确保目标活着。
	// runtime.KeepAlive(&x)
}

func main() {
	test(make([]byte, 10<<20))
}

// 10485760
// drop!
// exit.
```



#### **函数返回值**

借鉴动态语言的多返回值模式，让函数得以返回更多状态。

```go
func div(x, y int) (int, error) {
	if y == 0 {
		return 0, errors.New("division by zero")
	}
    
	return x / y, nil
}

func main() {
	z, err := div(6, 2)
	if err != nil {
		log.Fatalln(err)
	}

	println(z)
}
```



●用 _ 忽略不想要的返回值。
●多返回值可用作调用实参，或当结果直接返回。

```go
func log(x int, err error) {
	fmt.Println(x, err)
}

func test() (int, error) {
	return div(5, 0)        // 直接返回。
}

func main() {
	log(test())             // 直接当作多个实参。
}
```



●有返回值的函数，所有逻辑分支必须有明确 return 语句。
●除非 panic，或无 break 的死循环。

```go
func test(x int) int {
	if x > 0 {
		return 1
	} else if x < 0 {
		return -1
	}

	// missing return
} 
```

```go
func test(x int) int {
	for {
		break
	}

	// missing return
}
```



#### **返回值命名**

对返回值命名，使其像参数一样当作局部变量使用。

●函数声明更加清晰、可读。
●更好的代码编辑器提示。

```go
func paging(sql string, index int) (count int, pages int, err error) {
}
```



●可由 return 隐式返回。
●如被同名遮蔽，须显式返回。

```go
func add(x, y int) (z int) {
	z = x + y
	return
}
```

```go
func add(x, y int) (z int) {
	// 作为 “局部变量”，不能同级重复定义。
	{
		z := x + y
		// return // ~ result parameter z not in scope at return
		return z
	}
}
```



要么不命名，要么全部命名，否则编译器会搞不清状况。

```go
func test() (int, s string, e error) {
	// return 0, "", nil // ~ cannot use 0 as string value in return statement
	
}
```



如返回值类型能明确表明含义，就尽量不要对其命名。

```go
func NewUser() (*User, error)
```



### 匿名函数

匿名函数是指没有名字符号的函数。除此之外，和普通函数类似。

●可在函数内定义匿名函数，形成嵌套。
●可随定义直接传参调用，并返回结果。
●可保存到变量、结构字段，或作为参数、返回值传递。

●匿名函数可引用环境变量，形成闭包。
●不曾使用的匿名函数被当作错误。
●不能使用编译指令。（//go:noinline）

```go
func main() {
    // 定义时调用。
	_ = func(s string) string {
		return "[" + s + "]"
	}("abc")

	// --------------
    
    // 变量。
	add := func(x, y int) int {
		return x + y
	}

	_ = add(1, 2)
}
```

```go
func wrap(f func(string)string) func() {
	return func() {
		println(f("abc"))
	}
}

func main() {
	wrap(func(s string) string {
		return "[" + s + "]"
	})()
}
```

```go
func main() {
	// func(s string) { println(s) }
	// ~~~~ func is not used
}
```



编译器为匿名函数自动生成名字。

```go
func main() {
	func() {
		println("abc")
	}()
}

/*
$ go build -gcflags "-l"
$ go tool objdump -S -s "main\.main" ./test

	CALL main.main.func1(SB)
*/
```



普通函数和匿名函数都可作为结构体字段，或经通道传递。

```go
func main() {
	calc := struct {
		add func(int, int) int
		mul func(int, int) int
	}{
		add: func(x, y int) int { return x + y },
		mul: func(x, y int) int { return x * y },
	}

	_ = calc.add(1, 2)
}
```

```go
func main() {
	fn := make(chan func())

	go func() {
		defer close(fn)

		f := <- fn
		f()
	}()

	fn <- func(){ println("hi!") }
	<- fn
}
```

匿名函数是常见的重构手段：将大段代码分解成多个相对独立的逻辑单元。

●不提升代码作用域的前提下，分离流程和细节。
●依赖函数接口而非具体代码，逻辑层次更清晰。
●无状态逻辑与有状态变量分离，更利于测试。
●作用域隔离，修改代码时不会引发外部污染。



#### **闭包**

闭包（closure）是匿名函数和其引用的外部环境变量组合体。

```go
//go:noinline
func test() func() {
	x := 100
	println(&x, x)

	return func() {      // 返回闭包，而非函数。
		x++
		println(&x, x)   // 引用环境变量 x 。
	}
}

func main() {
	test()()
}

/*
0xc000018060 100
0xc000018060 101
*/
```



从上例输出结果看，闭包会延长环境变量生命周期。 所谓闭包，实质上是由匿名函数和（一到多个）环境变量指针构成的结构体。

编译器优化可能改变闭包返回和执行方式，以实际输出为准。

```go
/*

$ go build -gcflags "-m -S"

    moved to heap: x


---- func test ---------------------

    TEXT    "".test(SB), ABIInternal

x := 100

    LEAQ    type.int(SB), AX
    CALL    runtime.newobject(SB)
    MOVQ    AX, "".&x+16(SP)
    MOVQ    $100, (AX)

return func(){...}

    LEAQ    type.noalg.struct { F uintptr; "".x *int }(SB), AX
    CALL    runtime.newobject(SB)

    LEAQ    "".test.func1(SB), CX  ; 匿名函数。
    MOVQ    CX, (AX)

    MOVQ    "".&x+16(SP), CX       ; 环境变量 x 地址。
    MOVQ    CX, 8(AX)
    
    RET

*/
```



接收到的闭包，经专用寄存器（DX），将环境变量地址传递给匿名函数。

```go
/*

--- func main ----------------------

    TEXT    "".main(SB), ABIInternal

    CALL    "".test(SB)   ; 返回闭包。
    MOVQ    (AX), CX      ; 匿名函数。
    MOVQ    AX, DX        ; 传递给匿名函数。
    CALL    CX            ; 调用匿名函数。

--- func test.func1 ----------------------

    TEXT    "".test.func1(SB), NEEDCTXT|ABIInternal

    MOVQ    8(DX), AX         ; 从闭包中获取环境变量。
    MOVQ    AX, "".&x+8(SP)

    CALL    runtime.printlock(SB)
    MOVQ    "".&x+8(SP), AX
    CALL    runtime.printpointer(SB)
    CALL    runtime.printnl(SB)
    CALL    runtime.printunlock(SB)

    RET

*/
```



从实现机制看，拿到手的闭包除非执行，否则只是存有两个指针的结构体。 这会导致 “延迟求值”，一个初学者时常出错的问题。

```go
func test() (s []func()) {
	for i := 0; i < 2; i++ {
        // 循环使用的 i 始终是同一变量。
        // 闭包（存储 i 指针）被添加到切片内。
		s = append(s, func() {
			println(&i, i)
		})
	}
    
	return
}

func main() {
	for _, f := range test() {
        // 执行闭包中的匿名函数。
        // 它以指针读取环境变量 i 的值。
        // 自然是最后一次循环时 i = 2。
		f()
	}
}

// 0xc000018060 2
// 0xc000018060 2
```

```go

func test() (s []func()) {
	for i := 0; i < 2; i++ {
		// 用不同的环境变量。
		x := i
		s = append(s, func() {
			println(&x, x)
		})
	}
	return
}

func main() {
	for _, f := range test() {
		f()
	}
}

// 0xc000082000 0
// 0xc000082008 1
```



**闭包的应用**



静态局部变量。

```go
func test() func() {
	n := 0

	return func() {
		n++
		println("call:", n)
	}
}

func main() {
	f := test()
	f()
	f()
	f()
}

/*

call: 1
call: 2
call: 3

*/
```



模拟方法，绑定状态。

```go
func handle(db Database) func(http.ResponseWriter, *http.Request) {
	return func(w http.ResponseWriter, r *http.Request) {
		url := db.Get()
		io.WriteString(w, url)
	}
}

func main() {
	db := NewDatabase("localhost:5432")
    
	http.HandleFunc("/url", handle(db))
	http.ListenAndServe(":3000", nil)
}
```



包装函数，改变其签名或增加额外功能。

```go
package main

import (
	"fmt"
	"time"
)

// 修改签名。
func partial(f func(int), x int) func() {
	return func() {
		f(x)
	}
}

// 增加功能。
func proxy(f func()) func() {
	return func() {
		n := time.Now()
		defer func() {
			fmt.Println(time.Now().Sub(n))
		}()
		f()
	}
}

func main() {
	test := func(x int) {
		println(x)
	}
	var f func() = partial(test, 100)
	f()
	proxy(f)()
}

// 100
// 100
// 16.1µs
```



### 延迟调用

语句 defer 注册 稍后执行的函数调用。这些调用被称作 延迟调用，因为它们直到当前函数结束前（RET）才被执行。常用于资源释放、锁定解除，以及错误处理等操作。

●注册非 nil 函数，复制执行所需参数。
●多个延迟调用按 FILO 次序执行。
●运行时确保延迟调用总被执行。（`os.Exit` 除外）

```go
func main() {
	f, err := os.Open("./main.go")
	if err != nil {
		log.Fatalln(err)
	}
	defer f.Close()
	
	b, err := io.ReadAll(f)
	if err != nil {
		log.Fatalln(err)
	}
	println(string(b))
}
```

```go
func main() {
	defer println(1)   // 此处是注册，而非执行。
	defer println(2)   // 注册需提供执行所需参数。
	defer println(3)   // 按 FILO 次序执行。

	println("main")    
}                      // 退出前执行 defer。

/*
main
3
2
1
*/
```

```go
	var f func()

	// defer f() // ~ panic: invalid memory address or nil pointer dereference
}
```



正常退出，或以 panic 中断，运行时都会确保延迟函数被执行。

```go
func main() {
	defer println("defer!")
	panic("panic!")          // 换成 os.Exit，会立即终止进程，
}                            // 肯定不执行延迟调用。

/*
defer!
panic: panic!
*/
```



注意参数复制行为，必要时以指针或引用类型代替。

```go
func main() {
	x := 100
	defer println("defer:", x)

	x++
	println("main:", x)
}

/*
 main: 101
 defer: 100
*/
```



延迟调用以闭包形式修改命名返回值，但须注意执行次序。

```go
func test() (z int) {
	defer func() {
		z += 200
	}()
    
    return 100   // return = (ret_val_z = 100, call defer, ret)
}

func main() {
	println(test())  // 300
}
```



如果不是命名返回值，那么结果截然不同。

```go
func test() int {
	z := 0

	defer func() {
		z += 200    // 本地变量，与返回值无关。
	}()
    
    z = 100
    return z        // return = (ret_val = 100, call defer, ret)
}

func main() {
	println(test())  // 100
}
```



### 错误处理

没有结构化异常，以返回值标记错误状态。

```go
// builtin
type error interface {
	Error() string
}

// io
func ReadAll(r Reader) ([]byte, error)
```



●标志性错误：一种明确状态，比如 `io.EOF`。
●提示性错误：返回可读信息，提示错误原因。

必须检查所有返回错误，这也导致代码不太美观。

```go
func main() {
	file, err := os.Open("./main.go")
	if err != nil {
		log.Fatalln(err)
	}

	defer file.Close()

	content, err := io.ReadAll(file)
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Println(string(content))
}
```



标志性错误，通常以 全局变量（指针、接口）方式定义。而提示性错误，则直接返回 临时对象。

```go
// io
var EOF = errors.New("EOF")
```

```go
// src/errors/errors.go

package errors

func New(text string) error {
    return &errorString{text}   // pointer & interface
}

type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

●是否有错：`err != nil`
●具体错误：`err == ErrVar`
●错误匹配：`case ErrVal`
●类型转换：`e, ok := err.(T)`



自定义承载更多上下文的错误类型。

```go
type TestError struct {
	x int
}

func (e *TestError) Error() string {
	return fmt.Sprintf("test: %d", e.x)
}

var ErrZero = &TestError{ 0 }  // 指针! (以便判断是否同一对象)

// -----------------------------

func main() {
	var e error = ErrZero
	fmt.Println(e == ErrZero)

	if t, ok := e.(*TestError); ok {
		fmt.Println(t.x)
	}
}
```



函数返回时，须注意 error 是否真等于 nil，以免错不成错。

```go
func test() error {    
	var err *TestError

	// 结构体指针。
	println(err == nil)   // true
    
	// 接口只有类型和值都为 nil 时才等 于 nil。err 显然有类型信息。正确做法是直接 return nil。
    
	// 转型为接口。
	return err
}

func main() {
	err := test()
	println(err == nil)   // false
}
```




标准库：

●`errors.New`: 创建包含文本信息的错误对象。
●`errors.Join`: 打包一到多个错误对象，构建树状结构。
●`fmt.Errorf`: 以%w 包装一到多个错误对象。

包装对象：

●`errors.Unwrap`: 返回被包装错误对象（或列表）。
●`errors.Is`: 递归查找是否有指定错误对象。
●`errors.As`: 递归查找并获取类型匹配的错误对象。



创建错误树，返回完整的错误信息。

```go

func database() error {
	return errors.New("data")
}

func cache() error {
	if err := database(); err != nil {
		return fmt.Errorf("cache miss: %w", err)
	}

	return nil
}

func handle() error {
	return cache()
}

func main() {
	fmt.Println(handle())
}

// cache miss: data
```



从错误树获取信息。

```go
type TestError struct {}
func (*TestError) Error() string { return "" }

func main() {
    
    // 打包。
	a := errors.New("a")
	b := fmt.Errorf("b, %w", a)
	c := fmt.Errorf("c, %w", b)
    
    fmt.Println(c)                      // c, b, a
    fmt.Println(errors.Unwrap(c) == b)  // true

    // 递归检查。
    fmt.Println(errors.Is(c, a))        // true

	// -------------------------------

	x := &TestError{}
	y := fmt.Errorf("y, %w", x)
	z := fmt.Errorf("z, %w", y)
 
    // 提取（二级指针）类型匹配的错误对象。
	var x2 *TestError
	if errors.As(z, &x2) {
        fmt.Println(x == x2)           // true
	}
}
```

#### panic

与返回 `error` 相比，`panic`/`recover `的使用方式，很像结构化异常。

```go
func panic(v any)
func recover() any
```



恐慌（`panic`）立即中断当前流程，执行延迟调用。 在延迟调用中，恢复（`recover`） 捕获并返回恐慌数据。

●沿调用堆栈向外传递，直至被捕获，或进程崩溃。
●连续引发`panic`，仅最后一次可被捕获。

●先捕获`panic`，恢复执行，然后才返回数据。
●恢复之后，可再度引发`panic`，可再次捕获。

●无论恢复与否，延迟调用总会执行。
●延迟调用中引发`panic`，不影响后续延迟调用执行。

```go
func main() {
	defer func() {
        // 拦截panic，返回数据。
        // 数据未必是 error，也可能是 nil。
        // 无法回到 panic 后续位置继续执行。
        
		if r := recover(); r != nil {
			log.Fatalln(r)
		}
        
	}()
    
    func() {
		panic("p1")   // 终止当前函数，执行 defer。
	}()
	
	println("exit.")  // 即便 recover，也不再执行。
}

// p1
```

```go
func main() {
	defer func() {
        // 只有最后一次被捕获。
		if err := recover(); err != nil {
			log.Fatalln(err)
		}
        
	}()
    
    func() {
    	defer func(){
    		panic("p2")  // 第二次。
    	}()
		panic("p1")      // 第一次。
	}()
	println("exit.")
}

// p2
```

```go
func main() {
	defer func() {
		log.Println(recover())      // 捕获第二次：p2
	}()
    
    func() {
    	defer func(){
    		panic("p2")             // 第二次！
    	}()

    	defer func() {
    		log.Println(recover())  // 捕获第一次：p1
    	}()

		panic("p1")                 // 第一次！
	}()

	println("exit.")
}

// p1
// p2
```



恢复函数 `recover` 只能在延迟调用（`topmost`）内正确执行。 直接注册为延迟调用，或被延迟函数间接调用都无法捕获恐慌。

```go
func catch() {
	recover()
}

func main() {
	defer catch()                   // 有效！在延迟函数内直接调用。
	// defer log.Println(recover()) // 无效！作为参数立即执行。
	// defer recover()              // 无效！被直接注册为延迟调用。  

	panic("p!")
}
```



以 `*PanicNilError` 替代 `nil`，使其有具体含义。

在 1.21 之前，panic(nil) 和没有发生恐慌，recover 都返回 nil，无法区分这两种状态。

```go
func nopanic() {
	defer func() {
		e := recover()
		println(e == nil)   // true
	}()
}

func panicnil() {
	defer func() {
		e := recover()
		println(e == nil)   // false

		_, ok := e.(*runtime.PanicNilError)
		println(ok)         // true
	}()

	panic(nil)
}
```


适用场景：

●在调用堆栈任意位置中断，跳转到合适位置（恢复，参数为依据）。 
○`runtime.Goexit`：不能用于 main goroutine，进程崩溃。
○`os.Exit`：进程立即终止，延迟调用不会执行。 
●无法恢复的故障，输出调用堆栈现场。 
○文件系统损坏。
○数据库无法连接。
○……





## 数据

内置数据类型，使用及结构分析。

### 字符串

字符串是 不可变 字节序列，其本身是一个复合结构。

```go
 +-----------+          +---+---+---+---+---+
 |  pointer -|--------> | h | e | l | l | o |
 +-----------+          +---+---+---+---+---+
 |  len = 5  |          
 +-----------+          [...]byte, UTF-8
 
    header
```

```go
// runtime/string.go

type stringStruct struct {
	str unsafe.Pointer
	len int
}

type stringStructDWARF struct {
	str *byte
	len int
}
```




●编码 `UTF-8`，无 `NULL` 结尾，默认值 `""`。

●使用 \`raw string\` 定义原始字符串。
●支持` !=`、`==`、`<`、`<=`、`>=`、`>`、`+`、`+=`。

●索引访问字节数组（非字符），不能获取元素地址。
●切片返回子串，依旧指向原数组。
●内置函数 `len` 返回字节数组长度。





```go
func main() {
	s := "十二\x61\142\u0041"

	bs := []byte(s)
    rs := []rune(s)     // rune/int32: unicode code point

    fmt.Printf("%X, %d\n", s,  len(s))
	fmt.Printf("%X, %d\n", bs, utf8.RuneCount(bs))
	fmt.Printf("%U, %d\n",  rs, utf8.RuneCountInString(s))
}

// E5 8D 81 E4 BA 8C 61 62 41, 9
// E5 8D 81 E4 BA 8C 61 62 41, 5
// [U+5341 U+4E8C U+0061 U+0062 U+0041], 5
```

```go
func main() {
	s := "十二abc"

	fmt.Printf("%X\n", s[1])  // 8D

	// println(&s[1]) 
	// invalid operation: cannot take address of s[1]
}
```

```go
func main() {
	var s string
	println(s == "")  // true

	// println(s == nil) 
	// invalid operation: mismatched types string and untyped nil
}
```



原始字符串（raw string）内的转义、换行、前置空格、注释等，都视作内容，不与处理。

```go
func main() {
	s := `line\r\n,
  line 2`

	println(s)   // raw string
}

/*
line\r\n,
  line 2
*/
```



以加号连接字面量时，注意操作符位置。

```go
func main() {
	s := "ab" +           // 跨行时，加法操作符必须在上行结尾。
		 "cd"

	println(s == "abcd")  // true
	println(s > "abc")    // true
}
```



子串内部指针依旧指向原字节数组。

```go
func main() {
	s := "hello, world!"
	s2 := s[:4]

	p1 := (*reflect.StringHeader)(unsafe.Pointer(&s))
	p2 := (*reflect.StringHeader)(unsafe.Pointer(&s2))

	fmt.Printf("%#v, %#v\n", p1, p2)
}

// &StringHeader{ Data:0x497208, Len:13 }
// &StringHeader{ Data:0x497208, Len:4 }
```



遍历，分 `byte` 和 `rune`两种方式。

```go
func main() {
	s := "十二12"
    
    // byte
	for i := 0; i < len(s); i++ {         
		fmt.Printf("%d: %X\n", i, s[i])
	}
    
    // rune
	for i, c := range s {
		fmt.Printf("%d: %U\n", i, c)
	}
}
/*
0: E5
1: 8D
2: 81
3: E4
4: BA
5: 8C
6: 31
7: 32

0: U+5341
3: U+4E8C
6: U+0031
7: U+0032
*/
```



可作 `append`、`copy`参数。

```go
func main() {
	s := "de"

    bs := make([]byte, 0)
	bs = append(bs, "abc"...)
	bs = append(bs, s...)

	buf := make([]byte, 5)
	copy(buf, "abc")
	copy(buf[3:], s)

	fmt.Printf("%s\n", bs)   // abcde
	fmt.Printf("%s\n", buf)  // abcde
}
```




标准库相关：

`bytes`：字节切片。
`fmt`：格式化。
`strconv`：转换。
`strings`：函数。
`text`：文本（模版）。
`unicode`：码点。

#### 字符串转换

可在 `rune`、`byte`、`string`间转换。

单引号字符字面量是 `rune`类型，代表 Unicode 字符。

```go
func main() {
	var r rune = '我'

	var s  string = string(r)
	var b  byte   = byte(r)
	var s2 string = string(b)
	var r2 rune   = rune(b)
    
	fmt.Printf("%c, %U\n", r, r)
	fmt.Printf("%s, %X, %X, %X\n", s, b, s2, r2)
}

// 我, U+6211
// 我, 11, 11, 11
```



要修改字符串，须转换为可变类型（`[]rune` 或 `[]byte`），待完成后再转换回来。
但不管如何转换，都需重新分配内存，并复制数据。

```go

func main() {
	s := strings.Repeat("a", 1<<10)

    // 分配内存、复制。
	bs := []byte(s)
	bs[1] = 'B'

    // 分配内存、复制。
	s2 := string(bs)

	hs  := (*reflect.StringHeader)(unsafe.Pointer(&s))
	hbs := (*reflect.SliceHeader)(unsafe.Pointer(&bs))
	hs2 := (*reflect.StringHeader)(unsafe.Pointer(&s2))
	fmt.Printf("%#v\n%#v\n%#v\n", hs, hbs, hs2)
}

// &StringHeader{ Data:0xc0000a8000, Len:1024 }
// &SliceHeader { Data:0xc0000a8400, Len:1024, Cap:1024 }
// &StringHeader{ Data:0xc0000a8800, Len:1024 }

// 0x400 = 1024
```



验证编码是否正确。

```go
func main() {
	s := "十二"
	s2 := string(s[0:2] + s[4:])  // 非法拼接。
    
    fmt.Printf("%X, %X \n", s, s2)
    fmt.Println(utf8.ValidString(s2))
}

// E5 8D 81 E4 BA 8C, E5 8D BA 8C
// false
```

#### 字符串性能优化

使用 “不安全” 方法进行转换，改善性能。

不动底层数组，直接构建 `string` 或`slice` 头。
如果修改，注意内存安全。

`slice { data, len, cap }`  ->` string { data, len }`
`slice { string.data, string.len, string.len }`



动态构建字符串也容易造成性能问题。
加法操作符拼接字符串，每次都需重新分配内存和复制数据。
改进方法是预分配内存，然后一次性返回。

```go
package test

import (
	"bytes"
	"strings"
	"testing"
)

const N = 1000
const C = "a"

var S = strings.Repeat(C, N)


// 性能最差
func concat() bool {
	var s2 string
	for i := 0; i < N; i++ {
		s2 += C
	}
	return s2 == S
}

// 其次
func join() bool {
	b := make([]string, N)
	for i := 0; i < N; i++ {
		b[i] = C
	}

	return strings.Join(b, "") == S
}

// 最好
func buffer() bool {
	var b bytes.Buffer
	b.Grow(N)

	for i := 0; i < N; i++ {
		b.WriteString(C)
	}

	return b.String() == S
}

// ----------------------------------

func BenchmarkConcat(b *testing.B) {
	for i := 0; i < b.N; i++ {
		if !concat() {
			b.Fatal()
		}
	}
}

func BenchmarkJoin(b *testing.B) {
	for i := 0; i < b.N; i++ {
		if !join() {
			b.Fatal()
		}
	}
}

func BenchmarkBuffer(b *testing.B) {
	for i := 0; i < b.N; i++ {
		if !buffer() {
			b.Fatal()
		}
	}
}

```

```go
cpu: 12th Gen Intel(R) Core(TM) i7-12700
BenchmarkConcat-20         13267             93899 ns/op          530276 B/op        999 allocs/op
BenchmarkJoin-20          257335              4692 ns/op            1024 B/op          1 allocs/op
BenchmarkBuffer-20        463772              2501 ns/op            2048 B/op          2 allocs/op
PASS
```



 `strings.Join` 实现，和上面的 buffer 类似。

```go
// strings/strings.go

func Join(elems []string, sep string) string {
	switch len(elems) {
	case 0:
		return ""
	case 1:
		return elems[0]
	}
    
    // 计算总长度。
	n := len(sep) * (len(elems) - 1)
	for i := 0; i < len(elems); i++ {
		n += len(elems[i])
	}

    // 预分配，循环写入。
	var b Builder
	b.Grow(n)
	b.WriteString(elems[0])
	for _, s := range elems[1:] {
		b.WriteString(sep)
		b.WriteString(s)
	}
	return b.String()
}
```



编译器对字面量 + 号拼接会优化。
如此之外，还可以用 `fmt.Sprintf`、`text/template`等方式进行。

字符串某些看似简单的操作，都可能引发堆内存分配和复制。
事实上，编译器和运行时也会采取非常规手段进行优化。

### 数组

数组是单一内存块，并无其他附加结构。

```go
  +---+---+---+----//---+----+
  | 0 | 1 | 2 | ... ... | 99 |   [100]int
  +---+---+---+----//---+----+
```


数组长度必须是 非负整型常量 表达式。
长度是数组类型组成部分。

初始化多维数组，仅第一维度允许用 `...`。
初始化复合类型，可省略元素类型标签。

内置函数 `len` 和 `cap` 都返回一维长度。
元素类型支持 `==`、`!=`，那么数组也支持。

支持数组指针直接访问。
支持获取元素指针。
值传递，复制整个数组。

```go
func main() {
	var d1 [unsafe.Sizeof(0)]int  // 编译期计算。
	var d2 [2]int                 // 元素类型相同，长度不同！
    
	// d1 = d2 // ~ cannot use [2]int as type [8]int in assignment
}
```



灵活初始化方式。

```go
func main() {
	var a [4]int                 // 元素自动初始化为零。
    
	b := [4]int{2, 5}            // 未提供初始值的元素自动初始化为 0。
	c := [4]int{5, 3: 10}        // 可指定索引位置初始化。
	d := [...]int{1, 2, 3}       // 编译器按初始化值数量确定数组长度。
	e := [...]int{10, 3: 100}    // 支持索引初始化，数组长度与此有关。
    
	fmt.Println(a, b, c, d, e)
}

/*

a: [0 0 0 0]
b: [2 5 0 0]
c: [5 0 0 10]
d: [1 2 3]
e: [10 0 0 100]

*/
```

```go
func main() {
	type user struct {
		id   int
		name string
	}

	_ = [...]user{    // 这里不能省略。
		{1, "zs"},    // 元素类型标签可省略。
		{2, "ls"},
	}
}
```

```go
func main() {
	// var x [2]int = {2, 5} // ~ syntax error

	// var y [...]int = [2]int{2, 5} //~ invalid use of [...] array (outside a composite literal)
}
```



多维数组依旧是连续内存存储。

```go
func main() {
    x := [...][2]int{
        {1, 2},
        {3, 4},
    }

    y := [...][3][2]int{
        {
            {1, 2}, 
            {3, 4},
            {5, 6},
        },
        {
            {10, 11},
            {12, 13},
            {14, 15},
        },
    }

    fmt.Println(x, len(x), cap(x))  // 2, 2
    fmt.Println(y, len(y), cap(y))  // 2, 2
}

/*

(gdb) x/4xg &x
0xc000066df0:   0x0000000000000001      0x0000000000000002
0xc000066e00:   0x0000000000000003      0x0000000000000004

(gdb) x/12xg &y
0xc000066e30:   0x0000000000000001      0x0000000000000002
0xc000066e40:   0x0000000000000003      0x0000000000000004
0xc000066e50:   0x0000000000000005      0x0000000000000006
0xc000066e60:   0x000000000000000a      0x000000000000000b
0xc000066e70:   0x000000000000000c      0x000000000000000d
0xc000066e80:   0x000000000000000e      0x000000000000000f

*/
```



相等比较，须元素支持。

```go
func main() {
	var a, b [2]int
	println(a == b)     // true

	c := [2]int{1, 2}
	d := [2]int{0, 1}
	println(c == d)     // false

	// var e, f [2]map[string]int
	// println(e == f) 	// ~ [2]map[string]int cannot be compared
}
```



#### 指针

注意区别不同指针及称谓。

数组指针：`&array`，指向整个数组的指针。
元素指针：`&array[0]`，指向某个元素的指针。
指针数组：`[...]*int{ &a, &b }`，元素是指针的数组。

```go
func main() {
	d := [...]int{ 0, 1, 2, 3 }

	var p  *[4]int = &d       // 数组指针。
	var pe *int    = &d[1]    // 元素指针。

    p[0] += 10          // 相当于 (*p)[0]
	*pe += 20

	fmt.Println(d)
}

// [10 21 2 3]
```

```go
func main() {
	a, b := 1, 2

	d := [...]*int{ &a, &b }   // 指针数组。
	*d[1] += 10                // d[1] 返回 b 指针。

	fmt.Println(d)
	fmt.Println(a, b)
}

// [0xc000014080 0xc000014088]
// 1 12
```



#### 复制

鉴于数组传递会整个复制，可使切片或指针代替。

```go

func val(d [3]byte) [3]byte {
	fmt.Printf("val: %p\n", &d)

	d[0] += 100
	return d
}

func ptr(p *[3]byte) *[3]byte {
	fmt.Printf("ptr: %p\n", p)

	p[0] += 200
	return p
}

func main() {
	 d := [...]byte{ 1, 2, 3 }
	d2 := d
	d3 := *(&d)

	fmt.Printf(" d: %p\n", &d)
	fmt.Printf("d2: %p\n", &d2)
	fmt.Printf("d3: %p\n", &d3)

	// ---------------------

	d4 := val(d)
	fmt.Printf("val.ret: %p\n", &d4)

	p := ptr(&d)
	fmt.Printf("val.ret: %p\n", p)	

	// ---------------------

	fmt.Printf("d: %v\n", d)
}

/*
      d: 0xc000014080
     d2: 0xc000014083
     d3: 0xc000014086

    val: 0xc00001409b
val.ret: 0xc000014098

    ptr: 0xc000014080
val.ret: 0xc000014080

      d: [201 2 3]
*/
```

### 切片

切片以指针引用底层数组片段，限定读写区域。类似胖指针，而非动态数组或数组指针。

```go
  +---------+            +---+---+----//---+----+
  |  array -|----------> | 0 | 1 | ... ... | 99 |
  +---------+            +---+---+----//---+----+
  |  len    |            
  +---------+            array
  |  cap    |
  +---------+
  
     header
     
```

```go
// runtime/slice.go

type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```



引用类型，未初始化为 nil。
基于数组、初始化值或 make 函数创建。

函数 `len` 返回元素数量，`cap` 返回容量。
仅能判断是否为 `nil`，不支持其他 `==`、`!=` 操作。

以索引访问元素，可获取底层数组元素指针。
底层数组可能在堆上分配。

```go
+---+---+---+---+---+---+---+---+---+---+   
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |   array
+---+---+---+---+---+---+---+---+---+---+   
        |               |       |           slice: [low : high : max]
        |<--- s.len --->|       |           
        |                       |           len = high - low
        |<------- s.cap ------->|           cap = max  - low
```

```go
func main() {
	a := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9} // array
    
	s := a[2:6:8]   

	fmt.Println(s)
	fmt.Println(len(s), cap(s))

    // 引用原数组。
	fmt.Printf("a: %p ~ %p\n", &a[0], &a[len(a) - 1])
	fmt.Printf("s: %p ~ %p\n", &s[0], &s[len(s) - 1])
}

/*

 s: [2 3 4 5], len = 4, cap = 6

 a: 0xc00007a000 ~ 0xc00007a048
 s: 0xc00007a010 ~ 0xc00007a028
 
*/
```



`slice.len`：限定索引或迭代读取数据范围。
`slice.cap`：重新切片（reslice）允许范围。

```go
func main() {
	a := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s := a[2:6:8] 

	// fmt.Println(s[5]) // ~ index out of range [5] with length 4

	for i, x := range s {
		fmt.Printf("s[%d]: %d\n", i, x)
	}
}

/*

# 切片引用原数组，此图只为方便理解。


+---+---+---+---+---+---+---+---+---+---+   
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |   a: [10]int
+---+---+---+---+---+---+---+---+---+---+   
        .                       .
        +---+---+---+---+---+---+
        | 2 | 3 | 4 | 5 |   |   |           s: a[2:6:8]  // a[起始索引:结束索引:切片容量]
        +---+---+---+---+---+---+
        0   1   2   3   


s[0]: 2
s[1]: 3
s[2]: 4
s[3]: 5

*/
```

```go
func main() {
	a := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s := a[2:6:8]

	s1 := s[0:2:4]
	fmt.Println(s1, len(s1), cap(s1))

	s2 := s[1:6]
	fmt.Println(s2, len(s2), cap(s2))
    
	// _ = s[1:7] // ~ slice bounds out of range [:7] with capacity 6
}

/*

# 切片的数据来自底层数组，重切片只是调整引用范围。
# 重切片受原 cap 限制，而非 len。


        +---+---+---+---+---+
        | 3 | 4 | 5 | 6 | 7 |     s2: s[1:6]
        +---+---+---+---+---+
        .                   .
    +---+---+---+---+---+---+
    | 2 | 3 | 4 | 5 |   |   |     s
    +---+---+---+---+---+---+
    .               .
    +---+---+---+---+
    | 2 | 3 |   |   |             s1: s[0:2:4]
    +---+---+---+---+


s1: [2 3],       len = 2, cap = 4
s2: [3 4 5 6 7], len = 5, cap = 5

*/
```



构造切片的常见方法。

```go
func main() {
	a := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s := a[:]

    fmt.Println(s, len(s), cap(s))
}

/*

 expr       slice                  len   cap  
----------+-----------------------+----+-----+---------------
 a[:]       [0 1 2 3 4 5 6 7 8 9]   10    10    a[0:len(a)]
 a[2:5]     [2 3 4]                  3     8
 a[2:5:7]   [2 3 4]                  3     5
 a[4:]      [4 5 6 7 8 9]            6     6    a[4:len(a)]
 a[:4]      [0 1 2 3]                4     10   a[0:4]
 a[:4:6]    [0 1 2 3]                4     6    a[0:4:6]

*/
```

```go
func main() {
    
    // 按初始化值，自动分配底层数组。
	s1 := []int{ 0, 1, 2, 3 }
	fmt.Println(s1, len(s1), cap(s1))

    // 自动创建底层数组。
	s2 := make([]int, 5)
	fmt.Println(s2, len(s2), cap(s2))

	s3 := make([]int, 0, 5)
	fmt.Println(s3, len(s3), cap(s3))
}

/*

s1: [0 1 2 3],   len = 4, cap = 4
s2: [0 0 0 0 0], len = 5, cap = 5
s3: [],          len = 0, cap = 5

*/
```



作为引用类型，初始化与否很重要。

编译器可能将零长度对象`（len == 0 && cap == 0）`指向固定全局变量` zerobase`。

```go
func p(s []int) {
	fmt.Printf("%t, %d, %#v\n", 
		s == nil,
		unsafe.Sizeof(s),
		(*reflect.SliceHeader)(unsafe.Pointer(&s)))
}

func main() {
    
    // 仅分配 header 内存，未初始化。
	var s1 []int
    
    // 初始化。
	s2 := []int{}
    
    // 调用 makeslice 初始化。
	s3 := make([]int, 0)

	p(s1)
	p(s2)
	p(s3)
}

/*

s1: true,  24, &SliceHeader{Data:0x0,      Len:0, Cap:0}
s2: false, 24, &SliceHeader{Data:0x5521d0, Len:0, Cap:0}
s3: false, 24, &SliceHeader{Data:0x5521d0, Len:0, Cap:0}

$ nm test | grep 5521d0
00000000005521d0 B runtime.zerobase

*/
```



切片不支持比较操作。

```go
func main() {
	s1 := []int{ 1, 2 }
	s2 := []int{ 1, 2, 3 }

	println(s1 == nil)

	// println(s1 == s2)
	//         ~~~~~~~~ invalid: slice can only be compared to nil
}
```

函数 `clear` 仅将 `[0, len)` 范围内的元素重置为零值（memclr），不修改相关属性。

```go
func main() {
	a := [...]int{1, 2, 10: 100}
	s := a[:2:6]

	fmt.Printf("%v, len:%d, cap:%d, ptr: %p\n", s, len(s), cap(s), &s[0])

	clear(s)
	fmt.Printf("%v, len:%d, cap:%d, ptr: %p\n", s, len(s), cap(s), &s[0])

	fmt.Printf("%v", a)
}


/*
    [1 2], len:2, cap:6, ptr: 0xc00007e120
    [0 0], len:2, cap:6, ptr: 0xc00007e120
    [0 0 0 0 0 0 0 0 0 0 100]
*/
```



#### 切片转换

切片可直接转回数组或数组指针。

```go
func main() {
	var a [4]int = [...]int{ 0, 1, 2, 3 }
	
	// array -> slice: 指向原数组
	var s  []int = a[:]
	println(&s[0] == &a[0])     // true

	// slice -> array: 复制底层数组（片段）
	a2 := [4]int(s)
	println(&a2[0] == &a[0])   // false

	// slice -> *array: 返回底层数组（片段）指针
	p2 := (*[4]int)(s)
	println(p2 == &a)         // true
	
}
```

#### 切片指针

可获取元素指针，但不能以切片指针访问元素。

```go
func main() {
	a := [...]int{ 0, 1, 2, 3 }
	s := a[:]

	p := &s      // 切片指针
	e := &s[1]   // 元素指针

	// 数组指针直接指向元素所在内存。
	// 切片指针指向 header 内存。

	// _ = p[1] // ~ invalid: cannot index p (variable of type *[]int)
	_ = (*p)[1]


	// 元素指针指向数组。

	*e += 100

	fmt.Println(e == &a[1])  // true
	fmt.Println(a)           // [0 101 2 3]
}
```



指针相关操作。

```go
func main() {
	var a [4]int = [...]int{ 0, 1, 2, 3 }

	// 基于数组指针创建切片。
	var p *[4]int = &a
	var s []int   = p[:]

	println(&s[2] == &a[2])   // true

	// 基于非数组指针创建切片。
	p2 := (*byte)(unsafe.Pointer(&a[2]))  // 元素指针
	var s2 []byte = unsafe.Slice(p2, 8)
	
	fmt.Println(s2)
}

// [2 0 0 0 0 0 0 0]
```

```go
func main() {
	var a [3]byte = [...]byte{ 'a', 'b', 'c' }
	var s []byte  = a[:]

	// 返回切片底层数组首个元素指针。
	println(unsafe.SliceData(s) == &a[0])          // true

	// 构建字符串，返回底层数组指针。
	var str string = unsafe.String(&s[0], len(s))
	println(unsafe.StringData(str) == &a[0])      // true
}
```



切片本身只是 3 个整数字段的小对象，可直接值传递。 
另外，编译器尽可能将底层数组分配在栈上，以提升性能。

```go

package main

//go:noinline
func sum(s []int) (n int) {
	for _, v := range s {
		n += v
	}

	return
}

func main() {
	s := []int{ 1, 2, 3 }
	println(sum(s))
}

/*

$ go build -o test
$ go tool objdump -S -s "main\.main" ./test

TEXT main.main(SB)
func main() {
        s := []int{ 1, 2, 3 }
  0x462c20              MOVQ $0x1, 0x20(SP)  ; array
  0x462c29              MOVQ $0x2, 0x28(SP)
  0x462c32              MOVQ $0x3, 0x30(SP)
        println(sum(s))                      ; header {
  0x462c3b              LEAQ 0x20(SP), AX    ;   .array = AX
  0x462c40              MOVL $0x3, BX        ;   .len   = BX
  0x462c45              MOVQ BX, CX          ;   .cap   = CX
  0x462c48              CALL main.sum(SB)    ; }
  0x462c4d              MOVQ AX, 0x18(SP)    ; sum.return

  0x462c52              CALL runtime.printlock(SB)
  0x462c57              MOVQ 0x18(SP), AX
  0x462c60              CALL runtime.printint(SB)
  0x462c65              CALL runtime.printnl(SB)
  0x462c6a              CALL runtime.printunlock(SB)
}

*/
```



#### **交错数组**

如果元素也是切片，可实现类似 交错数组（jagged array）功能。

不同于多维数组元素等长，交错数组的元素可以是长度不等的数组。

```go
func main() {
	s := [][]int{
		{1, 2},
		{10, 20, 30},
		{100},
	}

	s[1][2] += 100

	fmt.Println(s[1])
}

// [10 20 130]
```



#### **追加**

内置函数 `append `向切片追加数据。

数据追加到 `s[len] `处。
返回新切片对象，通常复用原内存。`s = append(s, ...)`
超出 `s.cap` 限制，即便底层数组尚有空间，也会重新分配内存，并复制数据。
新分配内存大小，通常是 `s.cap * 2`。更大切片，会减少倍数，避免浪费。

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func pslice(name string, p *[]int) {
	fmt.Printf("%s: %#v\n", 
		name, *(*reflect.SliceHeader)(unsafe.Pointer(p)))
}

func main() {
	a := [...]int{ 0, 1, 2, 3, 99:0}
	fmt.Printf("a: %p ~ %p\n", &a[0], &a[len(a) - 1])

	s := a[:4:8]
	pslice("s", &s)

	// -----------------------------

    // 未超出 s.cap 限制。
	s = append(s, []int{4, 5, 6}...)
	pslice("a", &s)

    // 超出! 新分配数组，复制数据。
	s = append(s, []int{7, 8}...)
	pslice("a", &s)

	// -----------------------------

	fmt.Println("a:", a[:len(s)])
	fmt.Println("s:", s)
}

/*

a: 0xc00007a000 ~ 0xc00007a318
s: {Data:0xc00007a000, Len:4, Cap:8}

a: {Data:0xc00007a000, Len:7, Cap:8}
a: {Data:0xc00001e080, Len:9, Cap:16}

a: [0 1 2 3 4 5 6 0 0]
s: [0 1 2 3 4 5 6 7 8]

*/
```

```go
func main() {
	m := make([]int, 3, 4)
	a := append(m, 1)
	b := append(m, 2)
	c := append(m, 3, 4)
	fmt.Println(m, a, b, c)
	//0,0,0
	//0,0,0,2
	//0,0,0,2
}
```



为切片预留足够容量（cap），可有效减少内存分配和复制。

```go
s := make([]int, 0, 10000)
```



#### 切片拷贝

在两个切片间复制数据：

允许指向同一数组。
允许目标区间重叠。
复制长度以较短（`len`）切片为准。

```go
func main() {
	s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	// 在同一底层数组的不同区间复制。
	src := s[5:8]
	dst := s[4:]

	n := copy(dst, src) 
	fmt.Println(n, s)

	// 在不同数组间复制。
	dst = make([]int, 6) 

	n = copy(dst, src)
	fmt.Println(n, dst)
}

/*

3 [0 1 2 3 5 6 7 7 8 9]
3 [6 7 7 0 0 0]

*/
```



还可直接从字符串中复制数据到字节切片。

```go
func main() {
	b := make([]byte, 3)
	n := copy(b, "abcde")
	
	fmt.Println(n, b)
}

// 3 [97 98 99]
```



若切片引用大数组，那么应考虑新建并复制。及时释放大数组，避免内存浪费。

```go
package main

import (
	"runtime"
	"time"
)

func main() {
	d := [...]byte{ 100<<20: 10 }
	
	// -- 1 --------------
	s := d[:2]

	// -- 2 --------------
	// s := make([]byte, 2)
	// copy(s, d[:])

	for i := 0; i < 5; i++ {
		time.Sleep(time.Second)
		runtime.GC()
	}

	runtime.KeepAlive(&s)
}

/*

$ go build && GODEBUG=gctrace=1 ./test

-- 1 ------------
gc 5 @6.188s 0%: ..., 100->100->100 MB, 200 MB goal, ..., 2 P (forced)

-- 2 ------------
gc 5 @5.827s 0%: ..., 0->0->0 MB, 4 MB goal, ..., 2 P (forced)

*/
```



#### 切片应用

切片除作为非正式 “数组指针” 外，还充当 动态数组 或 向量（vector）角色。 当然，更可实现一些常用数据结构。

```go
// FILO: 先进后出，栈。

type Stack []int

func NewStack() *Stack {
	s := make(Stack, 0, 10)
	return &s
}

func (s *Stack) Push(v int) {
	*s = append(*s, v)
}

func (s *Stack) Pop() (int, bool) {
	if len(*s) == 0 {
		return 0, false
	}

	x, n := *s, len(*s)
	
	v := x[n - 1]
	*s = x[:n - 1]

	return v, true
}

// ---------------------------

func main() {
	s := NewStack()
    
	// push
	for i := 0; i < 5; i++ {
		s.Push(i + 10)
	}
    
	// pop
	for i := 0; i < 7; i++ {
		fmt.Println(s.Pop())
	}
}

/*

14 true
13 true
12 true
11 true
10 true
0 false
0 false

*/
```

```go
// FIFO：先进先出，队列。

type Queue []int

func NewQueue() *Queue {
	q := make(Queue, 0, 10)
	return &q
}

func (q *Queue) Put(v int) {
	*q = append(*q, v)
}

func (q *Queue) Get() (int, bool) {
	if len(*q) == 0 {
		return 0, false
	}

	x := *q
	v := x[0]

	// copy(x, x[1:])
	// *q = x[:len(x) - 1]
    
    *q = append(x[:0], x[1:]...)  // 等同上两行。

	return v, true
}

// ---------------------------

func main() {
	q := NewQueue()
    
	// put
	for i := 0; i < 5; i++ {
		q.Put(i + 10)
	}
    
	// get
	for i := 0; i < 7; i++ {
		fmt.Println(q.Get())
	}
}

/*

10 true
11 true
12 true
13 true
14 true
0 false
0 false

*/
```



容量固定，且无需移动数据的环状队列。

```go
import (
	"log"
	"sync"
	"math/rand"
	"time"
)

type Queue struct {
	sync.Mutex
	data []int
	head int
	tail int
}

func NewQueue(cap int) *Queue {
	return &Queue{ data: make([]int, cap) }
}

func (q *Queue) Put(v int) bool {
	q.Lock()
	defer q.Unlock()

	if q.tail - q.head == len(q.data) {
		return false
	}

	q.data[q.tail % len(q.data)] = v
	q.tail++

	return true
}

func (q *Queue) Get() (int, bool) {
	q.Lock()
	defer q.Unlock()

	if q.tail - q.head == 0 {
		return 0, false
	}

	v := q.data[q.head % len(q.data)]
	q.head++

	return v, true
}

// ---------------------------

func main() {
	rand.Seed(time.Now().UnixNano())

	const max = 100000
	src := rand.Perm(max)        // 随机测试数据。
	dst := make([]int, 0, max)

	q := NewQueue(6)

	// ------------------------

	var wg sync.WaitGroup
	wg.Add(2)

	// put
	go func() {
		defer wg.Done()
		for _, v := range src {
			for {
				if ok := q.Put(v); !ok {
					continue
				}
				break
			}
		}
	}()

	// get
	go func() {
		defer wg.Done()
		for len(dst) < max {
			if v, ok := q.Get(); ok {
				dst = append(dst, v)
				continue
			}
		}
	}()

	wg.Wait()

	// 转换成数组进行比较。
	if *(*[max]int)(src) != *(*[max]int)(dst) {
		log.Fatalln("fail !!!")
	}

	log.Printf("%+v\n", *q)
}

// {data:[99011 52214 53425 10572 82360 78821] head:100000 tail:100000}
```





### 字典

字典（哈希表）存储键值对，一种使用频率极高的数据结构。

```go

   pointer              header
   
  +-------+          +-----------+
  |  map -|--------> |  hmap     |
  +-------+          +-----------+
                     |  ...      |
                     +-----------+       +-----//-----+
                     |  buckets -|-----> | ...    ... |   array
                     +-----------+       +-----//-----+
```



引用类型，无序键值对集合。
以初始化表达式或 make 函数创建。

主键须是支持 `==`、`!=` 的类型。
可判断字典是否为 `nil`，不支持比较操作。

函数 `len `返回键值对数量。
访问不存在主键，返回零值。
迭代以随机次序返回。

值不可寻址（not addressable），需整体赋值。
按需扩张，但不会收缩（shrink）内存。



```go
func p(m map[string]int) {
    fmt.Printf("%t, %x\n", m == nil, *(*uintptr)(unsafe.Pointer(&m)))
}

func main() {

    // 字典变量是指针类型。
    // 未初始化，那就是空指针。
    var m1 map[string]int
    m2 := map[string]int{}
    m3 := make(map[string]int, 0)
    
    p(m1)
    p(m2)
    p(m3)
}

/*
m1: true,  0
m2: false, c000064da0
m3: false, c000064d70
*/
```

```go
func main() {
	m1 := map[string]int {
		"a": 1,
		"b": 2,
	}

	// 省略复合类型标签。
	m2 := map[string]struct {
		id   int
		name string
	}{
		"a": { 1, "u1" },
		"b": { 2, "u2" },
	}

    // 分配足够初始容量，可减少后续扩张。
	m3 := make(map[string]int, 10)
	m3["a"] = 1

	fmt.Println(m1, len(m1))
	fmt.Println(m2, len(m2))
	fmt.Println(m3, len(m3))
}

/*

m1: map[a:1 b:2]            len = 2
m2: map[a:{1 u1} b:{2 u2}]  len = 2
m3: map[a:1] 1              len = 1

*/
```

```go
func main() {
	m1 := map[int]int{}
	m2 := map[int]int{}

	// _ = m1 == m2 // ~ invalid: map can only be compared to nil
}
```



建议以 `ok-idiom `访问主键，确认是否存在。
删除（`delete`）不存在键值，不会引发错误。
清空（`clear`）字典，不会收缩内存。
可对 `nil `字典读、删除，但写会引发 `panic`！

```go
func main() {
	m := map[string]int{}

	// 是否有 { "a": 0 } 存在?
    
	x := m["a"]
	fmt.Println(x)       // 0

	x, ok := m["a"]
	fmt.Println(x, ok)   // 0 false
	
	m["a"] = 0
	x, ok = m["a"]
	fmt.Println(x, ok)   // 0 true
}
```

```go
func main() {
    var m map[string]int  // nil
	
	_ = m["a"]
	delete(m, "a")

	// m["a"] = 0 // ~ panic: assignment to entry in nil map
}
```

```go
func main() {
    m := map[string]int {
    	"a": 1,
    	"b": 2,
    }

	println(len(m))	// 2

	clear(m)
	println(len(m))	// 0
}
```



迭代字典，以 “随机” 次序返回键值。

以 `struct{}` 为可忽略值类型。
随机效果依赖键值对数量和迭代次数。
可用作简易版随机挑选算法。

```go
func main() {
	m := make(map[int]struct{})

	for i := 0; i < 10; i++ {
		m[i] = struct{}{}
	}

	for i := 0; i < 4; i++ {
		for k, _ := range m {
			print(k, ",")
		}

		println()
	}
}

/*

7,2,3,4,5,9,0,1,6,8,
1,6,8,9,0,3,4,5,7,2,
0,1,6,8,9,2,3,4,5,7,
9,0,1,6,8,7,2,3,4,5,

*/
```



因 内联存储、键值迁移 及 内存安全 需要，字典被设计成不可寻址。 不能直接修改值成员（结构或数组），应整体赋值，或以指针代替。

内联存储有更好的性能，减少 GC 压力。

```go
func main() {
	type user struct {
		name string
		age  byte
	}

    // 指针。
	m := map[int]*user{
		1: &user{"u1", 19},
	}

    // 返回指针，修改外部对象。
	m[1].age = 20
}
```

```go
func main() {
	m := map[string]int{ 
        "a": 1 ,
    }

	m["a"]++    // m["a"] = m["a"] + 1
}
```



迭代期间，新增或删除操作是安全的，但无法控制次序。

```go
func main() {
	m := make(map[int]int)

	for i := 0; i < 10; i++ {
		m[i] = i + 10
	}

	for k := range m {
		if k == 5 {
			m[100] = 1000
		}

		delete(m, k)
		fmt.Println(k, m)
	}
}
```




运行时会对字典并发操作做出检测。

启用竞争检测（`data race`）查找此类问题。
使用 `sync.Map` 代替。

```go
package main

func main() {
	m := make(map[string]int)

	// write
	go func() {
		for {
			m["a"] += 1
		}
	}()

	// read
	go func() {
		for {
			_ = m["b"]
		}
	}()

	select {}
}

// fatal error: concurrent map read and map write
```



#### 字典性能优化

预分配足够空间有助于提升性能，减少因扩张引发的内存分配和数据迁移操作。

```go
package main

import (
	"testing"
)

const max = 10000

//go:noinline
func test(cap int) map[int]int {
	m := make(map[int]int, cap)
    
	for i := 0; i < max; i++ {
		m[i] = i
	}
    
	return m
}

// -------------------------------

func BenchmarkTest(b *testing.B) {
	for i := 0; i < b.N; i++ {
		test(0)
	}
}

func BenchmarkTestCap(b *testing.B) {
	for i := 0; i < b.N; i++ {
		test(max)
	}
}

/*

$ go test -bench . -benchmem

cpu: 12th Gen Intel(R) Core(TM) i7-12700
BenchmarkTest-20            3079            382905 ns/op          670679 B/op        161 allocs/op
BenchmarkTestCap-20         5991            204367 ns/op          322265 B/op         12 allocs/op
PASS

*/
```



字典内联存储有长度限制。如果超出，则重新分配内存并复制。

1.18 上限是 128 字节。

```go
package main

import (
	"testing"
)

const max = 100

//go:noinline
func test[T any](v T) map[int]T {
	m := make(map[int]T, max)

	for i := 0; i < max; i++ {
		m[i] = v
	}

	return m
}

// -------------------------------

func Benchmark128(b *testing.B) {
	for i := 0; i < b.N; i++ {
		test([128]byte{1,2,3})
	}
}

func Benchmark129(b *testing.B) {
	for i := 0; i < b.N; i++ {
		test([129]byte{1,2,3})
	}
}
/*
cpu: 12th Gen Intel(R) Core(TM) i7-12700
Benchmark128-20           196035              6070 ns/op           41036 B/op          3 allocs/op
Benchmark129-20           237856              5045 ns/op           19848 B/op        103 allocs/op
PASS
*/

```



扩张的内存不会因键值删除而收缩，必要时应新建字典。

```go
package main

import (
	"runtime"
	"time"
)

const max = 1000000

//go:noinline
func test[T any](v T) map[int]T {
	m := make(map[int]T, max)

	for i := 0; i < max; i++ {
		m[i] = v
	}

	return m
}


func main() {
	m := test([128]byte{1,2,3})
	// m := test([128 + 1]byte{1,2,3})

	runtime.GC()

	clear(m)

	for i := 0; i < 5; i++ {
		time.Sleep(time.Second)
		runtime.GC()
	}

	runtime.KeepAlive(&m)
}

/*

$ go build && GODEBUG=gctrace=1 ./test

--- 128 -----------------

内联：字典内存无法收缩，时间较短。

gc 2 @1.023s 0%: 293->293->293 MB, (forced)
gc 3 @2.061s 0%: 293->293->293 MB, (forced)
gc 4 @3.066s 0%: 293->293->293 MB, (forced)
gc 5 @4.073s 0%: 293->293->293 MB, (forced)
gc 6 @5.076s 0%: 293->293->293 MB, (forced)

--- 128 + 1 -------------

外联：外部对象被回收，时间较长。

gc 4 @0.335s 15%: 175->175->175 MB, (forced)
gc 5 @1.420s  4%: 175->175->38  MB, (forced)
gc 6 @2.452s  2%: 38->38->38    MB, (forced)
gc 7 @3.474s  1%: 38->38->38    MB, (forced)
gc 8 @4.497s  1%: 38->38->38    MB, (forced)

*/
```



### 结构

结构体将多个字段序列打包成一个复合类型。

直属字段名必须唯一。
支持自身指针类型成员。
可用 _ 忽略字段名。

支持匿名结构和空结构。
支持匿名嵌入其他类型。
支持为字段添加标签。

仅所有字段全部支持，才可做相等操作。
可用指针选择字段，但不支持多级指针。

字段的名称、类型、标签及排列顺序属类型组成部分。
除对齐外，编译器不会优化或调整布局。



顺序初始化，必须包含全部字段值。而命名初始化，不用全部字段，也无关顺序。 建议使用命名方式，即便后续新增字段或调整排列顺序也不受影响。



```go
type node struct {
	id    int
	value string
	next  *node        // 自身指针类型。
}

func main() {

    // 顺序提供所有字段值，或全部忽略。
	// n := node{ 1, "a" } // ~ too few values in struct literal
    
    // 按字段名初始化。
	n := node{
		id   : 2,
		value: "abc",  // 注意结尾逗号
	}
}
```

```go
func main() {
    
    // 匿名结构
	user := struct {
		id   int
		name string
	}{
		id  : 1,
		name: "user1",
	}

	fmt.Printf("%+v\n", user)
    
    // --------------------

	var color struct {
		r int
		g int
		b int
	}

	color.r = 1
	color.g = 2
	color.b = 3

	fmt.Printf("%+v\n", color)
}

// {id:1 name:user1}
// {r:1 g:2 b:3}
```

```go
func main() {

	type file struct {
		name string
		attr struct {
			owner int
			perm  int
		}
	}

	f := file{
		name: "test.dat",

		// 因缺少类型标签，无法直接初始化。

		// attr: {          
		//    owner: 1,
		//    perm:  0755,
		// },
		// ~~~~~~~~~ missing type in composite literal
	}

	f.attr.owner = 1
	f.attr.perm = 0755

	fmt.Println(f)
}
```



**相等操作限制：**

全部字段支持。
内存布局相同，但类型不同，不能比较。
布局相同（含字段名、标签等）的匿名结构视为同一类型。

```go
func main() {
	type data struct {
		x int
		y map[string]int
	}

	d1 := data{ x: 100 }
	d2 := data{ x: 100 }

	// _  = d1 == d2 // ~ invalid: struct containing map cannot be compared
}
```

```go
func main() {
    // 类型不同
	type data1 struct {
		x int
	}
	type data2 struct {
		x int
	}

	d1 := data1{ x: 100 }
	d2 := data2{ x: 100 }

	// _  = d1 == d2 // ~ invalid: mismatched types data1 and data2
}
```

```go
func main() {
    // 匿名结构类型相同的前提是
    // 字段名、字段类型、标签和排列顺序都相同。
	d1 := struct {
		x int
		s string
	}{ 100, "abc" }
	var d2 struct {
		x int
		s string
	}

	d2.x = 100
	d2.s = "abc"

	println(d1 == d2) // true
}
```



不能以多级指针访问字段成员。

```go

func main() {
	type user struct {
		name string
		age  int
	}

    // 直接返回指针。
	p := &user{
		name: "u1",
		age:  20,
	}

	p.name = "u2"
	p.age++

    // 二级指针。
	p2 := &p
	
	// p2.age++ // ~ p2.age undefined (type **user has no field or method age)
}
```



#### 空结构

空结构（`struct{}`）是指没有字段的结构类型，常用于值可被忽略的场合。 无论是其自身，还是作为元素类型，其长度都为零，但这不影响作为实体存在。

```go
func main() {
	var a struct{}
	var b [1000]struct{}

	s := b[:]

	println(unsafe.Sizeof(a), unsafe.Sizeof(b))  // 0, 0
	println(len(s), cap(s))                      // 1000, 1000
}
```

```go
func main() {
    // 结束通知，值无关紧要。
	done := make(chan struct{})

	go func(){
		time.Sleep(time.Second)
		close(done)
	}()

	<- done
}
```

```go
func main() {
	nul := struct{}{}
	users := make(map[int]struct{})

	for i := 0; i < 100; i++ {
		users[i] = nul
	}

	// 利用字典随机选人，值不需要。
	for k, _ := range users {
		println(k)
		break
	}
}
```



#### 标签

标签（`tag`）不是注释，而是对字段进行描述的元数据。

不是数据成员，却是类型的组成部分。
内存布局相同，允许显式类型转换。

```go
func main() {
	type user struct {
		id int `id`
	}
	type user2 struct {
		id int `uid`
	}

	u1 := user{1}
	u2 := user2{2}

	// 类型不同。
	// _ = u1 == u2 // ~ mismatched types user and user2

	// 内存布局相同，支持转换。
	u1 = user(u2)
	fmt.Println(u1)
}
```



运行期，可用反射获取标签信息。常被用作格式校验，数据库关系映射等。

```go
func main() {
	type User struct {
		id   int     `field:"uid"  type:"integer"`
		name string  `field:"name" type:"text"`
	}

	t := reflect.TypeOf(User{})

    for i := 0; i < t.NumField(); i++ {
        f := t.Field(i)
        fmt.Println(f.Name, f.Tag.Get("field"), f.Tag.Get("type"))
    }	
}

// id: uid  integer
// name: name text
```



#### 匿名字段

所谓匿名字段（anonymous field），是指没有名字，仅有类型的字段。也被称作嵌入字段、嵌入类型。

隐式以类型名为字段名。
嵌入类型与其指针类型隐式字段名相同。
像直属字段那样直接访问嵌入类型成员。

```go
type Attr struct {
	perm int
}

type File struct {
	Attr            // Attr: Attr
	name string
}

func main() {
	f := File {
		name: "test.dat",
		// 必须显式名称初始化。

		// perm: 0644, // ~ unknown field 'perm' in struct literal of type File

		Attr: Attr{
			perm: 0644,
		},
	}

	// 像直属字段访问嵌入字段。
	fmt.Printf("%s, %o\n", f.name, f.perm)
}
```



嵌入其他包里的类型，隐式字段名字不含包名。

```go
type data struct {
	os.File
}

func main() {
	d := data{
		File: os.File{},
	}

	fmt.Printf("%#v\n", d)
}
```



除 **接口指针** 和 **多级指针** 以外的任何 **命名类型** 都可作为匿名字段。

```go
type data struct {
	*int              // int: *int（星号不是名字组成部分）
	// int 			  // ~ int redeclared

    fmt.Stringer
	// *fmt.Stringer  // ~ embedded field type cannot be a pointer to an interface
}

func main() {

	_ = data{
		int: nil,
		Stringer: nil,
	}
}
```



#### **匿名字段重名**

直属字段与嵌入类型成员存在重名问题。

编译器优先选择直属命名字段，或按嵌入层次逐级查找匿名类型成员。 如匿名类型成员被外层同名遮蔽，那么必须用显式字段名。

```go
type File struct {
	name []byte
}

type Data struct {
	File
	name string    // 和 File.name 重名。
}

func main() {
	d := Data{
		name: "data",
		File: File{ []byte("file") },
	}
    
	d.name = "data2"               // 优先选择直属命名字段。
	d.File.name = []byte("file2")  // 显式字段名。
    
	fmt.Println(d.name, d.File.name)
}
```



如多个相同层次的匿名类型成员重名，就只能用显式字段名，因为编译器无法确定目标。

```go
type File struct {
	name string
}

type Log struct {
	name string
}

type Data struct {
	File
	Log
}

func main() {
	d := Data{}
	
	// d.name = "name" // ~ ambiguous selector d.name

	d.File.name = "file"
	d.Log.name = "log"
}
```



Go 并非传统意义上的 OOP 语言（封装、继承和多态），仅实现了最小机制。
 匿名嵌入是 **组合**（Composition），而非 **继承**（Inheritance）。
 结构虽然承载了 class 功能，但无法 **多态**（Polymorphism），只能以方法集配合接口实现。



#### 结构内存

不管结构包含多少字段，其内存总是一次性分配，各字段（含匿名字段成员）在相邻地址空间按定义顺序（含对齐）排列。当然，对于引用类型、字符串和指针，结构内存中只包含其基本（头部）数据。

借助 `unsafe` 相关函数，输出所有字段的偏移量和长度。

```go
import (
    "fmt"
  . "unsafe"
)

type Point struct {
	x, y int
}

type Value struct {
	id    int    
	name  string 
	data  []byte 
	next  *Value 
	Point        
}

func main() {
	v := Value{
		id:    1,
		name:  "test",
		data:  []byte{1, 2, 3, 4},
		Point: Point{x: 100, y: 200},
	}

	fmt.Printf("%p ~ %p, size: %d, align: %d\n",
		&v, Add(Pointer(&v), Sizeof(v)), Sizeof(v), Alignof(v))

	s := "%p, %d, %d\n"

	fmt.Printf(s, &v.id,   Offsetof(v.id),   Sizeof(v.id))
	fmt.Printf(s, &v.name, Offsetof(v.name), Sizeof(v.name))
	fmt.Printf(s, &v.data, Offsetof(v.data), Sizeof(v.data))
	fmt.Printf(s, &v.next, Offsetof(v.next), Sizeof(v.next))
	fmt.Printf(s, &v.x,    Offsetof(v.x),    Sizeof(v.x))
	fmt.Printf(s, &v.y,    Offsetof(v.y),    Sizeof(v.y))
}

/*

     0xc00005c0a0 ~ 0xc00005c0e8, size: 72, align: 8

     field   address        offset   size
     ------+--------------+--------+--------+----------------------
     id      0xc00005c0a0   0        8        int
     name    0xc00005c0a8   8        16       string {ptr, len}
     data    0xc00005c0b8   24       24       slice  {ptr, len, cap}
     next    0xc00005c0d0   48       8        pointer
     x       0xc00005c0d8   56       8        int
     y       0xc00005c0e0   64       8        int


    +0 +----------+  0xc00005c0a0
       | id       |
     8 +----------+  0xc00005c0a8
       | name.ptr |
    16 +----------+
       | name.len |
    24 +----------+  0xc00005c0b8
       | data.ptr |
    32 +----------+
       | data.len |
    40 +----------+
       | data.cap |
    48 +----------+  0xc00005c0d0
       | next     |
    56 +----------+  0xc00005c0d8
       | Point.x  |
    64 +----------+  0xc00005c0e0
       | Point.y  |
    72 +----------+  0xc00005c0e8
     
*/

```



对齐以所有字段中最长的基础类型宽度为准。编译器这么做的目的，即为了最大限度减少读写所需指令，也因为某些架构平台自身的要求。

```go
import (
    "fmt"
  . "unsafe"
)

func main() {
	v1 := struct {
		a byte
		b byte
		c int32
	}{}
    
	v2 := struct {
		a byte
		b byte
	}{}
    
	v3 := struct {
		a byte
		b []int
		c byte
	}{}
    
	fmt.Printf("v1: %d, %d\n", Alignof(v1), Sizeof(v1))
	fmt.Printf("v2: %d, %d\n", Alignof(v2), Sizeof(v2))
	fmt.Printf("v3: %d, %d\n", Alignof(v3), Sizeof(v3))
}

/*

v1: 4, 8
v2: 1, 2
v3: 8, 40


(gdb) x/2xw &v1
0xc000088e88:	0x00000201	0x00000003

+---+---+---+---+---+---+---+---+
| a | b | ? | ? |       c       |
+---+---+---+---+---+---+---+---+
|<----- 4 ----->|<----- 4 ----->|

(gdb) x/2xb &v2
0xc000088e86:	0x01	0x02

+---+---+
| a | b |
+---+---+
0   1   2

(gdb) x/5xg &v3
0xc000088f48:	0x0000000000000001	0x000000c000088e90
0xc000088f58:	0x0000000000000003	0x0000000000000003
0xc000088f68:	0x0000000000000002

+---+-------+-----------+-----------+-----------+---+-------+
| a |  ...  |   b.ptr   |   b.len   |   b.cap   | c |  ...  |
+---+-------+-----------+-----------+-----------+---+-------+
|<--- 8 --->|<--- 8 --->|<--- 8 --->|<--- 8 --->|<--- 8 --->|

*/
```

```go
func main() {
	d := struct {
		a [3]byte
		x int32
	}{
		a: [3]byte{1, 2, 3},
		x: 100,
	}
	fmt.Println(d)
}

/*

+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | ? |      100      |
+---+---+---+---+---+---+---+---+
|<----- 4 ----->|<----- 4 ----->|

*/
```



#### 空结构的内存

如果空结构是最后一个字段，那么将其当做长度 `1 `的类型，避免越界。
其他零长度对象（`[0]int`）类似。

```go
import (
	"fmt"
  . "unsafe"
)

func main() {
	v := struct {
		a struct{}
		b int
		c struct{}
	}{}

	fmt.Printf("%p ~ %p, size: %d, align: %d\n",
		&v, Add(Pointer(&v), Sizeof(v)), Sizeof(v), Alignof(v))

	s := "%p, %d, %d\n"

	fmt.Printf(s, &v.a, Offsetof(v.a), Sizeof(v.a))
	fmt.Printf(s, &v.b, Offsetof(v.b), Sizeof(v.b))
	fmt.Printf(s, &v.c, Offsetof(v.c), Sizeof(v.c))
}

/*

     0xc000018070 ~ 0xc000018080, size: 16, align: 8
     
     field   address       offset    size
     ------+--------------+---------+---------
     a       0xc000018070  0         0
     b       0xc000018070  0         8
     c       0xc000018078  8         0


    +0 +----------+  0xc000018070
       |   a, b   |
     8 +----------+  0xc000018078
       |   c      |
    16 +----------+  0xc000018080
     
*/
```



如仅有一个空结构字段，那么按 `1` 对齐，只不过长度为 `0`，且指向 `zerobase `变量。

```go
func main() {
	v := struct {
		a struct{}
	}{}

	fmt.Printf("%p, %d, %d\n", &v, Sizeof(v), Alignof(v))
}

/*

0x5521d0, size: 0, align: 1

$ nm ./test | grep 5521d0
00000000005521d0 B runtime.zerobase

*/
```



### 指针

不能将内存 **地址** 与 **指针** 混为一谈。

地址是内存中每个字节单元的唯一编号，而指针则是实体。指针需分配内存空间，相当于一个专门用来保存地址的整型变量。

```go

            x: int          p: *int
   -------+---------------+--------------+-------
     ...  | 100           | 0xc000000000 |  ...    memory
   -------+---------------+--------------+-------
          0xc000000000    0xc000000008             address
```

```go
func main() {
	var x int
	var p *int = &x

	*p = 100

	println(p, *p)
    // 0xc000067f28 100
}
```



取址运算符 `&` 用于获取目标地址。
指针运算符 `*` 间接引用目标对象。
二级指针 `**T`，含包名则写成 `**package.T`。

指针默认值 `nil`，支持 `==`、`!=` 操作。
不支持指针运算，可借助 `unsafe `变相实现。

```go
func main() {

	// 空指针也会分配内存。
	var p *int
	println(unsafe.Sizeof(p))  // 8
/*
   p: *int
   +--------------+
   | 0            |
   +--------------+
   0xc000000008
*/
    
	var x int
/*
   p: *int                x: int
   +---------------+      +---------------+
   | 0             |      | 0             |
   +---------------+      +---------------+
   0xc000000008           0xc000000000
*/
    
    
	// 二级指针，指针的指针。
	var pp **int = &p
	*pp = &x
/*
                               +--------- *p -----------+
                               |                        | 
   pp: **int              p    |                 x      v 
   +---------------+      +---------------+      +---------------+
   | 0xc000000008 -|----->| 0xc000000000 -|----->| 0             |
   +---------------+      +---------------+      +---------------+
   0xc000000010           0xc000000008   ^       0xc000000000   ^
         |                               |                      |
         +-------- *pp ------------------+                      |
         |                                                      |
         +-------- **pp ----------------------------------------+
*/
    
    
	*p = 100
    **pp += 1
	println(**pp, *p, x)   // 101, 101, 101
}
```



并非所有对象都能进行取址操作。

```go
func main() {
	m := map[string]int{
		"a": 1,
	}

	// _ = &m["a"] // ~ invalid: cannot take address of m["a"]
}
```



支持相等运算符，但不能做加减法运算，不能做类型转换。
如两个指针指向同一地址，或都为 `nil`，那么它们相等。

```go
func main() {
	var b byte
	var p *byte = &b

	// p++ // ~ invalid: p++ (non-numeric type *byte) 
	// n := (*int)(p) // ~ cannot convert *byte to *int
}
```

```go
func main() {
	var p1, p2 *int
	println(p1 == p2)   // true

	var x int
	p1, p2 = &x, &x
	println(p1 == p2)   // true

	var y int
	p2 = &y
	println(p1 == p2)   // false
}
```



指针没有专门指向成员的 `->` 运算符，统一使用 `.` 选择表达式。

```go
func main() {
	a := struct {
		x int
	}{ 100 }
    
	p := &a
	p.x += 100
    
	println(p.x) // 200
}
```



零长度（0 byte）对象指针是否相等，与版本及编译优化有关，不过肯定不等于`nil`。

```go
func main() {
	var a, b struct{}
	var c [0]int

	pa, pb, pc := &a, &b, &c

	println(pa, pb, pc)
	println(pa == nil || pc == nil)
	println(pa == pb)              
}

/*

$ go build -gcflags "-N -l" && ./test

0xc00009cf56 0xc00009cf56 0xc00009cf50
false
true

$ go build && ./test

0xc000088f70 0xc000088f70 0xc000088f70
false
false

*/
```



借助 `unsafe`实现指针转换和运算，须自行确保内存安全。

●普通指针：`*T`，包含类型信息。
●通用指针：`Pointer`，只有地址，没有类型。
●指针整数：`uintptr`，足以存储地址的整数。

```go
func main() {
	d := [...]int{1, 2, 3}
	p := &d

	// *[3]int --> *int
    p2 := (*int)(unsafe.Pointer(p))

	// p2++
	p2 = (*int)(unsafe.Add(unsafe.Pointer(p2), unsafe.Sizeof(p[0])))
	*p2 += 100

	fmt.Println(d)  // [1 102 3]
}
```

普通指针（`*T`）和通用指针（`Pointer`）都能构成引用，影响垃圾回收。
而 `uintptr `只是整数，不构成引用关系，无法阻止垃圾回收器清理目标对象。



## 方法

方法（`method`）是与对象实例（`instance`）相绑定的特殊函数。

方法是面向对象编程的基本概念，用于维护和展示对象自身状态。对象是内敛的，每个实例都有各自不同的独立特征，以属性和方法来对外暴露。普通函数专注于算法流程，接收参数完成逻辑运算，返回结果并清理现场。也就是说，方法有持续性状态，而函数通常没有。

前置接收参数（`receiver`），代表方法所属类型。
可为当前包内除接口和指针以外的任何类型定义方法。
不支持静态方法（`static method`）或关联函数。
不支持重载（`overload`）。

```go
func (int) test() {} // ~ cannot define new methods on non-local type int

// -------------------------
type N *int
func (N) test() {} // ~ invalid receiver type N (pointer or interface type)

// -------------------------
type M int
func  (M) test(){}
func (*M) test(){} // ~ redeclared in this block
```



对接收参数命名无限制，按惯例选用简短有意义的名称。
如方法内部不引用实例，可省略接收参数名，仅留类型。

```go
type N int

func (n N) toString() string {
	return fmt.Sprintf("%#x", n)
}

func (N) test() {      // 省略接收参数名。
	println("test")
}

// -----------------------------

func main() {
	var a N = 25
	println(a.toString())
}
```



接收参数可以是指针类型，调用时据此决定是否复制（pass by value）。

注意区别：
不能为指针和接口定义方法，是说类型 `N` 本身不能是接口和指针。
这与作为参数列表成员的`receiver *N` 意思完全不同。

方法本质上就是特殊函数，接收参数无非是其第一参数。
只不过，在某些语言里它是隐式的 `this` 。

```go
type N int

func (n N) copy() {
	fmt.Printf("%p, %v\n", &n, n)
}

func (n *N) ref() {
	fmt.Printf("%p, %v\n", n, *n)
}

func main() {
	var a N = 25
	fmt.Printf("%p\n", &a)  // 0xc000014080

	a.copy()                // 0xc000014088, 25
	N.copy(a)               // 0xc0000140a0, 25

	a++
    
	a.ref()                 // 0xc000014080, 26
	(*N).ref(&a)            // 0xc000014080, 26
}
```



编译器根据接收参数类型，自动在值和指针间转换。
Go 文档不建议一个结构同时有值接收者和指针接收者

```go

type N int
func (n N) copy() {}
func (n *N) ref() {}

func main() {
	var a N = 25
	var p *N = &a

	a.copy()
    a.ref()     // (*N).ref(&a)

    p.copy()    // N.copy(*p)
	p.ref()
}

/*

$ go build -gcflags "-N -l"
$ go tool objdump -S -s "main\.main" ./test

TEXT main.main(SB)
func main() {
        var a N = 25
  0x455254              MOVQ $0x19, 0x8(SP)     
        var p *N = &a
  0x45525d              LEAQ 0x8(SP), CX        
  0x455262              MOVQ CX, 0x18(SP)       
        a.copy()
  0x455267              MOVQ 0x8(SP), AX        
  0x45526c              CALL main.N.copy(SB)    
        a.ref()
  0x455271              LEAQ 0x8(SP), AX        ; &a
  0x455276              CALL main.(*N).ref(SB)  
        p.copy()
  0x45527b              MOVQ 0x18(SP), CX       
  0x455282              MOVQ 0(CX), AX          
  0x45528a              CALL main.N.copy(SB)    
        p.ref()
  0x45528f              MOVQ 0x18(SP), AX       
  0x455294              CALL main.(*N).ref(SB)  
}

*/
```



不能以多级指针调用方法。

```go
type N int
func (n N) copy() {}
func (n *N) ref() {}

func main() {
	var a N = 25
	var p *N = &a

	p2 := &p
	// p2.copy() // ~ p2.copy undefined
	// p2.ref() // ~ p2.ref undefined

	(*p2).copy()
	(*p2).ref()
}
```



**如何确定接收参数（receiver）类型？**

修改实例状态，用 `*T`。
不修改状态的小对象或固定值，用 `T`。
大对象用 `*T`，减少复制成本。
引用类型、字符串、函数等指针包装对象，用 `T`。
含 Mutex 等同步字段，用 `*T`，避免因复制造成锁无效。
其他无法确定的，都用 `*T`。

### **匿名嵌入**

像访问匿名类型成员那样调用其方法，由编译器负责查找。

```go
type data struct {
	sync.Mutex
	buf [1024]byte
}

func main() {
	d := data{}
	d.Lock() // sync.(*Mutex).Lock()
	defer d.Unlock()
}
```



同名遮蔽。利用这种特性，实现类似覆盖（override）操作。

和匿名字段访问类似，按最小深度优先原则。
如两个同名方法深度相同，那么编译器无法作出选择（ambiguous selector），需显式指定。

```go
type E struct{}

type T struct {
	E
}

func   (E) toString() string { return "E" }
func (t T) toString() string { return "T." + t.E.toString() }

// ----------------------------------

func main() {
	var t T
	println(t.toString())    // T.E
	println(t.E.toString())  // E
}
```



同名，但签名不同的方法。

```go
type E struct{}

type T struct {
	E
}

func (E) toString() string { return "E" }
func (T) toString(s string) string { return "T: " + s }

// ------------------------------------------

func main() {
	var t T

	// println(t.toString()) // ~ not enough arguments in call to t.toString

    // 选择深度最小的方法。
	println(t.toString("abc"))   // T: abc

	// 明确目标。
	println(t.E.toString())      // E
}
```



匿名类型的方法只能访问自己的字段，对外层一无所知。

```go
type E struct {
	x int
}

type T struct {
	x string	
	E
}

func (e *E) do() {
	e.x = 100
	println(e.x)
}

func (t *T) do() {
	t.x = "abc"
	println(t.x)
}

// ------------------------------------------

func main() {
	var t T

	t.do()     // abc
	t.E.do()   // 100
}

/*

可以看出 t.E.do 的 receiver 仅限于自己那段内存。
即便把 E 作第一字断，各自方法内的寻址偏移也指向自己的内存区域。

$ go build -gcflags "-l"
$ go tool objdump -S -s "main\.main" ./test

func main() {
        var t T                                t: T
  0x455314    MOVQ $0x0, 0x8(SP)        0x8 +---------+
  0x45531d    MOVUPS X15, 0x10(SP)          | T.x.ptr |
                                       0x10 +---------+
        t.do()     // abc                   | T.x.len |
  0x455323    LEAQ 0x8(SP), AX         0x18 +---------+
  0x455328    CALL main.(*T).do(SB)         | T.E.x   |
                                            +---------+
        t.E.do()   // 100
  0x45532d    LEAQ 0x18(SP), AX
  0x455332    CALL main.(*E).do(SB)
}

*/
```



### 方法集

类型有个与之相关的方法集合（method set），这决定了它是否实现某个接口。
根据接收参数（receiver）的不同，可分为 `T` 和 `*T` 两种视角。

`T.set = T`
`*T.set = T + *T`

```go
type T int

func  (T) A() {}   // 导出成员，否则反射无法获取。
func  (T) B() {}
func (*T) C() {}
func (*T) D() {}

func show(i interface{}) {
	t := reflect.TypeOf(i)
	for i := 0; i < t.NumMethod(); i++ {
		println(t.Method(i).Name)
	}	
}

func main() {
	var n T = 1
	var p *T = &n

	show(n)         //  T = [A, B]
	show(p)         // *T = [A, B, C, D]
}
```



直接方法调用，不涉及方法集。编译器自动转换所需参数（receiver）。
而转换（赋值）接口（interface）时，须检查方法集是否完全实现接口声明。

```go
type Xer interface {
	B()
	C()
}

type T int

func  (T) A() {}
func  (T) B() {}
func (*T) C() {}
func (*T) D() {}

func main() {
	var n T = 1

    // 方法调用：不涉及方法集。
	n.B()         
	n.C()

    // 接口：检查方法集。
	// var x Xer = n // ~ T does not implement Xer (C method has pointer receiver)

	var x Xer = &n
	x.B()
	x.C()
}
```



首先，接口会复制对象，且复制品 不能寻址（unaddressable）。
如 `T` 实现接口，透过接口调用时，`receiver `可被复制，却不能获取指针（`&T`）。
相反，`*T` 实现接口，目标对象在接口以外，无论是取值还是复制指针都没问题。
这就是方法集与接口相关，且 `T = T`，`*T = T + *T `的原因。

除直属方法外，列表里还包括匿名类型（`E`）的方法。

`T{ E } = T + E`
`T{ *E } = T + E + *E`
`*T{ E | *E } = T + *T + E + *E`

```go
type E int

func  (E) V() {}
func (*E) P() {}

func show(i interface{}) {
	t := reflect.TypeOf(i)
	for i := 0; i < t.NumMethod(); i++ {
		println("  ", t.Method(i).Name)
	}	
}

func main() {
	println("T{ E }")
	show(struct{E}{})

	println("T{ *E }")
	show(struct{*E}{})

	println("*T{ E }")
	show(&struct{E}{})

	println("*T{ *E }")
	show(&struct{*E}{})
}

//  T{  E }: V
//  T{ *E }: P, V
// *T{  E }: P, V
// *T{ *E }: P, V
```



**别名扩展**

通过类型别名，对方法集进行分类，更便于维护。或新增别名，为类型添加扩展方法。

```go
type X int
func (*X) A() { println("X.A") }

type Y = X                           // 别名
func (*Y) B() { println("Y.B") }     // 扩展方法

func main() {
	var x X
	x.A()
	x.B()

	var y Y
	y.A()
	y.B()
}

```



通过反射，可以看到 “扩展” 被合并的效果。

```go
type X int
func (*X) A() { println("X.A") }

type Y = X
func (*Y) B() { println("Y.B") }

func main() {
	var n X
	t := reflect.TypeOf(&n)

	for i := 0; i < t.NumMethod(); i++ {
		fmt.Println(t.Method(i))
	}
}

// A: func(*main.X)
// B: func(*main.X)
```



需要注意，不同包的类型可定义别名，但不能定义方法。

```go
type X = bytes.Buffer

// func (*X) B() { println("X.b") }  // ~ cannot define new methods on non-local type
```



### 方法值

和函数一样，方法除直接调用外，还可赋值给变量，或作为参数传递。
依照引用方式不同，分为表达式（expression）和 值（value） 两种。

```go
type N int

func (n *N) ref() {
	fmt.Printf("%p, %v\n", n, *n)
}

func main() {
	var n N = 100

	// expression
	// var e = (*N).ref
	var e func(*N) = (*N).ref
	e(&n)

	// value
    // var v = n.ref
	var v func() = n.ref
	v()
}

// 0xc000014080, 100
// 0xc000014080, 100
```



表达式（expr）很好理解，将方法还原为普通函数，显式传递接收参数（receiver）。
而方法值（value）似乎打包了接收参数和方法，导致签名有所不同。

精简代码，看看具体如何实现。

```go
type N int
func (n N) copy() {
	println(n)
}
func (n *N) ref() {
	println(*n)
}
func test(f func()) {
	f()
}

func main() {
	var n N = 100
	var v func() = n.copy
	n++
	n.copy() // 101

	v()      // 100
	test(v)  // 100
}

/*
$ go build -gcflags "-N -l"
$ go tool objdump -S -s "main\.main" ./test

func main() {

    var n N = 100
    0x4552b4     MOVQ $0x64, 0x8(SP)   
    
    var v func() = n.copy
    0x4552c3     LEAQ 0x10(SP), CX               
    0x4552cf     LEAQ N.copy-fm(SB), DX     
    0x4552d6     MOVQ DX, 0x10(SP)               
    0x4552dd     MOVQ 0x8(SP), DX         0x0 +-------------+
    0x4552e2     MOVQ DX, 0x18(SP)            |             |
    0x4552e7     MOVQ CX, 0x20(SP)        0x8 +-------------+
                                              | n = 100     |
    n++                                  0x10 +-------------+     
    0x4552ec     MOVQ 0x8(SP), CX             | copy-fm     |
    0x4552f1     LEAQ 0x1(CX), AX        0x18 +-------------+     
    0x4552f5     MOVQ AX, 0x8(SP)             | 100         |
                                         0x20 +-------------+
    n.copy() // 101                           | ptr -> 0x10 |    
    0x4552fa     CALL N.copy(SB)              +-------------+
    
    v()      // 100
    0x4552ff     MOVQ 0x20(SP), DX
    0x455304     MOVQ 0(DX), CX
    0x455307     CALL CX          ; copy-fm
    
    test(v)  // 100
    0x455309     MOVQ 0x20(SP), AX       
    0x45530e     CALL test(SB)      
}
*/
```



方法值：`funcval { method-fm, receiver-copy }`。
换成`n.ref`，无非是复制 `*N` 而已。

```go
TEXT main.N.copy-fm(SB) <autogenerated>

    0x45535d     MOVQ 0x8(DX), AX      // receiver
    0x455361     MOVQ AX, 0x8(SP)
    0x455366     CALL N.copy(SB)


TEXT main.test(SB)

    0x455277     MOVQ 0(AX), CX       // N.copy-fm
    0x45527a     MOVQ AX, DX          // DX
    0x45527d     CALL CX              
```



对于空指针（`nil`），注意内存安全。

```go
type N int
func (n N) copy() {}
func (n *N) ref() {}

// -----------------------------

func main() {
	var p *N

	p.ref()         // value
	(*N)(nil).ref() // value
	(*N).ref(nil)   // expression
    
	// p.copy() // ~ runtime error: invalid memory address or nil pointer dereference
    // N.copy(*p) // ~ runtime error: invalid memory address or nil pointer dereference
}
```

