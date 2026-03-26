+++
date = '2024-07-21T16:40:55+08:00'
draft = false
title = 'HTML学习'
slug = "p4nw47z9x3"
description = "HTML学习"
categories = ["📒开发"]
tags = ["Html"]
summary = "HTML学习"
comments = true  
+++
﻿# 1.网络传输三大基石

三大基石：URL，HTTP协议，HTML

URL：在WWW上，每一信息资源都有统一的且在网上唯一的地址，该地址就叫URL（Uniform Resource Locator,统一资源定位符），它是WWW的统一资源定位标志，就是指网络地址。

HTTP协议：http是一个简单的请求-响应协议，它通常运行在TCP之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以ASCII码形式给出；而消息内容则具有一个类似MIME的格式。这个简单模型是早期Web成功的有功之臣，因为它使得开发和部署是那么的直截了当。

HTML：HTML称为超文本标记语言。

![image-20210216112253463](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241620106_0c5430.webp)

# 2.初识HTML

> 什么是HTML

- HTML
 - Hyper Text Markup Language（超文本标记语言）

> W3C标准

- W3C
  - **W**orld **W**ide **W**eb **C**onsortium
  - 成立于1994年，Web技术领域最权威和具影响力的国际**中立性技术标准机构**
  - http://www.w3.org/
  - http://www.chinaw3c.org/
- W3C标准包括
  - **结构**化标准语言（HTML、XML）
  - **表现**标准语言（CSS）
  - **行为**标准（DOM、ECMAScript）
# 3.网页基本信息

> 网页基本信息

```html
<!-- DOCTYPE：告诉浏览器，我们要使用什么规范 -->
<!DOCTYPE html>
<html>
        <!-- 这是一个注释，注释的快捷键是ctrl+shift+/-->
        <head>
                <!--页面标题-->
                <title>网页基本信息学习</title>
                <!--设置页面的编码，防止乱码现象
                        利用meta标签，
                        charset="utf-8" 这是属性，以键值对的形式给出  k=v a=b 
                        告诉浏览器用utf-8来解析这个html文档
                -->
                <meta charset="utf-8" /><!--简写-->
                <!--繁写形式：（了解）-->
                <!--<meta http-equiv="content-type" content="text/html;charset=utf-8" />-->
                <!--页面刷新效果-->
                <!--<meta http-equiv="refresh" content="3;https://www.baidu.com" />-->
                <!--页面作者-->
                <meta name="author" content="zy;22515@qq.com" />
                <!--设置页面搜索的关键字-->
                <meta name="keywords" content="赵羽;自学;" />
                <!--页面描述-->
                <meta name="description" content="赵羽学习页面" />
        </head>
        <!--
                body标签中：放入：页面展示的内容
        -->
        <body>
                Hello，World！
        </body>
</html>
```

【1】html标签
定义 HTML 文档，这个元素我们浏览器看到后就明白这是个HTML文档了，所以你的其它元素要包裹在它里面，标签限定了文档的开始点和结束点，在它们之间是文档的头部和主体。

【2】head标签---》里面放的是页面的配置信息
head标签用于定义文档的头部，它是所有头部元素的容器。<head> 中的元素可以引用脚本、指示浏览器在哪里找到样式表。文档的头部描述了文档的各种属性和信息，包括文档的标题、在 Web 中的位置以及和其他文档的关系等。绝大多数文档头部包含的数据都不会真正作为内容显示给读者。
下面这些标签可用在 head 部分：

```html
<title>、<meta>、<link>、<style>、 <script>、 <base>。
应该把 <head> 标签放在文档的开始处，紧跟在 <html> 后面，并处于 <body> 标签之前。
文档的头部经常会包含一些 <meta> 标签，用来告诉浏览器关于文档的附加信息。
```

【3】body标签---》里面放的就是页面上展示出来的内容
body 元素是定义文档的主体。body 元素包含文档的所有内容（比如文本、超链接、图像、表格和列表等等。）body是用在网页中的一种HTML标签，标签是用在网页中的一种HTML标签，表示网页的主体部分，也就是用户可以看到的内容，可以包含文本、图片、音频、视频等各种内容！

# 4.网页基本标签

```html
<!-- DOCTYPE：告诉浏览器，我们要使用什么规范 -->
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>基本标签学习</title>
    </head>
    <body>
        <!--标题标签-->
        <h1>一级标签</h1>
        <h2>二级标签</h2>

        <!--段落标签-->
        <p>你好！</p>

        <!--水平标签-->
        <hr/>

        <!--换行标签-->
        我喜欢学习<br />HTML
        <hr/>

        <!--加粗倾斜下划线-->
        <b>加粗</b><br />

        <i>倾斜</i><br />

        <u>下划线</u><br />

        <!--一箭穿心-->
        <del>你好 你不好</del><br />

        <hr/>
        <!--特殊符号-->

        空&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;格
        <hr/>
        <!--大于号-->
        &gt;
        <hr/>
        <!--小于号-->
        &lt;
        <hr/>
        <!--@版权所有-->
        &copy;版权所有
        <!--字体标签-->
        <hr/>
        <font color="#397655" size="7" face=楷体>床前明月光</font>
    </body>
</html>    
```

