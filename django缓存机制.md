# 缓存机制

### 一、概念

​      浏览器的缓存机制也就是我们说的http缓存机制，其机制是根据http报文的缓存标识进行的

​      1.**HTTP请求报文**，报文格式：**请求行-http头(通用信息头，请求头，实体头)-请求报文主体(只有POST才有报文主体)**

![](.\img\request.png)

![](.\img\请求主体.png)

​       2.**http响应报文**，报文格式位：**状态行-http头(通用信息头，响应头，实体头)-响应报文主体**

![](.\img\response.png)

![](.\img\响应主体.png)

注：**通用信息头**指的是请求和响应报文都支持的头域

分别为Cache-Control、Connection、Date、Pragma、Transfer-Encoding、Upgrade、Via;

**实体头**则是实体信息的实体头域，分别为Allow、Content-Base、Content-Encoding、Content-Language、Content-Length、Content-Location、Content-MD5、Content-Range、Content-Type、Etag、Expires、Last-Modified、extension-header.这里只是为了方便理解，将通用信息头，响应头/请求头，实体头都归位了HTTP头

### 二、过程

浏览器与服务器通信的方式为应答模式，即是：**浏览器发起HTTP请求-服务器响应该请求**。那么浏览器第一次向服务器发起该请求后拿到请求结果，会根据响应报文中HTTP头的缓存结果，是则将请求结果和缓存标识存入浏览器缓存中

![](.\img\第一次请求.png)

由上图我们可以知道

1、浏览器每次发起请求，都会先在浏览器缓存中查找请求的结果以及缓存标识

2、浏览器每次拿到返回的请求结果都会将该结果和缓存标识存入浏览器缓存中

以上两点结论就是浏览器缓存机制的关键，他确保了每个请求的缓存标识存入浏览器缓存中

根据是否需要向服务器重新发起http请求将缓存过程分为 **强制缓存** 和 **协商缓存** 两个部分



#### 强制缓存

**强制缓存就是向浏览器缓存查找该请求结果，并根据该结果的缓存规则来决定是否使用该缓存结果的过程**，强制缓存的情况主要三种

(1)不存在该缓存结果和缓存标识，强制缓存失效，则直接向服务器发起请求（跟第一次发起请求一致）

![](.\img\强制缓存1.png)

(2)存在该缓存结果和缓存标识，但是结果已经失效，强制缓存失效，则使用协商缓存

![](.\img\强制缓存2.png)

(3)存在该缓存结果和缓存标识，且该结果没有还没有失效，强制缓存生效，直接返回该结果

![](.\img\强制缓存3.png)

强制缓存的规则：当浏览器向服务器发送请求的时候，服务器会将**缓存规则**放入HTTP响应的报文的HTTP投中和请求结果一起返回给浏览器，**控制强制缓存的字段分别是Expires和Cache-Control**，其中Cache-Control的优先级比Expires高。

##### Expires

Expires是HTTP/1.0控制网页缓存的字段，其值为服务器返回该请求的结果缓存的到期时间，即再次发送请求时，如果客户端的时间小于Expires的值时，直接使用缓存结果

Expires是HTTP/1.0的字段，但是现在浏览器的默认使用的是HTTP/1.1，那么在HTTP/1.1中网页缓存还是否由Expires控制

到了HTTP/1.1，Expires已经被Cache-Control代替，原因在于Expires控制缓存的原理是使用**客户端的时间**与**服务端返回的时间**做对比，如果客户端与服务端的时间由于某些原因(时区不同；客户端和服务端有一方的时间不准确)发生误差，那么强制缓存直接失效，那么强制缓存存在的意义就毫无意义



##### Cache-control

在HTTTP/1.1中，Cache-Control是最重要的规则，主要用于控制网页缓存，主要取值为：

(1)**public**：所有的内容都将被缓存(客户端和代理服务器都可缓存)

(2)**private**：所有的内容只有客户端可以缓存，**Cache-Control**的默认取值

(3)**no-cache**：客户端缓存内容，但是是否使用缓存则需要经过协商缓存来验证决定

(4)**no-store**：所有的内容都不会被缓存，即不使用强制缓存，也不使用协商缓存

(5)**max-age=xxx(xxx is numeric)**：缓存内容将在xxx秒后失效

如图是个例子：

![](.\img\eg1.png)

由上面的例子我们可以知道：

(1)HTTP响应报文中expires的时间值，是一个绝对值

(2)HTTP响应报文中Cache-Control为max-age=600，是相对值

由于Cache-Control的优先级比expires，那么直接根据Cache-Control的值进行缓存，意思就是说在600秒内再次发起该请求，则会直接使用缓存结果，强制缓存生效

