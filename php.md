# 基础语法

## 初步语法

PHP是一种运行在服务器端的脚本语言，可以嵌入到HTML中。（HTML是通过浏览器解析，PHP是通过PHP引擎解析,那么嵌入到里面以后如何区分什么时候通过什么来解析？）

PHP代码标记：可通过多种标记来区分PHP脚本
ASP标记` <% php 代码 %>；` 
短标记 以上两种基本弃用，如使用，需在配置文件中开启
脚本标记 ：也不经常用

```php+HTML
<script language="php">
	echo 'hello world';	
</script>
```

标准标记：

```php+HTML
<？php
	echo 'hello world';	
?>
```

PHP注释：
行注释：
`//`或`# `
块注释：`/* */`
PHP语句分隔符：
1、在PHP中，代码以行为单位，系统需要通过判断行的结束，通常都用；表示结束。
注：PHP中标记结束符?>有自带语句结束符的效果，最后一行PHP代码可以没有语句结束符。
2、PHP中代码的书写并不是全嵌入到HTML中，而是单独存在，所以可以不用标记结束符?>

## 变量

```php
// 定义时不需要关键字，但必须使用$符号
$var1;
$var2 = 1;
echo $var2; // 访问变量，通过变量名找到数据，并显示
$var2 = 2; // 修改变量
echo '<hr/>',$var2; // hr/“ 为下划线，分隔符
unset($var2); // 删除变量，从内存中剔除
echo $var2; // 此时会报错，因为不存在变量
```

变量的命名规则

1、变量名字必须以"`$`"开头；
2、变量名可由数字、字母、下划线命名，但必须以字母和下划线开头
3、允许中文变量；

预定义变量
即提前定义的变量，由系统定义的变量，存储许多要用到的数据（预定义变量都是数组）。

可变变量
如果一个变量保存的值刚好是另外一个变量的名字，则可直接通过访问一个变量得到另外一个变量的值：但需在变量前多加一个`$`符号

```php
$a = 'b'; // a变量的内容正好是b变量的名称，故称a为可变变量
$b = 'bb';
echo $$a; // 使用时需加一个$符号
```

值传递

```PHP
// 值传递
<?php
// 2.1 执行此行，在栈区开辟一块内存存储$a，在数据段中开辟一块内存保存值1。然后将1所在位置赋值给a变量
$a = 1;
// 2.2 栈区开辟存储$b；发现是赋值运算，故会取出$a的值，
// 并在数据段重新开辟一块内存并保存，且再把新开辟地址赋值给栈区的变量b
$b = $a; // 值传递
// 2.3 执行该行
$b = 2;
echo $a,$b;//1,2
```

运行步骤：
1、代码装载：从脚本文件中将代码读取出来，进行编译，将编译结果存放到代码段（二进制）。
2、代码执行：从代码段中一行一行执行代码。
3、脚本执行结束：系统会回收所有内存（栈区、代码区）：因为数据段与栈区有关系，回收栈后，数
据段的内容无意义，相当于回收。

引用传递

```PHP
// 引用传递
$a = 1;
$b = &$a; 
$b = 2;
echo $a,$b; //2,2
```

## 常量

const/constant:是在程序运行中，不可改变的量（数据）；常量一旦定义，通常不可更改。

定义方式

1、使用定义常量的函数：define（'常量名',常量值）——类似于c++的 define
2、const

```php
<?php
// 使用函数定义常量
define('PI',3.1415); // 注意此处与c++不同，#define 为预处理命令，宏定义，无需加；。。
// 使用const关键字定义
const PI1 = 3;
// 定义特殊常量
define('^-^','smile');
// const ^-^ // 报错
// 访问常量
echo PI1;
echo ^-^ // 报错
constant('^-^'); // 特殊常量的访问
// 系统常量
echo '<hr/>',PHP_VERSION,'<br/>',PHP_INI_SIZE,'<br/>',PHP_INI_MAX; // 有符号整形
// 魔术常量
echo '<hr/>',__DIR__,'<br/>',__FILE__,'<br/>',__LINE__;
echo __LINE__; // 输出的行数会变
```

常量命名规则

