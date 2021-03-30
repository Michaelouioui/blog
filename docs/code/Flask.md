# 1 配置

这次使用动物系列书的FlaskWeb开发来进行学习

在构建虚拟环境的时候电脑的python出了一些问题，索性使用anaconda来进行python的安装以及虚拟环境的搭建，顺便还可以直接用他提供的库。

安装主要看的下面这一篇，讲的很全面。

https://blog.csdn.net/qq_41603102/article/details/84452375

------

好了，用anaconda之后后面的我又不知道咋办了

老老实实用按照书里的来

## 在git里面提取书里的代码

```
$git clone https://github.com/miguelgrinberg/flasky.git
```

然后使用git指令进行版本的切换

```
$cdflasky
$gitcheckout1a
```

## 创建虚拟环境

直接在cmd里面输入如下命令

```
py-3-mvenvvenv
```

即可创建一个命名为venv的文件夹，这个就是一个虚拟环境，包含这个项目专用的Python解释器

## 使用虚拟环境

激活命令

```
venv\Scripts\activate
```

输入之后cmd会有（venv）的前缀

## 使用pip安装python包

这里使用了清华大学的Python镜像库，快的一批

```
pip install flask -i https://pypi.tuna.tsinghua.edu.cn/simple
```

当切换到python之后能够正常导包之后就说明配置成功

## Pycharm配置

### 1 创建新项目

直接按照正常步骤创建一个新的project

### 2 创建虚拟环境

在创建项目的时候就可以直接生成虚拟环境（Pycharm牛逼！）

### 3 导包

File/Setting/Python interpreter

在+那里选择flask

可以用manage repository来切换下载源（经典清华大学镜像）

https://pypi.tuna.tsinghua.edu.cn/simple

导入其他包同理

### 4 在main函数中设置

```python
app = Flask(__name__)

if __name__ == '__main__':
    app.run()
```

# 2应用的基本结构

## 2.1初始化

所有Flask应用都必须创建一个应用实例

应用实例是Flask类的对象，通常由下述代码创建

```python
from flask import Flask
app = Flask(__name__)
```

## 2.2路由和视图函数

### 路由

客户端（例如Web浏览器）把请求发送给Web服务器，Web服务器再把请求发送给Flask应用实例。应用实例需要知道对每个URL的请求要运行哪些代码，所以保存了一个URL到Python函数的映射关系。

处理URL和函数之间关系的程序称为**路由**。

### app.route装饰器

定义路由最简便的方式是使用应用实例提供的**app.route装饰器**

```python
@app.route('/')
def index():
	return'<h1>Hello World!</h1>'
```

### 装饰器

装饰器是Python语言的标准特性。惯常用法是把函数注册为事件处理程序，在特定事件发生时调用。

### app.add_url_rule()方法

该方法接受三个参数：URL、端点名和视图函数

用app.add_url_rule()方法注册index()函数，其作用与前例相同：

```python
def index():
	return'<h1>HelloWorld!</h1>'
	
app.add_url_rule('/','index',index)
```

index()这样处理入站请求的函数称为**视图函数**。

如果应用部署在域名为www.example.com的服务器上，在浏览器中访问http://www.example.com后，会触发服务器执行index()函数。这个函数的返回值称为响应，是客户端接收到的内容。如果客户端是Web浏览器，响应就是显示给用户查看的文档。视图函数返回的响应可以是包含HTML的简单字符串，也可以是后文将介绍的复杂表单。

### app.route的动态表

### 示

```python
@app.route('/user/<name>')
defuser(name):
	return'<h1>Hello,{}!</h1>'.format(name)
```

路由URL中放在尖括号里的内容就是动态部分，任何能匹配静态部分的URL都会映射到这个路由上。调用视图函数时，Flask会将动态部分作为参数传入函数。在这个视图函数中，name参数用于生成个性化的欢迎消息。

路由中的动态部分默认使用字符串，不过也可以是其他类型。例如，路由/user/<int:id>只会匹配动态片段id为整数的URL，例如/user/123。Flask支持在路由中使用string、int、float和path类型。path类型是一种特殊的字符串，与string类型不同的是，它可以包含正斜线。

## 2.3一个完整的应用

```
git checkout 2a
```

## 2.4Web开发服务器

启动虚拟环境之后

```
(venv)$set FLASK_APP=hello.py
(venv)$flask run
```

服务器启动后便开始轮询，处理请求。直到按Ctrl+C键停止服务器，轮询才会停止。服务器运行时，在Web浏览器的地址栏中输入http://localhost:5000/。

## 2.5动态路由

通过添加动态路由实现访问时显示的个性化内容

```python
@app.route('/user/<name>')
defuser(name):
	return'<h1>Hello,{}!</h1>'.format(name)
```

```python
@app.route('/user/<name>')
defuser(name):
return'<h1>Hello,%s!</h1>'%name
```

上述两段代码的效果都是显示Hello，‘URL里面的name’

## 2.6调试模式

Flask应用可以在**调试模式**中运行。在这个模式下，开发服务器默认会加载两个便利的工具：**重载器**和**调试器**。

### 重载器