注：在无法确定客户端的时间是否与服务端的时间同步的情况下，Cache-Control相比于expires是更好的选择，所以同时存在时，只有Cache-Control生效



>  如何在浏览器中判断强制缓存是否生效

![](.\img\浏览器缓存.png)

这里我们以博客的请求为例，状态码为灰色的请求则代表使用了强制缓存，请求对应的Size值代表该缓存存放的位置，分别为**from memory cache**和**from disk cache**



> 那么 from memory cache 和 from disk cache 有分别代表的是什么，以及二者的使用场所

from memory cache代表使用内存中的缓存，from disk cache代表使用硬盘中的缓存，浏览器粗去缓存的顺序为memory->disk

以下为缓存读取：

访问网页 -> 200 -> 关闭博客的标签页 -> 重新访问网页 -> 200(from disk cache) -> 刷新 -> 200(from memory cache)

过程如下：

(1)访问博客网页

![](.\img\访问博客网站.png)

(2)关闭

(3)重新打开博客

![](.\img\重新打开博客.png)

(4)刷新

![](.\img\刷新.png)

关于同时存在着from disk cache和from memory cache

**内存缓存(from memory cache)**:内存缓存具有两个特点，分别是**快速读取**和**时效性**：

​     1、**快速读取**：内存缓存会将编译解析后的文件，直接存入该进程中，占据该进程的一定内存资源，以方便下次运行使用时的快速读取。

​     2、**时效性**：一旦该进程关闭，则该进程的内存会清空

**硬盘缓存(from disk cache)**:硬盘缓存则是直接将缓存写入硬盘文件中，读取缓存需要对该缓存存放的硬盘文件进行I/O操作，然后重新解析该缓存内容，读取复杂，速度比内存缓存慢



在浏览器中，浏览器会在**js和图片等文件解析执行后直接存入内存缓存中**，那么当刷新页面时只需要直接从内存缓存中读取(from memory cache);而**css文件则会存入硬盘文件**中，所以每次渲染页面都需要从硬盘读取缓存(from disk cache)

#### 协商缓存

协商缓存是强制缓存失效后，浏览器携带缓存标识向服务发起请求，由服务重启根据缓存标识决定是否使用缓存的过程，主要由两种情况：

(1)协商缓存生效。返回304，如下：

![](.\img\协商缓存生效.png)

(2)协商缓存失效，返回200和请求结果，如下

![](.\img\协商缓存生效.png)

同样，协商缓存的标识也是在响应报文的HTTP头中和请求结果一起返回给浏览器的，控制协商缓存的字段分别由：**Last-Modified/If-Modified-Since**和**Etag/If-None-match**,其中Etag/If-None-Match的优先级比Last-Modified/If-Modified-Since高。

##### Last-Modified/If-Modified-Since

(1)Last-Modified是服务器响应请求时，返回该资源文件在服务器最后被修改的时间，如下：

![](.\img\Last-modified.png)

(2)If-Modified-Since则是客户端再次发起该请求时，携带上次请求返回的Last-Modified值，通过此字段值告诉服务器资源上次请求返回的最后修改时间。服务器收到该请求，发现请求头含有If-Modified-Since字段，则会根据If-Modified-Since的字段值与该资源在服务器的最后被修改时间做对比，若服务器的资源最后被修改时间大于If-Modified-Since的字段值，则重新返回资源，状态码为200；否则返回304，代表资源无更新，可继续使用缓存文件，

![](.\img\if-modified-since.png)

##### Etag/If-None-Match

(1)Etag是服务器响应请求时，返回当前资源文件中的一个唯一标识(由服务器生成)，如下：

(2)If-None-Match是客户端再次发起请求时，携带上次请求返回的唯一标识Etag值，通过此字段值告诉服务器该资源上次请求返回的唯一标识值。服务器收到该请求后，发现请求头中含有If-None-Match,则会根据If-None-Match的字段值与该资源在服务器的Etag值座对比，一致则返回304，代表资源更新，继续使用缓存文件；不一致则重新返回资源文件，状态码200，如下

![](.\img\If-None-Match.png)

注：Etag/If-None-Match优先级高于Last-Modified/If-Modified-Since，同时存在则只有Etag/If-None-Match生效

#### 以上来源：https://www.cnblogs.com/chengxs/p/10396066.html





## Django缓存机制

---

#### 一、缓存介绍

- 在动态网站中，用户所有的请求，服务器都会去数据库中进行相应的增、删、改、查、渲染模板，执行业务逻辑，最后生成用户看到的页面

