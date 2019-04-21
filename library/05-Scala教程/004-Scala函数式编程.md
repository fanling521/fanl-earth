# 函数式编程📍

## 基础篇

### 函数的定义

#### 基本语法

```scala
def 函数名(参数名:类型):返回类型={
    return  可省略
}
//可变函数
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

1. lazy 只能修饰val，不能修饰var
2. 变量也可以被声明为lazy

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