启用重载器后，Flask会监视项目中的所有源码文件，发现变动时自动重启服务器。在开发过程中运行启动重载器的服务器特别方便，因为每次修改并保存源码文件后，服务器都会自动重启，让改动生效。

### 调试器

调试器是一个基于Web的工具，当应用抛出未处理的异常时，它会出现在浏览器中。此时，Web浏览器变成一个交互式栈跟踪，你可以在里面审查源码，在调用栈的任何位置计算表达式。

调试模式默认关闭，如需启用

```
setFLASK_DEBUG=1
flaskrun
```

使用app.run()方法启动服务器时，不会用到FLASK_APP和FLASK_DEBUG环境变量。若想以编程的方式启动调试模式，就使用app.run(debug=True)。

## 2.7命令行选项

------

#### 查看选项

```
flask--help
```

#### 打开Pythonshell会话

```
flaskshell
```

#### 在Web开发服务器中运行应用

```
flaskrun
```

#### 查看服务器接口

```
flask--host
```

------

## 2.8请求——响应循环

### 2.8.1应用和请求上下文

Flask从客户端收到请求时，要让视图函数能访问一些对象，这样才能处理请求。请求对象就是一个很好的例子，它封装了客户端发送的HTTP请求。

为了避免大量可有可无的参数把视图函数弄得一团糟，Flask使用上下文临时把某些对象变为全局可访问。

```python
fromflaskimportrequest

@app.route('/')
defindex():
	user_agent=request.headers.get('User-Agent')
	return'<p>Yourbrowseris{}</p>'.format(user_agent)
```

在上面的视图函数中，我们把request当作全局变量使用，事实上，request不可能是全局变量。

Flask使用上下文让特定的变量在一个线程中全局可访问，与此同时却不会干扰其他线程。

#### 线程

线程是可单独管理的最小指令集。进程经常使用多个活动线程，有时还会共享内存或文件句柄等资源。多线程Web服务器会创建一个线程池，再从线程池中选择一个线程处理接收到的请求。

#### 上下文

在Flask中有两种上下文：**应用上下文**和**请求上下文**

------

##### current_app

应用上下文，当前应用的应用实例

##### g

应用上下文，处理请求时用作临时存储的对象，每次请求都会重设这个变量

##### request

请求上下文，请求对象，封装了客户端发出的HTTP请求中的内容

##### session

请求上下文，用户会话，值为一个字典，存储请求之间需要“记住”的值

------

Flask在分派请求之前**激活**（或**推送**）**应用**和**请求上下文**，请求处理完成后再将其**删除**。应用上下文被推送后，就可以在当前线程中使用current_app和g变量。类似地，请求上下文被推送后，就可以使用request和session变量。如果使用这些变量时没有激活应用上下文或请求上下文，就会导致错误。

```python
>>>fromhelloimportapp
>>>fromflaskimportcurrent_app
>>>current_app.name
Traceback(mostrecentcalllast):
...
RuntimeError:workingoutsideofapplicationcontext
>>>app_ctx=app.app_context()
>>>app_ctx.push()
>>>current_app.name
'hello'
>>>app_ctx.pop()
```

上述例子中，没激活应用上下文之前就调用current_app.name会导致错误，但推送玩上下文之后就可以调用了。

注意，获取应用上下文的方法是在应用实例上调用app.app_context()。

### 2.8.2请求分派

应用收到客户端发来的请求时，要找到处理该请求的视图函数。为了完成这个任务，Flask会在应用的URL映射中查找请求的URL。URL映射是URL和视图函数之间的对应关系。

Flask使用app.route装饰器或者作用相同的app.add_url_rule()方法构建映射。

可以在Pythonshell中查看应用的URL映射

```python
(venv)$python
>>>fromhelloimportapp
>>>app.url_map
Map([<Rule'/'(HEAD,OPTIONS,GET)->index>,
<Rule'/static/<filename>'(HEAD,OPTIONS,GET)->static>,
<Rule'/user/<name>'(HEAD,OPTIONS,GET)->user>])
```

/和/user/<name>路由在应用中使用app.route装饰器定义。/static/<filename>路由是Flask添加的特殊路由，用于访问静态文件。

URL映射中的(HEAD,OPTIONS,GET)是请求方法，由路由进行处理。

每个请求都有对应的处理方式，这通常表示客户端想让服务器执行什么样的操作。Flask为每个路由都制定了请求方法，这样即使不同的请求方法发送到相同的URL上时，也会使用不同的视图函数处理。HEAD和OPTIONS方法由Flask自动处理，因此可以这么说，在这个应用中，URL映射中的3个路由都使用GET方法（表示客户端想请求信息，例如一个网页）

### 2.8.3请求对象

------

![image-20210304141828093](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210304141828093.png)

![image-20210304141846634](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210304141846634.png)

------

#### 2.8.4请求钩子

有时在处理请求之前或之后执行代码会很有用。例如，在请求开始时，我们可能需要创建数据库连接或者验证发起请求的用户身份。为了避免在每个视图函数中都重复编写代码，Flask提供了注册通用函数的功能，注册的函数可在请求被分派到视图函数之前或之后调用。