![image-20210216120407519](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241620696_4e9604.webp)

# 5.图像标签

```html
<!DOCTYPE html>
<html>
        <head>
                <meta charset="UTF-8">
                <title></title>
        </head>
        <body>
                <!--图片
                src:引入图片的位置
		                        引入本地资源
		                        引入网络资源
	                src：图像地址
					alt：图像的替代文字
					title：鼠标悬停提示文字
					width：图像宽度
					height：图像高度
	                        注意:一般高度和宽度只设置一个即可，另一个会按照比例自动适应
	                title:鼠标悬浮在图片上的时候的提示语，默认情况下（没有设置alt属性） 图片如果加载失败那么提示语也是title的内容
	                alt:图片加载失败的提示语
                -->
                ![图片加载失败](img/phone_1.jpg)
                ![](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/apic30792_7af9d6.webp)
                <!--音频和视频
				    src：资源路径
				    controls：控制条
				    autoplay ：自动播放
				    -->
		    <video src="audio/video.mp4" controls autoplay></video>
		
		    <audio src="audio/music.mp3" controls autoplay></audio>
        </body>
</html>
```

# 6.超链接标签应用

> 链接标签

- 文本超链接
- 图像超链接

```html
<a href="链接路径" target="目标窗口位置">链接文本或图像</a>
```

> 超链接

- 页面间链接：从一个页面跳转到另一个页面
- 锚链接
- 功能性链接

```html
<!DOCTYPE html>
<html>
	<head>
	    <meta charset="UTF-8">
	    <title>链接标签学习</title>
	</head>
	<body>
	    <!--使用name作为标记-->
	    <a name="top">顶部</a>
	
	    <!--锚标签
	    1.需要一个锚标记
	    2.跳转到标记
	    -->
	    <a href="#down">回到底部</a><br />
	
	    <!--a标签
		    href:必填，表示要跳转到那个页面
		    target:表示窗口在哪里打开
	        _blank:在新标签中打开
	        _self:在自己的网页中打开
	    -->
	    <a href="Hello.html" target="_blank">点击我跳转到页面</a><br />
	    
	    <a href="https://baidu.com" target="_self">点击我跳转到百度</a><br />

	    <!--锚标签
	    1.需要一个锚标记
	    2.跳转到标记
	    -->
	    <a href="#top">回到顶部</a>
	    <a name="down">底部</a>
		<br />
	</body>
</html>
```

# 7.列表标签

> 列表

- 什么是列表
  - 列表就是信息资源的一种展示形式。它可以使信息结构化和条理化，并以列表的样式显示出来，以便浏览者能更快捷的获得相应信息
- 列表的分类
  - 无序列表
  - 有序列表
  - 自定义列表

```html
<!DOCTYPE html>
<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <title>列表学习</title>
	</head>
	<body>
	
	    <!--有序列表:
            type:可以设置列表的标号：1,a,A,i,I
            start:设置起始标号
        -->
	    <ol type="I">
	        <li>窗前明月光</li>
	        <li>疑是地上霜</li>
	        <li>举头望明月</li>
	        <li>低头思故乡</li>
	    </ol>
	    <hr>
	    <!--无序列表:
            type:可以设置列表前图标的样式   type="square"
                        如果想要更换图标样式，需要借助css技术： style="list-style:url(img/act.jpg) ;"
        -->
	    <ul>
	        <li>窗前明月光</li>
	        <li>疑是地上霜</li>
	        <li>举头望明月</li>
	        <li>低头思故乡</li>
	    </ul>
	    <hr>
	    <!--自定义列表
	    dl：标签
	    dt：列表名称
	    dd：列表内容
	    -->
	    <dl>
	        <dt>静夜思</dt>
	        <dt>李白</dt>
	
	        <dd>窗前明月光</dd>
	        <dd>疑是地上霜</dd>
	        <dd>举头望明月</dd>
	        <dd>低头思故乡</dd>
	    </dl>
	</body>
</html>
```

