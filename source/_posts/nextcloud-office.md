---
title: docker搭建nextcloud个人网盘在线编辑office文件
tags:
  - nextcloud
  - 网盘
  - 在线文档
originContent: ''
categories:
  - 工具
toc: true
date: 2020-06-10 11:37:35
---

# 简述
使用nextcloud运行网盘服务，再加入onlyoffice/documentserver实现在线编辑功能。

# 运行nextcloud
将以下几个目录持久化到宿主机，防止配置和数据丢失
```bash
/var/www/html/data
/var/www/html/config
/var/www/html/custom_apps
```
同时，为容器传入redis环境变量，以触发nextcloud的缓存机制，缓存的环境变量名称可以通过 /var/www/html/config/redis.config.php 看到，内容如下：

```php
<?php
if (getenv('REDIS_HOST')) {
  $CONFIG = array (
    'memcache.distributed' => '\OC\Memcache\Redis',
    'memcache.locking' => '\OC\Memcache\Redis',
    'redis' => array(
      'host' => getenv('REDIS_HOST'),
      'port' => getenv('REDIS_HOST_PORT') ?: 6379,
      'password' => getenv('REDIS_HOST_PASSWORD'),
    ),
  );
}
```

可以看到如果存在REDIS_HOST这个环境变量，就可以触发缓存，我看网上很多文章的做法是修改apcu.config.php，手动加入以上redis配置代码块，感觉没这必要。
运行容器

```bash
docker run -d --name cloud -p 888:80 -v /var/data/nextcloud/apps:/var/www/html/custom_apps -v /var/data/nextcloud/config:/var/www/html/config -v /var/data/nextcloud/data:/var/www/html/data -e REDIS_HOST='192.168.11.83' -e REDIS_HOST_PORT='6380' -e REDIS_HOST_PASSWORD='your-password' nextcloud:17.0.1
```

启动后稍等片刻，访问http://ip:888，即可看到网盘安装界面。
推荐选择mysql作为数据库。

# 安装onlyoffice服务
```bash
docker run -i -t -d --name officeserver -p 51000:80 --restart=always onlyoffice/documentserver
```

如果运行没问题，进入http://ip:51000可以看到如下界面
![image.png](/images/2020/06/10/4f2961cb-a0ea-4b71-bd1f-096af7ade94e.png)

# 安装nextcloud插件
由于国内网络环境不够因特乃熊，网盘里的插件管理界面99%的时间是打不开的，等两个小时也只能显示白板（至少我这边是这样）。
所以使用离线方式安装插件。
## 下载插件
```bash
cd /var/data/nextcloud/apps
git clone https://github.com/ONLYOFFICE/onlyoffice-nextcloud.git onlyoffice
# 33是nextcloud访问目录和文件需要的权限
chown -R 33:sealin onlyoffice
```

## 配置插件
将插件源码下载到apps目录以后，最好重启下nextcloud,再去启用插件
```bash
docker restart cloud
```

重启后进入页面，到插件设置
![image.png](/images/2020/06/10/6bf4a030-6a95-44ad-b1a5-dcec1cf21edc.png)

启用列表中新出现的onlyoffice插件
![image.png](/images/2020/06/10/ea543d47-0989-4ccc-8d82-07c6c1d47d74.png)

启用完成后进入系统设置，左侧的选项会多出一个onlyoffice
![image.png](/images/2020/06/10/42687ab2-bab6-4860-b23a-532a69ad714c.png)

填写自己的documentserver服务IP和端口
![image.png](/images/2020/06/10/adb9827d-64ac-4646-bf1c-739c62571243.png)

点击保存后，如果验证51000端口的服务没问题，会出现允许使用在线编辑的格式选项。
![image.png](/images/2020/06/10/20cc66d3-c432-439b-8d26-f6ed3769b206.png)

勾选需要的格式后，就可以到网盘主界面的文件列表中添加一个比如word文档，来试试在线编辑了。
![image.png](/images/2020/06/10/9d35969c-0d32-43fc-9349-4e4f4bc38d0b.png)
可以看到多出来了几个新建文件的格式，试试word
![image.png](/images/2020/06/10/3aa69808-f4fb-4756-98d9-53d4f1fcf16d.png)
![image.png](/images/2020/06/10/13bc3a09-4146-4ff6-a760-596c973535f4.png)
带劲