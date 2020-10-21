---
title: 连接VPN后无法调试本地Dubbo服务的问题
tags:
  - JAVA
originContent: ''
categories:
  - 技术
  - JAVA
  - 经验
toc: true
date: 2020-09-13 15:29:22
---

# 问题
连接VPN或者创建多个网卡后(如安装虚拟机/docker等服务), 本机会存在多个IP地址, 这种情况下, 启动本地服务提供者后, 创建的dubbo服务绑定的网卡可能会无法准确找到物理网卡地址, 导致消费端启动后无法找到服务.

# 问题分析
通过查看dubbo的源码, 可以发现获取IP地址的事情是调用jdk提供的类: java.net.InetAddress#getLocalHost
主要逻辑如下:
```java
    public static InetAddress getLocalHost() throws UnknownHostException {

        SecurityManager security = System.getSecurityManager();
        try {
            String local = impl.getLocalHostName();

            if (security != null) {
                security.checkConnect(local, -1);
            }

            if (local.equals("localhost")) {
                return impl.loopbackAddress();
            }

            InetAddress ret = null;
            synchronized (cacheLock) {
                long now = System.currentTimeMillis();
                if (cachedLocalHost != null) {
                    if ((now - cacheTime) < maxCacheTime) // Less than 5s old?
                        ret = cachedLocalHost;
                    else
                        cachedLocalHost = null;
                }

                // we are calling getAddressesFromNameService directly
                // to avoid getting localHost from cache
                if (ret == null) {
                    InetAddress[] localAddrs;
                    try {
                        localAddrs =
                            InetAddress.getAddressesFromNameService(local, null);
                    } catch (UnknownHostException uhe) {
                        // Rethrow with a more informative error message.
                        UnknownHostException uhe2 =
                            new UnknownHostException(local + ": " +
                                                     uhe.getMessage());
                        uhe2.initCause(uhe);
                        throw uhe2;
                    }
                    cachedLocalHost = localAddrs[0];
                    cacheTime = now;
                    ret = localAddrs[0];
                }
            }
            return ret;
        } catch (java.lang.SecurityException e) {
            return impl.loopbackAddress();
        }
    }
```
大致流程为: 获取当前主机名 --> 检查主机名是否可以连接 --> 如果主机名为localhost, 直接获取本地回环地址(一般为127.0.0.1) --> 查询nameService获取当前主机名的实际地址

可以发现这个地址在获取的过程中, 是需要去查询域名解析服务的, 域名解析流程首先会从本地hosts文件中获取地址, 也就是说, 我们只需要设置一个跟主机名一样的地址到hosts文件中, 就可以让dubbo获取到自己想要的地址了.
![image.png](/images/2020/09/13/ada1bf87-958d-411e-ba23-722d8439d032.png)