# 8.表格标签

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
    </head>
        <body>
            <!--表格：4行4列
                    table:表格
                    tr:行
                    td:单元格
                    th:特殊单元格：表头效果：加粗，居中
                    	默认情况下表格是没有边框的，通过属性来增加表框：
                    border:设置边框大小
                    cellspacing：设置单元格和边框之间的空隙
                    align="center"  设置居中
                    background 设置背景图片 background="img/xx.jpg"
                    bgcolor :设置背景颜色
                    rowspan:行合并
                    colspan：列合并
            -->
            <table border="1px" cellspacing="2px" width="400px" height="300px" bgcolor=cadetblue >
                <tr bgcolor="bisque">
                    <th>学号</th>
                    <th>姓名</th>
                    <th>年纪</th>
                    <th>成绩</th>
                </tr>
                <tr>
                    <td align="center">192001212</td>
                    <td align="center">张三</td>
                    <td align="center">19</td>
                    <td rowspan="2" align="center">90.5</td>
                </tr>
                <tr>
                    <td align="center">192001213</td>
                    <td align="center">李四</td>
                    <td align="center">18</td>
                </tr>
                <tr>
                	<td colspan="4">备注：</td>
                </tr>
            </table>
        </body>
</html>
```

# 9.iframe内联框架

```html
<!DOCTYPE html>
<html>
	<head>
	    <meta charset="UTF-8">
	    <title>内联框架</title>
	</head>
	<body>
	    <!--iframe内联框架
	    src：地址
	    w-h：宽度高度
	    -->
	    <iframe src="http://baidu.com" frameborder="0" width="800px" height="300px">
	    </iframe>
	
	    <iframe src="" name="hello" frameborder="0"></iframe>
	    <a href="Hello.html" target="hello">点击跳转</a>
	</body>
</html>
```

frameset 元素可定义一个框架集。它被用来组织多个窗口（框架）。每个框架存有独立的文档。在其最简单的应用中，frameset 元素仅仅会规定在框架集中存在多少列或多少行。您必须使用 cols 或 rows 属性。

里面如果只有一个框架用frame标签
如果多个框架用frameset标签
用cols 或 rows进行行，列的切割

```html
<!DOCTYPE html>
<html>
        <head>
                <meta charset="UTF-8">
                <title></title>
        </head>
        <!--框架集合：和body是并列的概念，不要将框架集合放入body中-->
        <frameset rows="20%,*,30%">
                <frame />
                <frameset cols="30%,40%,*">
                        <frame />
                        <frame src="index.html"/>
                        <frame />
                </frameset>
                <frameset cols="50%,*">
                        <frame />
                        <frame />
                </frameset>
        </frameset>
