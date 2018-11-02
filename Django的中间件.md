# Django的中间件

### 中间件介绍

#### 什么是中间件?

##### 官方说法:中间件是一个用来处理Django的请求和响应的框架级别的钩子.它是一个轻量,低级别

的插件系统,用于在全局范围内改变Django的输入和输出.每个中间件组件都负责做一些特定的功能.

但是由于其影响的是全局,所以需要谨慎使用,使用不当会影响性能.

中间件就是帮助我们在视图函数执行之前和之后都可以做一些额外的操作,它本质上就是一个自定义

类,类中定义了几个方法,Django框架会在处理请求的特定的时间去执行这些方法.

中间件在Django项目的settings.py文件里面的MIDDLEWARE

```python
# Django中间件配置
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

```

MIDDLEWARE配置项是一个列表,列表是一个个字符串,这些字符串其实是一个个类,也就是一个个中间件.

### 自定义中间件

##### 中间件可以定义五个方法,分别是(主要是process_request和process_response:

>- process_request(self,request)
>- process_response(self,request,response)
>- process_ view(self,request,view_func,view_args,view_kwargs)
>- process_exception(self,request,response)
>- process_template(self,request,response)

以上方法的返回值可以是None或一个HttpResponse对象,如果是None,则继续按照Django

定义的规则向后继续执行,如果是HttpResponse对象,则直接将该对象返回给用户.

主要注意这五种方法的执行时间,参数,返回值,执行顺序.

### 自定义一个中间件示例

```python
# 1.在项目app文件夹里面写一个py文件
# 2.文件里面的写自定义内容
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import render, HttpResponse


class MD1(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD1中的process_request')

    def process_response(self, request, response):
        print('这是MD1中的process_response')
        return response


class MD2(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD2中的process_request')

    def process_response(self, request, response):
        print('这是MD2中的process_response')
        return response
# 3.注册中间件
```

## process_request

process_request有一个参数,就是request,这request和视图函数的request是一样的.

它的返回值可以是None也可以是HttpResponse对象.返回值是None的话,按照正常流

程继续走,交给下一一个中间件处理,如果是HttpResponse对象,Django将不执行视图

函数,而将响应对象返回给浏览器.

当有多个中间件时

```python
class MD1(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD1中的process_request')
        # return HttpResponse('这是MD1')

    def process_response(self, request, response):
        print('这是MD1中的process_response')
        # print(id(request))  # 测试返回值的
        return response


class MD2(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD2中的process_request')

    def process_response(self, request, response):
        print('这是MD2中的process_response')
        return response
```

中间件注册顺序:

```pythoin
# 中间件配置
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'app01.my_middleware.MD1',  # 配置自定义中间件
    'app01.my_middleware.MD2',  # 配置自定义中间件
]
```



执行结果:

```python
这是MD1中的process_request
这是MD2中的process_request
这是index函数
61170576
这是MD2中的process_response
这是MD1中的process_response
```

如何把MD1和MD2的注册顺序调换之后执行结果:

```python
这是MD2中的process_request
这是MD1中的process_request
这是index函数
63398800
这是MD1中的process_response
这是MD2中的process_response
#  执行结果相反
```

看结果我们知道:视图函数还是执行的,MD2比MD1先执行自己的process_request方法

在打印一下两个自定义中间件process_request方法中的request参数,会发现它们是同

一个对象.

总结: 

-  1.中间件的process_request方法是在执行视图函数之前执行的,在路由匹配之前.

- 2.当配置多个中间件时,会按照MIDDLESARE中注册顺序,也就列表的索引值,从前到后依次执行 

- 3.不同中间件之间传递的request都是同一个对象

- 4.返回值可以是None或者HttpReponse对象,如果是HttpReponse对象,将不执行Django视图

  ​	函数,而将响应对象返回给浏览器.

## process_response

它有两个参数,一个是request,一个是response,request就是和process_request中的例子一样的对象

reponse是视图函数返回的HttpResponse对象.该方法的返回值也必须是HttpResponse对象.

给MD1和MD2加上process_response方法:

```python
class MD1(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD1中的process_request')
        # return HttpResponse('这是MD1')

    def process_response(self, request, response):
        print('这是MD1中的process_response')
        # print(id(request))  # 测试返回值的
        return response


class MD2(MiddlewareMixin):
    def process_request(self, request):
        print('这是MD2中的process_request')

    def process_response(self, request, response):
        print('这是MD2中的process_response')
        return response
```

访问视图,执行结果:

```python
这是MD1中的process_request
这是MD2中的process_request
这是index函数
61170688
这是MD2中的process_response
这是MD1中的process_response
```

看执行结果可知:

process_reaponse方法是在视图函数之后执行的,并且顺序是MD2比MD1先执行(中间件注册

中MD1比MD2先注册)

多个中间件的process_response方法是按照MIDDLEWARE中注册顺序执行的,也就是说第一个

中间件process_request方法方法先执行,而它的procsee_response方法最后执行,最后一个

中间件的process_request方法最后一个执行,它的process_response方法是最先执行.

