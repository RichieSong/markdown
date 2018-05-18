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

dataType:’JSON’，//新加参数，返回的数据自动转换成obj

traditional:true, // data中的value是数组的话，需要加此参数

success:function(arg){

// arg是个obj

}

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





a->b

b->c

c->d

b->>d





