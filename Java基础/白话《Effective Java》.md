+ [一、引言](#chapter1)
+ [二、创建和销毁对象](#chapter2)
   - [考虑用静态工厂方法代替构造器](#rule1)
   - [遇到多个构造器参数时要考虑用构建器（Builder）](#rule2)
   - [用私有构造器或枚举类型强化Singleton属性](#rule3)
   - [通过私有构造器强化不可实例化的能力](#rule4)
   - [避免创建不必要的对象](#rule5)
   - [消除过期对象的引用](#rule6)
   - [避免使用终结方法](#rule7)
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


### <span id="rule2">第2条： 遇到多个构造器参数时考虑使用构建器（Builder）</span>
当构造对象时有多个可选参数，构造器和和静态工厂方法都有很大的局限。要么考虑每种情形，根据情形提供不同的方法，这样做始终赶不上变化，灵活度不够。要么在方法声明时提供所有参数，让调用者自由选择，但这样做会产生大量冗余参数。   
举个栗子：   
汉堡包（Hamburger）由很多原料组成，加鸡肉（Chicken）可以做出鸡肉汉堡，加牛肉（Beef）可以做出牛肉汉堡，两样都加可以做出鸡肉+牛肉汉堡。以下提供三种方式实现这个汉堡。

1. 重叠构造器模式
>这种方式加入了很多冗余参数，不仅造成阅读困难，还有可能因为调用者传错参数而造成项目出错。

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
>对象创建与设值分开，容易造成对象状态不一致，因为对象在设值的时候有可能被其他对象修改。   
JavaBeans模式阻止了把类做成不可变的状态，天生不是线程安全的，因为它提供了setter方法供外界修改其状态。

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
>相比JavaBeans模式，Builder模式可以在在构造出对象之前设置属性，因此它可以保证对象状态的一致性。   
相比重叠构造器模式，Builder模式可以有多个可变参数，调用者可以根据需要自由选择，不会产生参数冗余。   
但是Builder模式也有缺点，它多创建了Builder对象，性能会稍微有点降低。另外它的写法也比较冗长，因此多个参数（4个以上）时才适合使用。   
要使用Builder模式，最好一开始就使用，而不是等参数增多时才改成Builder模式，因为这种修改可能会造成调用者的旧API出现问题。

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

### <span id="rule3">第3条： 用私有构造器或者枚举类型增强Singleton属性</span>
JDK1.5之前，实现Singleton有两种方式：一种是通过公有的静态常量向外界提供本对象的实例，另一种通过公有的静态工厂方法向外界提供本对象的实例。这两种方式都要求构造方法私有。   
但是这样依然有可能通过反射来破坏单例属性，因此需要在构造方法中加非法判断，在存在多个实例时做相应处理。   
还有一种破坏单例属性的方法是序列化和反序列化，为了保证Singleton，必须声明所有实例域都是transient（瞬时的），并提供readResolve方法。反序列化时会调用readResolve方法，并返回里面的对象，不用创建新对象了。   
JDK1.5之后还可以通过enum实现Singleton。   

1. 静态常量
>实现单一，只能通过赋值来实现，如果要改变单例的行为（比如为一个线程返回单一实例等），可能需要调用者修改代码

```Java
//单例类
public class Singleton{
  public static final Singleton INSTANCE = new Singleton();

  private Singleton(){}

  public Object readResolve(){
    return INSTANCE;
  }
}

//获取单例对象
Singleton s = Singleton.INSTANCE;
```

2. 静态工厂方法
>优点：   
1、JVM会将静态工厂方法的调用内联化。一般的方法调用需要入栈出栈，而内联化就是将静态工厂方法的内容移到调用方法中，省去了频繁出入栈的开销。   
2、在不改变API的前提下，可以改变该单例类的行为，弥补静态常量的缺点。

```Java
public class Singleton{
  private static final Singleton INSTANCE = new Singleton();

  private Singleton(){}

  public Object readResolve(){
    return INSTANCE;
  }

  public static Singleton newInstance(){
    return INSTANCE;
  }
}

//获取单例对象
Singleton s = Singleton.newInstance();
```

3. 枚举
>单元素的enum类型已经成为实现Singleton的最佳方法，无论是反射还是反序列化，都无法改变其单例属性。   
首先，enum类型天生就无法被实例化，因此反射无法破坏其单例属性。   
再者，对枚举来说，反序列化得到的枚举对象`==`被序列化的枚举对象，因此反序列化时无需额外的操作来保障其单例属性。

```Java
public enum Singleton{
  INSTANCE;

  public void xxx(){
    ......
  }
}

//获取单例对象
Singleton s = Singleton.INSTANCE;
```


### <span id="rule4">第4条： 通过私有构造器强化不可实例化的能力</span>
强化类不可实例化的能力通常有两种做法：    
1. 将类做成抽象类
>这是行不通的，抽象类可以被子类化，子类可以被实例化。   
这种类是专门为了继承而设计的，父子类之间不是`is-a`关系，可能会带来很多问题。

2. 私有构造器
>如果类不包含显式构造器，编译器会生成默认的构造器，因此不提供显式构造器的类也可以被实例化。为了强化不可实例化的能力，我们需要显式地私有化它的构造函数，并且在构造函数中抛异常。

```Java
//工具类
public class Util{
  private Util(){
    throw new AssertionError();
  }

  //一系列静态工具方法
  ...
}

```

### <span id="rule5">第5条： 避免创建不必要的对象</span>
1. 如果对象是不可变的，它始终可以被重用
>对于String，JVM提供了重用机制。如果常量池中不存在该字符串，就会创建一个字符串对象并存入常量池中；否则直接从常量池中返回该字符串对象。

```Java
//始终创建新对象，不建议使用
String s = new String("abc");
//只有在常量池不存在该对象时才创建新对象
String s = "abc";
```

2. 除了重用不可变对象之外，还可以重用不会被修改的可变对象   
   * 不会被修改的可变对象（常量对象）

```Java
//小明 v1.0
//他的生日不会改变
//每次调用isBirthday都创建一个Date实例，开销大
public class XiaoMing{
  private Date birthday;

  //判断某个日期是否是1小明的生日
  public boolean isBirthday(Date date){
    birthday = new Date(1996, 05, 16);
    return birthday.equals(date);
  }
}

//小明 v2.0
//他的生日不会改变
//只需在类加载时创建一个Date实例，大大地节省了开销
public class XiaoMing{
  private static final Date BIRTHDAY = new Date(1996, 05, 16);

  //判断某个日期是否是1小明的生日
  public boolean isBirthday(Date date){
    return BIRTHDAY.equals(date);
  }
}

//小明 v3.0
//他的生日不会改变
//延迟初始化，只需在第一次调用isBirthday时创建一个Date实例，大大地节省了开销
public class XiaoMing{
  private static final Date BIRTHDAY;

  //判断某个日期是否是1小明的生日
  public boolean isBirthday(Date date){
    if (BIRTHDAY == null) {
      BIRTHDAY = new Date(1996, 05, 16);
    }
    return BIRTHDAY.equals(date);
  }
}
```

  * 考虑适配器的情形
   >适配器是指这样一个对象：它把功能委托给一个后备对象，从而为后备对象提供一个可替代的接口。适配器有时候也叫视图，例如：Map接口的entrySet方法返回该Map对象的Set视图

```Java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
  transient Node<K,V>[] table;
  transient Set<K> keySet;

  //由于视图直接使用外部类的table变量，
  //因此当table发生变化时这个视图也能感知
  //所以只需要一个keySet对象即可，无需每次调用都新建对象
  public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
  }

  final class KeySet extends AbstractSet<K> {
    public final void forEach(Consumer<? super K> action) {
      //遍历table数组及Entry链表
    }
  }
}
```

3. 使用基本类型而不是装箱类型，要当心无意识的自动装箱
>自动装箱会新建对象，带来性能上的开销

```Java
//不建议
Long l = 1;
//建议
long l = 1;
```

**前面说到避免创建对象，很容易让人联想到对象池。但是对象池的维护成本大，而且占用更多的内存，因此数据库连接、线程等重量级对象才适合使用对象池，一般轻量级对象不适合采用对象池维护**

### <span id="rule6">第6条： 消除过期对象的引用</span>
不清除过期对象的引用，有可能会造成内存泄露。内存泄漏包含以下几种情形：   
1. 维护过期引用
>过期引用：永远也不会再被解除的引用
```Java
//栈类 v1.0
//因为elements数组会一直保持对象的引用，所以会造成内存泄漏，需要手动对过期引用置为null
public class Stack{
  private Object[] elements;
  private int size;

  public Object pop(){
    return elements[--size];
  }
}

//栈类 v2.0
//pop时解除引用，不会造成内存泄漏
public class Stack{
  private Object[] elements;
  private int size;

  public Object pop(){
    Object result = elements[--size];
    elements[--size] = null;
    return result;
  }
}
```

2. 使用缓存
>一旦把对象放到缓存中，就很容易被遗忘掉，从而使得它很长时间没有被使用并存留在缓存中。   
如果缓存中的对象只要没有被外部引用就过期，那么可以使用WeakHashMap替代缓存。   
一般情况下，缓存中的对象随着时间推移，就会越来越没有价值，因此可以开启后台线程定时清理。

3. 监听器和其他回调
>如果客户端在API中注册回调，但是释放对象时没有显式地取消，这些回调会积聚起来，造成内存泄漏。解决办法是只保存这些回调的弱引用，比如将它们保存成WeakHashMap中的键。

### <span id="rule7">第7条： 避免使用终结方法</span>
如果需要在对象终结时回收资源，需要提供一个显式的终结方法（如：InputStream的close()），而不是在finalize方法中回收资源。
#### finalize方法的缺点：   
1. 终结方法不确保会被及时执行，甚至不确保会被执行，虽然System.runFinalizersOnExit()与Runtime.runFinalizersOnExit()能确保终结方法被执行，但是这两个方法有致命缺陷，已经被废弃。

2. 终结方法会屏蔽未被捕获的异常。

3. 终结方法有严重的性能损失，覆盖了finalize方法的对象在销毁时会耗费更多的时间。

#### finalize方法的合法用途
1. 当调用者忘了调用显式的终结方法回收资源时，finalize可以充当“安全网”。

2. 回收本地对等体的资源
>本地对等体是一个native对象，普通对象的native方法需要委托给native对象执行，因为native对象不是普通对象，所以它是不会被垃圾回收器知道的。而finalize方法是native方法，它可以回收native对象拥有的关键资源。

#### 正确使用finalize方法
1. 终结方法链不会自动执行
>如果子类覆盖了finalize方法，那么必须在子类的finalize中手动地调用父类的finalize方法。   
为了确保父类的finalize被执行，需要在finally中调用父类的finalize方法。
```Java
@Override
protected void finalize() throws Throwable{
  try{
    ...
  }catch(Exception e){
    ...
  }finally{
    super.finalize();
  }
}
```

2. 为了弥补子类忘了调用父类finalize带来的缺陷，可以设置终结方法守卫者
>终结方法守卫者其实是个匿名类的实例，该实例被外部类对象的成员变量引用，继承finalize方法来释放外部类对象中的资源。当外部类对象被回收时，它的成员变量指向的对象也会被回收，因此终结方法守卫者的finalize方法可以被执行。

```Java
public class Parent{
  private Object o = new Object(){
    @Override
    protected void finalize() throws Throwable{
    //释放外部类资源
    ...
  }
}

```
