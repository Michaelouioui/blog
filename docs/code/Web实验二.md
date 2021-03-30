# Web实验二



## 功能一：验证用户名是否已经存在

### AJAX

#### Servlet

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");

        //使用ACcountService中的方法通过用户名查找Account
        AccountService accountService = new AccountService();
        Account result = accountService.getAccount(username);

        //作为普通的字符串输出
        resp.setContentType("text/plain");

        //用PrintWriter进行输出
        PrintWriter out = resp.getWriter();

        if (result == null){
            //用户名可用，输出Not Exist
            out.print("Not Exist");
        } else{
            //用户名不可用，输出Exist
            out.print("Exist");
        }

        out.flush();  //关闭流
        out.close();  //释放资源
    }
```

#### JSP（HTML部分）

```jsp
<td>
	<input 
           id = "usernameID" （设置id方便对这玩意儿进行操作） 
           type="text"  
           name="username" 
           onblur="checkUsername();" （设置鼠标动作）
     />
	<span id = "usernameTips"></span>  （在同一行的后面显示）
</td>
```



#### JSP （样式设计）

```css
<style>
	.okTips{
		color: green;
	}

	.errorTips{
		color: red;
	}
</style>
```

#### JSP （JavaScript部分）

```jsp
<script>
	var xhr;  

	function checkUsername(){
		var username = document.getElementById("usernameID").value; //获得username的值

		// alert(username);  //弹窗警告
		// console.log(username);  //输出到console的log里面

		xhr = new XMLHttpRequest(); //new一下
		xhr.onreadystatechange = fun1; //回调函数

		xhr.open("GET", "usernameExist?username=" + username, true); //调用Servlet
		xhr.send(null);

	}

	function fun1(){
		if (xhr.readyState == 4){
			if (xhr.status == 200){
				var tips = document.getElementById("usernameTips"); //对应输入框后面那玩意儿
				var responseInfo = xhr.responseText;
				if (responseInfo == "Exist"){
					tips.className = 'errorTips';  //设置字体样式
					tips.innerText = "Invalid";  //设置文本内容
				} else if (responseInfo == "Not Exist"){
					tips.className = 'okTips';  //设置字体样式
					tips.innerText = "Available";  //设置文本内容
				}

			}
		}
	}
</script>
```

### 优化

#### 用JQuery对原有代码进行优化

```javascript
$(function (){
		$('#usernameID').on('blur', function (){  //确定对象，on后面带事件，后接function
			$.ajax({
				type	: "GET",  
				url		: "usernameExist?username=" + this.value,
				success	: function (data){
					console.log(data);
					if (data.code == 1){
						$('#usernameTips').attr("class", "errorTips").text('Invalid');
					} else if (data.code == 0){
						$('#usernameTips').attr("class", "okTips").text('Available');
					}
				}
			})
		});
	});
```

#### 使用Json对Servlet进行优化

##### result类

```java
private int code;
    private String msg;

    public Result(){

    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
```

##### Servlet

```java
	   resp.setContentType("text/json");  //输出为json格式的字符串，以便后面用fastJson进行转换

        PrintWriter out = resp.getWriter();

        Result result1 = new Result();

        if (result == null){
            //用户名可用
            result1.setCode(0);
            result1.setMsg("Not Exist");
        } else{
            result1.setCode(1);
            result1.setMsg("Exist");
            //用户名不可用
        }

        String str = JSON.toJSONString(result1);  //使用fastJson进行转换
```

## 功能二：实现商品搜索的自动补全

这里直接用了JQuery UI 来做

### 导入css

```css
<link rel="StyleSheet" href="css/jquery-ui.css" type="text/css" media="screen"/>
```

### 导入JQuery UI

```jsp
<script src="//code.jquery.com/ui/1.10.4/jquery-ui.js"></script>
<script src="http://libs.baidu.com/jquery/2.1.4/jquery.min.js"></script>
```

### 编写样式（抄的）

```jsp
<style>
    .ui-autocomplete-category {
        font-weight: bold;
        padding: .2em .4em;
        margin: .8em 0 .2em;
        line-height: 1.5;
    }
</style>
```

### 具体实现（官网文档）

```jsp
<script>
                $( function() {
                $.widget( "custom.catcomplete", $.ui.autocomplete, {
                _create: function() {
                this._super();
                this.widget().menu( "option", "items", "> :not(.ui-autocomplete-category)" );
                },
                _renderMenu: function( ul, items ) {
                var that = this,
                currentCategory = "";
                $.each( items, function( index, item ) {
                var li;
                if ( item.category != currentCategory ) {
                ul.append( "<li class='ui-autocomplete-category'>" + item.category + "</li>" );
                currentCategory = item.category;
                }
                li = that._renderItemData( ul, item );
                if ( item.category ) {
                li.attr( "aria-label", item.category + " : " + item.label );
                }
                });
                }
                });
                var data = [
                    { label: "Amazon Parrot", category: "Birds" },
                    { label: "Finch", category: "Birds" },
                    { label: "Koi", category: "Fish" },
                    { label: "Goldfish", category: "Fish" },
                    { label: "Angelfish", category: "Fish" },
                    { label: "Tiger Shark", category: "Fish" },
                    { label: "Bulldog", category: "Dogs" },
                    { label: "Chihuahua", category: "Dogs" },
                    { label: "Dalmation", category: "Dogs" },
                    { label: "Poodle", category: "Dogs" },
                    { label: "Golden Retriever", category: "Dogs" },
                    { label: "Labrador Retriever", category: "Dogs" },
                    { label: "Iguana", category: "Reptiles" },
                    { label: "Rattlesnake", category: "Reptiles" },
                    { label: "Persian", category: "Cats" },
                    { label: "Manx", category: "Cats" }
                ];

                $( "#inputID" ).catcomplete({
                delay: 0,
                source: data
                });
                } );
                </script>
```

## 功能三：修改数量弹窗直接完成