请求钩子通过装饰器实现，Flask支持以下4种钩子

------

##### before_request

注册一个函数，在每次请求之前运行

##### before_first_request

注册一个函数，只在处理第一个请求之前运行，通过这个钩子添加服务器初始化任务

##### after_repuest

注册一个函数，如果没有未处理的一场抛出，在每次请求之后运行

##### teardown_request

注册一个函数，即使有未处理的异常抛出，也在每次请求之后运行

------

在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量g。

例如：before_request处理程序可以哦那个数据库中加载已登录用户，并将其保存到d.user中，随后调用视图函数时，便可以通过g.user获取用户。

### 2.8.5响应

Flask调用视图函数后，会将其返回值作为响应的内容。多数情况下，响应就是一个简单的字符串，作为HTML页面回送客户端。

但HTTP协议需要的不仅是作为请求响应的字符串。HTTP响应中一个很重要的部分是状态码，Flask默认设为200，表明请求已被成功处理。

如果视图函数返回的响应需要使用不同的状态码，可以把数字代码作为第二个返回值，添加到响应文本之后。例如，下述视图函数返回400状态码，表示请求无效：

```python
@app.route('/')defindex():
	return'<h1>BadRequest</h1>',400
```

如果不想返回由1个、2个或3个值组成的元组，Flask视图函数还可以返回一个响应对象。

make_response()函数可接受1个、2个或3个参数（和视图函数的返回值一样），然后返回一个等效的响应对象。有时我们需要在视图函数中生成响应对象，然后在响应对象上调用各个方法，进一步设置响应。下例创建一个响应对象，然后设置cookie：

```python
fromflaskimportmake_response

@app.route('/')
defindex():
	response=make_response('<h1>Thisdocumentcarriesacookie!</h1>')
response.set_cookie('answer','42')
	returnresponse
```

#### Flask响应对象

------

![image-20210304141747722](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210304141747722.png)

------

#### 重定向

响应有个特殊的类型，称为重定向，这种响应没有页面文档，只会告诉浏览器一个新URL，用以加载新页面，重定向经常在Web表单中使用，状态码通常是302，在Location首部中提供目标URL。重定向响应可以使用3个值形式的返回值生成，也可在响应对象中设定，不过，由于使用频繁，Flask提供了redirect()辅助函数，用以生成这种响应

```python
fromflaskimportredirect

@app.route('/')
defindex():
	returnredirect('http://www.example.com')
```

还有一种特殊的响应由abort()函数生成，用于处理错误。在下面这个例子中，如果URL中动态参数id对应的用户不存在，就返回状态码404：

```python
fromflaskimportabort

@app.route('/user/<id>')
defget_user(id):
	user=load_user(id)
	ifnotuser:
		abort(404)
	return'<h1>Hello,{}</h1>'.format(user.name)
```

注意，abort()不会把控制权交还给调用它的函数，而是抛出异常。

## 2.9Flask扩展

全是废话

------

# 3模板

以用户注册账号为例，用户在表单中输入相关信息并且提交的过程称为**业务逻辑**，服务器接收到请求之后所作出的一系列反应称为**表现逻辑**

把表现逻辑移到模板中能提升应用的可维护性

模板是包含相应文本的文件，其中包含用占位变量表示的动态部分，其具体值旨在请求的上下文中才能知道，使用真实值替换变量，在返回最终得到的响应字符串，这一过程称为渲染。为了渲染模板，Flask使用一个名为Jinja2的强大模板引擎。

## 3.1Jinja2模板引擎

形式最简单的Jinja2模板就是一个包含响应文本的文件。

#### 示例3-1

```html
<h1>Hello World!</h1>
```

这个和之前index()视图函数的响应一样

#### 示例3-2

```html
<h1>Hello,{{ name }}!</h1>
```

这个实例包含一个变量，和之前含有变量的视图函数一样

### 3.1.1 渲染模板

默认情况下，Flask在应用目录中的templates子目录里寻找模板。

#### 实例3-3

```python
fromflaskimportFlask,render_template

#...

@app.route('/')
defindex():
	return render_template('index.html')

@app.route('/user/<name>')
defuser(name):
	return render_template('user.html',name = name)
```

Flask提供的render_template()函数把Jinja2模板引擎集成到了应用中。这个函数的第一个参数是模板的文件名，随后的参数都是键–值对，表示模板中变量对应的具体值。在这段代码中，第二个模板收到一个名为name的变量。

name=name,左边的name表示参数名，也就是模板中的占位符，右边的name是当前作用域中的变量，表示同名参数的值。

### 3.1.2 变量

{{ name }}表示一个变量，这是一种特殊的占位符，告诉模板引擎这份位置的值从渲染模板时使用的数据中获取。

Jinja2能识别所有类型的变量，包括列表、字典、对象等

#### 实例

```python
<p>Avaluefromadictionary:{{mydict['key']}}.</p>
<p>Avaluefromalist:{{mylist[3]}}.</p>
<p>Avaluefromalist,withavariableindex:{{mylist[myintvar]}}.</p>
<p>Avaluefromanobject'smethod:{{myobj.somemethod()}}.</p>
```

