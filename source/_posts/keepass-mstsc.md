---
title: Keepass实现远程桌面(mstsc)自动登录
tags:
  - 工具
categories:
  - 工具
toc: true
date: 2019-07-08 20:02:26
---

无需插件.

# 配置:
## 添加记录
## 记录选项卡配置:
![image.png](/images/2019/07/08/4be2fa10-a176-11e9-bf34-997617e8f98c.png)
## 地址栏:
```shell
cmd://"C:\Windows\System32\cmd.exe" /c cmdkey.exe /generic:TERMSRV/{S:SERVER} /user:{S:DOMAIN}{USERNAME} /pass:"{PASSWORD}" & mstsc.exe /v:{S:SERVER} & cmdkey.exe /delete:TERMSRV/{S:SERVER}
```
## 高级选项卡配置
![image.png](/images/2019/07/08/b1de0080-a176-11e9-bf34-997617e8f98c.png)

## 使用方法:
添加好记录后, 选中记录, 右键 -> 网址 -> 打开, 或者点击左上角的快捷方式, 将自动运行mstsc, 并使用设置的账号密码进行登录.
![image.png](/images/2019/07/08/1ae6c6b0-a178-11e9-bf34-997617e8f98c.png)

# 简化
## URL替换
Keepass提供了一个URL替换的功能, 我们可以按照上述原理, 简化我们的配置过程, 把创建windows认证这一步交给keepass自动完成.
## 配置界面
工具 -> 选项 -> 集成 -> URL 替代(在界面下角)
![image.png](/images/2019/07/11/11813d00-a37e-11e9-9e9d-7bf2a7893497.png)
## 自定义规则
添加一条自己的规则, 配置如下:
```shell
cmd://cmd /c cmdkey /generic:TERMSRV/{URL:RMVSCM} /user:{USERNAME} /pass:"{PASSWORD}" && mstsc /v:{URL:RMVSCM} && cmdkey /delete:TERMSRV/{URL:RMVSCM}
```
![image.png](/images/2019/07/11/42f6fb40-a37e-11e9-9e9d-7bf2a7893497.png)
## 添加记录
配置好自定义替换规则后, 我们就可以像添加普通记录一样添加远程桌面的账号密码记录了, 并且这种方式不用在高级选项卡添加变量了, 需要注意的是比如你的服务器地址是192.168.1.250, 那么在URL一项应填写: rdp://192.168.1.250, 让我们刚刚添加的替换规则能够命中.添加好以后一样点击打开URL图标, 就可以自动打开mstsc然后自动登录了.
![image.png](/images/2019/07/11/dd735240-a37e-11e9-9e9d-7bf2a7893497.png)
![image.png](/images/2019/07/11/ee4c10c0-a37e-11e9-9e9d-7bf2a7893497.png)