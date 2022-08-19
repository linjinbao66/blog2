---
title: 简单工程模式
date: 2019-10-14
---

## 2019-10-14-工厂模式

## 含义

* 简单工厂模式又叫静态方法模式（因为工厂类定义了一个静态方法）
* 现实生活中，工厂是负责生产产品的；同样在设计模式中，简单工厂模式我们可以理解为负责生产对象的一个类，称为“工厂类”。

## 解决的问题

将“类实例化的操作”与“使用对象的操作”分开，让使用者不用知道具体参数就可以实例化出所需要的“产品”类，从而避免了在客户端代码中显式指定，实现了解耦。

## 步骤

1. 创建抽象产品类 & 定义具体产品的公共接口；
2. 创建具体产品类（继承抽象产品类） & 定义生产的具体产品；
3. 创建工厂类，通过创建静态方法根据传入不同参数从而创建不同具体产品类的实例；
4. 外界通过调用工厂类的静态方法，传入不同参数从而创建不同具体产品类的实例

## 示例

1. 创建抽象产品类

```java
abstract class Product{
    public abstract void Show();
}
```

1. 创建具体产品类

```java
//具体产品类A
class  ProductA extends  Product{

    @Override
    public void Show() {
        System.out.println("生产出了产品A");
    }
}

//具体产品类B
class  ProductB extends  Product{

    @Override
    public void Show() {
        System.out.println("生产出了产品C");
    }
}

//具体产品类C
class  ProductC extends  Product{

    @Override
    public void Show() {
        System.out.println("生产出了产品C");
    }
}
```

1. 创建工厂类，通过创建静态方法从而根据传入不同参数创建不同具体产品类的实例

```java
class  Factory {
    public static Product Manufacture(String ProductName){
//工厂类里用switch语句控制生产哪种商品；
//使用者只需要调用工厂类的静态方法就可以实现产品类的实例化。
        switch (ProductName){
            case "A":
                return new ProductA();

            case "B":
                return new ProductB();

            case "C":
                return new ProductC();

            default:
                return null;

        }
    }
}
```

1. 外界通过调用工厂类的静态方法，传入不同参数从而创建不同具体产品类的实例

```java
//工厂产品生产流程
public class SimpleFactoryPattern {
    public static void main(String[] args){
        Factory mFactory = new Factory();

        //客户要产品A
        try {
//调用工厂类的静态方法 & 传入不同参数从而创建产品实例
            mFactory.Manufacture("A").Show();
        }catch (NullPointerException e){
            System.out.println("没有这一类产品");
        }

        //客户要产品B
        try {
            mFactory.Manufacture("B").Show();
        }catch (NullPointerException e){
            System.out.println("没有这一类产品");
        }

        //客户要产品C
        try {
            mFactory.Manufacture("C").Show();
        }catch (NullPointerException e){
            System.out.println("没有这一类产品");
        }

        //客户要产品D
        try {
            mFactory.Manufacture("D").Show();
        }catch (NullPointerException e){
            System.out.println("没有这一类产品");
        }
    }
}
```
