---
title: Java Native Interface(JNI)小记
tags:
  - JAVA
originContent: ''
categories:
  - 技术
  - JAVA
toc: true
date: 2019-07-02 23:29:49
---

    看JAVA源码的时候发现很多类使用到native关键字,查了下发现跟C#的import作用差不多,就是用来调用动态链接库dll文件方法的关键字。

自己试试才更好理解，以下是我折腾这个的过程：

一：

    先用JAVA写好一个调用类（就当这个DLL已经存在了）。

```

packageorg.uroot.jni;

/**

* Created by Sealin on 2017-12-06.

* http://java.uroot.org/

*/

public classHelloJNI {

    public native voidsay();

    static{

       //这里引用名要和生成的DLL文件名一致，引用的时候不写.dll

        System.loadLibrary("org_uroot_jni_HelloJNImpl");

    }

    public static voidmain(String[] args) {

        HelloJNI jni =newHelloJNI();

        jni.say();

    }

}

```

二：编译这个类，生成.class文件

```

javac org.uroot.jni.HelloJNI

```

三：使用javah生成这个class文件的*.h文件（C语言的头文件），我生成的文件名是【org_uroot_jni_HelloJNI.h】

```

javah -jni HelloJNI

```

如果一切正常，那*.h文件已经生成在当前目录了。这个文件一般不要去修改它，因为它生成了对应class文件的结构和调用方法的引用等信息，和JAVA文件中的say方法结构，结构如下：

```

JAVA_完整包路径_类名_方法名

//比如我这个示例生成的.h抽象方法名是：

Java_org_uroot_jni_HelloJNI_say (JNIEnv *, jobject);

```

四：新建一个c或者cpp文件，这里叫【org_uroot_jni_HelloJNImpl.cpp】，引入生成的头文件，并实现上述头文件的方法部分

```

#include "org_uroot_jni_HelloJNI.h"

#include

#include "jni.h"

#include "stdafx.h"

/**

*Class:    org_uroot_jni_HelloJNI

* Method:    say

* Signature: ()V

*/

JNIEXPORT void JNICALL Java_org_uroot_jni_HelloJNI_say

(JNIEnv *, jobject) {

    printf("Hello, I'm Java Native Interface\n");

    return;

}

void main() {

    Java_org_uroot_jni_HelloJNI_say(nullptr, NULL);

}

```

五：把【%JAVA_HOME%\include\jni.h】和【%JAVA_HOME%\include\win32\jni_md.h】放到当前文件夹，编译【org_uroot_jni_HelloJNImpl.cpp】，我这里因为装了VS，所以直接用cl工具进行DLL编译

```

cl /LD org_uroot_jni_HelloJNImpl.cpp

```

我在这里弄了好久，因为一直提示找不到jni.h,打开javah生成的头文件看了下，发现生成的引入方式是#include ，因为在path环境变量里边没有加入include和include/win32，所以导致了这个问题，将我们生成的头文件和新建的实现文件此处引用都改为【 #include "jni.h"】，编译通过。

六：确认*.class和刚刚生成的*.dll都在同一个目录中后，就可以运行试试效果了

```

java org.uroot.jni.HelloJNI

```

这里加入包名后可能会出现无法找到DLL的情况，确认引用名和DLL文件名一样后还是说找不到，但是写java文件的时候不使用包就不存在这个问题，到网上查了下，程序在执行的时候会在PATH环境变量中去查找引用DLL，所以加入了个文件夹到系统PATH，然后把刚刚生成的DLL文件放到里边，再执行就可以了

```

_>java org.uroot.jni.HelloJNI

_>Hello, I'm Java Native Interface

```