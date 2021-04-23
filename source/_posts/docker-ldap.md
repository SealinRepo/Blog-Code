---
title: NextCloud集成OpenLdap实现单点登录
tags:
  - 基础服务
  - nextcloud
  - LDAP
categories:
  - 折腾实录
toc: true
date: 2021-04-23 14:13:16
---

# LDAP部署

openLdap是一套开源的AD域管理工具(不过看起来很复古, 不知道有没有更好用的替代方案)

```
docker run \
    -d \
    -p 389:389 \
    -p 636:636 \
    -v /data/openldap/ldap:/var/lib/ldap \
    -v /data/openldap/slapd.d:/etc/ldap/slapd.d \
    --env LDAP_ORGANISATION="org" \
    --env LDAP_DOMAIN="sealin.net" \
    --env LDAP_ADMIN_PASSWORD="123456" \
    --name openldap \
    --hostname openldap-host\
    osixia/openldap:1.4.0
```

版本建议用1.4.0, 更高版本经测试在nextcloud中获取分组存在问题

# phpLdapAdmin安装

phpLdapAdmin是管理ldap数据的客户端

```
docker run \
-p 8080:80 \
--privileged \
--name ldap-admin \
--env PHPLDAPADMIN_HTTPS=false \
--env PHPLDAPADMIN_LDAP_HOSTS=192.168.11.84 \
--detach osixia/phpldapadmin
```

使用时将 `192.168.11.84`替换为ldap的部署地址

## 使用

在浏览器打开phpLdapAdmin: http://ip:8080/

## 登陆

用户名格式:

```
cn=admin,dc=sealin,dc=net
```

dc为启动ldap实例时的 --env LDAP_DOMAIN="sealin.net"值, 以.分隔为多个dc,如:

```
# 如果域名为: sealin.net.cn, 对应的用户名为
cn=admin,dc=sealin,dc=net,dc=cn
```

## 创建用户

`Create new entry here`

先创建:`Samba: Domain`, 产生一个SID
接着创建分组:`Samba: Group Mapping`
然后创建用户: `Samba: Account`
为用户添加显示名称字段:
在左侧选中刚刚添加的用户, 在界面上打开此功能: `Add new attribute`, 在下拉框中选择`displayName`, 填写要用于显示的名称.

# 配置NextCloud

首先从`设置-应用`界面启用插件, 默认是关闭的:
`LDAP user and group backend`
启用该插件后, 在`设置`界面左侧, 会多出一项`LDAP-AD整合`

## 填写服务器地址

填入服务地址(不填写端口), 然后点`检测端口`, 没问题的话会检测出我们启动的端口:389

## 填写用户信息

按照上述用户名格式, 填写用户和密码, 接着点`保存凭据`

## 测试连接

依次点下方的`检测基础DN` -- `测试基础DN`
如无意外, 下方的状态标签会变成`配置完成`, 并附带绿色圆点

## 用户选项卡

上一步通过后, 点`继续`按钮, 在`用户`选项卡的`只有这些对象类：`选项中, 选择`inetOrgPerson`, 选好以后可以点击左下方的`验证设置和统计用户`, 如果旁边显示`发现 1 个用户`, 说明配置通过, 可以`继续`

## 登陆属性选项卡

在此处, 可以配置自定义的字段用于nextcloud的登陆用户名, 选择需要的字段, 点`继续`

## 群组选项卡

此选项卡, 可以限制nextcloud从哪些组获取ldap中的用户信息, 选择好需要的分组后, 点`验证设置和统计分组数`, 提示`发现 * 个分组`, 说明配置无误.此时, 我们的nextcloud已经集成了ldap的数据, 不过有些小问题需要处理

## 显示自定义用户名

在nextcloud的用户管理界面, 可以看到ldap的用户名称上显示了一串UUID, 不易于用户理解.
我们可以回到nextcloud中的ldap配置界面, 打开右侧的`砖家`功能.
修改`用户 UUID 属性：`这一项的值, 改成ldap用户信息中存在的字段, 比如默认带的`sn`字段, 填上sn
改完以后, `清除用户-LDAP用户映射`, 再回到nextcloud用户管理界面, 可以发现用户名已显示正常

# 结语

ldap支持为用户创建自定义属性, 结合nextcloud配置LDAP界面`高级`-`特殊属性`设置, 基本用户同步过来后可以初始化所有信息, 如配额, 邮箱等.当然也有一些限制, 在nextcloud中创建用户无法选择ldap中的分组, 也就是只能通过LDAP创建用户给cloud用, cloud无法向ldap中添加用户, 如果用户较多的话无法让各组分而治之, 管理ldap的人亚历山大.

