Less（Leaner Style Sheets）是一门向后兼容的 CSS 扩展语言。

注释格式为 //

### 变量（variables）
通过`@变量名:变量值；`来定义变量，变量可在任意处定义；
作为属性值使用的用法为`@变量名`；
作为选择器、链接、属性名使用的用法为`@{变量名}`；
使用`$类名`作为属性值可以直接复用该类名的属性值；

### 嵌套（nesting）
Less中可通过嵌套来体现元素间的关系；
如以下CSS代码：
```css
#header {
color: black; 
}
#header .navigation { 
font-size: 12px; 
}
```
在Less中可写为：
```less
#header { 
color: black; 
.navigation { 
font-size: 12px; 
}
}
```
&符号用于指代父元素，
如
```less
.p1{
	&:hover{
	}
}
```
即表示
`.p1:hover{}`


### 扩展（extend）
`:extend()`伪类用于对一个元素的样式进行扩展；
如
```less
.p1{
	width: 100px;	
}
.p2:extend(p1){
	color: red;
}
```
可以实现以下效果：
```css
.p2{
	width: 100px;
	color: red;
}
```


### 混合（mixins）
混合是一种将一组属性从一个规则集包含（或混入）到另一个规则集的方法。
定义一个混合类，
```less
.bordered(){ 
	border-top: dotted 1px black; 
	border-bottom: solid 2px black; 
}
```
使用如下
```less
#menu a{ 
	color: #111; 
	.bordered(); 
}
```
即可将`.bordered( )`的样式应用到`#menu a`中，括号表示`.bordered`不会被解析到CSS文件中。

#### 混合函数
定义函数：
```less
.函数名(@参数1名,@参数2名,...){
	属性：@参数1名；
	......
}
```
使用函数：
```less
div{
	.函数名(参数1,参数2,...)
}
```