1、常量不需要使用"$"符号，一旦使用被认为是变量
2、变量名可由数字、字母、下划线命名，但必须以字母和下划线开头（const定义）
3、常量的名字通常是以大写字母为主（与变量以示区别）
4、变量命名的规则比变量要松散，可以使用一些特殊字符（define函数）
5、变量通常不区分大小写，但可以区分（define函数的第三个参数）

系统常量

魔术常量：由双下划线+常量名+双下划线组成，其值会随着环境变化而变化，用户无法改变。

`__DIR__` : 当前被执行脚本所处文件夹的绝对路径。
`__FILE__` : 当前被执行脚本的绝对路径。
`__LINE__` : 当前所属行数。
`__NAMESPACE__` : 当前所属命名空间。
`__CLASS__` : 当前所属的类。
`__METHOD__` : 当前所属的方法。

## 数据类型

数据类型：data type，在PHP中指的是数据本身的类型，而不是变量的类型。PHP是一种弱类型语言，变量本身没有数据类型。

基本数据类型
boolean（布尔型）
integer（整型）
float（浮点型）
string（字符串型）

复合数据类型
array（数组）
object（对象）

特殊数据类型
resource（资源）
NULL 空值（null）

转换方式
1、自动转化：系统根据自己的需求判断，自己转化（用的较多、但效率较低）。
2、强制转换（手动）：在变量之前增加一个()，并在括号里面写上对于的类型，其中NULL特殊，需用
unset。

转换说明：
1、以字母开头的字符串，转换成数值永远为0；
2、以数字开头的字符串，取到碰到字符串为止。（第二个小数点属于字符串）

```php
<?php
// 数据类型
// 创建数据
$a = 'abcd1.1.1';
$b = '1.1.1abc';
// 自动转换。算术运算，系统先转化为数值类型，然后运算
echo $a+$b; // 结果为1.1（0+1.1）
// 强制转换
echo '<br/>',(float)$a,(float)$b; // 0  1.1
// 类型判断
echo '<hr/>';
var_dump(is_int($a)); // bool(FALSE)
var_dump(is_string($a)); // bool(TRUE)
// 获取数据类型
echo '<hr/>';
echo gettype($a); // string
// 设置类型
// var_dump 输出展示展示代码内容，结构与类型。该函数作可以窥探所有内容的类型，以及内部信息
var_dump(settype($b,'int')); // 先将字符串转换为int型，转换成功返回true，var_dump判断是否为bool型，故显示 bool（true）
echo gettype($b),$b; // interger1
```

整型

```php
<?php
/*
十进制转换二进制----->除以2
10 1010 注：不管结果ruhr，均需补足32位：00000000 00000000 00000000 00001010
*/
// php中提供了很多函数进行转换：
// Decbin():十进制转二进制
var_dump(decbin(107)); // 结果：string(7) "1101011"
// 同理，还有Decoct():十进制转八进制
// Dechex():十进制转十六进制
// Bindec():二进制转十进制
*/

```

浮点数类型
问：为什么浮点数和整型均占用四个字节，为什么比整型表示的范围大？
整型数据的32位均通过*2转化为十进制。而浮点型中，前八位的后七位为指数，所以表示的范围要大。
实际使用时，尽量不用浮点型数字做精确判断，且计算机中凡是小数基本上均不准确。

```php
<?php
// 浮点数的定义
$f1 = 1.23;
$f2 = 1.23e10;
$f3 = PHP_INT_MAX+1; // 若整型超过自身存储的大小之后会自动改为浮点型存储
var_dump($f1,$f2,$f3);
// 结果：float(1.23)float(1.2300000000) float(214748348)
// 浮点数判断
$f4 = 0.7;
$f5 = 2.1;
$f6 = $f5/3;
var_dump($f6 == $f4); // 结果：bool(false),,因此其不能进行精确判断
```

## 运算符

运算符：是一种将数据进行运算的特殊符号，在PHP中一共有十多种运算符。
算术运算符 `+-*/%`
比较运算符` >  >=  <  <=  ==(数据大小相同即可，无需考虑数据数据类型) !=     ===(全等于，大小及数据类型均等)  !==`

```php
<?php
$a = '123'; // 字符串
$b = 123; // 整型
var_dump($a == $b); // 结果： bool(true)
var_dump($a === $b); // 结果： bool(false) 不全等于
```

