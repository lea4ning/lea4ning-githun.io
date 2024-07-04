## 【零】补充方法

### 【1】Django项目测试

```python
if __name__ == '__main__':
    import os
    import django
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'BookSystem.settings')
    django.setup()
    '''测试代码'''
```

### 【2】项目使用模块版本导入与导出

```shell
# 查看
pip freeze
```

```shell
# 导出
pip freeze > requirements.txt  # requirements文件名可以自行更改，但约定俗称
```

```shell
# 导入
pip install -r requirements.txt  # 安装文件中的模块及版本
```

## 【一】使用静态文件

### 【1】配置静态文件

```python
# settings.py
STATIC_URL = 'static/'
STATICFILES_DIRS = [
    # 存放静态文件的路径
    os.path.join(BASE_DIR, 'static')

```

### 【2】加载静态文件

```python
# 前端
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    {#    加载静态文件中的js文件#}
    <script src="{% static 'xxx.js' %}"></script>
    {#    加载静态文件中的css文件#}
    <link rel="stylesheet" href="{% static 'xxx.css' %}">
        
</head>
<body>
{#加载静态文件中的文件#}
{% static 'xxx' %}
</body>
</html>
```

## 【二】request对象

### 【1】属性

- `request.method`: 返回HTTP请求方法，如GET、POST等。
- `request.path`: 返回请求的路径部分。
- `request.GET`: 包含GET请求参数的字典。
- `request.POST`: 包含POST请求参数的字典。
- `request.FILES`: 包含上传文件的字典。
- `request.session`: 返回一个与请求关联的会话对象。
- `request.user`: 返回当前登录用户的对象。
- `request.META`: 包含HTTP请求的元数据的字典，如请求头信息、IP地址等。
- `request.COOKIES`: 包含请求中的cookie的字典。

### 【2】方法

1. `request.get()`: 获取GET请求参数的值。
2. `request.post()`: 获取POST请求参数的值。
3. `request.get_full_path()`: 返回包含查询参数的完整路径。
4. `request.is_secure()`: 返回一个布尔值，指示请求是否通过HTTPS协议进行。
5. `request.build_absolute_uri()`: 构建并返回请求的绝对URL。
6. `request.get_host()`: 返回请求的主机名。
7. `request.get_port()`: 返回请求的端口号。
8. `request.get_raw_uri()`: 返回原始请求URI。
9. `request.is_ajax()`: 返回一个布尔值，指示请求是否是通过Ajax发送的。
10. `request.session.get()`: 获取会话数据。

## 【三】ORM(object ralational mapping)

### 【1】配置数据库

```python
# settings.py

'''django默认使用小型数据库sqlite3'''

DATABASES = {
    'default': {
        # 选择数据库引擎
        'ENGINE': 'django.db.backends.sqlite3',
        # 选择库名
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

```python
# settings.py
'''MySQL数据库配置'''
DATABASES = {
    'default': {
        # 选择数据库引擎
        'ENGINE': 'django.db.backends.mysql',
        # 库名
        'NAME': '库名',
        # 登录的用户名
        'USER': '用户名',
        # 用户名对应的密码
        'PASSWORD': '密码',
        # 主机地址
        'HOST': '127.0.0.1',
        # 端口，一般是3306
        'PORT': 3306
    }
}
```

#### 【1.1】PyCharm配置数据库

![image-20240312172612314](https://res.lea4ning.top/res/202403121726496.png)

#### 【1.2】报错情况

```python
'''django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.'''
```

- 解决

```python
# 猴子补丁：任意一个文件夹中的 __init__.py
import pymysql
pymysql.install_as_MySQLdb()
```

```shell
# 安装第三方模块
pip install mysqlclient
```

### 【2】数据迁移文件

- 在Django中，对数据库操作依赖于`from django.db import models`组件
- 在`models.py`文件中创建表后，需要执行数据迁移使数据对数据库生效

```shell
# 创建数据迁移文件
python manage.py makemigration
# 执行数据迁移
python manage.py migrate
# 【注】执行manage.py操作，需要在该文件所在位置执行命令
```

### 【3】对应关系

```python
from django.db import models

class TableName(models.Model):
    # 字段 = models.字段类型(约束条件)
    field1 = models.CharField(max_length=32, unique=True, null=True, verbose_name='描述')

    class Meta():
        db_table = '自定义表名'
        
# 每一个实例化类得到的对象可以对应数据库中的记录（或行）。
# 每个对象代表数据库表中的一条记录，并且对象的属性对应表中的字段
```

## 【四】请求生命周期流程图

![1](https://res.lea4ning.top/res/202403121745593.png)

![img](https://res.lea4ning.top/res/202403121744234.png)

<!-- ##{"timestamp":1664874653}## -->