变量的值可以使用过滤器修改，过滤器添加在变量名之后，二者以竖线分隔，例如，下述模板把name变量的值编程首字母大写的形式

```python
Hello,{{name|capitalize}}
```

#### 常用Jinja2变量过滤器

![image-20210304141647440](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210304141647440.png)

safe过滤器值得特别说明一下。默认情况下，出于安全考虑，Jinja2会转义所有变量。例如，如果一个变量的值为'<h1>Hello</h1>'，Jinja2会将其渲染成'&l（）t;h1&g（）t;Hello&lt（）;/h1&gt（）;'，浏览器能显示这个h1元素，但不会解释它。很多情况下需要显示变量中存储的HTML代码，这时就可使用safe过滤器。

### 3.1.3 控制结构

#### if-else

```python
{%if user %}
	Hello,{{ user }}!
{% else %}
	Hello,Stranger!
{% endif %}
```

#### for

```python
<ul>
{%forcommentincomments%}
	<li>{{comment}}</li>
{%endfor%}
</ul>
```

#### 宏（类似Python代码中的函数）

```python
{%macrorender_comment(comment)%}
	<li>{{comment}}</li>
{%endmacro%}
<ul>
	{%forcommentincomments%}
		{{render_comment(comment)}}
	{%endfor%}
</ul>
```

#### 可以把宏保存在单独的文件，在需要使用的模板导入

```python
{%import'macros.html'asmacros%}
<ul>
	{%forcommentincomments%}
		{{macros.render_comment(comment)}}
	{%endfor%}
</ul>
```

#### 需要在多出重复使用的模板代码片段可以写入单独的文件，再引入到所有模板中，以避免重复

```python
{%include'common.html'%}
```

#### 模板继承（类似Python的类继承）

##### 创建一个名为base.html的基模板

```html
<html>
<head>
	{%blockhead%}
	<title>
	{%blocktitle%}{%endblock%}-My-Application
	</title>
	{%endblock%}
</head>
<body>
	{%blockbody%}
	{%endblock%}
</body>
</html>
```

基模板中定义的区块可在衍生模板中覆盖。Jinja2使用**block**和**endblock**指令在基模板中定义内容区块。在本例中，我们定义了名为head\title和body的区块。注意，title包含在head中。

##### 基模板的衍生模板

```python
{%extends"base.html"%}//声明该模板衍生自base.html

{%blocktitle%}Index{%endblock%}

{%blockhead%}
	{{super()}}  //引用基模版中同名块中的内容
	<style>
	</style>
{%endblock%}
{%blockbody%}
<h1>Hello,World!</h1>
{%endblock%}
```

在 extends 指令之后，基模板中的3个区块被重新定义，模板引擎会将其插入适当的位置。如果基模板和衍生模板中的同名区块中都有内容，衍生模板中的内容将显示出来。

## 3.2 使用Flask-Bootstrap集成Bootstrap

Bootstrap时客户端框架，因此不会直接涉及服务器。服务器需要做的只是提供引用了Bootstrap的CSS和JavaScript文件的HTML响应，并在HTML、CSS、JavaScript代码中实例化所需的用户界面元素。这些操作最理想的执行场景就是模板。

#### 安装Flask-Bootstrap

```
(venv) $ pip install flask-bootstrap
```

#### 初始化Flask-Bootstrap

```python
from flask_bootstrap import Bootstrap 
# ...
bootstrap = Bootstrap(app)
```

#### 实例 3-5

```html
{% extends "bootstrap/base.html" %} 

{% block title %}Flasky{% endblock %} 

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation"> 
	<div class="container"> 
		<div class="navbar-header">
			<button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse"> 					<span class="sr-only">Toggle navigation</span>
				<span class="icon-bar"></span> 
				<span class="icon-bar"></span> 
				<span class="icon-bar"></span>
			</button> 
			<a class="navbar-brand" href="/">Flasky</a> 		</div>
		<div class="navbar-collapse collapse"> 
			<ul class="nav navbar-nav"> 
				<li><a href="/">Home</a></li>
			</ul> 
		</div> 
	</div> 
</div> 
{% endblock %}

{% block content %}
<div class="container"> 
	<div class="page-header"> 
		<h1>Hello, {{ name }}!</h1>
	</div> 
</div>
{% endblock %}
```

#### Flask-Bootstrap基模板中定义的区块

![image-20210305213212026](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210305213212026.png)

#### 注意

Bootstrap 的 CSS 和 JavaScript 文件在 styles 和 scripts 区块中声明。如果应用需要向 已经有内容的块中添加新内容，必须使用 Jinja2 提供的 super() 函数。

如果要在衍生模板中添加新的 JavaScript 文件，需要这么定义 scripts 区块：

```html
{% block scripts %} 
{{ super() }}
<script type="text/javascript" src="my-script.js"></script>
{% endblock %}
```

## 3.3 自定义错误页面

使用app.errirhandler装饰器为这两个错误提供自定义的处理函数

