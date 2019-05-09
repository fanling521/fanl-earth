# Scala函数式编程📍

## 基础篇

### 函数的定义

#### 基本语法

```scala
def 函数名(参数名:类型):返回类型={
    return  可省略
}
//可变函数参数类型📍
def fun(n1:Int,args:Int*)={
    
}
// 没有参数的，没有
def demo="hello"
//等价
def demo()={
    "hello"
}
```

#### 递归函数说明

1. 程序执行一个函数的时候，会创建一个新的受保护的函数栈
2. 函数的局部变量是独立的，不会相互影响
3. 不可以使用类型推导

#### 函数使用细节

1. 函数的形参可以有多个，如果没有，调用时候可以不写()
2. 形参列表和返回值列表的数据类型可以是值类型和引用类型
3. 函数可以根据函数最后一行推导函数返回值类型，可以省略return和返回值类型
4. 声明Unit，即为无返回值，不明确返回值类型，可以用Any

### 过程

将函数的返回**类型为Unit**的函数成为过程，如果函数没有明确返回值，那么等号可以省略。

```scala
def demo(n:Int){
    
}
```

### 惰性函数

惰性计算是许多函数式编程语言的特效，惰性集合在需要时提供其元素，无需预先计算他们。

当函数返回值被声明为lazy时，函数的执行将被推迟执行，直到首次取值，该函数才会被执行。

注意细节：

1. `lazy`只能修饰val，不能修饰var
2. 变量也可以被声明为`lazy`

**应用场景**：将耗时的计算推迟到绝对需求的时候。

```scala
object LazyDemo {

  def main(args: Array[String]): Unit = {
    lazy val res = sum(12, 12);
    printf("----------")
    printf("sum=" + res)
  }

  def sum(n1: Int, n2: Int): Int = {
    printf("执行了sum===")
    n1 + n2
  }
}
```

### 异常📍

Scala和Java的异常区别，其他的都可以参考Java的内容，finally表示资源的释放。

1. 只能有一个catch
2. Scala中的catch有多个case，=>表示处理的异常的代码块
3. 没有编译时异常，只有运行时异常

```scala
object ThrowDemo {
  def main(args: Array[String]): Unit = {
/*    val t = test
    printf("1.===" + t.toString)
    printf("2....")*/

    try{
      test
    }catch {
      case exception: Exception=>{
        printf("3333")
      }
    }
  }

  def test: Unit = {
    throw new Exception("异常")
  }
}
```

### 练习题

```scala
def myPrint(n: Int): Unit = {
  for (i <- 1 to n) {
    for (j <- 1 to i) {
      printf("%d * %d = %d\t", j, i, i * j)
    }
    println()
  }
}
```
## 高级篇

### 偏函数

`collect`函数支持偏函数， 当使用偏函数时，会遍历集合的所有元素，编译器执行流程时先执行isDefinedAt()，如果为true ,就会执行 apply, 构建一个新的Int 对象返回，执行isDefinedAt() 为false 就过滤掉这个元素，即不构建新的Int对象。

```scala
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4, "abc", 4)
    //偏函数PartialFunction接收参数Any,返回类型Int
    val pfn = new PartialFunction[Any, Int] {
        //判断元素
      override def isDefinedAt(x: Any): Boolean = {
        x.isInstanceOf[Int]
      }
        //isDefinedAt 是true执行
      override def apply(v1: Any): Int = {
        v1.asInstanceOf[Int] + 1
      }
    }
      //collect
    println(list.collect(pfn))
  }
//简写
   val list3 = list.collect{
      case i:Int => i + 1
      case j:Double => (j * 2).toInt
      case k:Float => (k * 3).toInt
    }

```

### 作为参数的函数

函数作为一个变量传入到了另一个函数中，那么该作为参数的函数的类型是：function1，即：(参数类型) => 返回类型。PS：有n个参数的函数类型为\<`function`**n**\>

前面的map(xx)就是这样的函数。

### 匿名函数

没有名字的函数就是匿名函数，可以通过函数表达式来设置匿名函数。

```scala
    val puls = (x: Int, y: Int) => {
      x + y
    }
```

### 高阶函数

能够接受函数作为参数的函数，叫做高阶函数。

```scala
    def sum(n: Int): Int = {
      n + 1
    }

    def add(n: Int): Int = {
      n * 2
    }

    def hofn(f1: Int => Int, f2: Int => Int, n: Int): Int = {
      f1(f2(n))
    }

    println(hofn(sum, add,5))
```

高阶函数返回匿名函数

```scala
  def main(args: Array[String]): Unit = {

    def high(x: Int) = {
      (y: Int) => x + y
    }

    println(high(1)(3))
  }
```

### 类型推断

1. 参数类型是可以推断时，可以省略参数类型
2. 当传入的函数，只有单个参数时，可以省去括号
3. 如果变量只在`=>`右边只出现一次，可以用`_`来代替

### 闭包（closure）

闭包就是一个函数和与其相关的引用环境组合的一个整体（实体）。

也就是说闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量

```scala
def high(x: Int) = {
  (y: Int) => x + y
}
//f 就是闭包，high函数y与外部的x形成一个整体
val f = high(20)
println(f(1))
```

### 函数柯里化（curry）

1. 函数编程中，接受多个参数的函数都可以转化为接受单个参数的函数，这个转化过程就叫**柯里化**
2. 柯里化就是证明了函数只需要一个参数而已。其实我们刚才的学习过程中，已经涉及到了柯里化操作

```scala
  def main(args: Array[String]): Unit = {
    /**
      * 编写一个函数，接收两个整数，可以返回两个数的乘积，要求:
      *
      * 使用常规的方式完成
      * 使用闭包的方式完成
      * 使用函数柯里化完成
      */
    def mult1(x: Int, y: Int) = x * y

    println("01-" + mult1(2, 3))

    def mult2(x: Int) = {
      (y: Int) => x * y
    }

    println("02-" + mult2(2)(3))

    def mult3(x: Int)(y: Int) = x * y

    println("03-" + mult3(2)(3))
  }
```

### 控制抽象

1. 参数是函数
2. 函数参数没有输入值也没有返回值