## 【五】路由层

### 【1】路由匹配

```python
# urls.py
'''django1.10版本引入了path函数，1.x版本使用正则匹配【url(r'正则/')】'''
from django.urls import path
from views import func

urlpatterns = [
    # path('路径/',视图函数)
    path('xxx/',views.func)
]
```

### 【2】正则匹配

```python
# urls.py
from django.urls import re_path
from views import func

urlpatterns = [
    # 正则匹配分为有名分组和无名分组
    
    '''无名分组匹配到的路径在后端可以通过【*args】中的参数接受'''
    re_path(r'^正则表达式/',views.func)  # 无名分组
    
    '''有名分组匹配到的路径在后端可以通过【**kwargs】或指定【key】接受数据'''
    re_path(r'(?P<name>正则表达式)')  # 有名分组
]
```

### 【3】路径转换器

- **`<int>`**：匹配一个整数。
- **`<str>`**：匹配一个字符串，但不包含路径分隔符 `/`。
- **`<slug>`**：匹配一个 Slug，由字母、数字、连字符或下划线组成。
- **`<uuid>`**：匹配一个 UUID 格式的字符串。
- **`<path>`**：匹配包含路径分隔符 `/` 的字符串。

```python
# urls.py

'''路径转换器与正则匹配的有名分组类似，且可以作类型转换'''
'''普通的路由匹配拿到的路径都是字符串，而通过路径转换器后端可以拿到指定格式的数据'''

from django.urls import path
from views import func

urlpatterns = [
    # < int:id >
    path('根路径/< 路径转换器:name >',视图函数)
]
```

### 【4】反向解析

- 在 Django 中进行反向解析时，必须要给 URL 模式或视图函数命名

#### 【4.1】无名反向解析

```python
# urls.py
from django.urls import path
from views import func

urlpatterns = [
    path('路径/'，视图函数,name='view_name')
]
```

```python
# views.py
from django.shortcuts import HttpResponse, reverse

def func(request):
    url = reverse('view_name')
    return HttpResponse(url)
	# '/view_name/' 可以通过该反向解析的url获取到该name对应的视图函数
```

```html
<!-- 前端使用反向解析 -->
<body>
    <!-- 比较常用于为按钮或链接添加跳转链接，跳转至指定视图函数 -->
    <a href='{% url 'view_name'%}'>link</a>
</body>
```

#### 【4.2】有名反向解析

```python
# urls.py
from django.urls import path,re_path
from views import func

urlpatterns = [
    # 路径转换器
    path('路径/<int:id>'，视图函数,name='view_name'),
    # 无名分组
    re_path('路径/(?P<bid>(\d+)/)',视图函数,name = 'view_name')
]
```

```python
# views.py
from django.shortcuts import HttpResponse, reverse

def func(request,**kwargs):
    '''当路由中设置为有名时，后端需要设置形参接受，否则会报错'''
    
    url = reverse('view_name',kwargs={k:v})
    return HttpResponse(url)
	# '/view_name/value/' 可以通过该反向解析的url获取到该name对应的视图函数
```

```html
<!-- 前端 -->
<body>
    <!-- 需要将参数传递放置在view_name后，否则将报错 -->
    <a href='{% url 'view_name' value %}'>link</a>
</body>
```

### 【5】路由分发

- 为了对路由解耦合，保证代码清晰可读

```python
# urls

from django.urls import path
from django.urls.conf import include

urlpatterns = [
    # 一般会为每一个应用程序创建一个urls文件
    path('路径/',iuclude('指定文件夹.urls'))
]
```

## 【六】视图层

### 【1】响应三板斧

#### 【1.1】HttpResponse

- 将字符串转换为HttpResponse对象
- 【注】视图函数所返回的对象必须是HttpResponse对象或者渲染的前端页面

```python
# views.py
from django.shortcuts import HttpResponse

def func(request):
    return HttpResponse('Hello World!')
```

#### 【1.2】render

- 返回渲染的前端界面

```python
# views.py
from django.shortcuts import render

def func(request):
    return render(request=request,templates_name='前端文件.html'，context='上下文')
```

#### 【1.3】redirect

- 重定向至指定视图函数

```python
# views.py
from django.shortcuts import redirect

def func(request):
    return redirect(to='view_name'，context='上下文',permanent=301/302)
	# permanent可以指定永久重定向【301】还是临时重定向【302】
    # 默认302，因为301将会缓存至浏览器，下一次将优先从缓存中获取界面
```

### 【2】JsonResponse

- 返回json格式的响应数据

```python
# views.py
from django.http import JsonResponse

def func(request):
    data = {
        'k':v
    }
    return JsonResponse(data)
```

- `JsonResponse`的其他参数
  - `data`: 要转换为 JSON 的数据。默认情况下，由于安全漏洞，在 EcmaScript 5 之前只允许传递 `dict` 对象。有关更多信息，请参见 `safe` 参数。
  - `encoder`: 应该是一个 JSON 编码器类。默认为 `django.core.serializers.json.DjangoJSONEncoder`。
  - `safe`: 控制是否只有 `dict` 对象可以被序列化。默认为 `True`。
  - `json_dumps_params`: 传递给 `json.dumps()` 的关键字参数字典

### 【3】获取文件数据

- form表单中input标签设置type属性为file时，上传的数据需要进一步处理才能获取

```html
<!-- 前端上传数据 -->
<body>
    <!-- 指定【enctype】参数 -->
<form action="" method="post" enctype="multipart/form-data">
    file: <input type="file" name='filename'>
</form>
</body>
```