#### 实例 3-6 hello.py

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404
```

与视图函数一样，错误处理函数也返回一个响应。此外，错误处理函数还要返回与错误对 应的数字状态码。状态码可以直接通过第二个返回值指定。

#### 二级基模板

可以将一个继承自bootstrap/base.html的模板作为其他模板的二级基模板

#### 示例 3-7  templates/base.html

```html
{% extends "bootstrap/base.html" %}

{% block title %}Flasky{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Flasky</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
<div class="container">
    {% block page_content %}{% endblock %}
</div>
{% endblock %}
```

#### 实例 3-8 templates/404.html

```html
{% extends "base.html" %}

{% block title %}Flasky - Page Not Found{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Not Found</h1>
</div>
{% endblock %}
```

## 3.4 链接

#### url_for()

使用应用的URL映射中保存的信息生成URL

以视图函数名（或者app.add_url_route()定义路由时使用的端点名）作为参数，返回对应的URL。

例如：

调用url_for('index')得到的结果是/，调用url_for('index', _external = True)返回值是http://localhost:5000/

使用url_for()生成动态URL时，将动态部分作为关键字参数传入。例如，url_for('user', name = 'John', _external = True)的返回结果是

http://localhost:5000/user/John

传给 url_for() 的关键字参数不仅限于动态路由中的参数，非动态的参数也会添加到查询字符串中。

例如，url_for('user', name='john', page=2, version=1) 的返回结果是 

/user/john?page=2&version=1。

## 3.5 静态文件

默认设置下，Flask 在应用根目录中名为 static 的子目录中寻找静态文件。如果需要，可在 static 文件夹中使用子文件夹存放文件。服务器收到映射到 static 路由上的 URL 后，生成的响应包含文件系统中对应文件里的内容。

#### 实例 3-10

```html
{% block head %}
{{ super() }}
<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
<link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
{% endblock %}
```

## 3.6 使用Flask-Moment本地化日期和时间

```html
main.py
from datetime import datetime

index.html
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}.</p>

```



# 4 Web 表单

## 4.1 配置

安装flask-wtf

要求应用配置一个密钥

```python
app = Flask(__name__) 
app.config['SECRET_KEY'] = 'hard to guess string'
```

## 4.2 表单类

使用 Flask-WTF 时，在服务器端，每个 Web 表单都由一个继承自 FlaskForm的类表示。这 个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个验证函数。验证函数用于验证用户提交的数据是否有效。

StringField 类表示属性为 type="text" 的 HTML <input> 元素。SubmitField 类表示属性 为 type="submit" 的 HTML <input> 元素。字段构造函数的第一个参数是把表单渲染成HTML时使用的标注（label）

```
class NameForm(FlaskForm): 
	name = StringField('What is your name?', validators=[DataRequired()])
	submit = SubmitField('Submit')
```

StringField 构造函数中的可选参数 validators 指定一个由验证函数组成的列表，在接受 用户提交的数据之前验证数据。验证函数 DataRequired() 确保提交的字段内容不为空。

#### 表 4-1：WTForm支持的HTML标准字段

![image-20210308170150485](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308170150485.png)

#### 表 4-2：WTForm验证函数

![image-20210308170230593](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308170230593.png)

## 4.3 把表单渲染成HTML

表单字段是可调用的，在模板中调用后会渲染成 HTML。假设视图函数通过 form 参数把 一个 NameForm 实例传入模板，在模板中可以生成一个简单的HTML表单

调用字段时传入的任何关键字参数都将转换成字 段的HTML属性。例如，可以为字段指定 id 或 class 属性，然后为其定义 CSS 样式：

Flask-Bootstrap 扩展提供了一个高层级的 辅助函数，可以使用Bootstrap 预定义的表单样式渲染整个 Flask-WTF 表单，而这些操作
只需一次调用即可完成。使用 Flask-Bootstrap，上述表单可以用下面的方式渲染：

```html
{% import "bootstrap/wtf.html" as wtf %} 
{{ wtf.quick_form(form) }}
```

## 4.4 在视图函数中处理表单

使用 GET 和 POST 请求方法处理 Web 表单

```python
@app.route('/', methods=['GET', 'POST']) 
def index(): 
	name = None
	form = NameForm()
	if form.validate_on_submit(): 
		name = form.name.data 
		form.name.data = ''
	return render_template('index.html', 
							form=form, 										name=name)
```

app.route 装饰器中多出的 methods 参数告诉 Flask，在URL映射中把这个视图函数注册为 GET 和 POST 请求的处理程序。如果没指定 methods参数，则只把视图函数注册为 GET 请求的处理程序。

## 4.5 重定向和用户会话

这种需求的实现方式是，使用重定向作为 POST 请求的响应，而不是使用常规响应。重定向 是一种特殊的响应，响应内容包含的是 URL，而不是HTML代码的字符串。浏览器收到 这种响应时，会向重定向的 URL 发起 GET 请求，显示页面的内容。这个页面的加载可能 要多花几毫秒，因为要先把第二个请求发给服务器。除此之外，用户不会察觉到有什么不 同。现在，前一个请求是 GET 请求，所以刷新命令能像预期的那样正常运作了。这个技巧称为 Post / 重定向 /Get 模式。

#### 用重定向对上一小节的代码进行优化

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    name = None
    age = 20
    form = NameForm()
    if form.validate_on_submit():
        session['name'] = form.name.data
        session['age'] = form.age.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'), age=session.get('age'))
```

## 4.6 闪现消息

#### 实例 4-6 main.py

```python
flash('Looks like you have changed your name!')
```

#### 实例 4-7 base.html

```html
{% block content %}
<div class="container">
    {% for message in get_flashed_messages() %}
    <div class = "alert alert-warning">
        <button type="button" class="close" data-dismiss="alert">×</button>
        {{ message }}
    </div>
    {% endfor %}

    {% block page_content %}{% endblock %}
</div>
{% endblock %}
```

这个实例使用Bootstrap提供的CSS alert 样式渲染警告消息

# 5 数据库

## 5.5 使用 Flask-SQLAlchemy管理数据库

Flask-SQLAlchemy 是一个 Flask 扩展，简化了在 Flask 应用中使用 SQLAlchemy 的操作。 SQLAlchemy 是一个强大的关系型数据库框架，支持多种数据库后台。SQLAlchemy 提供了高层ORM，也提供了使用数据库原生 SQL 的低层功能。

在 Flask-SQLAlchemy 中，数据库使用 URL 指定。

![image-20210308184434558](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308184434558.png)

在这些 URL 中，hostname 表示数据库服务所在的主机，可以是本地主机（localhost），也 可以是远程服务器。数据库服务器上可以托管多个数据库，因此 database 表示要使用的数 据库名。如果数据库需要验证身份，使用 username 和 password 提供数据库用户的凭据。
SQLite 数据库没有服务器，因此不用指定 hostname、username 和 password。URL中的 database 是磁盘中的文件名。

应用使用的数据库 URL 必须保存到 Flask 配置对象的SQLALCHEMY_DATABASE_URI 键中。 Flask-SQLAlchemy 文档还建议把 SQLALCHEMY_TRACK_MODIFICATIONS 键设为 False，以便在 不需要跟踪对象变化时降低内存消耗。其他配置选项的作用参阅 Flask-SQLAlchemy 的文档。

#### 配置数据库

```python
import os
from flask_sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))

app.config['SQLALCHEMY_DATABASE_URI'] =\
    'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

db 对象是 SQLAlchemy 类的实例，表示应用使用的数据库，通过它可获得 Flask-SQLAlchemy 提供的所有功能。

## 5.6 定义模型

模型这个术语表示应用使用的持久化实体。在ORM中，模型一般是一个 Python 类，类中的属性对应于数据库表中的列。

Flask-SQLAlchemy 创建的数据库实例为模型提供了一个基类以及一系列辅助类和辅助函 数，可用于定义模型的结构。图 5-1 中的 roles 表和 users 表可像示例 5-2 那样，定义为Role 和 User 模型。

#### 实例 5-2 happy.py：定义Role和User模型

```python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.String(64), unique=True)

    def __repr__(self):
        return '<Role %r>' % self.name

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)

    def __repr__(self):
        return '<User %r' % self.username
