# HTML

```html
<!DOCTYPE html>
<html>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>XXX</title>
<head>

<br />下一行
&nbsp;    空格
<hr />分段横线
<code>插入代码
<pre> 同上，显示空格和回车
<ul><li>文本前显示点</li></ul>
<ol><li>自动排123
<div>区分逻辑部分
<div id=" name">
<div class="class">
/*说明*/
<！--说明-->

```

## 设置

```html
↓嵌入式css
< head>  <style type="text/css">  
span{ color: blue} 
Body{ line-height:1.6em} 
    
↓内联式css
<p style="color:red;font-size:12px">这里文字是红色。</p>
    
↓外联式 css 
<link href="base.css" rel="stylesheet" type="text/css" />

```

## 选择器

优先级 内联式 > 嵌入式 > 外部式

```html
↓类选择器
<span class="stress">胆小如鼠</span>
.stress{color:red;}	/*类前面要加入一个英文圆点*/ ←与span一样用
↓ID选择器
<span id="stress">胆小如鼠</span>
#stress{font-family:"宋体";}
↓子选择器
.某class>span{border:1px solid red;}
.某class span{border:1px solid red;} ←后代选择器
* {color:red;}   ←通用选择器
某标签:hover{color:red;} ←伪类选择器 鼠标划过变色
各标签用,隔开
优先级 id＞类＞标签 →   !important优先级最高
相等等级 最后的覆盖前面的

```

## 表格设置

```html
<style type="text/css">
Table tr td, th{borde:1px solid #000;
p{
   font-size:12px;
   color:red;
   font-weight:bold; ←加粗
   font-family:"宋体";
   font-style:italic ; ←斜体
   text-decoration:underline; ←下划线
   text-decoration:line-through; ←划掉
   text-indent:2em;  ←首行缩进2个文字大小
   line-height:2em;  ←行间距
   letter-spacing:50px; ←文字间距
   word-spacing:50px; ←单词间距
   text-align:center/ right/ left; ←居中
   }  

```

## **表格**

```html
<table>
<tbody>←非必要，看不懂
<captain>←表格标题
<tr>   ←这是行
<th>加粗字体</th>
<td>正常字体</td>
</tr>
</table>
<table summary="对表格的形容">

dashed（虚线）| dotted（点线）| solid（实线）
border-bottom/top/ right/ left← 单边框
padding:10px;  ←填充
margin-bottom:30px; ←表格外填充
------超链接------
<a href="目标网址" 
title=" 鼠标悬浮显示"
target="_blank"  ←新标签页
>
文本666
</a>
<a href="mailto:123@qq.com? subject=主题& body=内容>
文本</a>

<img src="图片地址" alt="下载失败时的替换文本" title = "提示文本">

```

## *表单标签*

```html
<form    method="post"   action="save.php">
        <label for="username">用户名:</label>
        <input type="text" name="username" />   ←type="text"输入框为文本输入框
        <label for="pass">密码:</label>
        <input type="password" name="pass" />  ←type="password"为密码输入框
</form>
 <form>
  <label for="male">男</label>
  <input type="radio" name="gender" id="male" />

```



## *文本域*

```html
<textarea  rows="行数" cols="列数">文本</textarea>
```



## *选择框*

```html
<input   type="radio/checkbox"   value="值"    name="名称"   checked="checked"/>
   当 type="radio" 时，控件为单选框
   当 type="checkbox" 时，控件为复选框
<input   type="submit"   value="提交"> ←确定按钮
<input type="reset" value="重置"> ←重置按钮
<option value= "提交值 selected="selected">默认选项</option>
< select multiple="multiple"> ← 多选

```



### *常用的块状元素有：*

```html
<div>、<p>、<h1>...<h6>、<ol>、<ul>、<dl>、<table>、<address>、<blockquote> 、<form>
设置块状元素:span{display:block;}

```



### *常用的内联元素有：*

```html
<a>、<span>、<br>、<i>、<em>、<strong>、<label>、<q>、<var>、<cite>、<code>
设置内联元素: div{display:inline;}

```



### 常用的内联块状元素有：

```html
<img>、<input>
设置内联块状元素 display:inline-block

```

# css

**绝对定位**(position: absolute)

left:100px;  ←距离左边

top:50px;  ←距离上面

**相对定位**(position: relative)

Left:100px; ←相对左边

top:50px;  ←相对上面

**固定定位**(position: fixed)

bottom:0;  ←弹窗

right:0;   ←弹窗