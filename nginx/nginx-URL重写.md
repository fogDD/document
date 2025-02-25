# nginx-URL重写

### URL重写（rewrite）介绍

URL重写是指将一个URL请求重新写成网站可以处理的另一个URL的过程。这样说可能不是很好理解，举个例子来说明一下，在开发中可能经常遇到这样的需求，比如通过浏览器请求的http://localhost:8080/getUser?id=1，但是需要通过SEO优化等等原因，需要把请求的地址重写为http://localhost:8080/getUser/1这样的URL，从而符合需求或者更好的被网站阅读。

当遇到这种请求的时候，就需要使用到UrlRewrite重写或者使用一些网关路由，如SpringCloud的Gateway，Zuul，又或者是Nginx来实现这个功能。

和apache等web服务软件一样，rewrite的主要功能是实现URL地址的重定向。Nginx的rewrite功能需要PCRE软件的支持，即通过perl兼容正则表达式语句进行规则匹配的。默认参数编译nginx就会支持rewrite的模块，但是也必须要PCRE的支持。

**rewrite和location的功能有点相像，都能实现跳转，主要区别在于rewrite常用于同一域名内更改获取资源的路径，而location是对一类路径做控制访问和反向代理，可以proxy_pass到其他服务器。**

Nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。 rewrite只能放在server{},location{},if{}中， 并且只能对域名后边的除去传递的参数外的字符串起作用。

Rewrite主要的功能就是实现URL的重写，Nginx的Rewrite规则采用Pcre，perl兼容正则表达式的语法规则匹配，如果需要Nginx的Rewrite功能，在编译Nginx之前，需要编译安装PCRE库。 通过Rewrite规则，可以实现规范的URL、根据变量来做URL转向及选择配置。



### rewrite语法格式及参数语法说明如下:

` rewrite`   `<regex>`  `<replacement>`  `[flag]`;

  关键字         正则            替代内容         flag标记

 

  **关键字：其中关键字error_log不能改变**

  **正则：perl兼容正则表达式语句进行规则匹配**

  **替代内容：将正则匹配的内容替换成replacement**

  **flag标记：rewrite支持的flag标记**

**** 

**flag标记说明：**

**last #本条规则匹配完成后，继续向下匹配新的location URI规则**

**break #本条规则匹配完成即终止，不再匹配后面的任何规则**

**redirect #返回302临时重定向，浏览器地址会显示跳转后的URL地址**

**permanent #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址**



### rewrite参数的标签段位置：

server,location,if



### 例子：

rewrite ^/(.*) http://www.czlun.com/$1 permanent;

说明：                    

rewrite为固定关键字，表示开始进行rewrite匹配规则

regex部分是 ^/(.*) ，这是一个正则表达式，匹配完整的域名和后面的路径地址

replacement部分是http://www.czlun.com/$1 $1，是取自regex部分()里的内容。匹配成功后跳转到的URL。

flag部分 permanent表示永久301重定向标记，即跳转到新的 http://www.czlun.com/$1 地址上



### regex 常用正则表达式说明

| **字符**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| **\**         | 将后面接着的字符标记为一个特殊字符或一个原义字符或一个向后引用。如“\n”匹配一个换行符，而“\$”则匹配“$” |
| **^**         | 匹配输入字符串的起始位置                                     |
| **$**         | 匹配输入字符串的结束位置                                     |
| *****         | 匹配前面的字符零次或多次。如“ol*”能匹配“o”及“ol”、“oll”      |
| **+**         | 匹配前面的字符一次或多次。如“ol+”能匹配“ol”及“oll”、“oll”，但不能匹配“o” |
| **?**         | 匹配前面的字符零次或一次，例如“do(es)?”能匹配“do”或者“does”，"?"等效于"{0,1}" |
| **.**         | 匹配除“\n”之外的任何单个字符，若要匹配包括“\n”在内的任意字符，请使用诸如“[.\n]”之类的模式。 |
| **(pattern)** | 匹配括号内pattern并可以在后面获取对应的匹配，常用$0...$9属性获取小括号中的匹配内容，要匹配圆括号字符需要\(Content\) |



### rewrite 企业应用场景

Nginx的rewrite功能在企业里应用非常广泛：

可以调整用户浏览的URL，看起来更规范，合乎开发及产品人员的需求。

为了让搜索引擎搜录网站内容及用户体验更好，企业会将动态URL地址伪装成静态地址提供服务。u 网址换新域名后，让旧的访问跳转到新的域名上。例如，访问京东的360buy.com会跳转到jd.com

根据特殊变量、目录、客户端的信息进行URL调整等



### 配置展示

```
    server {
      listen      80;
      server_name dd.com;
      rewrite ^/(.*)http://www.dd.com/$1 permanent;
```

确认没问题重启nginx，



### 验证

打开浏览器访问dd.com

页面打开后，URL地址栏的dd.com变成了www.dd.com说明URL重写成功。