</html>
```

# 10.表单

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621036_6fdca7.webp)

表单在 Web 网页中用来给访问者填写信息，从而能采集客户端信息，使网页具有交互的功能。一般是将表单设计在一个Html 文档中，当用户填写完信息后做提交(submit)操作，于是表单的内容就从客户端的浏览器传送到服务器上，经过服务器上程序处理后，再将用户所需信息传送回客户端的浏览器上，这样网页就具有了交互性。这里我们只讲怎样使用Html 标志来设计表单。
所有的用户输入内容的地方都用表单来写，如登录注册、搜索框。
一个表单一般应该包含用户填写信息的输入框,提交按钮等，这些输入框,按钮叫做控件,表单很像容器,它能够容纳各种各样的控件。

```html
<form action＝"url" method=get|post name="myform" ></form>
```

-name：表单提交时的名称
-action：提交到的地址
-method：提交方式，有get和post两种，默认为get

```html
<!DOCTYPE html>
<html>
        <head>
            <meta charset="UTF-8">
            <title>表单元素学习</title>
        </head>
        <body>
        	<!--表单form
    			action：表单提交的位置，可以是网站，也可以是一个请求处理地址
    			method：post，get 提交方式
        		get方式提交：外面可以在url中看到外面提交的信息，不安全，高效
        		post方式提交，比较安全，可以传输大文件
    		-->
            <form action="" method="get">
                    
                <p>
                	<!--文本框:
                            input标签使用很广泛，通过type属性的不同值，来表现不同的形态。
                            type="text"  文本框，里面文字可见
                            	表单元素必须有一个属性：name 有了name才可以提交数据,才可以采集数据
                            	然后提交的时候会以键值对的形式拼到一起。
                            value:就是文本框中的具体内容
                            	键值对：name=value的形式
                            	如果value提前写好，那么默认效果就是value中内容。
                            	一般默认提示语：用placeholder属性，不会用value--》value只是文本框中的值。
                            size：指定表单元素的初始宽度。
                            	但type为text或password时，表单元素的大小以字符为单位。
                            	对于其他类型，宽度以像素为单位。
                            maxlength：type为text或password时，输入的最大字符数。
                            checked：type为radio或checkbox时，指定按钮是否被选中。
                            readonly只读：只是不能修改，但是其他操作都可以，可以正常提交
                            disabled禁用：完全不用，不能正常提交
                            
                        	写法：
	                            readonly="readonly"
	                            readonly
	                            readonly = "true"
                    -->
                	<input type="text" name="uname"  placeholder="请录入身份证信息"/>
	                <hr/>
	                <input type="text" name="uname2" value="123123" readonly="true"/>
	                <hr/>
	                <input type="text" name="uname3" value="456456" disabled="disabled"/>
	                <hr/>
	                <!--密码框:效果录入信息不可见-->
	                <input type="password" name="pwd"  />
	                <hr/>
                </p>
                    
                <p>
                	<!--单选按钮：
		                            注意：一组单选按钮，必须通过name属性来控制，让它们在一个分组中，然后在一个分组里只能选择一个
		                            正常状态下，提交数据为：gender=on ，后台不能区分你提交的数据
		                            不同的选项的value值要控制为不同，这样后台接收就可以区分了               
		                            默认选中：checked="checked"
                    -->
                	<a>性别：</a>
                	<input type="radio" name="gender" value="1" checked="checked"/>男
                	<input type="radio" name="gender" value="0"/>女
                	<hr />
                </p>	
                    
                <p>
                	<!--多选按钮:
			                            必须通过name属性来控制，让它们在一个分组中，然后在一个分组里可以选择多个
			                            不同的选项的value值要控制为不同，这样后台接收就可以区分了
			                            多个选项提交的时候，键值对用&符号进行拼接：例如下：
                            favlan=1&favlan=3
                    -->
                	<a>你喜欢的语言：</a>
                	<input type="checkbox" name="favlan" value="1" checked="checked"/>汉语
	                <input type="checkbox" name="favlan" value="2" checked="checked"/>英语
	                <input type="checkbox" name="favlan" value="3"/>法语
	                <input type="checkbox" name="favlan" value="4"/>韩语
	                <hr />
	                <!--文件-->
	                <input type="file" />
	                <hr />
	                <!--普通按钮：普通按钮没有什么效果，就是可以点击，以后学了js，可以加入事件-->
	                <input type="button" value="普通按钮" />
	                <hr />
	                <!--特殊按钮：重置按钮将页面恢复到初始状态-->
	                <input type="reset" />
	                <hr />
	                <!--特殊按钮：图片按钮-->
	                ![](img/phone_1.jpg)
	                <input type="image" src="img/phone_1.jpg" />
	                <hr />
	                <!--下拉列表  默认选中：selected="selected" 多选：multiple="multiple"-->  
	                <a>请选择你喜欢的城市：</a>    
	                <select name="city" multiple="multiple">
	                        <option value="0">---请选择---</option>
	                        <option value="1">哈尔滨市</option>
	                        <option value="2" selected="selected">青岛市</option>
	                        <option value="3">郑州市</option>
	                        <option value="4">西安市</option>
	                        <option value="5">天津市</option>
	                </select>
	                <hr />
                </p>    
           			      
                <p>
                	<!--多行文本框  利用css样式来控制大小不可变：style="resize: none;"-->  
                	<a>自我介绍：</a>
                	<textarea style="resize: none;" rows="10" cols="30">请在这里填写信息。。</textarea>
                	<br />
                	<hr />
                </p>

			    <p>
			    	<!--文本域-->
			    	<a>反馈：</a>
			        <textarea name="textarea" id="" cols="30" rows="10"></textarea>
			        <hr />
			        <!--文件域-->
			        <input type="file" name="files">
		        	<input type="button" value="上传" name="upload">
		        	<hr />
			    </p>
                	
                <p>
                	<!--label标签
                        	一般会在想要获得焦点的标签上加入一个id属性，然后label中的for属性跟id配合使用。
                	-->
                	<label for="uname">用户名：</label><input type="text" name="uername" id="uname"/>
                	<input type="submit" />
                	<hr />
                </p>
                
                
		        <p>
		        	<!--邮件验证-->
		        	<a>邮件：</a>
		            <input type="email" name="email"><br />
		            
		            <!--URL-->
		            <a>URL：</a>
		            <input type="url" name="url"><br />
		            
		            <!--数字-->
		            <a>数字：</a>
		            <input type="number" name="num" max="100" min="0" size="10"><br />
		            
		            <!--滑块-->
		            <a>音量：</a>
		            <input type="range" name="voice" min="0" max="100" step="2"><br />
		            
		            <!--搜索框-->
		            <a>搜索：</a>
		            <input type="search" name="search"><br />
		        </p>
            </form>
        </body>
</html>
```

效果图：

![image-20210216145456639](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621527_2e1c6b.webp)

![image-20210216145517151](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621704_d00e24.webp)

