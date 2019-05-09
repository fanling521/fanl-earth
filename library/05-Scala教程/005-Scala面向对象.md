# Scala面向对象

## 基础篇

### 类与对象

此描述和Java中是一样的，要再次指出Scala语言是纯粹的面向对象的，在Scala中一切皆对象。

#### 类的定义

注意事项：Scala中类不需要声明为public，默认为public，一个文件中可以有多个类。

```scala
class Xxx{
    
}
// 使用
val|var xx:类型=new 类型()
val|var xx:Xxx=new Xxx()
// 可以省略类型，但是多态适合需要指明。把子类赋值给父类对象。
```

#### 对象

大体的思和Java是一样的。

对象的创建过程：

1. 加载类的信息
2. 在内存开辟空间
3. 使用父类的构造器开始初始化
4. 使用主/辅助构造器为属性初始化
5. 将此对象的地址赋值给变量

#### 方法

Scala类的方法就是函数，其创建等内容和函数是一样的。

#### 构造器

Scala的构造器和Java一样构造对象的适合需要调用构造方法，也可以有多个构造方法，但是在写法和逻辑上有一定的区别，Scala的构造器分为主构造器和辅助构造器。

##### 主构造器

主构造器直接放在类名后面，如果需要设置为私有，可以添加private，主构造器会执行类中所有的语句，即构造器也是函数。没有参数的主构造器可以省略()

```scala
class Dog(){
    ......
}

class Dog(str:String){
    ......
}

class Dog private(){
    ......
}
```

##### 辅助构造器📍

辅助构造器的名称就是this，都会直接或者间接调用主构造器，this()需要放在第一行。

```scala
def this(name:String){
    this()
    .......
}
```

#### 属性

Scala类中的主构造器的形参没有使用任何修饰符，那么这个形参就是局部变量，实例化对象无法访问到的。

如果使用val 修饰，这个形参就是类的私有的只读属性，而用var修饰则是私有的可读可写的属性。

```scala
object ClassDemo {

  def main(args: Array[String]): Unit = {
    val dog = new Dog("tom")
    dog.DogName
    dog.name
  }

  class Dog(val name: String) {
    var DogName = name
  }
}
```

注意：给某个属性添加@BeanProperty会自动生产get/set方法。

```scala
  class Cat{
    @BeanProperty
    var catName:String=""
  }
```

### 包

Scala和Java的包管理是一样的，但是其更加复杂。

源文件和指定包路径可以不匹配，编译后会按照源文件的包路径存放。

```scala
// com.fanling
package com.fanling{
    // 一个源文件多个package
    // com.fanling.demo1
	package demo1{
    
	}
	// com.fanling.demo2
    package demo2{
    
	}
}
```

#### 包对象

```scala
package com.fanling {
    
  package object scala {
    var name = "fanl"
  }
    
  package scala {
    class App {
      def main(args: Array[String]): Unit = {
        val m = new My
        m.test()
      }
    }
    class My {
      def test(): Unit = {
        println(name)
      }
    }
  }
}

```

注意事项：

1. 包对象需要在父类包中定义
2. private修饰包对象，只能在类内部和伴生对象中使用
3. protected 修饰的只能子类中使用
4. 可以定义访问范围，如 `private[com] val name=“name”`

#### 包的引入📍

1. 包的引入可以出现在任何地方，缩小包的作用范围，提高效率
2. 所有的包的通配符是`_`
3. 可以用选择器选择引入的类，如`import scala.collection.mutable.{HashMap,HashSet}`
4. 可以重命名引用的类，防止出现歧义，如`import java.util.{HashMap=>JavaHashMap,List}`
5. 不想使用某个类，可以隐藏掉，如`import java.util.{HashMap=>_,_}`

### 面向对象的特征

#### 封装

封装就是抽象的数据和对数据的操作封装在一起，数据被保护在内部，只能通过被授权的操作才能对数据进行访问。

#### 继承

和Java一样是单继承，重写需要用`override`，调用超类方法需要用`super`