```

#### 表 5-2 最常用的SQLAlchemy列类型

![image-20210308190212512](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308190212512.png)

![image-20210308190225163](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308190225163.png)

db.Column的其余参数指定属性的配置选项

#### 表 5-3：最常用的SQLAlchemy列选项

![image-20210308190316807](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308190316807.png)

Flask-SQLAlchemy 要求每个模型都定义主键，这一列经常命名为 id。

虽然没有强制要求，但这两个模型都定义了 __repr()__ 方法，返回一个具有可读性的字符串表示模型，供调试和测试时使用。

## 5.7 关系

```python
users = db.relationship('User', backref = 'role')

role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```

#### 表 5-4 常用的SQLAlchemy关系选项

![image-20210308191455399](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308191455399.png)

除了一对多之外，还有其他几种关系类型。一对一关系可以用前面介绍的一对多关系表 示，但调用 db.relationship() 时要把 uselist 设为 False，把“多”变成“一”。多对一 关系也可使用一对多表示，对调两个表即可，或者把外键和 db.relationship() 都放在 “多”这一侧。最复杂的关系类型是多对多，需要用到第三张表，这个表称为关联表（或联结表）。

## 5.8 数据库操作

### 5.8.1 创建表

首先要让Flask-SQLAlchemy 根据模型类创建数据库。db.create_all() 函数将寻找所有 db.Model 的子类，然后在数据库中创建对应的表：

#### shell

```shell
#创建对应的表
from main import db
db.create_all()