逻辑运算符`&&    ||`
连接运算符` .(将字符串连接一起) .=(将左边内容与右边内容连接起来并重新赋值)`

```php
$e = 'hello';
$f = 123;
echo $e . $f; // hello 123（注意，此处有强制类型转换）
$e .= $f;
echo $e; // hello 123
```

错误抑制符：`@`(可能出错的表达式)，在PHP中有一些错误可以提前预知，但又不想报错，这就需要错误抑制符。

```php
$g = 0;
echo $f % $g; // 此时会报错
echo @($f % $g); // 不会报错
```

三目运算符`(问号表达式) 表达式1 ？ 表达式2：表达式3`
自操作运算符 `++ --`(前置或后置如果只有自操作，则效果一致)
位运算符`& | ~(按位取反) ^(按位异或) >>(右移) <<(左移)`

## 流程控制

顺序结构、分支结构（if分支与switch分支）、循环结构

```php
<?php
// 分支结构——if分支
$day = '星期天';
if($day == '星期1')
{
	echo 'go out';
}
else
{
	echo 'work';
}
// switch 分支：同一条件下，有多个值，且每个值对应一种操作
/*
switch(条件表达式)
{
	case 值1:
		代码;
		break;
	case 值1:
		代码;
		break;
	default:
		代码;
		break;
}
*/
// 循环结构 for循环、while循环、Do-While循环、foreach循环（针对数组）
for($i = 0;i<10;i++)
{
	echo $i;
}
// while Do-while循环
while($i <= 10)
{
    echo $i++ ;
}
// 循环控制
// 1、中断控制：重新开始从头循环 continue(需求，输出1-100的5的倍数)
$i = 1;
while($i <= 100)
{
	if($i % 5 != 0)
	{
		$i++ ;
		continue;
	}
	echo $i++;
}
```

流程控制替代语法

左大括号{使用冒号代替
右大括号}使用end+对应的标记

## 文件操作

文件包含：在一个PHP脚本中，将另外一个文件包含进来，合作完成一件事情。

文件包含作用
1、要么使用被包含文件中的内容，实现代码共享，向上包含（索要）
2、要么自己的东西可被使用，向下包含（给予）：自己有某个东西需要别的脚本显示。
最大的作用是分工协作，共同完成一件事情。

文件包含的四种形式

```php
<?php
// 包含文件：使用数据
// 包含文件：include include_once(系统自动判断文件包含当中，是否已被包含过) require require_once
// 向上包含：使用已准备好的文件
include 'include1.php'; // 包含当前文件所在文件夹下的include1.php文件
echo $a,PI; // include1.php中已定义这个变量和常量，故可直接使用
// 向下包含：类似于调用了子函数
$a = 10;
const PI = 3.14;
include_once 'display.php'; // 该文件中可输出a和PI
```

文件加载原理

PHP代码执行流程：
1、读取代码文件（相当于PHP程序）；
2、编译：将PHP代码转化成字节码（二进制），生成opcode（php可解析的代码）；
3、针对引擎来解析opcode，按照细节码进行逻辑运算；
4 转化成对应的html代码。

1、在文件加载（include 或 require）时，系统会自动嵌入对应的include位置
2、在PHP中，被包含的文件是单独编译的。。
若编译过程中报错，则会失败，不执行。但若是被包含文件有错误，则系统执行到include语句时，才回报错。

**include 和 require区别**
include会执行多次，导致报错（重复定义变量）。。而include_once不会出现这种情况。
require和include区别在于：若未包含文件，则报错形式不一样。（require包含错误文件，则include后不再执行；include未包含文件，会警告，但是仍会执行后面的。）

**文件加载路径**

文件加载时需指定文件路径，才能保证PHP正确找到对应的文件。

```php
<?php
// 文件加载路径
// 相对路径加载(只供演示，不考虑多次加载)
include 'include1.php'; // 不写路径，默认在当前文件夹下
include './include1.php'; // 另一种形式
include '../hostdoc/include1.php'; // ../代表当前文件夹的上一个文件夹，hostdoc为当前文件
夹
// 绝对路径
include 'E:/server/apache/htdocs/include1.php'; // 绝对路径，不会出错
```

**文件嵌套包含**

嵌套包含容易出现相对路径出错的问题。

## 函数

将实现某一功能的代码块封装到一个结构中，从而实现代码的复用。

