# Scala中的类、对象、继承以及特质
## 类
-  类的使用
```
class Calculator{
    val brand: String = "HP"
    def add(m: Int, n: Int):
        Int = m + n
}
val calc = new Calculator
cal.add(1,2)//Int = 3
calc.brand//String = "HP"
```
>这里方法和字段都默认是公有

>调用无参方法时，可以写上圆括号，也可以不写（一般对于改值器方法写()，取值器方法不写()，如果声明中不写()，则使用时不能有()）

- [getter和setter的意义](http://www.zhihu.com/question/21401198)

这两个方法可以方便增加额外功能（比如验证）
1. 内部存储和外部表现不同。
1. 可以保持外部接口不变的情况下，修改内部存储方式和逻辑。
1. 任意管理变量的生命周期和内存存储方式。
1. 提供一个debug接口。
1. 能够和模拟对象、序列化乃至WPF库等融合。
1. 允许继承者改变语义。
1. 可以将getter、setter用于lambda表达式。（大概即作为一个函数，参与函数传递和运算）
1. getter和setter可以有不同的访问级别。


Scala会自动为字段提供getter和setter方法，定义一个公有字段
```
class Person{
    var age = 0
}
```
这里面书上写错了吧。。。。
应该是其中age为公有字段，其中在jvm中也生成了getter和setter方法，这两个方法由于没有将age声明为private，因此也是公有的。同理，对于私有字段，其中的getter和setter方法也是私有的。

>getter和setter方法在使用和定义的时候都分别使用形式age和age_(这里书上也印错了。。。)

[统一访问原则：](http://blog.csdn.net/shijiebei2009/article/details/38666201)某个模块提供的所有服务都应该能通过统一的表示法访问到，至于它们是通过存储还是通过计算来实现的，从访问方式上应无所获知。因此Scala对于每个字段都默认定义getter和setter方法。

>三个原则：

>如果字段是私有的，则getter和setter方法也是私有的

>如果字段是val，则只有getter方法被生成

>如果不需要任何getter或setter，可以将字段声明为private[this]

- 对象私有字段 

在Scala中，方法可以访问类的所有对象的私有字段。下面程序只需保证other同样为Counter类的对象即合法
```
class Counter{
    private var value = 0
    def increment(){value += 1}
    def isLess(other: Counter) = value < other.value
    //可以访问另一个对象的私有字段
}
```
>对定义添加更加严格的访问限制，可以通过private[this]实现

>将字段或方法声明为protected，则该成员可以被子类访问，但是不能从其他位置看到

protect[this]
```
private[this] var value = 0
```
这样Counter类的方法只能访问到当前对象的value字段，而不能访问同样是Counter类型的其他对象字段，称为对象私有字段，对于对象私有字段，Scala根本不会生成getter和setter方法。

==public，private和protect的区别==

- Bean属性
加@BeanProperty自动生成四个方法
```
name: String
name_=(new Value: String): Unit
getName():String
setName(newValue: String):Unit 
```
---

1.主构造器

在Scala中，每个类都有主构造器，主构造器不以this方法定义， 而是与类定义交织在一起
主构造器的参数直接放在类名之后
```
class Person(val name: String, val age: Int){
//
}
```
==主构造器会执行类定义中的所有语句==
如下例
```
class Person(val name: String, val age: Int){
    println("Just constructed another person")
    def description = name + "is" + age + " years old"
}
```
其中println是主构造器中的一部分，每当有对象被构造出来的时候，上述代码就会被执行一次，同理
```
class MyProg{
    private var props = new Properties
    props.load(new FileReader("myprog.properties"))
}
```
上面代码中类名没有参数，该类具有一个无参主构造器。这样的构造器的目的仅仅是简单执行类中的所有语句而已。

通常可以在主构造器中使用默认参数类避免过多使用辅助构造器

*不带val或val的参数与带的区别
```
class Person(val name: String, private var age:Int)
```
这段代码将声明并初始化如下字段
```
val name: String
private var age: Int
```
如果不带val或var的参数至少被一个方法所使用，它将被升格为字段（对象私有的）；否则，该参数将不被保存为字段，它仅仅是一个可以被主构造器中的代码访问的普通参数。

==主构造器一般就是把类定义和构造器定义放在一起==


2.辅助构造器

类可以有任意多的辅助构造器，与java/c++两点区别
- 辅助构造器名称为this。
- 每一个辅助构造器都必须以一个对先前已定义的其他辅助构造器或主构造器的调用开始
```
class Person{
    private var name = ""
    private var age = 0
    def this(name: String){
        this()//调用主构造器！
        this.name = name
    }
    def this(name: String, age: Int){
        this(name)//调用前一个辅助构造器
        this.age = age         
    }
}
```
使用
```
val p1 = new Person//主构造器
val p2 = new Person("Fred")//第一个辅助构造器
val p3 = new Person("Fred", 42)//第二个辅助构造器
```


* 补：区别函数和方法


[函数和方法的区别](http://www.bubuko.com/infodetail-346478.html)

理解：函数是对象，方法就是对象里面的方法。。。



- 嵌套类 
示例
```
import scala.collection.mutable.ArrayBuffer
class Network{
    class Member(val name: String){
        val contacts = new ArrayBuffer[Member]
    }
    private val members = new ArrayBuffer[Member]
    
    def join(name: String)={
        val m= new Member(name)
        members += m
        m
    }
}
```
定义两个实例
```
val chatter = new Network
val myFace = new Network
```
现在每个实例都有自己的Member类，就和它们有自己的字段一样，chatter.Member和myFace.Member是不同的两个类
要构建一个新的内部对象，只需要new chatter.Member


## 对象
- 单例对象

应用于想使用类的静态字段和静态方法时
```
object Accounts｛
    private var lastNumber = 0
    def newUniqueNumber() = {
        lastNumber += 1
        lastNumber
    }
｝
```
对象的构造器在该对象第一次被使用时调用（这个叫构造器？？）。在本例中，Accounts的构造器Accounts.newUniqueNumber()在首次调用时执行。

==对象本质上可以拥有类的所有特性，但是不能提供构造器参数==
- 伴生对象
```
class Account{
    val id = Account.newUniqueNumber()
    private var balance = 0.0
    def deposit(amount: Double){balance += amount}
    ...
}
object Account{//伴生对象
    private var lastNumber = 0
    private def newUniqueNumber() = {lastNumber += 1; lastNumber}
}
```
类和它的伴生对象必须存在于一个源文件中，且可以相互访问私有特性（我的理解就是相当一个java里面成员都是静态的类，是否实例化都行的这种）

- apply方法

当遇到如下形式时，apply方法就会被调用。

>Object(参数1，参数2，...,参数N)

通常，这样一个apply方法返回的是伴生类的对象

==使用对象的apply方法与类的构造器区别在于使用方式不同，类的构造器赋值需要用new==
>Array(100)调用的是apply(100)，输出一个单元素的Array[Int];第二个表达式是构造器this(100)，结果是Array[Nothing]，包含了100个null元素

- 枚举

Scala没有枚举类型。不过标准类库提供了一个助手类用于产生枚举
```
object TrafficLightColor extends Enumeration{
    val Red, Yellow, Green = Value
}
```
类的对象并以Value方法调用初始化枚举中的所有可选值。随后可以通过import TrafficLightColor调用这些枚举值


## 继承
在子类定义中给出子类需要而父类没有的字段和方法，或者重写父类的方法。（将类声明为final，则它不能被扩展，将方法或字段声明为final，则不能被重写，但是是变量不是常量）
* 重写

在Scala中重写一个非抽象方法必须使用override，重写抽象方法时则不需要使用override

Scala中调用父类的方法
```
public class Employee extends Person{
    ...
    override def toString = super.toString + "[salary" + salary + "]"
}
```
其中super.toString会调用父类的toString方法，即Person.toString
* 类型检查和转换

测试某个对象是否属于某个给定的类
p.isInstanceOf(Employee)
val s = p.asInstanceOf[Employee]//将对象p强制转换为Employee类型

* 超类的构造

类有一个主构造器和任意数量的辅助构造器，而每个辅助构造器都必须以对先前定义的辅助构造器或主构造器的调用开始。因此辅助构造器永远都不可能直接调用超类的构造器，子类的辅助构造器最终都会调用主构造器，只有主构造器可以调用超类的构造器。（绕）
```
class Employee(name: String, age: Int, val salary: Double) extends Person(name,age)
```
这段代码定义了一个子类和一个调用超类构造器的主构造器，其中Employee类的三个参数，name,age和salary其中有两个被传递到了超类。

* 重写字段

Scala的字段由一个私有字段和取值器/改值器方法构成。可以用另一个同名的val字段重写一个val(或不带参数的def)。
如：
```
abstract class Person{
    def id: Int
}
class Student(override val id: Int) extends Person
```
使用val重写抽象的def
限制如下：
1. def只能重写另一个def
1. val只能重写另一个val或不带参数的def
1. val只能重写另一个抽象的var

>用var一般都不能被重写

* 匿名子类

可以通过包含带有定义或重写的代码块的方式创建一个匿名子类，比如：
```
val alien = new Person("Fred"){
    def greeting = "Greetings, Earthling! My name is Fred"
}
```
这将会创建出一个结构类型的对象，该类型标记为Person{def greeting: String}。可以用这个类型作为参数类型的定义：
```
def meet(p: Person{def greeting: String}){
    println(p.name + "says: " + p.greeting)
}
```
* 抽象类和抽象字段
和Java一样，可以用abstract关键字标记不能被实例化的类，通常这是因为它的某个或某几个方法没有被完整定义。
```
abstract class Person(val name: String){
    def id : Int
}
```
在子类中重写超类的抽象方法时，不需要使用override关键字
```
class Employee(name:String) extends Person(name){
    def id = name.hashCode
}
```
==抽象字段就是一个没有初始值的字段==
```
abstract class person{
    val id: Int//一个带有抽象getter方法的抽象字段
    var name: String//带有抽象getter方法和setter方法的抽象字段
}
```
==具体的子类必须提供具体字段（即带有初始值)==

和方法一样，在子类中重写超类中的抽象字段时，不需要override关键字
>可以随时用匿名类型定制抽象字段???
```
val fred = new Person{
    val id = 1729
    var name = "Fred"
}
```
* Scala继承层级
1. 与Java中基本类型相对应的类，以及Unit类型，都扩展自AnyVal
1. 所有其他类都是AnyRed的子类，AnyRed是Java中Object类的同义词。
1. AnyVal和AnyRef都扩展自Any类，而Any类是整个继承层级的根节点。
1. Any类定义了isInstanceOf，asInstanceOf方法，以及用于相等性判断和哈希码的方法。
1. AnyVal没有追加任何方法。它只是所有值类型的一个标记
1. AnyRef类追加了来自Object类的监视方法wait和notify/notifyAll。同时提供了一个带函数参数的方法synchronized。这个方法等同于Java中的synchronized块。

所有的Scala类都实现ScalaObject这个接口，这个接口没有定义任何方法。

继承层级的另一端是Nothing和Null类型

Null类型的唯一实例是null值。可以将null复制给任何引用，但不能复制给值类型的变量，即不能将Int设为null

Nothing类型没有实例。

![继承层级](http://7nj1qk.com1.z0.glb.clouddn.com/@/scala/class/scala_hierarchy.jpg)

- 判断对象相等

在Scala中AnyRef的equals方法调用eq，可以用eq检查两个引用是否指向一个对象。具体的equals一般自己重写根据实际需要定义
```
final override def equals(other: Any)={
    val that = other.asInstanceOf[Item]
    if(that == null) false
    else description == that.description && price == that.price
}
```
---

## scala中的特质(trait)
Scala中特质存在的意义主要是为了解决多继承的问题

trait支持部分实现，也就是说可以在trait中实现部分方法，因此特质即可以对应java中接口的功能，但是形式是与类相似
```
trait Logger{
    def log(msg: String)//抽象方法
}
```
这里不需要将方法声明为abstract，特质中未被实现的方法默认就是抽象的。
子类可以给出实现：
```
class ConsoleLogger extends Logger{//用extends，而不是implements
    def log(msg: String){println(msg)}//不需要写override
}
```
在重写特质的抽象方法时不需要给出override关键字
```
class ConsoleLogger extends Logger with Cloneable with Serializable
```
所有的java接口都可以作为Scala特质使用，和Java一样，Scala类只能有一个超类，但可以有任意数量的特质

特质中的方法不一定要求是抽象的

* 带有特质的对象
```
trait Logged{
    def log(msg: String){}
}
```
类定义
```
class SavingsAccount extends Account with Logged{
    def withdraw(amount: Double){
        if(amount > balance) log("Insufficient funds")
        else ...
    }
    ...
}
```
特质的特质
```
trait ConsoleLogger extends Logged{
    override def log(msg: String){println(msg)}
}
```
现在就可以在构造具体对象是加入一个特质

```
val acct = new SavingsAccount with ConsoleLogger
```
当在acct对象上调用log方法时，ConsoleLogger特质的log方法就会被执行
* 叠加在一起的特质

当类或对象添加多个互相调用的特质时，顺序是从最后一个开始

添加时间戳的类
```
trait TimestampLogger extends Logged{
    override def log(msg: String){
        super.log(new java.util.Data() + " " +msg)
    }
}
```
截断过长信息
```
trait ShortLogger extends Logged{
    val maxLength = 15
    override def log(msg: String){
        super.log( if(msg.length <= maxLength)) msg else mse.substring(0, maxLength-3) + "..."
    }
}
```
在上面的例子中，每一个log方法都将修改过的消息传递给super.log，对特质而言，super.log的含义与类不同，super.log调用的是特质层级中的下一个特质，一般来说，特质从最后一个开始被处理。

举例
```
val acct1 = new SavingsAccount with ConsoleLogger with TimestampLogger with ShortLogger

val  acct2 = new SavingsAccount with ConsoleLogger with ShortLogger with TimestampLogger
```
从acc1中得到的消息为
```
Sun Feb 06 17：45：45 ICT 2011 Insufficient...
```
ShortLogger的log方法首先被执行，然后它的super.log调用的是TimestampLogger（这个调用层级没有太看懂）；而从acct2取款时输出的是
```
Sun Feb 06 1...
```
其中TimestampLogger在特质列表中最后出现，log方法首先被调用，结果在之后被截断。

* 在特质中重写抽象方法

10.5突然加上一个abstract。。。？？

- 特质中的具体和抽象字段
1. 具体字段
```
trait ShoratLogger extends Logged{
    val maxLength = 15//具体的字段
}
```
>混入该特质的类自动获得一个maxLength字段。通常，对于特质中的每一个具体字段，使用该特质的类都会获得一个字段与之对应。这些字段不是被继承的，而只是简单地被加到了子类当中。

>在JVM中，一个类只能扩展一个超类，因此来自特质的字段不能以相同的方式继承。由于这个限制，因此来自特质的字段是被直接加入进去。因此具体的特质字段可以被当作是针对使用该特质的类的“装配指令”。任何通过这种方式被混入的字段都自动成为该类自己的字段。


特质中未被初始化的字段在具体的子类中必须被重写

2. 抽象字段
```
trait ShortLogger extends Logged{
    val maxLength: Int//抽象字段

}
```
当在一个具体的类中使用该特质时，必须提供maxLength字段：
```
class SavingAccount extends Account with ConsoleLogger with ShortLogger{
    val maxLength = 20//不需要写override
    ...
}
```
- 特质构造顺序

特质的构造器由字段的初始化和其他特质体中的语句构成
```
trait FileLogger extends Logger{
    val out =new PrintWriter("app.log")//特质构造器的一部分
    out.println("#" + new Date().toString)//同样是特质构造器的一部分
    def log(msg: String){
        out.println(msg);
        out.flusth()
    }
}
```

这些语句在任何混入该特质的对象在构造时都会被执行

构造器以如下顺序执行：
1. 首先调用超类的构造器
1. 特质构造器在超类构造器之后、类构造器之前执行
1. 特质由左到右被构造
1. 每个特质中，父特质首先被构造
1. 如果多个特质公有一个父特质，而那个父特质已经被构造，则不会被再次构造
1. 所有特质构造完毕，子类被构造

考虑如下一个类
```
class SavingAccount extends Account with FileLogger with ShortLogger
```
构造器将按照如下的顺序执行：
1. Account(超类)
1. Logger(第一个特质的父特质)
1. FileLogger(第一个特质)
1. ShortLogger(第二个特质)，它的父特质Logger已经被构造
1. SavingsAccount（类）

- 初始化特质中的字段

==特质不能有构造器参数。每个特质都有一个无参数的构造器==
>缺少构造器参数是特质与类之间唯一的技术差别。除此之外，特质可以具备类的所有特性，比如具体的和抽象的字段以及超类

比如一个文件日志生成器，想要指定日志文件
```
val acct = new SavingsAccount with FileLogger("myapp.log")//error，特质不能指定构造器参数
```
可是使用FileLogger一个抽象的字段用来存放文件名
```
trait FileLogger extends Logger{
    val filename: String
    val out = new PrintStream(filename)
    def log(msg: String){out.println(msg); out.flush}
}
```
这样使用该特质的类就可以重写filename字段，不过不能直接使用
```
val acct = new SavingsAccount with FileLogger{
    val filename = "myapp.log"//error
}
```
因为FileLogger构造器先于子类构造器执行。new语句构造的是一个扩展自SavingsAccount（超类）并混入了FileLogger特质的匿名类的实例。filename的初始化只发生在这个匿名的子类中，实际上并不会发生，因为在轮到子类之前，FileLogger的构造器会抛出一个空指针异常。

解决方法,在FIleLogger构造器中使用懒值
```
trait FileLogger extends Logger{
    val filename: String
    lazy val out = new PrintStream(filename)
    def log(msg: String){out.println(msg)}//不需要写override
}
```
这样out字段只有在初次被使用的时候才会初始化，而那个时候filename已经被设好值了

- 扩展类的特质（绕）

特质可以扩展另一个特质，而由特质组成的继承层级也很常见。特质也可以被类扩展，特质的超类会自动地成为我们的类的超类。

示例，LoggedException特质扩展自Exception类
```
trait LoggedException extends Exception with Logged{
    def log(){ log(getMessage()) }
}
```
LoggedException有一个log方法用来记录异常的消息。注意log方法调用了从Exception超类继承下来的getMessage()方法

创建一个混入上面特质的类
```
class UnhappyException extends LoggedException{//该类扩展自一个特质
    override def getMessage() = "arggh!"
}
```
特质的超类也自动地成为我们的类的超类

如果该类已经扩展了另一个类的话，只要那是特质的超类的一个子类即可，如
```
class UnhappyException extends IOException with LoggedException
```
这里的UnhappyException扩展自IOException，而这个IOException已经是扩展自Exception了。混入特质的时候，它的超类已经存在，因此不需要额外添加

如果类扩展自一个不相关的类，那么就不可能混入这个特质了。
```
class UnhappyFrame extends JFrame with LoggedException//error
```
无法同时将JFrame和Exception作为超类


* 吐血推荐下面对特质的解释

下面一个例子中定义了一个抽象类Animal表示所有的动物，然后定义了两个trait：Flyable和Swimable分别表示会飞和会游泳两种特征
Animal的实现
```
abstract class Animal{
    def walk(speed:Int)
    def breathe()={
        println("animal breathes")
    }
}
```
这里的抽象类Animal定义了walk方法，并实现了breathe方法
```
trait Flyable{
    def hasFearher = true
    def fly
}
trait Swimable{
    def swim
}
```
以上实现了两个trait，其中一个是hasFeather方法，这个方法已经实现了，另一个方法是fly方法，这个方法只定义没有实现而Swimable trait也只定义了一个方法，也没有实现

下面定义一种即会飞也会游泳的动物鱼鹰
```
class FishEagle extends Animal with Flyable with Swimable{
    def walk(speed:int)=println("fish eagle walk with speed" + speed)
    def swim() = println("fish eagle swim fast")
    def fly() = println("fish eagle fly fast")
}
```
FishEagle类继承自Animal，后面有两个with, with Flyable 和 with Swimable，表示也具备两种特征。在类的视线中需要实现抽象类Animal的walk方法，也需要实现两个特征中定义的未实现方法。
下面是main方法代码：
```
object App{
    def main(args: Array[String]){
        val fishEagle = new FishEagle
        val flyable:Flyable = fishEagle
        flyable.fly
        
        val swimmer:Swimable = fishEagle
        swimmer.swim
    }
}
```
在main方法中，我们首先初始化了一个FishEagle对象，然后通过Flyable和Swimable trait来分别调用其fly和swim方法，输出结果如下：
```
fish eagle fly fast
fish eagle swim fast
```
综上，抽象类可以做的任何事情，trait都可以做，它的长处在于可以多继承。

**trait和抽象类的区别在于抽象类是对一个继承链的，类和类之间确实有父子类的继承关系，而trait则如其名字，可以多继承**

## 操作符
- apply和update方法

在Scala中，对于下面代码
```
f(arg1,arg2, ..)
```
如果f不是函数或方法，那么这个表达式就等同于调用
```
f.apply(arg1,arg2,...)
```
当它出现在赋值语句的等号左侧。表达式
```
f(arg1,arg2,...) = value
```
对应如下调用
```
f.update(arg1,arg2,...,value)
```
这个机制被用于数组和映射，如
```
val scores = new scala.collection.mutable.HashMap[String, Int]
scores("Bob") = 100//调用scores.update("Bob",100)
val bobsScore = scores("Bob")//调用scores.apply("Bob")
```
apply方法也经常被用在伴生对象中，用来构造对象而不用显式使用new。
```
class Fraction(n: Int, d:Int){
    ...
}
object Fraction{
    def apply(n: Int, d: Int) = new Fraction(n, d)
}
```
因为有了apply方法，就可以用Fraction(3,4)构造出一个分数，而不用new Fraction(3,4)。