##### 类型检查和转换

要测试某个对象是否属于某个给定的类，可以用`isInstanceOf[]`方法，`asInstanceOf[]`方法将引用转换为子类的引用，用`classOf[]`获取对象的类名。

##### 超类的构造

只有子类的主构造器才能调用父类的构造器。

##### 覆写字段

Scala中可以通过使用`override`覆写父类的字段，这和Java中是不同的。

##### 抽象类

在Scala中，通过abstract关键字标记抽象类，方法不用标记，只要省略方法体即可。

默认情况下，抽象类是不能实例化的，但是你通过实例化，其实就是实现该实现中的所有抽象方法，抽象类可以没有抽象方法，抽象方法不能有方法主体，不能用abstract修饰，一个类继承了抽象类，则需要实现所有的抽象方法。

##### 匿名子类

和Java中类似，语法方式不同。

```scala
object Dog {
  def main(args: Array[String]): Unit = {
    val dog=new Dog {
      override def say(): Unit = {
        println("哇哇哇哇")
      }
    }
    dog.say()
  }
}

abstract class Dog{
  def say();
}
```

#### 多态

```scala
  def testWork(employee: Employee): Unit = {
    if (employee.isInstanceOf[Manager]) {
      employee.asInstanceOf[Manager].manager()
    } else if (employee.isInstanceOf[Worker]) {
      employee.asInstanceOf[Worker].work
    }
  }
```

## 高级篇

### 静态属性和静态方法

#### 伴生对象和伴生类

当在同一个文件中，有 `class ScalaPerson` 和 `object ScalaPerson`，`class ScalaPerson` 称为伴生类,将非静态的内容写到该类中，`object ScalaPerson` 称为伴生对象,将静态的内容写入到该对象（类）

 `class ScalaPerson` 编译后底层生成 `ScalaPerson`类 `ScalaPerson.class`

 `object ScalaPerson` 编译后底层生成 `ScalaPerson$`类 `ScalaPerson$.class`

注意点：

1. 伴生对象中声明的全是 "静态"内容，可以通过伴生对象名称直接调用。

2. 如果 class A 独立存在，那么A就是一个类， 如果 object A 独立存在，那么A就是一个"静态"性质的对象[即类对象]

#### 用Apply创建对象

```scala
class Pig(name: String) {
  @BeanProperty
  var pName = name
}

object Pig {
  def apply(name: String): Pig = new Pig(name)

  def apply(): Pig = new Pig("小组")
}


object TestObject {
  def main(args: Array[String]): Unit = {
    val p = Pig()
    println(p.getPName)
  }
}
```

### Trait特质

Scala是纯面向对象的语言，在Scala中，没有接口，多个类具有相同的特征（特征）时，就可以将这个特质（特征）独立出来，采用关键字trait声明，理解trait 等价于（interface + abstract class），Java中的接口可以当做特质使用。

特质中可以定义具体字段，如果初始化了就是具体字段，如果不初始化就是抽象字段，未被初始化的字段在具体的子类中必须被重写。

特质可以继承类，以用来拓展该特质的一些功能。

```scala
trait TraitTest {
  def getConnect()
}

// 用extends关键字，多个特质用with连接
class Connect extends TraitTest {
  override def getConnect(): Unit = {
    println("呵呵")
  }
}
```

##### 动态混入

动态混入可以在不影响原有的继承关系的基础上，给指定的类扩展功能。

```scala
trait Operate {
  def insert(): Unit ={
    println("insert")
  }
}


class OracleDB {

}

abstract class MySQLDB {
  def scan()
}

object TestObject {
  def main(args: Array[String]): Unit = {
    val mySQL = new MySQLDB with Operate {
      override def scan(): Unit = {
        println("scan")
      }

      override def insert(): Unit = {
        println("insert2")
      }
    }
    mySQL.insert()
    mySQL.scan()

    val oracle = new OracleDB with Operate
    oracle.insert()
  }
}
```

**问题**：Scala创建对象是方式有哪些？