函数定义语法（与c的差别在于可在任意位置调用子函数）

```php
/*
Function 函数名(参数){
函数体
返回值
}
*/
// 函数的定义
// 1、函数不会自动运行，必须调用才可
// 2、代码执行阶段，遇到函数名字才回调用，不是在编译阶段
// 3、函数调用可在声明之前
function display()
{
echo 'hello world'; // 没有返回值
}
// 函数的调用（）
diaplay(); // 若函数有参数，则需加参数
// 函数命名规范：字母数字下划线、但不能数字开头。
// 一半遵循以下规则：1、驼峰法：除第一个单词外，其余首字母大写。showParent() 2、下划线方式
// 在一个脚本函数周期中，不允许出现同名函数。
```

函数参数

```php
<?php
// 函数参数
// 定义函数:定义函数时使用的参数，形参
function add($arg1,$arg2)
{
	echo arg1+arg2;
}
// 调用函数时使用的参数，实参
$num1 = 10;
add($num1,20); // 1、实参个数可以多于形参（不能少于），只是函数不用而已 2、理论上实参个数没有限制
/* 调用过程：1、系统调用add函数时，会去内存中找是否有add函数
2、系统在栈区开辟内存空间运行函数add
3、系统查看函数本身是否有形参
4、系统判断调用函数时是否有实参
5、系统默认会将实参$num、20分别赋值给形参
6、执行函数体、运行
7、返回值
*/
// 默认值：形参的默认值。。若调用时没有提供实参，则函数使用默认值执行函数
// 注：1、默认值定义时，应放在后边，不能左边有默认值，而右边没有
function moren($num1 = 0,$num2 =0) // 当前的num1是形参，编译时不执行。且如果外部有同名子变量，也不会冲突
{
	echo $num1-$num;
}
// 上述实参形参的传递相当于值传递，函数内部改变变量的内容，不会影响外面变量的内容
// 引用传递：可在函数内部改变外部变量
function yinyong($a,&$b) // 函数要的是地址，故将外部变量b存储的地址取出赋值给了形参
{
	$b = $b - 1;
	$a = $a -1;
}
$a = 10;
$b = 5;
yinyong($a,$b); // 注意：此处不取地址..另，引用传递不可传入数字（常量中存储的不是地址）
```

函数返回值 类似于JS

作用域（与c差别在于全局变量不能直接被函数调用）
作用域：通常是指变量可以被访问的区域。
在PHP中，作用域严格分为两种，以及内部定义的一种
1、全局变量：所属全局空间，在PHP中只允许在全局空间使用，函数内部不可用。（c++可使用）
2、局部变量：函数内部的变量
3、超全局变量：预定义变量（系统定义的）,没有访问限制，能够帮助局部去访问全局变量。

```php
<? php
// php中作用域
// 默认的代码空间：全局空间
$global = 'global area';
// 局部变量（函数内部定义）
function display()
{
	$inner = 1;
// 访问全局变量
	echo $global; // 函数内部不能访问全局变量
// 转化为超全局变量，使得函数内部可以访问
	echo $GLOBALS['global']; // 这样可访问
}
display();
```

想在函数内部访问全局变量，可通过$GLOBALS，也可使用引用传值。

还有一种方式既可从全局访问局部、也可从局部访问全局。即，global关键字：
1、若使用global定义的关键字在外部存在，那么系统在函数内部定义的变量直接指向外部变量所指向的内存空间（同一个变量）。
2、若其定义的变量在外部不存在，系统会自动在全局空间定义一个与局部变量同名的全局变量。
本质为：在函数的内部和外部，对一个同名变量使用同一块内存地址保存数据。

```php
<? php
// global关键字的应用
// 默认的代码空间：全局空间
$global = 'global area';
// 局部变量（函数内部定义）
function display()
{
	// 访问全局变量
	echo $global; // 函数内部不能访问全局变量
	// 1、全局变量存在
	global $global;
	echo $global; // 此时可以调用全局变量
	//2、全局变量不存在
	global $local = 'inner';
}
echo $local; // 访问局部变量
display();
```

静态变量

静态变量：static是在函数内部定义的变量，使用static关键字修饰，用来实现夸函数共享数据的变量（注：跨函数是指同个函数多次调用）。

