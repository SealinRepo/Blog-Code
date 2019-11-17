---
title: Nginx+PHP实现无后缀访问
tags:
  - 基础服务
  - Nginx
  - PHP
originContent: ''
categories:
  - 技术
toc: false
date: 2019-11-12 10:02:59
---

其实这方面的资料在网上找了下不算多,但是也算很基本的东西吧,博主之前一直在思考能不能不通过服务器转发或者跳转就达到目的,也就是通过http://xxxx.com/test 这样的方式访问http://xxxx.com/test.php之类的页面,最初尝试直接在php中实现,发现php的配置文件是不能这样解析的,在php的配置文件中写上正则表达式会导致php-fpm直接挂掉.
所以只能从nginx配置入手,nginx的配置文件是支持正则的,所以要么通过请求地址判断是否301转发,此外也可以在php的关联配置下新增一个配置组

```c
#这里写需要正常解析的后缀,这里加了解析类型后别忘了在php-fpm的配置文件中启用security.limit_extensions = .php .php7
location ~ (.php7|.php)$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /$document_root$fastcgi_script_name;
    include        fastcgi_params;
}
#这里为新增的解析类型,如果需要准确匹配可以自行修改正则,此处为示例,解析除上述配置外的所有文件类型,也包括无后缀文件
location ~ $ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /$document_root$fastcgi_script_name.php;
    include        fastcgi_params;
}
```
经测试这样是可行的
![image.png](/images/2019/11/12/57f493d0-04f0-11ea-a4e8-15b79e6f3bf7.png)
当然也可以通过301转向实现.