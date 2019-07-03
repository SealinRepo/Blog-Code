---
title: Idea+JRebel全自动热部署
tags:
  - 工具
  - IDEA
originContent: ''
categories:
  - 技术
toc: true
date: 2019-07-02 23:24:19
---

注意:Idea插件库的版本已经和谐服务器注册的方式了,没办法激活的,所以到这下载吧.
链接：https://pan.baidu.com/s/1eT418Ls 密码：2q1w
Idea离线安装插件就不占篇幅了,大概如下:
下载完以后是个ZIP压缩包,打开IDEA,关闭所有项目,Settings -- plugins -- install from disk,选择刚刚下载的ZIP就行了.
装完以后打开IDEA,Help-JRebel-Activation
![Active0.png.png](https://upload-images.jianshu.io/upload_images/8936944-b94e13ae6aae4e45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注册服务器: http://lic.sealin.net/{username}
这里username和email随意填就行
![Active.png](https://upload-images.jianshu.io/upload_images/8936944-da83325205c70e4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用及自动部署:
依次打开View - Tool Windows - JRebel
![Usage0.png](https://upload-images.jianshu.io/upload_images/8936944-1a26a98d2447d564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
就能看到如下窗口了:
![Usage1.png.png](https://upload-images.jianshu.io/upload_images/8936944-f56d3031326c3786.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在需要纳入JRebel管理的项目后边勾选,运行项目记得使用多出的两个按钮运行,分别是运行和调试.
![Debug.png](https://upload-images.jianshu.io/upload_images/8936944-1cc49dd7e745783a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候已经实现热部署了,不过需要每次修改源码后手动执行UPDATE操作,我们可以结合Idea自身的Debug Hot swap功能,在"运行和调试配置"中,配置自动执行UPDATE操作.
![DebugConf.png](https://upload-images.jianshu.io/upload_images/8936944-9cdcb205e8728ee9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样我们改完源码后,切换到浏览器的时候后台就会自动帮我们部署项目了,再也不用手动点一下左边的UPDATE按钮,这里如果没有[Update classes and resource]选项的话,说明你部署的artifact包,在Deployment选项将WEB包更改为exploded就可以了.