```php
function display(){
    $local = 1;
    static $count = 1;
    echo $local++,$count++,'<br/>';
}
display();//11
display();//12
display();//13
```

**可变函数**

**匿名函数**

```php
<?php
// 匿名函数：没有名字的函数，
// 定义基本匿名函数
// function() // 没有办法运行，故须加一个变量名字
// 变量保存匿名函数，本质得到的是一个对象（closure类中的对象）
$func = function()
{
	echo 'hello world';
}; // 因为相当于变量的赋值，所以需加一个分号
// 调用匿名函数
$func();
```

**闭包函数**
闭包：要执行的代码块（由于自由变量被包含在代码块中，这些自由变量以及它们引用的对象没有被释放）和为自由变量提供绑定地计算环境。（简单理解就是说，函数内部的一些局部变量即要执行的代码块，在执行完毕后没有被释放）。没有被释放的原因是：在函数内部，还有对应的函数被引用，通常为匿名函数。

```php
// 闭包函数
function display()
{
	$name == __FUNCTION__;
// 定义匿名函数
// $innerfunction = function()// $name 相对于匿名函数来说是外部变量，故不能直接用
// 1、使用匿名函数 2、使用关键字use
	$innerfunction = function() use($name) // use就是将局部变量 保留给内部使用（形成了闭包，使display运行完 毕后，不会被释放）
{
	echo $name;
}
$innerfunction();
// 3、 匿名函数返回给外部使用
return $innerfunction(); // 为验证局部变量未被释放而返回
};
$closure = display(); // 理论上此处局部变量被释放
$closure(); // 结果却输出了name，说明上一行并未释放局部变量
// 如何证明局部变量在函数使用完之后没有被释放？（三步法）
// 1、使用内部匿名函数
// 2、匿名函数使用句变量：use
// 3、匿名函数返回给外部使用。
```

## 错误处理

错误处理：指系统或用户在对某些代码进行执行的时候，发现有错误，就会通过错误处理的形式告诉程序员。

### **错误分类**

1、语法错误：书写代码不符合PHP语法规范，会导致代码在编译中不允许，故也不会执行（parseerror）；
2、运行时错误：代码编译通过，但在执行时会出现一些条件不满足从而导致的错误。（runtime error取空数组的第几位数）
3、逻辑错误：写代码不规范、但逻辑性错误，导致虽可正常运行，但得不到预期结果。

### **错误代号**

系统代号在PHP中均被定义为了系统常量，故可直接使用：
1、系统错误（系统使用的代号）：
E_PARSE：编译错误，代码不会运行
E_ERROR：fatal error致命错误，会在出错的位置断掉
E_WARNING：warning警告错误，不影响执行，但可能得不到预期结果
E_NOTICE：notice，通知错误、不影响代码执行
2、用户错误（用户使用的代号）：E_USER_ERROR、E_USER_WARNING、E_USER_NOTICE用户在使用自定义错误出发的时候，会使用道德错误代号。
3、E_ALL：代表所有错误所有E开头的错误常量都由一个字节（8位）存储，且每一种错误占用一个位，故可进行位操作。
排除通知级别notice：E_ALL & ~E_NOTICE 。。假设ALL全为1，那么与NOTICE取反再取与就可把其剔除
只要警告和通知：E_WARNING | E_NOTICE

### **错误触发**

程序运行时触发：主要针对代码的语法错误和运行时错误。
人为触发：知道某些逻辑可能会出错，从而使用对应的代码编号来判断

```php
<?php
// php错误处理
// 人为触发
// 处理脚本让浏览器按照指定字符集解析
// header('Content-type:text/html;charset=utf-8');
$b = 0;
if($b == 0)
{
	trigger_error('除数不能为0')
}
echo $a / $b;
```



## 字符串类型

### 字符串定义

1、引号定义:比较适合定义较短的或无结构要求的字符串

```php
$str1 = 'hello';
$str2 = "hello";
var_dump($str1,$str2); // 两种方式显示的结果一致
```



### 结构化定义

 2、heredoc字符串：没有单引号的单引号字符串

```php
$str3 = <<<EOD
		HELLO
		EOD;
```

3、nowdoc 结构

```php
$str4 = <<<'EOD' // eod只是边界符，可自己定义
			hello
			EOD;
var_dump($str3,$str4);
```

