---
title: Java常用的设计模式
date: 2020-04-01 10:10:02
tags:
- Java
- 设计模式
categories:
- 面试笔试
description: 🛴最近软件工程课程上到了这个内容，正好总结一下，对面试笔试有点帮助的
top_img: https://file.buildworld.cn/img/4gj334.jpg
cover: https://file.buildworld.cn/img/20200401101919.png

---

### 一、变种Builder模式（构造者模式）

#### 1、构造者模式包含如下角色

- Builder：抽象建造者
- ConcreteBuilder：具体建造者
- Director：指挥者
- Product：产品角色

#### 2、对Builer模式使用方法的总结：

> 对于习惯使用get、set方法的人来说，构造者模式多此一举，但是用起来是真的香啊。 🤣 看看代码多么优雅呢！

- （1）、外部类的构造函数私有，且参数为静态内部类；
- （2）、静态内部类拥有外部类相同的属；
- （3）、为每一个属性，写一个方法，返回的是Builer；
- （4）、最后一个方法是build方法，用于构建一个外部类；

```java
// 详情可以看《effective java》这本书

public class Person {

    //必要参数
    private final int id;
    private final String name;

    //可选参数
    private int age;
    private String sex;
    private String phone;
    private String address;
    private String desc;

    public Person(Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.age = builder.age;
        this.sex = builder.sex;
        this.phone = builder.phone;
        this.address = builder.address;
        this.desc = builder.desc;
    }

    // 1、在要构建类的内部，创建一个静态内部类Builder；
    public static class Builder{
    
    // 2、静态内部类的属性要与构建类的属性一致；
    
        //必要参数
        private final int id;
        private final String name;

        //可选参数
        private int age;
        private String sex;
        private String phone;
        private String address;
        private String desc;

    // 3、构建类的构造参数是静态内部类，使用静态内部类的变量为构建类逐一赋值；
    
        //初始构造器，强制设置必要参数
        public Builder(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public Builder sex(String sex) {
            this.sex = sex;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public Builder desc(String desc) {
            this.desc = desc;
            return this;
        }

    // 4、静态内部类提供参数的setter方法，并且返回值是当前Builder对象；
    
        // Person调用builder方法，返回Builder对象
        public Person builder(){
            return new Person(this);
        }

    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", phone='" + phone + '\'' +
                ", address='" + address + '\'' +
                ", desc='" + desc + '\'' +
                '}';
    }

    //测试
    public static void main(String[] args) {
        Person person = new Person.Builder(1,"michong").sex("男").builder();
        System.out.println(person.toString());
    }

}
```
### 二、单例模式
>当需要控制一个类的实例只能有一个，而且客户只能从一个全局访问点访问它时，可以选用单例模式，这些功能恰好是单例模式要解决的问题。

#### 五种实现方式
##### 1、懒汉式
>先判断，再生成对象
```java
/**
 * 懒汉式单例实现方式
 * 
 * @author Administrator
 *
 */
public class Singleton {
    /**
     * 定义一个变量来存储创建好的实例
     */
    private static Singleton instance = null;

    /**
     * 私有化构造方法，可以在内部控制创建实例的数目
     */
    private Singleton() {
    }

    /**
     * 定义一个方法为客户端提供类实例
     * 
     * @return
     */
    public static synchronized Singleton getInstance() {
        // 判断存储实例的变量是否有值
        if (instance == null) {
            // 如果没有，就创建一个类实例，并把值赋给存储类实例的变量
            instance = new Singleton();
        }

        return instance;
    }

}
```
##### 2、饿汉式
> 二话不说，直接生成对象

```java


/**
 * 饿汉式单例
 * 
 * @author Administrator
 *
 */
public class Singleton2 {

    /**
     * 定义一个变量来存储创建好的类实例，直接在这里创建类实例，只能创建一次
     */
    private static Singleton2 instance = new Singleton2();

    /**
     * 私有化构造方法，可以在内部控制创建实例的数目
     */
    private Singleton2() {
    }

    public static Singleton2 getInstance() {
        return instance;
    }

}

```
##### 3、双重加锁

```java
public class Singleton3 {

    /**
     * 对保存实例的变量添加volatile的修饰
     */
    private volatile static Singleton3 instance = null;

    private Singleton3() {
    }

    public static Singleton3 getInstance() {
        // 先检查实例是否存在，如果不存在才金如意下面的模块
        if (instance == null) {
            // 同步块，线程安全得创建实例
            synchronized (Singleton3.class) {
                // 再次检查实例是否存在，如果不存在才真正地创建实例
                if (instance == null) {
                    instance = new Singleton3();
                }
            }
        }
        return instance;
    }
}
```
##### 4、静态内部类

