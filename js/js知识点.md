#js知识点



### ajax语法	

```
$.ajax({

url:’’,

type:’’,

data:{},

success:function(arg){

// arg是个字符串类型

var obj = JSON.parse(arg)

}

})

$.ajax({

url:’’,

type:’’,

data:{},

dataType:’JSON’，//新加参数，返回的数据自动转换成obj 接受的类型

traditional:true, // data中的value是数组的话，需要加此参数

success:function(arg){

// arg是个obj
}
})

$.ajax({
    dataType: 'json',
    contentType: 'application/json', //发送复杂对象类型，比如嵌套
    data: JSON.stringify({a: [{b:1, a:1}]}) //序列号成字符串传输，服务器端JSON.parse还原即可
})
```



### 事件委托 常用语编辑功能

$(‘要绑定的标签的上级标签’).on(‘click’,’要绑定的标签’，function(){})

###ajax发送数据要点

- 如果data中的v是字符串，正常传就行
- 如果data中v是数组，需要加参数traditional:true,

### 前端语法：

- 序列化：obj-> str 

​    	— JSON.stringfy(obj)

- 反序列化： str ->obj

​       — JSON.parse(str)

## 新url方式 vs 对话框

- 新url方式更新：
  - 适合数据量大	
  - 独立页面
  - 提交时，保留新添的内容？？没做
- 对话框方式更新：
  - 数据量少或者条目少
  - 增加，编辑
    - ajax：考虑当前页，td中自定义属性
    - 页面
    - data:$(‘#form_lable’).serialize() 提交所有表单数据
- 删除：对话框即可

###django组件相关

-  django 的form组件：
- - 对用户请求的验证
  - - ajax
    - Form
  - 生成html：input缓存记忆功能
- djangoform组件：
- - 创建个class
  - 创建字段
  - GET
  - - obj=Form()
    - obj.user =>生成html
  - POST
  - - obj=Form(request.POST)
    - if obj.is_valid():
    - -  obj.cleaned_data
    - else:
    - - obj.error
      - return obj

### prop() vs attr() vs text() vs val() vs html()

- prop() 方法设置或返回被选元素的属性和值。

  当该方法用于**返回**属性值时，则返回第一个匹配元素的值。

  当该方法用于**设置**属性值时，则为匹配元素集合设置一个或多个属性/值对。

  **注意：**prop() 方法应该用于检索属性值,返回值是布尔型数据 true or false

- text() - 设置或返回所选元素的 **文本内容** 

- html() - 设置或返回所选元素的**内容（包括 HTML 标记）**

- val() - 设置或返回**表单字段的值(value=?)**

- attr() 方法也用于**设置/改变属性值**。













