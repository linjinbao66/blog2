---
title: jdk的代理Proxy使用
date: 2019-10-12
---

## 使用jdk的代理-Proxy动态代理

核心思想：反射，不改变原有代码，使用代理对象执行原有方法，侵入原始类内部 用处：解耦，切面编程

CalculatorProxy.java

```java
public class CalculatorProxy {

    /**
       *    代理
     * @param calcultor
     * @return
     */
    public static Calcultor getCaluctor(Calcultor calcultor) {
        ClassLoader loader = calcultor.getClass().getClassLoader();
        Class<?>[] interfaces = calcultor.getClass().getInterfaces();
        InvocationHandler h = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("代理对象执行中，，，，，，，，，");
                return method.invoke(calcultor, args);
            }
        };
        return (Calcultor) Proxy.newProxyInstance(loader, interfaces, h);
    }

}
```

TestAOP.java

```java
public class TestAOP {
    @Test
    public void test01() {
        Calcultor calcultor = new MyMathCaluctor();
        Calcultor Proxy = CalculatorProxy.getCaluctor(calcultor);
        Proxy.add(2, 3);
    }
}
```

## Proxy动态代理缺陷

1. 编写困难
2. 如果目标对象没有实现任何接口，则无法创建代理对象`Proxy.newProxyInstance(loader, interfaces, h);` `interfaces` 无法传参