### **转义字符串**

\r：回车 \n：换行 \t:四个空格
双引号内的字符串能够识别。

### **双引号中变量识别规则：**

```php
$a = 'hello';
$b = "avc{$a}a";
```

### **字符串长度问题**

```php
<?php
header('Content-type:text/html;charset = utf-8');
// 定义字符串
$str1 = 'abcefjdoifaoi';
$str2 = '你好中国123';
echo strlen($str1),'<br/>',strlen($str2); // 13 15(中文在utf下占3个字节)
// 多字节字符串的长度问题：包含中文的长度
// 多字节字符串扩展模块：mbstring扩展(mb:Multi Bytes)
// 首先需加载PHP的mbstring扩展（php.ini中去注释即可）
// 使用mbstring
echo mb_strlen($str1),'<br/>',mb_strlen($str2); // 13 15(与之前一致)
// 长度并未改变，MBstring针对不同的字符集有不同的统计结果
echo mb_strlen($str1),'<br/>',mb_strlen($str2),'<br/>',mb_string($str2,'utf-8'); //13 15 7
```

## 数组

### 定义语法

```php
<?php
// php数组：可以是一种或多种类型的数据，这与c++很不一样.类似于哈希表
// 定义数组：array
$arr1 = array('1',2,'hello');
var_dump($arr1); // 结果：
// array(3) ([0]=>string(1) "1" [1]=>int(2)[2]=>string(5) "hello")

// 定义数组：[]
$arr2 = ['1',2,'hello'];
var_dump($arr2);
// array(3) ([0]=>string(1) "1" [1]=>int(2) [2]=>string(5) "hello")

// 定义数组：隐型数组
$arr3[] = 1; // 默认给数组第0个元素赋值
$arr3[10] = 100; // 第10个元素赋值
$arr3[] = '1'; // 第11个。默认下标是从当前最大下标
$arr3['key'] = 'key'; // 第key个
$arr3[1] = 'value' // 第1个，但不会自动调整，还是会处于最后一个位置
// 结果为：array(4) ([0]=>int(1) [10]=>int(100) [11]=>string(1) "1"["key"]=>string(3)"key" [1]=>string(5) "value")
```

### **php数组特点**

1、可以整数下标或者字符串下标
若数组下标均为整数，则称为索引数组
若数组下标均为字符串，则称为关联数组。
混合下标的话称为混合数组
2、数组元素的顺序以放入顺序为准，与下标无关
3、数字下标的增长特性：从0开始自动增长，若中间手动加入较大的下标，则后面则会从当前最大下标+1增长。
4、特殊值下标的自动转换

```php
<?php
// 特殊下标自动转换
$arr1[false] = false;
$arr1[true] = true;
$arr1[NULL] = NULL;
var_dump($arr1); // array(3)([0]=>bool(false),[1]=>bool(true) [""]->NULL)
```

5、PHP数组中类型元素没有限制。
6、PHP中数组元素没有长度限制。类似于c++ vector
补充：PHP中数组是很大的数据，故会存储在堆区。

### 多维数组

类似于C++ vector
二维数组，数组中的所有元素都是一维数组。

```php
$info = array(
	array("name" => "jim","age" => 30),
	array("name" => "tom","age" => 31),
    array("name" => "xiaoming","age" => 32)
)
```

### 数组的遍历

类似于JS

```php
$info = array(
	array("name" => "jim","age" => 30),
	array("name" => "tom","age" => 31),
    array("name" => "xiaoming","age" => 32)
);
echo $info[0]["name"]; // tom
```

**foeach**

```php
<?php
// 数组遍历 foreach
$arr = array(1,2,3,4,5,6,7,8,9,10);
// foreach
foreach($arr as $a)
{
	echo $a,'<br/>'; // 依次输出
}
foreach($arr as $a => $v)
{
	echo 'key',$a,'== value',$v,'<br/>'; // 依次输出key0 == value1 等
}

// 二维数组
$arr = array(
	0 => array('name' =>'Tom','age' => 10),
	1 => array('name' => 'Jim','age' => 11)
);
// 通过foreach遍历二维元素
foreach($arr as $a)
{
	echo 'name is:',$a['name'],'age is:',$a['age'],'<br/>';
// name is:TOM age is:10
// name is:TOM age is:10
}
```