#删除表
db.drop_all()
```

### 5.8.2 插入行

创建一些角色和用户

```
from main import Role, User
admin_role = Role(name='Admin')
mod_role = Role(name='Moderator')
user_role = Role(name='User')
user_john = User(username='john', role=admin_role)
user_susan = User(username='susan', role=user_role)
user_david = User(username='david', role=user_role)
```

模型的构造函数接受的参数是使用关键字参数指定的模型属性初始值。注意，role 属性 也可使用，虽然它不是真正的数据库列，但却是一对多关系的高级表示。新建对象时没有明确设定 id 属性，因为在多数数据库中主键由数据库自身管理。现在这些对象只存在于Python 中，还未写入数据库。因此，id 尚未赋值：

```
>>> print(admin_role.id) 
None 
>>> print(mod_role.id) 
None 
>>> print(user_role.id)
None
```

对数据库的改动通过数据库会话管理，在 Flask-SQLAlchemy 中，会话由 db.session 表示。 准备把对象写入数据库之前，要先将其添加到会话中：

```
>>> db.session.add(admin_role) 
>>> db.session.add(mod_role) 
>>> db.session.add(user_role) 
>>> db.session.add(user_john) 
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```

简写为

```
>>> db.session.add_all([admin_role, mod_role, user_role, ... user_john, user_susan, user_david])
```

为了把对象写入数据库，我们要调用 commit() 方法提交会话：

```
>>> db.session.commit()
```

提交数据后再查看 id 属性，现在它们已经赋值了：

```
>>> print(admin_role.id) 
1
>>> print(mod_role.id) 
2
>>> print(user_role.id)
3
```

数据库会话 db.session 和第 4 章介绍的 Flask session 对象没有关系。数据库会话也称为事务。

数据库会话能保证数据库的一致性。提交操作使用原子方式把会话中的对象全部写入数据 库。如果在写入会话的过程中发生了错误，那么整个会话都会失效。如果你始终把相关改 动放在会话中提交，就能避免因部分更新导致的数据库不一致。

数据库会话也可回滚。调用 **db.session.rollback()** 后，添加到数据库会话
中的所有对象都将还原到它们在数据库中的状态。

### 5.8.3 修改行

在数据库会话上调用 add() 方法也能更新模型。我们继续在之前的 shell 会话中进行操作， 下面这个例子把 "Admin" 角色重命名为 "Administrator"：

```
>>> admin_role.name = 'Administrator' 
>>> db.session.add(admin_role)
>>> db.session.commit()
```

### 5.8.4 删除行

数据库会话还有个 delete() 方法。下面这个例子把 "Moderator" 角色从数据库中删除：

```
>>> db.session.delete(mod_role) 
>>> db.session.commit()
```

注意，删除与插入和更新一样，提交数据库会话后才会执行。

### 5.8.5 查询行

Flask-SQLAlchemy为每个模型类都提供了query对象。最基本的模型查询是使用all ( ) 方法 取回对应表中的所有记录：

```
>>> Role.query.all() 
[<Role 'Administrator'>, <Role 'User'>] 
>>> User.query.all()
[<User 'john'>, <User 'susan'>, <User 'david'>]
```

使用过滤器可以配置 query 对象进行更精确的数据库查询。下面这个例子查找角色为 "User" 的所有用户：

```
>>> User.query.filter_by(role=user_role).all() 
[<User 'susan'>, <User 'david'>]
```

若想查看 SQLAlchemy 为查询生成的原生 SQL 查询语句，只需把 query 对象转换成字符串：

```
>>> str(User.query.filter_by(role=user_role)) 
'SELECT users.id AS users_id, users.username AS users_username, users.role_id AS users_role_id \nFROM users \nWHERE :param_1 = users.role_id'
```

如果你退出了 shell 会话，前面这些例子中创建的对象就不会以 Python 对象的形式存在， 但在数据库表中仍有对应的行。如果打开一个新的 shell 会话，要从数据库中读取行，重新创建 Python 对象。下面这个例子发起一个查询，加载名为 "User" 的用户角色：

```
>>> user_role = Role.query.filter_by(name='User').first()
```

注意，这里发起查询的不是 all() 方法，而是 first() 方法。all() 方法返回所有结果构成 的列表，而 first() 方法只返回第一个结果，如果没有结果的话，则返回 None。因此，如果知道查询最多返回一个结果，就可以用这个方法。

filter_by() 等过滤器在 query 对象上调用，返回一个更精确的 query 对象。多个过滤器可以一起调用，直到获得所需结果。

#### 表 5-5：常用的SQLAlchemy查询过滤器

![image-20210308194310294](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308194310294.png)

在查询上应用指定的过滤器后，调用 all() 方法将执行查询，以列表的形式返回结果。

#### 表 5-6：最常用的SQLAlchemy查询执行方法

![image-20210308194408526](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308194408526.png)

```
>>> users = user_role.users 
>>> users
[<User 'susan'>, <User 'david'>] 
>>> users[0].role
<Role 'User'>
```

## 5.9 在视图函数中操作数据库

#### 实例 5-5：首页路由的新版本

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            db.session.commit()
            session['known'] = False
        else:
            session['known'] = True
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'),
                           known=session.get('known', False))
```

实例 5-6 index.html

```html
{% if not known %}
    <p>oui oui</p>
    {% else %}
    <p>oui</p>
    {% endif %}
```

## 5.10 集成Python shell

可以让flask shell命令自动导入这些对象

若想把对象添加到导入列表中，必须使用 app.shell_context_processor 装饰器创建并注册 一个 shell 上下文处理器

```python
@app.shell_context_processor
def make_shell_context():
    return dict(db=db, User=User, Role=Role)
```