## process_view

process_view(self, request,  view_func, view_args,  view_kwargs)

该方法有四个参数

- request是HttpResponse对象
- view_func是Django即将使用的视图函数.(它是实际的函数对象,而不是函数的名称作为字符串)
- view_args是将传递给试图的位置参数的列表
- view_kwargs是将传递给视图的关键字参数的字典.view_args和view_kwargs都不包含第一个

​        视图参数(request)

-  Django会在调用视图函数之前调用process_view方法

  它应该返回一个None或一个HttpResponse对象.如果返回None,Django将继续处理这个请求\

  执行其它中间件的process_view方法,然后在执行相应的视图.如果他返回一个HttpResponse

  对象,Django不会调用适当的视图函数.它将执行中间件的process_response方法并将应用到该

  HttpResponse并返回结果.

  给MD1和MD2添加process_view方法:

  ```pytyhon
  class MD1(MiddlewareMixin):
  
      def process_request(self, request):
          print('这是MD1中的process_request')
          # return HttpResponse('这是MD1返回值')  # 直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD1中的process_response')
          # print(id(request))  # 测试返回值的
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD1里面的process_view')
          print(view_func)   # 要执行的视图函数
          print(view_args)   # 位置参数
          print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD1返回的process_view')
  
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print('这是MD2中的process_request')
          # return HttpResponse('这是MD2返回值')  #  直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD2中的process_response')
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD2里面的process_view')
          # print(view_func)  # 要执行的视图函数
          # print(view_args)  # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD2返回的process_view')
  ```

  url设置:

  ```pytyhon
   # url(r'^index/(\d+)/', views.index),  # 接收位置参数
      url(r'^index/(?P<num>\d+)/', views.index),  # 接收关键字参数
  ```

  执行结果:

  ```python
  这是MD1中的process_request
  这是MD2中的process_request
  MD1里面的process_view
  <function index at 0x00000000039802F0>
  ()
  {'num': '233'}
  MD2里面的process_view
  这是index函数
  63333600
  233
  这是MD2中的process_response
  这是MD1中的process_response
  ```

  process_view方法是在process_response之后,在视图函数之前,执行顺序按照MIDDLEWARE中

  的注册顺序从前到后顺序执行的.

  >- 返回值可以是None或者HttpResponse对象,如果是HttpResponse对象,

  ### process_exception

  process_exception(self, request, ecception)

  该方法两个参数:

  一个HttpRequest对象

  一个exception是视图函数出现异常了才执行,它返回的值可以是一个None也可以是一个

  HttpResponse对象.如果是HttpResponse对象,Django将调用模板和中间件的

  process_response方法,并返回浏览器,否者将默认异常处理.如果返回一个None,则交给

  下一个中间件的process_exception方法来处理异常.它的顺序也是按照中间件注册顺序

  的倒序执行.

  给MD1和MD2添加这个方法:

  ```python
  class MD1(MiddlewareMixin):
  
      def process_request(self, request):
          print('这是MD1中的process_request')
          # return HttpResponse('这是MD1返回值')  # 直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD1中的process_response')
          # print(id(request))  # 测试返回值的
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD1里面的process_view')
          print(view_func)   # 要执行的视图函数
          print(view_args)   # 位置参数
          print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD1返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD1中的process_exception')
  
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print('这是MD2中的process_request')
          # return HttpResponse('这是MD2返回值')  #  直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD2中的process_response')
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD2里面的process_view')
          # print(view_func)  # 要执行的视图函数
          # print(view_args)  # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD2返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD1中的process_exception')
  
  ```

  如果视图函数中无异常,process_exception方法不执行

  在视图函数中加入一个异常:

  ```python
  
  def index(request, num):
      print('这是index函数')
      print(id(request))
      int('xxxx')  # 模拟视图函数出现的异常
      print(num)
      ret = HttpResponse('ok')    
      return ret
  
  ```

  执行结果:

  ```python
  这是MD1中的process_request
  这是MD2中的process_request
  MD1里面的process_view
  <function index at 0x00000000039602F0>
  ()
  {'num': '233'}
  MD2里面的process_view
  这是index函数
  63407384
  invalid literal for int() with base 10: 'xxxx'
  MD1中的process_exception
  invalid literal for int() with base 10: 'xxxx'
  MD1中的process_exception
  ```

  在MD1的process_exception中返回一个响应对象:

  ```python	
  
  class MD1(MiddlewareMixin):
  
      def process_request(self, request):
          print('这是MD1中的process_request')
          # return HttpResponse('这是MD1返回值')  # 直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD1中的process_response')
          # print(id(request))  # 测试返回值的
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD1里面的process_view')
          # print(view_func)   # 要执行的视图函数
          # print(view_args)   # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD1返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD1中的process_exception')
          return HttpResponse(str(exception))
  
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print('这是MD2中的process_request')
          # return HttpResponse('这是MD2返回值')  #  直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD2中的process_response')
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD2里面的process_view')
          # print(view_func)  # 要执行的视图函数
          # print(view_args)  # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD2返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD2中的process_exception')
  ```

  执行结果:

  ```python
  这是MD1中的process_request
  这是MD2中的process_request
  MD1里面的process_view
  MD2里面的process_view
  这是index函数
  60750592
  invalid literal for int() with base 10: 'xxxx'
  MD2中的process_exception
  invalid literal for int() with base 10: 'xxxx'
  MD1中的process_exception
  这是MD2中的process_response
  这是MD1中的process_response
  ```

  注意:

  ​	这里并么有没执行MD2的process_exception方法,因为MD1中的process_exception方法

  直接返回一个响应对象.

  浏览器执行效果:

  ```python
  invalid literal for int() with base 10: 'xxxx'
  ```

  

  ### process_template_response

  process_template_response(self, request, response)

  它的`参数一个HttpResponse对象,response是TemplateResponse对象

  (由视图函数或者中间件产生)

  process_template_response是在视图函数执行完成后立即执行,但是它有一个前提条件,那

  就是视图函数返回对象有一个render()方法(或者表明该对象是一个TemplateResponse对象或等价方法)

  在MD1和MD2中添加:

  ```python
  
  class MD1(MiddlewareMixin):
  
      def process_request(self, request):
          print('这是MD1中的process_request')
          # return HttpResponse('这是MD1返回值')  # 直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD1中的process_response')
          # print(id(request))  # 测试返回值的
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD1里面的process_view')
          # print(view_func)   # 要执行的视图函数
          # print(view_args)   # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD1返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD1中的process_exception')
          return HttpResponse(str(exception))
  
      def process_template_response(self, request, response):
          print("MD1 中的process_template_response")
          return response
  
  
  class MD2(MiddlewareMixin):
      def process_request(self, request):
          print('这是MD2中的process_request')
          # return HttpResponse('这是MD2返回值')  #  直接跳过视图,走response
  
      def process_response(self, request, response):
          print('这是MD2中的process_response')
          return response
  
      def process_view(self, request, view_func, view_args, view_kwargs):
          # print('*' * 120)
          print('MD2里面的process_view')
          # print(view_func)  # 要执行的视图函数
          # print(view_args)  # 位置参数
          # print(view_kwargs)  # 关键字参数
          # return HttpResponse('MD2返回的process_view')
  
      def process_exception(self, request, exception):
          print(exception)
          print('MD2中的process_exception')
  
      def process_template_response(self, request, response):
          print("MD2 中的process_template_response")
          return response
  
  ```

  在视图函数中:

  ```pythion
  def index(request, num):
      print('这是index函数')
      print(id(request))
      # int('xxxx')  # 模拟视图函数出现的异常
      print(num)
      
      def render():
          print('in index/render')
          return HttpResponse('没问题')
      ret = HttpResponse('ok')
      ret.render = render
      return ret
  
  ```

  执行结果:

  ```python
  这是MD1中的process_request
  这是MD2中的process_request
  MD1里面的process_view
  MD2里面的process_view
  这是index函数
  63341792
  233
  MD2 中的process_template_response
  MD1 中的process_template_response
  in index/render
  这是MD2中的process_response
  这是MD1中的process_response
  ```

  浏览器返回:

  ```python
  没问题
  ```

  分析:视图函数执行完之后，立即执行了中间件的process_template_response方法，顺序是倒序，先执行MD1的，在执行MD2的，接着执行了视图函数返回的HttpResponse对象的render方法，返回了一个新的HttpResponse对象，接着执行中间件的process_response方法。

  

  ## 中间件的执行流程

  请求到达中间件之后，先按照正序执行每个注册中间件的process_reques方法，process_request方法返回的值是None，就依次执行，如果返回的值是HttpResponse对象，不再执行后面的process_request方法，而是执行当前对应中间件的process_response方法，将HttpResponse对象返回给浏览器。也就是说：如果MIDDLEWARE中注册了6个中间件，执行过程中，第3个中间件返回了一个HttpResponse对象，那么第4,5,6中间件的process_request和process_response方法都不执行，顺序执行3,2,1中间件的process_response方法。 

  ![](C:\Users\Administrator\Desktop\中间件执行流程.png)

   

  process_request方法都执行完后，匹配路由，找到要执行的视图函数，先不执行视图函数，先执行中间件中的process_view方法，process_view方法返回None，继续按顺序执行，所有process_view方法执行完后执行视图函数。假如中间件3 的process_view方法返回了HttpResponse对象，则4,5,6的process_view以及视图函数都不执行，直接从最后一个中间件，也就是中间件6的process_response方法开始倒序执行。 

  ![](C:\Users\Administrator\Desktop\中间件执行过程2.png)

  

  process_template_response和process_exception两个方法的触发是有条件的，执行顺序也是倒序。总结所有的执行流程如下： 

  

![](C:\Users\Administrator\Desktop\中间件触发执行过程1.png)

![](C:\Users\Administrator\Desktop\中间件触发执行2.png)



Django请求流程图:

![](C:\Users\Administrator\Desktop\Django请求流程2.png)