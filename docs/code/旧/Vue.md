script 中 # 是id选择器

#### 示例1 利用选择器与函数对标签对象进行操作

```html
<head>
    <title>hello</title>
</head>

<body>

    <div id="app">
        <h2 @click="eat">{{ food }}</h2>
        <input type="button" id="inputbtn" value="hello" @click="input">
    </div>


    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

    <script>
        var app = new Vue({
            el: '#app',
            data : {
                food : "beef"
            },
            methods : {
                eat:function(){
                    this.food += " eat";
                },
                input:function(){
                    alert("hhh");
                }
            }
        })
    </script>
</body>
```

#### 示例2 更改标签的显示情况

```html
<head>
    <title>hello</title>
</head>

<body>

    <div id="app">
        <h2 @click="eat">{{ food }}</h2>
        <input type="button" id="inputbtn" value="hello" @click="input">
        <input type="button" @click="showBtn" value="点击更改显示">
        <h2 v-show="isShow" >hhhhhhhhhhhhhhhhhhh</h2>
    </div>


    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

    <script>
        var app = new Vue({
            el: '#app',
            data : {
                food : "beef",
                isShow : false
            },
            methods : {
                eat:function(){
                    this.food += " eat";
                },
                input:function(){
                    alert("hhh");
                },
                showBtn:function(){
                    if (this.isShow) {
                        this.isShow = false;
                    } else {
                        this.isShow = true;
                    }
                }
            }
        })
    </script>
</body>
```

