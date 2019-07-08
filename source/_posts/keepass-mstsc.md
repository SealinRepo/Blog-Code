---
title: Keepass实现远程桌面(mstsc)自动登录
tags:
  - 工具
originContent: ''
categories:
  - 工具
toc: true
date: 2019-07-08 20:02:26
---

无需插件.

# 添加记录
## 记录选项卡配置:
![image.png](/images/2019/07/08/4be2fa10-a176-11e9-bf34-997617e8f98c.png)
## 地址栏:
```shell
cmd://"C:\Windows\System32\cmd.exe" /c cmdkey.exe /generic:TERMSRV/{S:SERVER} /user:{S:DOMAIN}{USERNAME} /pass:"{PASSWORD}" & mstsc.exe /v:{S:SERVER} & cmdkey.exe /delete:TERMSRV/{S:SERVER}
```
## 高级选项卡配置
![image.png](/images/2019/07/08/b1de0080-a176-11e9-bf34-997617e8f98c.png)

# 使用方法:
添加好记录后, 选中记录, 右键 -> 网址 -> 打开, 或者点击左上角的快捷方式, 将自动运行mstsc, 并使用设置的账号密码进行登录.
![image.png](/images/2019/07/08/1ae6c6b0-a178-11e9-bf34-997617e8f98c.png)