- 当一个网站的用户访问量很大的时候，每一次的后台操作，都会消耗很多的服务端资源，所以必须使用缓存来减轻后端服务器的压力
- **缓存是将一些常用的数据保存到内存或者memcache中**，在一定的时间内有人来访问这些数据时，则不再去执行数据库及渲染等操作，而是直接从内存或memcache的缓存中去取得数据，然后返回给用户，可以局部缓存，也可以全站缓存(可以放在中间件中)

#### 二、Django的6种缓存方式

- 开发调试缓存
- 内存缓存
- 文件缓存
- 数据库缓存
- Memcache缓存(使用python-memcache模块)
- Memcache缓存(使用python模块)

#### 三、 Django的6中缓存配置

##### 开发调试(此模式为开发调试使用，实际上不执行任何操作)

``` python
# seting.py文件配置
CACHES ={
    'default':{
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache', # 缓存后台使用的引擎
        
        'TIMEOUT': 300, # 缓存超时时间（默认300秒，None表示永不过期， 0表示立即过期
        
        'OPTIONS':{
            'MAX_ENTRIES': 300, #最大缓存记录数量（默认300）
            'CULL_FREQUENTCY': 3, 
            #缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY(默认)
        }
    }
}
```

##### 内存缓存(将缓存内容保存至内存区域中)

``` python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache', # 指定缓存使用的引擎
        'LOCATION': 'unique-snowflake', #写在内存中的变量的唯一值
        'TIMEOUT': 300, #缓存超时时间(默认300，None表示永不过期)
        'OPTIONS':{
            'MAX_ENGIRIES': 300, # 最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3, # 缓存到达最大个数之后，剔除缓存个数的比例。即：1/CULL_FREQUENCY(默认)
        }
    }
}
```

##### 文件缓存（把缓存数据存储在文件中）

```python
CACHES = {
    'default':{
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache', #指定缓存使用的引擎
        'LOCATION': '/var/tmp/django_cache', #指定缓存的路径
        'TIMEOUT': 300,  # 缓存超时时间（默认为300秒，None表示永不过期
        'OPTIONS':{
            'MAX_ENTRIES': 300,  #最大缓存记录的数量（默认300）
            'CULL_FREQUENCY': 3, #缓存到达最大个数之后，剔除缓存个数的比例。即：1/CULL_FREQUENCY(默认)
        }        
    }
}
```

##### 数据库缓存（把缓存数据存储在数据库中）

``` python
CACHES = {
    'default':{
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache', #指定缓存引擎
        'LOCATION': 'cache_table',     # 数据库表
        'OPTIONS':{
            'MAX_ENTRIES': 300,
            'CULL_FREQUENCY': 3,
        }
    } 
}
# 注意：创建缓存的数据库表使用的语句
python manage.py createcachetable
```

##### Memcache缓存（使用python-memcached模块连接memcache）

Memcached是Django原生支持的缓存系统，要使用Memcached，需要下载Memcached的支持库python-memcache或pylibmc

```python
CACHES = {
    'default':{
        'BACKEND': 'django.core.cache.backends.memcached.MemcacheCache', # 指定缓存使用的引擎
        'LOCATION': '127.0.0.1:PORT', #指定Memcache缓存服务器的IP地址和端口
        'OPTIONS':{
            'MAX_ENTRIES': 300,
            'CULL_FREQUENCY': 3,
        }
    }
}
# LOCATION也可以配置成如下：
'LOCATION': 'unix:/tmp/memcached.sock‘，#指定局域网内的主机名加socket套接字为Memcache缓存服务器
'LOCATION': {   #指定一台或多台其他主机ip地址加端口为Memcache缓存服务器
    '127.0.0.1:8000'
}
```

##### Memcache缓存（使用pylibmc模块连接memcache）

``` python
CACHE = {
    'default': 'django.core.cache.backends.memcached.PyLibMCCache',
    'LOCATION: 'ip:port', # 指定本机的port端口为Memcache缓存服务器
    'OPTIONS':{
        "MAX_ENTIRES":300,
        'CULL_FREQUENCY':3,
    }
}

# LOCATION也可以配置成如下：
'LOCATION': 'unix:/tmp/memcached.sock‘，#指定局域网内的主机名加socket套接字为Memcache缓存服务器
'LOCATION': {   #指定一台或多台其他主机ip地址加端口为Memcache缓存服务器
    '127.0.0.1:8000'
}
# Memcached是基于内存的缓存，数据存储在内存中，所以如果服务器死机的话，数据就会丢失，所以Memcached一般与其他缓存配合使用
```

#### 四、Django缓存的应用

Django提供了不同粒(力)度的缓，可以缓存某个页面，可以只缓存一个页面的某个部分，甚至可以缓存整个网站

