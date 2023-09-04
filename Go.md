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
