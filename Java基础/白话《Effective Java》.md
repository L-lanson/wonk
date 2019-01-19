+ [一、引言](#chapter1)
+ [二、创建和销毁对象](#chapter2)
   - [考虑用静态工厂方法代替构造器](#rule1)
   - [遇到多个构造器参数时要考虑用构建器（Builder）](#rule2)
   - [用私有构造器或枚举类型强化Singleton属性]()
   - [通过私有构造器强化不可实例化的能力]()
   - [避免创建不必要的对象]()
   - [消除过期对象的引用]()
   - [避免使用中介方法]()
+ [三、所有对象都通用的方法]()
   - [覆盖equals请遵守通用约定]()
   - [覆盖equals时总要覆盖hashCode]()
   - [始终要覆盖toString]()
   - [谨慎地覆盖clone]()
   - [考虑实现Comparable接口]()
   - []()
   - []()
   - []()
+ [四、类和接口]()
   - [使类和成员的可访问性最小化]()
   - [在公有类中使用访问方法而非公有域]()
   - [使可变性最小化]()
   - [复合优先于继承]()
   - [要么为继承而设计，并提供文档说明，要么禁止继承]()
   - [接口优先于抽象类]()
   - [接口只用于定义类型]()
   - [类层次优先于标签类]()
   - [用函数对象表示策略]()
   - [优先考虑静态成员类]()
+ [五、泛型]()
+ [六、枚举和注解]()
+ [七、方法]()
+ [八、通用程序设计]()
+ [九、异常]()
+ [十、并发]()
+ [十一、序列化]()

## <span id="chapter1">一、引言</span>
Java很多人都会用，但是如何用精、写出在大牛眼中无可挑剔的代码却难倒了很多人。《Effective Java》这本书从10个角度描述了78条Java开发规则，遵守这些规则能写出更健壮的代码。

## <span id="chapter2">二、创建和销毁对象</span>
>讲述了从创建对象到销毁对象应该遵循的规则

### 第1条： <span id="rule1">考虑用静态工厂方法代替构造器</span>
静态工厂方法：一个返回类的实例的静态方法
```Java
public class Person{
  private String name;
  private String gender;

  private Person(String name){
    this.name = name;
  }

  private Person(String name, String gender){
    this(name);
    this.gender = gemder;
  }

  //静态工厂方法
  public static Person createPerson(String name){
    return new Person(name);
  }

  //静态工厂方法
  public static Person createPersonWithGender(String name, String gender){
    return new Person(name, gender);
  }
}
```

##### 使用静态工厂方法的优点
1. 我们可以根据静态工厂方法的作用给它们起合适的名字
>构造器有个缺点：每个构造器都必须与类名同名，如果类中有多个构造器，除非在源码中作了注释，否则不知道这个构造器有什么作用，在什么情况下使用这个构造器。   
静态工厂方法很好地弥补了这个缺陷，我们可以根据静态工厂方法的作用给它起合适的名字，使用者可以根据名字来调用他们需要的方法构造对象。

2. 不必在每次调用静态工厂方法的时候都创建新对象
>调用构造器时，肯定会有一个新对象被创建出来。   
而使用静态工厂方法，我们可以返回一个被缓存的对象。该对象可能是在类加载时被创建出来的，也有可能是在第一次调用静态工厂方法时被创建出来的，总之创建后就被缓存起来，后续通过静态工厂方法从缓存中取对象。

3. 静态工厂方法可以返回原返回类型的任意子类型对象
>调用构造方法只能创建特定类型的对象，如`new Person()`，就只能创建`Person`对象。   
根据Java的多态特性，子类型可以用父类型去接收。比如有两个类：`Animal`和`Dog`，`Dog`是`Animal`的子类，我们可以这样接收`Dog`的实例：`Animal animal = new Dog()`。   
同样，静态工厂方法上的返回类型可以是实际返回类型的父类或者接口。这样做对日后的扩展或修改有很大的好处。在静态工厂方法里面修改实际返回类型，对调用者是透明的，无需调用者做任何改动。还是拿`Animal`和`Dog`举例：

```Java
//迭代一
public static Animal createAnimal(){
  return new Dog();
}

--- 产品经理萌改变主意了，他要养猫而不是养狗

//迭代二，createAnimal()方法的变化对调用者而言是透明的
public class Cat extends Animal{...}

public static Animal createAnimal(){
  return new Cat();
}
```

4. JDK1.7之前，在创建参数化类型的时候，使用静态工厂方法可以使代码变得更简洁

```Java
//直接new创建对象，JDK1.7之前
Map<String, Person> persons = new HashMap<String, Person>();

//使用静态工厂方法
public static <K, V> HashMap<K, V> newInstance(){
  return new HashMap<K, V>();
}
Map<String, Person> persons = HashMap.newInstance();

//JDK1.7之后
Map<String, Person> persons = new HashMap<>();
```

##### 静态工厂方法的缺点
1. 如果类中不含public或者protected的构造器，就不能被子类化。
>这样子可以因祸得福，它鼓励程序员使用复合来扩展类而不是继承。

2. 静态工厂方法与其他静态方法实际上没有任何区别
>除非有很好的文档说明或者名字一目了然（如：newInstance、createXxx），否则我们都不知道这是个静态工厂方法。


### 第2条： 遇到多个构造器参数时考虑使用构建器（Builder）
当构造对象时有多个可选参数，构造器和和静态工厂方法都有很大的局限。要么考虑每种情形，根据情形提供不同的方法，这样做始终赶不上变化，灵活度不够。要么在方法声明时提供所有参数，让调用者自由选择，但这样做会产生大量冗余参数。   
举个栗子：   
汉堡包（Hamburger）由很多原料组成，加鸡肉（Chicken）可以做出鸡肉汉堡，加牛肉（Beef）可以做出牛肉汉堡，两样都加可以做出鸡肉+牛肉汉堡。以下提供三种方式实现这个汉堡。

1. 重叠构造器模式
>这种方式加入了很多冗余参数，不仅造成阅读困难，还有可能因为调用者传错参数而造成项目出错

```Java
//层叠构造器的方式
//有很多冗余参数，而且代码量大
public class Hamburger{
  private Chicken chicken;
  private Beef beef;

  public Hamburger(Chicken chicken){
    this(chicken, null);
  }

  public Hamburger(Beef beef){
    this(null, beef);
  }

  public Hamburger(Chicken chicken, Beef beef){
    this.chicken = chicken;
    this.beef = beef;
  }
}

//鸡肉汉堡
Hamburger chickenHamburger = new Hamburger(chicken);
//鸡肉+牛肉汉堡
Hamburger chickenBeefHamburger = new Hamburger(chicken, beef);
```

2. JavaBeans模式
>对象创建与设值分开，容易造成对象状态不一致，因为对象在设值的时候有可能被其他对象修改   
JavaBeans模式阻止了把类做成不可变的状态，天生不是线程安全的，因为它提供了setter方法供外界修改其状态

```Java
public class Hamburger{
  private Chicken chicken;
  private Beef beef;

  public Hamburger(){}

  public void setChicken(Chicken chicken){
    this.chicken = Chicken;
  }

  public void setBeef(Beef beef){
    this.beef = beef;
  }
}

//鸡肉+牛肉汉堡
Hamburger hamburger = new Hamburger()
hamburger.setChicken(chicken);
hamburger.setBeef(beef);
```

3. 构建器（Builder）模式
>相比JavaBeans模式，Builder模式可以在在构造出对象之前设置属性，因此它可以保证对象状态的一致性   
相比重叠构造器模式，Builder模式可以有多个可变参数，调用者可以根据需要自由选择，不会产生参数冗余   
但是Builder模式也有缺点，它多创建了Builder对象，性能会稍微有点降低。另外它的写法也比较冗长，因此多个参数（4个以上）时才适合使用   
要使用Builder模式，最好一开始就使用，而不是等参数增多时才改成Builder模式，因为这种修改可能会造成调用者的旧API出现问题

```Java
//构建器接口
public interface Builder<T>{
  public T build();
}

//汉堡包类
public class Hamburger{
  private Chicken chicken;
  private Beef beef;

  private Hamburger(Builder<? extends Hamburger> builder){
    this.chicken = builder.chicken;
    this.beef = builder.beef;
  }

  //内部静态类作为构建器
  public static Builder implements Builder<Hamburger>{
    private Chicken chicken;
    private Beef beef;

    public Builder chicken(Chicken chicken){
      this.chicken = chicken;
      return this;
    }

    public Builder beef(Beef beef){
      this.beef = beef;
      return this;
    }

    public Hamburger build(){
      return new Hamburger(this);
    }
  }
}

//鸡肉+牛肉汉堡
Hamburger hamburger = new Hamburger.Builder()
                                   .chicken(chicken)
                                   .beef(beef)
                                   .build();
```