```java
/**
 * 使用静态内部类实现单例模式
 * 
 * @author Administrator
 *
 */
public class Singleton4 {

    /**
     * 没有绑定关系，而且只有被调用到时才会装载，从而实现延迟加载
     * 
     * @author Administrator
     *
     */
    private static class SingletonHolder {
        /**
         * 静态初始化器，由JVM来保证线程安全
         */
        private static Singleton4 instance = new Singleton4();
    }

    private Singleton4() {
    }

    public static Singleton4 getInstance() {
        return SingletonHolder.instance;
    }
}
```
##### 5、枚举

```java
/**
 * 使用枚举来实现单例
 * 
 * @author Administrator
 *
 */
public enum Singleton5 {

    // 定义 一个枚举的元素，它就代表了Singleton的实例
    instance;

    public static void main(String[] args) {
        System.err.println(Singleton5.instance.hashCode());
        System.err.println(Singleton5.instance.hashCode());
    }
}
```
### 三、简单工厂模式
#### 1、定义
> 提供一个`创建对象实例`的功能，而无须关心其具体实现。被创建的类型可以是`接口`、`抽象类`，也可以是`具体类`

#### 2、选择的时机
- 完全封装隔离具体实现，让外部只能通过接口来操作封装体
- 想要把对创建对象的职责集中管理和控制

#### 3、示例代码
##### Api

```java
/**
 * 接口的定义，该接口可以 通过简单工厂创建
 *
 */
public interface Api {

    /**
     * 具体功能方法的 定义
     * 
     * @param s 需要的参数
     */
    public void operation(String s);
}
```
##### ImplA

```java
/**
 * 接口具体实现A
 * 
 *
 */
public class ImplA implements Api {

    @Override
    public void operation(String s) {
        // 实现功能的代码
        System.out.println("ImplA s=" + s);
    }
}
```
##### ImplB

```java
/**
 * 接口具体实现B
 * 
 * @author Administrator
 *
 */
public class ImplB implements Api {

    @Override
    public void operation(String s) {
        // 实现功能的代码
        System.out.println("ImplA s=" + s);
    }
}
```
##### Factoty

```java
/**
 * 工厂类，用来创建Api对象
 * 
 * @author Administrator
 *
 */
public class Factory {

    private Factory() {
    }

    public static Factory getInstance() {
        return FactoryHolder.factory;
    }

    /**
     * 具体创建Api对象的方法
     * 
     * @param condition
     *            从外部传入的参数条件
     * @return 创建好的APi对象
     */
    public Api createApi(int condition) {
        Api api = null;
        if (condition == 1) {
            api = new ImplA();
        } else if (condition == 2) {
            api = new ImplB();
        }
        return api;
    }

    static class FactoryHolder {
        public static final Factory factory = new Factory();
    }
}
```
##### Client

```java
/** 
 * 客户端，使用Api接口
 * 
 * @author Administrator
 *
 */
public class Client {

    public static void main(String[] args) {
        // 通过简单工厂来获取接口对象
        Api api = Factory.getInstance().createApi(1);
        api.operation("正在使用简单工厂~");
    }
}
```

### 四、工厂方法模式

#### 1、定义
> 定义一个用于创建对象的借口，让子类决定实例化哪一个类，FactoryMethod使一个类的实例化延迟到子类。


#### 2、结构
`Product`:定义工厂方法所创建的对象的接口，也就是实际需要使用的对象的接口。

`ConcreteProduct`:具体的Product接口的实现对象。

`Creator`:创建器，声明工厂方法，工厂方法通常会返回一个Product类的实例对象，而且多是抽象方法

`ConcreteCreator`:具体的创建器对象，覆盖实现Creator定义的工厂方法，返回具体的Product实例

#### 3、示例代码
##### Product

```java
/**
 * 工厂方法所创建的对象的接口
 * 
 * @author Administrator
 *
 */
public interface Product {

}
```

##### ConcreteProduct

```java
/**
 * 具体的product对象
 * 
 * @author Administrator
 *
 */
public class ConcreteProduct implements Product {

}
```
##### Creator

```java
/**
 * 创建器，声明工厂方法
 * 
 * @author Administrator
 *
 */
public abstract class Creator {

    /**
     * 创建Product的工厂方法
     * 
     * @return 对象
     */
    protected abstract Product factoryMethod();

    /**
     * 实现某些功能的方法
     */
    public void someOperation() {

        // 通常在这些方法实现需要调用工厂方法来获取Product对象
        Product product = factoryMethod();
    }
}
```
##### ConcreteCreator
```java
public class ConcreteCreator extends Creator {

    @Override
    protected Product factoryMethod() {
        return new ConcreteProduct();
    }

}
```
### 五、抽象工厂模式（Abstract Factory Pattern）
#### 1、定义
>提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，属于对象创建型模式。

#### 2、模式结构
- `AbstractFactory`：抽象工厂
- `ConcreteFactory`：具体工厂
- `AbstractProduct`：抽象产品
- `Product`：具体产品
![image](https://file.buildworld.cn/img/20200331231328.png)



### 六、其他
[更多的设计模式请看菜鸟教程](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)