## 5.11 使用Flask-Migrate实现数据库迁移

SQLAlchemy 的开发人员编写了一个迁移框架，名为 Alembic。除了直接使用 Alembic 之 外，Flask 应用还可使用 Flask-Migrate 扩展。这个扩展是对 Alembic 的轻量级包装，并与flask 命令做了集成。

### 5.11.1 创建迁移仓库

安装Flask-Migrate

添加数据库迁移支持

```
flask db init
```

### 5.11.2 创建迁移脚本

#### upgrade ( )

把迁移中的改动应用到数据库中

#### downgrade ( )

将改动删除

Alembic经常有添加和删除改动的能力，意味着数据库可重设到修改历史的任意一点

我们可以使用 revision 命令手动创建 Alembic 迁移，也可使用 migrate 命令自动创建。 手动创建的迁移只是一个骨架，upgrade() 和 downgrade() 函数都是空的，开发者要使用 Alembic 提供的 Operations 对象指令实现具体操作。自动创建的迁移会根据模型定义和数据库当前状态之间的差异尝试生成 upgrade() 和 downgrade() 函数的内容。

#### 使用 Flask-Migrate 管理数据库模式变化的步骤如下。

1. 对模型类做必要的修改
2. 执行flask db migrate命令，自动创建一个迁移脚本
3. 检查自动生成的脚本，根据对模型的实际改动进行调整
4. 把迁移脚本纳入版本控制
5. 执行flask db upgrade 命令用于自动创建迁移脚本

flask db migrate 子命令用于自动创建迁移脚本：

```
flask db migrate -m "initial migration"
```

### 5.11.3 更新数据库

检查并修正好迁移脚本之后，执行flask db upgrade 命令，把迁移应用到数据库中

如果你按照之前的说明操作过，那么已经使用 db.create_all() 函数创建了 数据库文件。此时，flask db upgrade 命令将失败，因为它试图创建已经存 在的数据库表。一种简单的处理方法是，把 data.sqlite 数据库文件删掉，然 后执行 flask db upgrade 命令，通过迁移框架重新创建数据库。另一种方法 是不执行 flask db upgrade 命令，而是使用 flask db stamp 命令把现有数据库标记为已更新。

### 5.11.4 添加几个迁移

修改数据库的步骤

1. 对数据库模型做必要的修改。
2. 执行 flask db migrate 命令，生成迁移脚本。
3. 检查自动生成的脚本，改正不准确的地方。
4. 执行 flask db upgrade 命令，把改动应用到数据库中。

实现一个功能时，可能要多次修改数据库模型才能得到预期结果。如果前一个迁移还未提 交到源码控制系统中，可以继续在那个迁移中修改，以免创建大量无意义的小迁移脚本。在前一个迁移脚本的基础上修改的步骤如下。

1. 执行 flask db downgrade 命令，还原前一个脚本对数据库的改动（注意，这可能导致 部分数据丢失）。
2. 删除前一个迁移脚本，因为现在已经没什么用了。
3. 执行 flask db migrate 命令生成一个新的数据库迁移脚本。这个迁移脚本除了前面删 除的那个脚本中的改动之外，还包括这一次对模型的改动。
4. 根据前面的说明，检查并应用迁移脚本。

# 6 电子邮件

#### 使用Flask-Mail提供电子邮件支持

Flask-Mail 连接到简单邮件传输协议（SMTP，simple mail transfer protocol）服务器，把邮 件交给这个服务器发送。如果不进行配置，则 Flask-Mail 连接 localhost 上的 25 端口，无须验证身份即可发送电子邮件。表 6-1 列出了可用来设置 SMTP 服务器的配置。

#### 表 6-1：Flask-Mail SMTP服务器的配置

![image-20210308203824326](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210308203824326.png)

#### QQ邮箱的相关配置

首先在qq邮箱中打开SMTP服务，获取授权码，后面项目中的PASSWORD就是这个授权码

```python
# 设置电子邮件相关配置
app.config['MAIL_SERVER'] = 'smtp.qq.com'
app.config['MAIL_PORT'] = 465
app.config['MAIL_USE_SSL'] = True  #qq邮件使用SSL，必须要选
app.config['MAIL_USE_TLS'] = False
app.config['MAIL_USERNAME'] = '649193646@qq.com'
app.config['MAIL_PASSWORD'] = 'fzwuxnwlqurqbdcb'
app.config['MAIL_DEFAULT_SENDER'] = '649193646@qq.com'

mail = Mail(app)
```

```
from main import *
from flask_mail import Message
msg = Message('oui', sender = '649193646@qq.com', recipients=['michaellinht@outlook.com'])
msg.body = 'oui oui'
with app.app_context():
    mail.send(msg)
```











# 文档

#### Moment.js

http://momentjs.com/docs/#/displaying/



#### 清华镜像

https://pypi.tuna.tsinghua.edu.cn/simple



# 安装的插件

- Flask
- Flask-Bootstrap
- Flask-Moment
- Flask-SQLAlchemy
- Flask-WTF
- Flask-Migrate
- Flask-Mail