```python
# views.py
# 后端获取数据

def func(request):
    if request.method == 'POST':
        # 当表单提交数据时，获取到表单中的文件数据
        file_obj = request.FILES.get('filename')  # 当有多个文件时，根据name数据设置的键取值
        # 当前file_obj只是内存中的临时缓存，不能直接写入文件，需要通过【.chunks()】方法
        file = file_obj.chunks()
        with open('文件路径', mode='wb') as fp:
            for data in file:
                fp.write(data)
    return render(request, '前端页面.html')
```

- `InMemoryUploadedFile` 和 `TemporaryUploadedFile` 对象并不是真正的文件对象，而是内存中的临时对象或临时文件对象。这些对象的主要作用是临时存储上传的文件内容，而不是直接写入文件系统中

- `chunks()` 方法返回一个生成器（generator），每次迭代都会产生一块文件内容的数据。

#### 【3.1】文件对象常用的属性和方法

一些常用的属性包括：

1. `name`: 上传文件的名称。
2. `size`: 上传文件的大小。
3. `content_type`: 上传文件的内容类型。
4. `charset`: 上传文件的字符集。
5. `content_type_extra`: 上传文件的额外内容类型信息。
6. `file`: 上传文件的内容（BytesIO 对象）。

一些常用的方法包括：

1. `read()`: 读取文件的内容。
2. `readline()`: 读取文件的一行。
3. `chunks(chunk_size=None)`: 生成器函数，按块读取文件内容。
4. `seek(offset, whence=0)`: 移动文件指针到指定位置。
5. `close()`: 关闭文件。
6. `multiple_chunks()`: 检查文件是否分成多个块。
7. `temporary_file_path()`: 获取文件的临时文件路径（如果存在）。

### 【4】CBV视图类

#### 【4.1】FBV和CBV

- CBV（Class-Based Views）和 FBV（Function-Based Views）是 Django 中两种常用的视图编写方式，它们分别基于类和函数。

```python
# views.py
from django.shorcuts import render,HttpResponse
'''CBV视图类需要继承【View类】'''
from django.views import View


def fbv(request):
    # FBV基于函数
    return HttpResponse('FBV')


class Cbv(View):
    # CBV基于类
    
    def get(self,request):
        '''当【get请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - get')
    
    def post(self,request):
        '''当【post请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - post')
```

```python
# urls.py
'''在路由中，FBV与CBV的调用方式有所不同'''

from django.urls import path
from views import fbv,Cbv

urlpatterns = [
    # FBV就是平时使用的最多的视图函数
    path('路径/',fbv),
    # CBV视图类调用时，需要使用【as_view()】方法调用
    # 具体原因请看源码分析
    path('路径',Cbv.as_view(),name='CBV')
]
```

#### 【4.2】CBV源码分析

![CBV源码解析](https://res.lea4ning.top/res/202403121947187.png)

### 【5】装饰器

#### 【5.1】为FBV添加装饰器

- FBV就是普通的函数，所以与给函数加装饰器语法一致

```python
# 装饰器
def auth(func):
    def inner(request,*args,**kwargs):
        # 需要注意，因为视图函数的第一个对象都是request，所以一般情况下将第一个形参设置为request
        # 当然，如果愿意，也可以从【args[0]】中获取
        res = func(request,*args,**kwargs)
        return res
    return inner

# 加了装饰器视图函数
@auth
def func(request):
    return HttpResponse('oi')
```

#### 【5.2】为CBV添加装饰器

> [[装饰类 | Django 文档 | Django (djangoproject.com)](https://docs.djangoproject.com/zh-hans/3.2/topics/class-based-views/intro/#decorating-the-class)](https://docs.djangoproject.com/zh-hans/3.2/topics/class-based-views/intro/#decorating-the-class)

- CBV视图类，因为是类，当调用普通的装饰器时，第一个参数将会错乱，可能时`self`也可能时`cls`，request对象将不太好寻找
- 为CBV添加装饰器需要借助`from django.utils.decorators import method_decorator`组件

##### 【5.2.1】加装饰器在类上

```python
# 装饰器
def auth(func):
    '''装饰器函数的创建是一致的'''
    def inner(request,*args,**kwargs):
        res = func(request,*args,**kwargs)
        return res
    return inner


@method_decorator(decorator='装饰器名',name='需要加装装饰器的函数')
class Cbv(View):

    def get(self, request):
        '''当【get请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - get')

    def post(self, request):
        '''当【post请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - post')
```

##### 【5.2.2】加装饰器在函数上

```python
class Cbv(View):
   
    @method_decorator(decorator='装饰器名')
    def get(self, request):
        '''当【get请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - get')
    
    
	@method_decorator(decorator='装饰器名')
    def post(self, request):
        '''当【post请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - post')
```

##### 【5.2.3】加装饰器在函数dispatch

- 根据源码我们可以看到在执行我们写的方法前，会先执行dispatch方法，所以如果重写父类中的dispatch方法，并添加装饰器，那么在我们视图类中的所有方法都可以加上装饰器

```python
class Cbv(View):
    def get(self, request):
        '''当【get请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - get')

    def post(self, request):
        '''当【post请求】调用视图类时执行的代码'''
        return HttpResponse('CBV - post')

    @method_decorator(decorator='装饰器名')
    def dispatch(self, request, *args, **kwargs):
        '''不对dispatch作其他操作，直接继承父类中的方法'''
        return super().dispatch(request, *args, **kwargs)
```

##### 【5.4】当装饰器有多个时

```python
'''当装饰器有多个时，可以构建一个列表或元组'''
decorators = [never_cache, login_required]

# 传参时将上述的列表或元组传入即可
@method_decorator(decorators, name='dispatch')
class ProtectedView(View):
    template_name = 'secret.html'
```

<!-- ##{"timestamp":1664935853}## -->