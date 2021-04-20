- 数据的编辑和删除

``` python
# 查询所有数据
   res = models.User.objects.fitler()
   res = models.User.objests.all()
#  编辑数据
"""
1.获取用户想要编辑的数据主键值

2.后端查询出对应的数据对象展示到前端

3.提交post请求修改数据

"""

# 批量更新
models.User.objects.filter(id=edit_id).update(**kwargs)
#单个更新
user_obj = models.User.objects.filter(id=edit_id).first()
user_obj.username = 'jason'
user_obj.password = '666'
user_obj.save()
"""
   数据过多时，效率低
"""
# 删除数据
"""
数据并不是真正意义上的删除，我们在创建表的时候会加一个用来标示是否被删除的字段
is_delete
is_alive
is_status
...
删数据其实就是修改字段的状态 之后通过代码筛选没有删除的状态数据即可
(删除数据应该有一个二次确认sweetalaert)
"""
```

- 创建表关系

``` python
class Publish:
    pass

class User:
    models.ForeignKey(to="Publish")
    models.ForeignKey(to=Publish)
"""
一对多
   models.ForeignKey(to='')
   models.ForeignKey(to=关联表名)     关联表明必须出现在上方
      1.在django1.x版本中外键默认就是级联更新删除的
      2.会自动给字段加_id后缀 无论你有没有加
一对一
   models.OneToOneField(to='关联表名')
      1.在django1.x版本中外键默认就是级联更新删除的
      2.会自动给字段加_id后缀 无论你有没有加
多对多
  models.ManyToManyField(to='关联表名')
      1.在django1.x版本中外键默认就是级联更新删除的
      2.该字段是个虚拟字段不会真正的在表中展示出来，而用来告诉Django orm当前表和关联表是多对多的外键关系 ，需要自动创建第三张关系表
      3.在Django orm中多对多的表关系有好几种（三种）创建方式
      4.外键建在任意一方均可 但是推荐你建在查询频率较高的表中（orm查询方便）
      
判断表关系的方式：换位思考
"""
```

- Django请求

    ``` python
"""
浏览器
    发送请求（HTTP协议）

web服务网关接口
     1，请求来的时候解析封装
        响应走的时候在打包处理
     2. Django默认的wsgiref和uwsgi的关系
        WSGI是网关协议
        wsgiref和uwsgi是实现该协议的功能模块
        
Django后端
    1. django中间件
        
    2.urls.py             路由层
        识别路由匹配对应的视图函数
        
    3.view.py             视图层
        网页整体的业务逻辑
        
    4.templates文件夹      模板层
        网站所有的html文件
        
    5.models.py           模型层
        ORM
        
  额外扩展：缓存数据库的作用
"""
    ```

- 路由分发

``` python
"""
url()方法第一参数是正则表达式
一旦匹配成功了 直接触发正则后面的视图函数的运行

#首页
url(r"^$",)
#尾页
url(r'',)
"""
# Django路由匹配的时候其实可以匹配两次 第一次如果url后面没有加斜杠 django会让浏览器加斜杠再发一次请求
配置文件
APPEND_SLASH = True/False
```

- 有无名分组

``` python
# 无名分组
url(r'^index/(\d+)/',view.index)
"""
将括号内正则表达式匹配的内容当作位置参数传递给后面的视图函数
index(request,year=111)
"""

# 无名和有名不能混合使用

# 单个分组可以重复使用
```

- 反向解析

``` python
"""
本质：通过一些方法得到一个结果 该结果可以访问到对应的url从而触发视图函数的运行
"""
# 最简单的情况 url第一个参数里面没有正则符号
  url('^index/',view.index,name='xxx')
# 前端
  {% url "xxx" %}
# 后端
  from django.shortcuts import reverse
  reverse('xxx')
"""
别名不能出现冲突！！！！
"""
```

- 无名有名分组的反向解析

``` python

```

