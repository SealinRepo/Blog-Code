---
title: Mac安装软件后打不开解决方案
tags:
  - MAC
  - 操作系统
categories:
  - 经验
toc: true
date: 2020-03-06 09:46:09
---

# 不受信任的开发者
打开finder -》 应用程序 -》 在app上右键(或按住control再点击APP)， 在弹出的选项中打开即可。

# 软件已损坏
```bash
sudo xattr -d com.apple.quarantine /Applications/xx.app
```
将xx.app替换为提示损坏的app名称， 如qq.app

# 因为Apple无法检查其是否包含恶意软件

启用任何来源
```bash
sudo spctl --master-disable
```

关闭任何来源
```bash
sudo spctl --master-enable
```