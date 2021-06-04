# HTML 标签属性

## input

###### placeholder

在输入框中默认显示的内容

```html
<label>日期：</label>
<input type="text" name="date_field" 
placeholder="請輸入 DD/MM/YYYY的日期格式" />
```

![image-20210331081913292](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20210331081913292.png)

当用户开始输入时，placeholder就会消失

注意：设计时不要把 placeholder 来取代栏位的名称

提示输入信息也不要放到这里面，placeholder 适合用来放一个实例