**for**

```php
<?php
// for循环遍历数组
// 数组特点：1、索引数组 2、下标规律
$arr = array(1,2,3,4,5,6,7,8);
for($i = 0; $i<count($arr);$i++)
{
	echo 'key is:',$i,'value is:',$arr[$i];
}
```

数组相关函数

排序函数

```php
// 排序函数
$arr = array(3,1,5,2,0);
print_r(sort($arr)) // 结果为1
print_r($arr); // 排序后，索引变为01234
```

指针函数(链表)

```php
// 指针函数
//reset:将数组的内部指针指向第一个单元；
//end():将数组指针指向最后一个元素；
// next(): prev():指针上移
//current():获取当前指针对应的元素值 
//key()获取当前指针对应的key
// 判断是否指针移动
echo current($arr),'<br/>';
echo key($arr),'<br/>'; //若是第一个元素。则当前数组指针未移动
echo next($arr),next($arr),'<br/>'; //15
echo prev($arr);//1
// 注意事项：next，prev会移动指针，可能导致超出数组，此时再使用next、prev便不能再返回数组，只能通过end/reset
```

其他函数

```php
// 其他函数：count
```

栈/队列

```php
//前面array_shift(),array_unshift();
//后面array_push(),array_pop();
array_push($arr,3);
array_push($arr,2);
array_push($arr,1);

array_pop($arr);
```



# Composer

## 安装

先升级包管理器

```bash
sudo apt update
```

然后下载需要的一些软件

```bash
sudo apt install php-cli unzip
```

用`curl`下载东西

```bash
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
```

检测一下sha-384，没啥用但是挺好玩。

```bash
HASH=`curl -sS https://composer.github.io/installer.sig`
```

```bash
echo $HASH

# Output
# e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a
```

```bash
php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Output
# Installer verified
```

安装`compser` 用php直接安装，可以设置路径和文件名。

```bash
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

安装完了运行

```bash
composer
```

```bash
# Output
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.10.5 2020-04-10 11:44:22

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi                     Force ANSI output
      --no-ansi                  Disable ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...
```



## Demo

以`slugify`为例  也可以去https://packagist.org/找其他的插件

初始化composer

```bash
mkdir composer
cd composer
composer init
```

会自己创建一个`composer.json`文件

```bash
composer require cocur/slugify
```

```bash
#Output
Using version ^4.0 for cocur/slugify
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Installation request for cocur/slugify ^4.0 -> satisfiable by cocur/slugify[v4.0.0].
    - cocur/slugify v4.0.0 requires ext-mbstring * -> the requested PHP extension mbstring is missing from your system.
...
```

报错，说需要PHP extension mbstring

```bash
sudo apt install php-mbstring
```

重新安装

```bash
composer require cocur/slugify
```

```bash
#Output
Using version ^4.0 for cocur/slugify
./composer.json has been created
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing cocur/slugify (v4.0.0): Downloading (100%)         
Writing lock file
Generating autoload files
```

然后会生成文件

```bash
ls -l

#output
drwxrwxr-x 4 work work 4096 Nov 23 12:06 ./
drwxrwxr-x 3 work work 4096 Nov 23 11:48 ../
-rw-rw-r-- 1 work work  198 Nov 23 11:49 composer.json
-rw-rw-r-- 1 work work 3114 Nov 23 11:49 composer.lock
drwxrwxr-x 2 work work 4096 Nov 23 11:48 src/
drwxrwxr-x 4 work work 4096 Nov 23 12:08 vendor/
```

`composer.lock`是用来储存已经安装的包信息的。如果别人clone了这个项目，可以通过这个文件来确保你们的依赖一致。
`vendor`存放已经安装的包的文件。

```bash
cat composer.json

{
    "name": "work/src",
    "description": "test",
    "require": {
        "cocur/slugify": "^3.0"
    }
}
```

```bash
vim test.php
```

```php
<?php
require __DIR__.'/vendor/autoload.php';

use Cocur\Slugify\Slugify;
$slugify = new Slugify();

echo $slugify->slugify('hello world');
```

```bash
php test.php

#output
hello-world
```

升级composer插件的命令

```bash
composer update vendor/package vendor2/package2
```