**数据库**（以下以该数据库为例）

``` python
class Book(models.Model):
    name=models.CharField(max_length=32)
    price=models.DeciamlField(max_digits=6,decimal_places=1)
```

##### 视图函数使用缓存

**视图**：

``` python
from django.views.decorators.cache import cache_page
import time
from .models import *

@cache_page(15)          #超时时间为15秒
def index(request):
　　t=time.time()      #获取当前时间
　　bookList=Book.objects.all()
　　return render(request,"index.html",locals())
```

**模板(index.html)**

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h3>当前时间:-----{{ t }}</h3>

<ul>
    {% for book in bookList %}
       <li>{{ book.name }}--------->{{ book.price }}$</li>
    {% endfor %}
</ul>

</body>
</html>
```

上面的例子是基于内存的缓存配置，基于文件的缓存该怎么配置呢

**更改setting.py的配置**

```python
CACHES = {
 'default': {
  'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache', # 指定缓存使用的引擎
  'LOCATION': 'E:\django_cache',          # 指定缓存的路径
  'TIMEOUT': 300,              # 缓存超时时间(默认为300秒,None表示永不过期)
  'OPTIONS': {
   'MAX_ENTRIES': 300,            # 最大缓存记录的数量（默认300）
   'CULL_FREQUENCY': 3,           # 缓存到达最大个数之后，剔除缓存个数的比例，即：1/CULL_FREQUENCY（默认3）
  }
 }
}
```

然后再次刷新浏览器,可以看到在刚才配置的目录下生成的缓存文件

通过实验可以知道,Django会以自己的形式把缓存文件保存在配置文件中指定的目录中.

##### 全站使用缓存

既然是*全站缓存，当然要使用Django中的中间件*

用户的请求通过中间件，经过一系列的认证等操作，如果请求的内容在缓存中存在，则使用FetchFromCacheMiddleware获取内容并返回给用户

当返回给用户之前，判断缓存中是否已经存在，如果不存在，则UpdateCacheMiddleware会将缓存保存至Django的缓存之中，以实现全站缓存

``` python
# 缓存整个站点，这是最简单的缓存方法-------默认是放在内存中

#在 MIDDLEWARE_CLASSES中加入“update”和“fetch”中间件
MIDDLEWARE_CLASSES=(
    'django.middleware.cache.UpdateCacheMiddleware', # 第一
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware', # 最后
)

#"update" 必须配置在第一个
#"fetch"   必须配置在最后一个
```

**修改settings.py配置文件 **

``` python
MIDDLEWARE_CLASSES = (
    'django.middleware.cache.UpdateCacheMiddleware',   #响应HttpResponse中设置几个headers
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',   #用来缓存通过GET和HEAD方法获取的状态码为200的响应

)

CACHE_MIDDLEWARE_SECONDS=10
```

**视图函数**

```python
from django.views.decorators.cache import cache_page
import time
from .models import *


def index(request):
　　　print(1111111111)   #遇到中间件就直接返回了，在刷新页面，在终端就不会再打印1111111111了
     t=time.time()      #获取当前时间
     bookList=Book.objects.all()
     return render(request,"index.html",locals())

def foo(request):
    t=time.time()      #获取当前时间
    return HttpResponse("HELLO:"+str(t))
```

**模板(index.html)**

``` python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h3 style="color: green">当前时间:-----{{ t }}</h3>

<ul>
    {% for book in bookList %}
       <li>{{ book.name }}--------->{{ book.price }}$</li>
    {% endfor %}
</ul>

</body>
</html>
```

其余代码不变，刷新浏览器是10秒，页面上的时间变化一次，这样就是先了全站缓存

##### 局部视图缓存

例如，刷新页面时，整个网页有一部分实现缓存(如果是CBV可以继承一个类，对于FBV可以使用装饰器)

**views视图函数**

``` python
from django.views.decorators.cache import cache_page
import time
from .models import *@cache_page(10)       #缓存某一个视图10秒
def index(request):
     t=time.time()      #获取当前时间
     bookList=Book.objects.all()
     return render(request,"index.html",locals())
```

**模板(index.html)**

```html
{% load cache %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
 <h3 style="color: green">不缓存:-----{{ t }}</h3>
<!--缓存某一部分，这样就可以将视图函数的缓存装饰器去掉了-->
{% cache 2 'name' %}         <!--缓存某一部分2秒，缓存的名字可以随便写，一般可以写的有意义点-->
 <h3>缓存:-----:{{ t }}</h3>
{% endcache %}

</body>
</html>
```

#### 以上来自于https://www.cnblogs.com/sui776265233/p/9900954.html#_label3