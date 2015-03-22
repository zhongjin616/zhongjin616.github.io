hi there ~:

i will talk about Nginx varible  ngx.var.variable_name. what exactly variable_name can be?

1. Nginx 变量的值只有一种类型---字符串。没有值的变量也有两种特殊的值： invalid & not found

    invalid : 创建却并未被赋值。 如$foo
    
    not found: URL的query string中没有xxx这个参数，则ngx.var.arg_xxx内建变量的值就是not found.

    这两种特殊值和空字符串（“”）和 0 以及 false都不是相等的。 这和js中undefined 和null； lua中的nil是一个道理。

2. nginx.conf 中的set指令是标准ngx _ rewrite 模块的配置指令。echo指令也是第三方ngx _ echo模块的配置指令。

3. nginx变量一旦创建，则在整个nginx.conf中都是可见的。但他不是全局变量，不能跨出块的区域，但**可以通过内部跳转**实现 ngx _ echo :echo _ exec
    通过rewrite指令可以发起301和302这样的“外部跳转”。

4. 使用set指令隐式创建的Nginx变量，称为“用户变量”。而由Nginx核心模块和其他各个模块提供的“预定义变量”称为”内建变量“（builtin variables）
    如$uri 和 $request_uri.

5. 特别常用的$arg_xxx变量群。他是当前请求的URI中的query string参数(不区分大小写)，而且还是未解码的原始形式。如hello%20world
    可以使用第三方ngx_set_misc提供的set_unescape_uri配置指令： set_unescape_uri $name $arg_name

    类似的变量群还有很多，如取cookie值的$cookie_xxx变量群， 取请求头的$http_xxx变量群，取响应头的$sent_http_xxx变量群，都来自ngx_http_core模块。

6. 内建变量都是**只读**的，不应该对他们进行赋值操作。

7. 但有一个特例：$args支持改写。他返回当前请求的uri参数**串**。但改写后会改变$arg_xxx的值。