1. new 对象
2. 匿名子类
3. 特质的动态混入
4. 伴生对象的apply方法

##### 叠加特质

构建对象的同时如果混入多个特质，称之为叠加特质，声明从左到右，实现从右到左。

##### 富接口

即该特质中既**有抽象方法**，又有**非抽象方**法。

##### 自身类型

```scala
this: Exception =>
```

### 嵌套类

（1）使用内部类和静态内部类

```scala
class MyDog {
  class MyInnerDog{}
}
object MyDog{
  class MyStaticInnerClass{}
}

object Test01 {
  def main(args: Array[String]): Unit = {
    val myDog1:MyDog=new MyDog
    val myDog2:MyDog=new MyDog
    //内部类
    val inner01=new myDog1.MyInnerDog
    //内部静态类
    val inner02=new outclass.MyDog.MyStaticInnerClass
  }
}
```

（2）在内部类中实现调用外部类的属性

```scala
class MyCat {
  //别名
  myouter =>
  var name = "cat"

  class MyInnerCat {
    def info: Unit = {
      println("name=" + myouter.name)
    }
  }

}
//测试类
val mycat01 = new MyCat
val myInnerCat = new mycat01.MyInnerCat
myInnerCat.info
```

（3）类型投影（外部类调用内部类）

```scala
//调用内部类
def ic(myInnerCat: MyCat#MyInnerCat): Unit = {
  println(myInnerCat.age)
}

val mycat01 = new MyCat
val myInnerCat = new mycat01.MyInnerCat
mycat01.ic(myInnerCat)
```

### 隐式转换⭐

#### 隐式函数

隐式转换函数是以`implicit`关键字声明的带有单个参数的函数。这种函数将会自动应用，将值从一种类型转换为另一种类型。

```scala
  def main(args: Array[String]): Unit = {
    implicit def f1(d: Double): Int = {
      d.toInt
    }

    val num: Int = 3.5
    println(num)
  }
}
```

#### 隐式值

隐式值也叫隐式变量，将某个形参变量标记为implicit，所以编译器会在方法省略隐式参数的情况下去搜索作用域内的隐式值作为缺省参数。

```scala
object Demo2 {
  def main(args: Array[String]): Unit = {
    implicit val name = "jack"

    def info(implicit n: String = "tom"): Unit = {
      println(n)
    }
    info
  }
}
```

#### 隐式类

可以使用`implicit`声明类，隐式类的非常强大，同样可以扩展类的功能，写法可以参考隐式函数。

1. 其所带的构造参数有且只能有一个
2. 隐式类必须被定义在类，伴生对象和包对象里
3. 隐式类不能是case class（case class在定义会自动生成伴生对象与2矛盾）
4. 作用域内不能有与之相同名称的标示符

## 泛型

如果我们要求函数的参数可以接受任意类型。可以使用泛型。

```scala
object Demo07 {
  def main(args: Array[String]): Unit = {
    val msg1 = new Message[String]("12")
    println(msg1.f)

    val msg2 = new Message[Int](12)
    println(msg2.f)
  }
}

class Message[T](n: T) {
  def f = n
}
```

### 上界

在 Scala 里表示某个类型是 A 类型的子类型，也称上界或上限，使用 `<:` 关键字，语法如下：

`[T <: A]`或用通配符：`[_ <: A]`

对于下界，可以传入任意类型。

### 下界

在 scala 的下界或下限，使用 `>:` 关键字，语法如下：

`[T >: A]`或用通配符：`[_ >: A]`

### 协变、逆变、不变

Scala的协变(+)，逆变(-)，协变covariant、逆变contravariant、不可变invariant

对于一个带类型参数的类型，比如 List[T]，如果对A及其子类型B，满足 List[B]也符合List[A]的子类型，那么就称为covariance(协变) ，如果 List[A]是 List[B]的子类型，即与原来的父子关系正相反，则称为contravariance(逆变)。如果一个类型支持协变或逆变，则称这个类型为variance(翻译为可变的或变型)，否则称为invariance(不可变的)