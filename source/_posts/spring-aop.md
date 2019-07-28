---
title: Spring boot 使用AOP对方法进行增强和统一异常处理
tags:
  - JAVA
  - Spring
  - AOP
categories:
  - 技术
  - JAVA
toc: false
date: 2019-07-28 01:28:46
---

# 声明切入点
```java
@Aspect
public class AopPointCut {
    /**
     * 切入点
     */
    @Pointcut("execution(void net.sealin.web.demo.behavior.aop.bean.AopCalc.calcDiv(int, int))")
    void pointCut(){}
}
```
注意类上声明的@Aspect注解
execution表达式的值为方法完整签名, 访问级别可以忽略.
# 注册到容器
可以在配置类中使用@Bean注解手动将切面类加入容器, 或者标记为@Component扫描此包路径
# 方法增强
## 用法
在切面类中加入对应时机的处理方法, 可以实现目标方法的增强.
```java
    @After(value = "pointCut()")
    public void after() {
        System.out.println("@After 被执行了");
    }

```

## 支持的注解
|注解|说明|
|-|-|
|@Before|方法执行前调用|
|@After|方法执行后调用|
|@AfterReturning|方法返回后调用|
|@AfterThrowing|方法抛出异常后调用|
|@Around|方法环绕, 全权接管目标方法的调用过程|

# 异常处理
想通过AOP实现全局异常处理, 使用@Around并且将调用过程用try-catch包裹即可.
```java
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("@Around 开始");
            Object result = joinPoint.proceed();
            System.out.println("@Around 结束");
            return result;
        } catch (Throwable throwable) {
            System.out.println(String.format("异常已被拦截, 异常信息为: %s", throwable.getMessage()));
        }
        return null;
    }
```
## 获取方法信息
执行时机的方法声明时, 可以声明一个JoinPoint类型的入参, 此对象中包含方法调用的很多属性, 包括参数, 调用类型, 调用目标等等, 具体输出结果如下:
```java
getKind: method-execution
getSingnature: void net.sealin.web.demo.behavior.aop.bean.AopCalc.calcDiv(int,int)
getSourceLocation: org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint$SourceLocationImpl@3e27aa33
getStaticPart: execution(void net.sealin.web.demo.behavior.aop.bean.AopCalc.calcDiv(int,int))
getTarget: net.sealin.web.demo.behavior.aop.bean.AopCalc@2e385cce
getThis: net.sealin.web.demo.behavior.aop.bean.AopCalc@2e385cce

```
