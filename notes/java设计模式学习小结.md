## 设计模式
### 1、设计模式的分类
#### 
网上关于设计模式的介绍已近有好多高质量的资料，本篇主要是自己在学习过程中的一些小结

#### 1、设计模式原则：
>1、开闭原则（Open Close Principle）:可扩展，不可修改已有类和接口

>2、里氏代换原则（Liskov Substitution Principle）：父类出现的地方子类也可出现

>3、依赖倒转原则（Dependence Inversion Principle）：依赖于抽象的接口或者父类

>4、接口隔离原则（Interface Segregation Principle）：尽量使用多个单一的接口代替一个复杂接口，降低耦合

>5、迪米特法则（最少知道原则）（Demeter Principle）：一个实体类和别的类之间的交集尽量的少，提高内聚性

>6、合成复用原则（Composite Reuse Principle）：尽量使用组合，而不是继承


#### 2、设计模式介绍：
>1、创造型：5种
>>1、工厂模式(Factory)：不同实例类实现同一个接口，工厂类只是按照特定的规则去创建实例对象。一般分为三种，普通/多工厂方法模式，静态工厂方法模式，最常用的是 ==静态工厂== 模式，代码如下：

```
// interface：Eat
// class implement interface：Person  Pig、
// 缺点：当需要扩展时，需要修改该工厂类，违反了闭包原则
public class EatFactory {  
      
    public static Eat eatPerson(){  
        return new Person();  
    }  
      
    public static Eat eatPig(){  
        return new Pig();  
    }  
}  
```
>>2、抽象工厂模式(Abstract Factory)：解决工厂模式扩展时的缺点，
>>>1、一个工厂类对应一个实例类

>>>2、工厂类和实现类分别实现两个接口，相互解耦，扩展时，实现类和工厂类直接实现各自的接口就ok了。

>>>3、缺点：当需要创建的实例很多时，类的数量程倍数增加。

>>3、单例模式(Singleton)：单例模式在jvm中只保存一个实例对象的存在。单例模式看似简单，其实实现一个高性能，高可靠的单例还是很难的，这里主要说下单例模式的基本条件。
>>>1、空的私有构造方法，防止对象被实例化

>>>2、一个公有静态方法，由于获取单例对象

>>>3、私有静态为空的实例对象(可选)

>>>4、根据前3个条件去实现单例模式，不同的使用场景使用不同的单例实现方式，各个实现之间的主要不同点在于 ** 第二个 ** 条件的实现不同，也是由这个条件，单例模式分为好几种模式，所以在实现单例时，根据对==高性能==和==多线程安全==等实际情况的要求去实现。

```
// 下面是网上的一种实现
public class Singleton {  
    private Singleton() {  
    }  
  
    // 使用内部类构建单例对象
    private static class SingletonFactory {  
        private static Singleton instance = new Singleton();  
    }  
  
    public static Singleton getInstance() {  
        return SingletonFactory.instance;  
    }  
}  
```
>>4、创造者模式（Builder）：创造一个 复杂的的对象，一般是有好多个主要部分和次要部分组成的。
>>>1、将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

>>5、原型模式（Prototype）：利用object对象的clone方法对实例对象进行浅复制。

```
public class Proto implements Cloneable {  
  
    public Object clone() throws CloneNotSupportedException {  
        Proto proto = (Proto) super.clone();  
        return proto;  
    }  
}  
```
>>1、浅复制：对象中包含的基本类型进行复制，引用类型还是使用原来的引用对象，object对象中的clone方法是浅复制
>>2、深复制：复制对象和原始对象是两个完全独立的对象，需要子类中实现深复制方法，一般是对引用对象的递归实现，或者是直接使用串行化进行深度复制（利用java原始的串行化性能较低）
```
public Object deepClone() throws IOException,
  OptionalDataException,ClassNotFoundException{
    //将对象写到流里
    ByteArrayOutoutStream bo=new ByteArrayOutputStream();
    ObjectOutputStream oo=new ObjectOutputStream(bo);
    oo.writeObject(this);
    //从流里读出来
    ByteArrayInputStream bi=new ByteArrayInputStream(bo.toByteArray());
    ObjectInputStream oi=new ObjectInputStream(bi);
    return(oi.readObject());
 }

```

>2、结构型：7种

>3、行为型：11种

>4、并发型模式和线程池模式