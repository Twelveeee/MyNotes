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

