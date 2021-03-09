# JavaScript

```html
<head>
<script type="text/JavaScript">  //定义下文使用的是JAVA 可用在head or body
document. write("text")；
</script>
<script src="script.js"></script>  //引用文件

```

```javascript
// 注释掉
/* 里面的注释掉*/
var 变量名 
变量名="JavaScript";  //可以使用任意多个英文字母、数字、下划线(_)或者美元符($)组成
var 变量名 = 6;      //在JS中区分大小写
alert(字符串或变量) //弹窗、
```

||← 或者 
&&← 并且
！ ← 非




### 数组操作

```javascript
var myarr=new Array(); //定义数组
 myarr[0]=80; 
 myarr[1]=60;
 myarr[2]=99;
//第一种方法：
var myarray = new Array(66,80,90,77,59);//创建数组同时赋值
//第二种方法：
 var myarray = [66,80,90,77,59];//直接输入一个数组（称 “字面量数组”）

var arr=[98,76,54,56,76];  // 包含5个数值的数组
document.write(arr.length); //显示数组的长度5
arr[15]=34;  //增加元素，使用索引为15,赋值为34
alert(arr.length);  //显示数组的长度16
arr.length=20;  //增大数组的长度
alert(arr.length);  //显示数组的长度20

```



## If 

```javascript
if(score >=60)
{ 条件成立时执行的代码 }
else
{ 条件不成立时执行的代码 }break

中断循环
for(初始条件;判断条件;循环后条件值更新)
{
  if(特殊情况)
  {break;}
  循环代码
}
continue; 
for(初始条件;判断条件;循环后条件值更新)
{
  if(特殊情况)
  { continue; }
 循环代码
}

```

### If嵌套

```javascript
if(条件1)
{ 条件1成立时执行的代码}
else  if(条件2)
{ 条件2成立时执行的代码}
else  if(条件n)
{ 条件n成立时执行的代码}
else
{ 条件1、2至n不成立时执行的代码}

```

```javascript
var myweek =1
switch(myweek)
{
 case 1:
 case 2:
 document.write("学习理念知识");
 break;
 case 3:
 case 4:
 document.write("到企业实践");
 break;
 case 5:
 document.write("总结经验");
 break;
 case 6:
 case 7:
 document.write("周六、日休息和娱乐");
}

```

### For循环

```javascript
var num=1;
for (num=1;num<=6;num++)  //初始化值；循环条件；循环后条件值更新
{   document.write("取出第"+num+"个球<br />");
}

```

### While 循环

```javascript
while (num<=6)   //条件判断
{
  document.write("取出第"+num+"个球<br />");
  num=num+1;  //条件值更新
}

```

### Do while 循环

```javascript
<script type="text/javascript">
   num= 1;
   do
   {
     document.write("数值为:" +  num+"<br />");
     num++; //更新条件
   }
   while (num<=5) //满足条件时循环
</script>

```

## 输出内容（document.write）

```javascript
<script type="text/javascript">
  document.write("I love JavaScript！"); //内容用""括起来，""里的内容直接输出。
</script>
<script type="text/javascript">
  var mystr="hello world!";
  document.write(mystr);  //直接写变量名，输出变量存储的内容。
</script>
<script type="text/javascript">
  var mystr="hello";
  document.write(mystr+"I love JavaScript"); //多项内容之间用+号连接
</script>
<script type="text/javascript">
  var mystr="hello";
document.write(mystr+"<br>");//输出hello后，输出一个换行符
  document.write("JavaScript");
</script>

```

## 窗口指令

window.open([URL], [窗口名称], [参数字符串])
URL：可选参数
窗口名称：可选参数，被打开窗口的名称
_blank：在新窗口显示目标网页
        _self：在当前窗口显示目标网页
        _top：框架网页中在上部窗口中显示目标网页

Eg：

```javascript
window.open('http://www.imooc.com','_blank','width=300,height=200,menubar=no,toolbar=no, status=no,scrollbars=yes')
//大小为300px * 200px，无菜单，无工具栏，无状态栏，有滚动条窗口


window.close();   //关闭本窗口
<窗口对象\变量>.close();   //关闭指定的窗口

```

### 选择对话窗口

```javascript
var 001=confirm("你喜欢JavaScript吗?");
    if(001==true)
    {   document.write("很好,加油!");   }
    else
{  document.write("JS功能强大，要学习噢!");   }

```

### 询问消息对话框

```javascript
var myname=prompt("请输入你的姓名:");
if(myname!=null)
  {   alert("你好"+myname); }
else
  {  alert("你好 my friend.");  }

```

##  Dom操作

```javascript
//通过ID提取信息
document.getElementById(“id”) 
//通过ID设置字体
<script>
   var 变量 = document.getElementById("ID");
   变量.style.color="red";
   变量.style.fontSize="20";
   变量.style.backgroundColor ="blue";
</script>

```

## Function 指令

```javascript
function add2(x,y)
{
   sum = x + y;
   return sum; 
}


<input type="button"  value="点击我" onclick="函数名()" />
```

### 事件

```javascript
Onclick  鼠标单击事件
Onmouseover 鼠标经过事件
Onmouseout 鼠标移开事件
Onchange文本框内容改变事件
Onselect文本框内容被选中事件
Onfocus光标聚集
Onblur光标离开
Onload网页导入
Onunload关闭网页
<input type="button"  value="点击我" onclick="函数名()" />

```

