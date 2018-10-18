### Django用户认证

我们在开发一个网站的时候，无可避免的需要设计实现网站的用户系统。此时我们需要实现包括用户注册、用户登录、用户认证、注销、修改密码等功能，这还真是个麻烦的事情呢。

Django作为一个完美主义者的终极框架，当然也会想到用户的这些痛点。它内置了强大的用户认证系统--auth，它默认使用 auth_user 表来存储用户数据。

## 创建超级用户

用法:

```python
python manage.py createsuperuser
# 设置账户和密码
```

进入admin窗口,输入账户和密码

## auth模块

```python
from django.contrib import auth
```

atuh中提供了许多实用方法:

### authenticate()认证 校验和密码验证

提供了用户认证功能，即验证用户名以及密码是否正确，一般需要username 、password两个关键字参数。

如果认证成功（用户名和密码正确有效），便会返回一个 User 对象。

authenticate()会在该 User 对象上设置一个属性来标识后端已经认证了该用户，且该信息在后续的登录过程中是需要的。

用法：

```pyth
user = auth.authenticate(request,username='theuser',password='thepassword')
```

### login(HttpRequest, user)

该函数接受一个HttpRequest对象，以及一个经过认证的User对象。

该函数实现一个用户登录的功能。它本质上会在后端为该用户生成相关session数据。

用法：

```python
def login(request):
    # form_obj = RegForm()
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        obj = auth.authenticate(request, username=username, password=password)
        # print(obj)
        # print(obj, type(obj))
        # 如果校验成功,就登录成功
        if obj:
            # 记录登录状态
            auth.login(request, obj)
            return redirect('/index/')

    return render(request, 'login.html')

```

## login_requierd()

auth 给我们提供的一个装饰器工具，用来快捷的给某个视图添加登录校验。

用法：

```python
# 跳转页面
@login_required()  # 加装饰器,让页面只能登录之后跳转
def index(request):
    return render(request, 'index.html')
```

### logout(request)

该函数接受一个HttpRequest对象，无返回值。

当调用该函数时，当前请求的session信息会全部清除。该用户即使没有登录，使用该函数也不会报错。

用法：

```python
# 注销页面
def logout(request):
    auth.logout(request)   # 注销的方法,就是删除session数据
    return redirect('/login/')  # 注销之后跳转到登录页面
```

### is_authenticated()

用来判断当前请求是否通过了认证。

用法：

```python
# 跳转页面
# @login_required()  # 加装饰器,让页面只能登录之后跳转
def index(request):
    # 登录之后的状态放在user中
    print(request.user.is_authenticated())  # 判断登录状态,匿名对象没有认证返回False
    return render(request, 'index.html')
```

### create_user()    创建普通用户

auth 提供的一个创建新用户的方法，需要提供必要参数（username、password）等。

用法：

```python
# 注册用户
def register(request):
    form_obj = RegForm()
    if request.method == 'POST':
        form_obj = RegForm(request.POST)
        if form_obj.is_valid():
            form_obj.cleaned_data.pop('re_password')  # 删除第二次校验的密码
            User.objects.create_user(**form_obj.cleaned_data)   # 将数据存入数据
            return redirect('/login/')

```

### create_superuser()  创建超级用户

```python
# 注册用户
def register(request):
    form_obj = RegForm()
    if request.method == 'POST':
        form_obj = RegForm(request.POST)
        if form_obj.is_valid():   
            form_obj.cleaned_data.pop('re_password')  # 删除第二次校验的密码

            # 创建超级用户,必须要有email
            User.objects.create_superuser(email='', **form_obj.cleaned_data)
            return redirect('/login/')
```

### check_password(password)

auth 提供的一个检查密码是否正确的方法，需要提供当前请求用户的密码。

密码正确返回True，否则返回False。

用法：

```python
print(request.user.check_password('11111111'))   # 判断密码是否正确
# 返回True或者False
```

### set_password(new_password)

auth 提供的一个修改密码的方法，接收 要设置的新密码 作为参数。

注意：设置完一定要调用用户对象的save方法！！！

用法：

```python
print(request.user.check_password('11111111'))   # 判断密码是否正确
    if request.user.check_password('11111111'):
        request.user.set_password('22222222')   # 修改之后的新密码
        request.user.save()  # 更新到数据库
```

### User对象属性

User对像属性: username, password

is_staff:    用户是否拥有网站的管理权限

is_active:   是否允许用户登录,设置为False,可以在不删除用户的前提下禁止用户登录

email:  用户邮箱,超级用户必须设置默认值必须有邮箱

### 扩展默认的auth_user表

这内置的认证系统这么好用，但是auth_user表字段都是固定的那几个，我在项目中没法拿来直接使用啊！

比如，我想要加一个存储用户手机号的字段，怎么办？

聪明的你可能会想到新建另外一张表然后通过一对一和内置的auth_user表关联，这样虽然能满足要求但是有没有更好的实现方式呢？

答案是当然有了。

我们可以通过继承内置的 AbstractUser 类，来定义一个自己的Model类。

这样既能根据项目需求灵活的设计用户表，又能使用Django强大的认证系统了。

```python
from django.db import models
from django.contrib.auth.models import AbstractUser


# 继承AbstractUser
class UserInfo(AbstractUser):
    phone = models.CharField(max_length=11,)

```

注意:

按上面的方式扩展了内置的auth_user表之后，一定要在settings.py中告诉Django，我现在使用我新定义的UserInfo表来做用户认证。写法如下： 

```pthon
# 引用Django自带的User表，继承使用时需要设置
AUTH_USER_MODEL = "app01.UserInfo"
```

再次注意:

一旦我们指定了新的认证系统所使用的表，我们就需要重新在数据库中创建该表，而不能继续使用原来默认的auth_user表了。 