---
layout: post
title: 写给Java程序员的Lice教程
category: Lisp
tags: Lisp
keywords: Lisp,Java,Lice
description: Intro to lice for java developers
---

# 写给Java程序员的Lice教程

### 本文仅适用于 Lice 1.X

本文大部分内容都是一本正经介绍语言，有少量的不易察觉的卖萌。<br/>
—— by [千里冰封](http://ice1000.org)

（第一次写这么长的技术文章啊）

## Lice是什么

这是一门运行在JVM上的解释性Lisp方言。

下文中，有些地方会把Lice称为Lisp。可能是因为写的比较仓促，见谅。

## Lice有哪些特色

+ 轻量级
+ 熟悉语法只需要五分钟
+ 没有运行时，标准库可定制
+ 解释器是纯Kotlin实现，跨到所有JVM存在的平台
+ 和Java有良好的互操作性
+ 支持动态元编程
+ 动态解释性语言不需要编译，可以代替一些频繁改动的逻辑
+ 支持全角括号，全角分号，全角引号，全角逗号等初学者易混淆的东西

## 可以看看它长啥样吗

可以。简单的程序：

```lisp
(print (str-con "Hello World"))
```

普通的程序：

```lisp
(require lice.math)

(print (<- PI)) ; It's in lice.math

(while (> 10 (<-> i 0))
  (|> (print (<- i))
   (type (<- i))
   (-> i (+ 1 (<- i)))))
```

复杂的程序：

```lisp
(require lice.io)
(require lice.str)

(for-each
  i
  (.. 1000 1500)
  (thread|> (force|>
    (println (<- i))
    (write-file
      (file (format "%d.html" (<- i)))
      (read-url (url (str-con
        "https://vijos.org/p/" (<- i))))))))
```

## 学习Lice需要什么基础

这取决于你想做到什么程度。

### 将Lice作为学习Lisp所采用的语言

+ 那么你不需要拥有编程基础
+ 但是你需要会用电脑，以及会安装JRE。

### 将Lice用于平时写一些小玩具用

+ 少量的编程基础
+ 知道啥是函数调用，啥是返回值

### 将Lice作为自己项目的脚本语言

+ 最基本的Java基础，会写匿名内部类或者Lambda表达式。

### 参与Lice语言的开发

+ 扎实的Kotlin基础
+ 函数式编程基础
+ 面对几千行高度压缩过的代码还能硬着头皮去啃的精神
+ 怒怼千里冰封的勇气

~~参与维护还有机会收到女装照~~

## 环境搭建

下载安装JRE，并将它添加到环境变量中。在命令行输入以下指令以检查环境安装：

```shell
C:\Users\ice1000>java
```

如果出现大量说明文字那么安装成功。

如果出现`'java' 不是内部或外部命令，也不是可运行的程序或批处理文件。`那么说明安装失败，再找找教程吧。

### 下载Lice

你可以：

#### 通过网页下载

在GitHub的release界面下载[Lice的交互式解释器](https://github.com/ice1000/lice/releases)。

交互式解释器(下文简称repl)是一个可执行的jar文件。

#### 通过源码构建

```shell
git clone https://github.com/ice1000/lice.git
cd lice
git gc
```

~~讲道理，我也不知道怎么命令行编译，你还是用IntelliJ IDEA吧。~~

## 首次运行GUI的repl

首先打开repl，可以看到一个窗口。

![repl](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/1.PNG)

位于最上方的按钮Clear Screen用于清空屏幕。

可以看到屏幕上的三行说明信息，以及最下面的一行提示字符：

```lisp
Lice >
```

你会发现这个窗口并不能进行编辑。
这是很正常的，因为下面有一个专门用于输入的输入框，用于输入代码。
回车以将输入的代码提交到repl中。

你可以在下面的窗口输入以下代码检查repl是否正常工作。

```lisp
(+ 1 1)
```

`输入`这一动作应该是很好理解的，但是有些人就是不知道`输入`是什么意思（这种人按理说早就被孙明琦劝退了），于是我解释一下：

就是按照下图这样输入。我截取了刚才的repl窗口的一部分。

![input](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/2.PNG)

这是一段**基于括号的S表达式**的代码，其意义等同于Java中的

```java
1 + 1
```

但是，这里有两个区别。

0. `(+ 1 1)`是合法的、可以运行的代码。`1 + 1`只是你Java代码中的一部分，不能单独运行。
0. `(+ 1 1)`中的+符号是一个函数，它接收多个数值参数，返回他们相加的结果。而`1 + 1`的+不是函数，是Java语言语法的一部分。 

可以看到，Lice是支持使用一些奇奇怪怪的字符作为函数名的。

好了，现在按下回车可以看到你输入的代码（蓝色）、输出（下面的`2 => java.lang.Integer`），以及新的光标。

![output](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/3.PNG)

根据这种格式，下文中的代码都以以下形式提供（就是直接把repl里的东西抄进来）。

```lisp
Lice > (+ 1 1)
2 => java.lang.Integer
```

也就是说，`Lice > `后面的东西就是你该输入的代码。下面是输出。

## 使用命令行repl

这一部分不是写给编程新手看的，编程新手看完上面的部分请直接进入下面的“Hello World”部分。

下载jar，命令行输入

```shell
User > java -jar Lice.jar -cui
```

你会看到输出：

```shell
Lice language repl v1.1-SNAPSHOT
see: https://github.com/ice1000/lice

for help please input: help

Lice > (+ 1 1)
2 => java.lang.Integer
```

好了废话了半天，我们开始使用Lice编程吧。

## Hello World

首先我们要像TCPL里面说的那样，使用一个输出“Hello World”字符串的程序来开启我们的编程之旅。

```lisp
Lice > (print "Hello World")
Hello World
Hello World => java.lang.String
```

好了我们来看一下输出。第一行是这个print函数的输出，print函数接收任意数量的任意类型的参数，输出他们的toString后的值。

第二行是整个程序的返回值（在这里，就是print函数，因为整个程序只有一个函数的调用），前面是它的toString，`=>`后面是它的类型。类型直接采用Java的类型。

我们可以再试试：

```lisp
Lice > (+ 1 1)
2 => java.lang.Integer

Lice > (print "boy " "next " "door")
boy next door
door => java.lang.String

Lice > (print 4 5 0)
450
0 => java.lang.Integer

Lice > (print 0xFFF)
4095
4095 => java.lang.Integer
```

这说明：

+ print函数输出它的参数
+ print函数原封不动地返回它输出的最后一个内容

写到这里我发现进度太慢（初中都下课了），下文将会稍微快节奏一些。这里预警一下。

那么这就是print函数了，对应的还有println函数，可以直接观察它的行为来了解它的工作机制。

```lisp
Lice > (println 100 200 300)
100
200
300
300 => java.lang.Integer
```

## Lice的语法

Lice的语法很简单。

0. 注释只有一种： 分号开头，直到行末都是注释。 ; 比如这就是一个注释。
0. 函数调用： (函数名 参数1 参数2 ...)
0. 字面量： 100 0xFF 0b1010 011 true false null () "字符串" 以上是Lice仅有的几种字面量类型。
0. 空格和逗号用于区分不同元素（就是函数名啦，字面量啦，下同）。空格和括号语义完全相同，你可以按照自己喜欢的方式使用。

注意：

0. 字符串不支持逃逸字符，但是支持在源码中写成跨多行的形式。
0. 整数字面量支持 2 8 10 16 三种进制， 0b 开头是2进制， 0 开头是8进制 0x开头是16进制（字母不区分大小写）其余被视为十进制。
0. 整数后面加个n（模糊大小写），可以变成BigInteger
0. 不能直接在括号中写一个值，括号里第一个元素必须是函数名，后面是参数。
0. 每个函数都有返回值。
0. Lice没有变量的概念，使用一个全局符号表取而代之，你可以理解为是只支持全局变量。缺点是无法构建大型程序，优点是实现极其轻量级。
0. 不能`(+ 1(+ 1 1))`，必须`(+ 1 (+ 1 1))`。否则`1(+`会被视为一个单独的词法元素。
0. 这是一个bug，代码文件第一个字符是分号的话这个字符会被视为函数而不是注释。我认为无可厚非于是没去管，毕竟你只需要不在第一行写注释就行了。

语法就这么点。你现在已经学会Lice这门语言了，但是你肯定还不知道怎么编程。毕竟if while这些结构你都不知道咋整。
其实这些都是标准库函数，将在下文介绍。

~~你还可以通过Lice强(la)大(ji)的动态元编程能力来把他们取消定义掉，这样这语言就没法用了。~~

## 基本的标准库函数

### 数值运算

这部分很简单，就是整数运算。支持加减乘除之类的操作。

```lisp
Lice > (+ 1 1 1)
3 => java.lang.Integer

Lice > (- 10 5 1)
4 => java.lang.Integer

Lice > (* 2 2 2 2)
16 => java.lang.Integer

Lice > (/ 100 10)
10 => java.lang.Integer
```

这些函数都可以嵌套：

```lisp
Lice > (+ 1 1 (* 2 2 (/ 100 10)))
42 => java.lang.Integer
```

为了解决这个问题可是费劲了我的脑子：

```lisp
Lice > (- 1N 10)
-9 => java.math.BigInteger

Lice > (- 10N 1)
9 => java.math.BigInteger

Lice > (+ 1 1)
2 => java.lang.Integer

Lice > (+ 9999999999999999999999999999999999999N 9999999999N)
10000000000000000000000000009999999998 => java.math.BigInteger

Lice > (- 9999999999999999999999999999999999999N 9999999999N)
9999999999999999999999999990000000000 => java.math.BigInteger

Lice > (+ 1N (int->double 1))
2.0 => java.math.BigDecimal

Lice > (* 1 1N)
1 => java.math.BigInteger

Lice > (* 1 10 1290 )
12900 => java.lang.Integer

Lice > (* 129308 (int->double 130))
1.681004E7 => java.lang.Double

Lice > (* 129308 (int->double 130) 1N)
1.681004E+7 => java.math.BigDecimal

```

还有比较函数，返回布尔值，下面是它们的行为实例。它涉及到了一些复杂行为：

```lisp
Lice > (== 10 10)
true => java.lang.Boolean

Lice > (== 10 10 10 10)
true => java.lang.Boolean

Lice > (== 19 21)
false => java.lang.Boolean

Lice > (< 1 2 3 4)
true => java.lang.Boolean

Lice > (<= 1 1 2 3 100 0xFF)
true => java.lang.Boolean

Lice > (< 1 1 2 3)
false => java.lang.Boolean

Lice > (!= 1 2 3 4)
true => java.lang.Boolean

Lice > (!= 1 2 1 3)
true => java.lang.Boolean

Lice > (!= 1 2 2 3)
false => java.lang.Boolean
```

### 布尔运算

我觉得我不需要多说什么。

```lisp
Lice > (&& true true)
true => java.lang.Boolean

Lice > (&& true false)
false => java.lang.Boolean

Lice > (&& true null)
type mismatch: expected: Boolean, found: java.lang.Object
at line: 1

Lice > (|| false false true)
true => java.lang.Boolean

Lice > (! true)
false => java.lang.Boolean
```

### 包含库函数

为了防止符号表变得太大，我把常用函数分开放置，简单模块化了一下。
使用一个模块的话需要这样包含它。

```lisp
Lice > (require lice.str)
true => java.lang.Boolean

Lice > (require lice.deep.dark.fantasy)
lice.deep.dark.fantasy not found!
null => java.lang.Object
```

第二个例子是当你试图包含一个不存在的库时，解释器的行为。

### 字符串操作

字符串操作的大部分函数都在 lice.str 里面。使用它们的话首先需要包含这个函数库。

```lisp
Lice > (require lice.str)
true => java.lang.Boolean

Lice > (str-con "deep " "dark " "fantasy")
deep dark fantasy => java.lang.String

Lice > (str-con (str-con "My name " "is Van") " I'm an artist")
My name is Van I'm an artist => java.lang.String
```

str-con 就是连接字符串， string-connect 的缩写。

还有更多的。这里介绍了大量标准库函数，读者不必完全记忆：

```lisp
Lice > (split "My name is Van, I'm an artist, I'm an performance artist" " ")
[My, name, is, Van,, I'm, an, artist,, I'm, an, performance, artist] => java.util.ArrayList

Lice > (int->oct 100)
0144 => java.lang.String

Lice > (int->hex 100)
0x64 => java.lang.String

Lice > (int->bin 100)
0b1100100 => java.lang.String

Lice > (str->int "02135")
1117 => java.lang.Integer

Lice > (str->int "100")
100 => java.lang.Integer

Lice > (str->int "-0xFFab219")
-268087833 => java.lang.Integer

Lice > (str->int "-0b1010110")
-86 => java.lang.Integer

Lice > (format "My name is %s" "Van")
My name is Van => java.lang.String

Lice > (->chars "Fucking slaves get your ass back here")
[F, u, c, k, i, n, g,  , s, l, a, v, e, s,  , g, e, t,  , y, o, u, r,  , a, s, s,  , b, a, c, k,  , h, e, r, e] => java.util.ArrayList
```

### 流程控制语句

你现在终于要学会怎么使用流程控制了。讲道理，这里看完，你就可以使用Lice写出稍微复杂一点的代码了。

```lisp
Lice > (if (== 1 1) (print "Oh my shoulder") (print "Give up? ha?"))
Oh my shoulder
Oh my shoulder => java.lang.String

Lice > (if (!= 1 1) (print "Oh my shoulder") (print "Give up? ha?"))
Give up? ha?
Give up? ha? => java.lang.String
```

很简单是不是？ if函数需要两个必须提供的参数，第一个是一个布尔值，是条件。第二个是“true分支”，如果为真那么调用并返回它。
第三个参数是可选的参数，是“else”分支。

上面第一段代码对应的Java代码如下：

```java
if (1 == 1) {
    System.out.print("Oh my shoulder");
} else {
    System.out.print("Give up? ha?");
}
```

while也很简单，接收两个参数，第一个是条件，第二个是如果条件为真时执行的代码。因为这里还没有涉及到变量的定义，因此我们暂时只能写死循环：

```lisp
Lice > (while true (print 1))
1
1
1
1
1
1
... 无限循环，省略
```

就语法来讲还是很简单的。

~~什么？for循环是什么？~~

### 引入副作用

有时你需要同时执行多个语句。比如，if和while每个条件分支都只提供了一个参数来提供给你执行代码。如果你需要实现以下Java代码：

```java
if (true) {
  System.out.println("I have a mmp");
  System.out.println(" and ");
  System.out.println("I have not idea whether I should say");
}
```

在这里你需要用到一个这样的函数：

```lisp
Lice > (|> (println "billy") (type "billy"))
billy
java.lang.String
billy => java.lang.String
```

`|>`，看起来很奇怪是不是？这和Clojure中的run含义完全相同。我给它起这个名字是因为这个符号看起来像各种IDE中表示`运行`的三角形箭头：

![run](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/4.PNG)

而且在Fira Code字体打开ligature的时候可以看到这两个字符的ligature：

![ligature](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/5.PNG)

于是有：

```lisp
Lice > (|> (println "I have a mmp") (println " and ") (println "I have not idea whether I should say"))
I have a mmp
 and
I have not idea whether I should say
I have not idea whether I should say => java.lang.String
```

同时，还有两个比较类似的函数，一个是`thread|>`，传入的所有表达式将会被在一个单独的线程里求值，并返回最后一个。

```lisp
Lice > (require lice.thread)
true => java.lang.Boolean

Lice > (thread|> (print 1) (print 2) (print 3))
null => java.lang.Object
1
Lice > 
2
3
```

看到了没？由于求值被放在了另外的线程中，repl的输出被打乱了。

还有一个是`force|>`，执行里面所有的代码并在产生Exception的时候退出执行，返回最后一次成功求值的结果。
可以用于强制执行一段代码，比如爬取网页的时候配合`thread|>`使用。

```lisp
(require lice.io)
(require lice.str)

(for-each i
  (.. 1000 1500)
  (thread|> (force|>
    (println (<- i))
    (write-file
      (file (format "%d.html" (<- i)))
      (read-url (url (str-con "https://vijos.org/p/" (<- i))))
    )))
)
```

上面的代码可以爬取vijos的1000到1500号题目到当前目录。

那么你可能会问了，上面的`<-`和`->`是啥意思啊？

### 值的保存

终于到了“变量”了！实际上Lice是不支持变量的（因为我不知道符号表怎么做成栈式的，我菜爆了），但是使用了另一种方法作为替代方案。

这个方案很挫，挫到爆，但是还是有那么一点点用处的。

Lice有一个全局的字符串到值的映射（就是一个HashMap），你可以通过符号来访问它的值。

Lice有两个函数，它们长得非常形象：

+ `->`，用于把一个值塞进符号
+ `<-`，用于把一个值从符号中取出
+ `<->`，用于取出一个值，并提供默认值。如果取出后得到null，那么就填入默认值并返回该值。

我们可以直接从repl里面看到它的能力。

```lisp
Lice > (-> name-of-variable "Pachouli Go")
Pachouli Go => java.lang.String

Lice > (<- name-of-variable)
Pachouli Go => java.lang.String

Lice > (require lice.io)
true => java.lang.Boolean

Lice > (-> file (file "sample/tests/test2.lice"))
sample\tests\test2.lice => java.io.File

Lice > (<- file)
sample\tests\test2.lice => java.io.File

Lice > (<- undefined-name)
null => java.lang.Object

Lice > (<-> undefined-but-with-default "deep")
deep => java.lang.String

Lice > (<-> name-of-variable "Fucking slaves")
Pachouli Go => java.lang.String

Lice > (<-> undefined-but-with-default "dark")
deep => java.lang.String
```

讲道理我觉得还是很简单的。

我仔细看了一遍上文，我觉得介绍的差不多了，现在你基本已经可以自由地使用Lice编程了。等到啥时候我有闲心了给你们写一篇讲标准库的。

先给个列表操作的代码：

```lisp
(require lice.thread)

; run this in the command line repl.

(for-each
  i
  (.. 1 10)
  (|>
    (println (<- i))
    (sleep (* 100 (<- i)))
    (print "I'm alive")))
```

## 元编程

讲道理，目前的元编程只有一点点。比如有一个Ruby风格的undef：

```lisp
Lice > (print "Bye, World!")
Bye, World!
Bye, World! => java.lang.String

Lice > (undef print)
null => java.lang.Object

Lice > (print "give me an error!")
undefined function: print

at line: 1
```

虽然我感觉没什么卵用，但是拿来作为奇技淫巧玩玩还是可以的。

## 和Java的交互

现在你将看到如何使用Java扩展Lice语言。由于本文是面向Java程序员，我假定你可以看懂下面的代码。

```java
import lice.Lice;
import lice.compiler.model.ValueNode;
import lice.compiler.util.SymbolList;
import java.io.File;

class TheJavaClass {
  public static void main(String[] args) {
    // 创建一个符号表，可以往里面添加方法。
    // 构造函数调用默认的就行了
    SymbolList sl = new SymbolList();
    // 定义一个方法
    sl.defineFunction(
        // 方法的名字
        "java-api-invoking",
        // 方法体，一个Lambda
        // 参数说明：
        // ln是这段代码在源码中的行号，类型是int，可以在报错的时候给出恰当的信息
        // ls是List<Node>，每个Node都是传入的参数，按顺序放好了的。
        // 有一个eval()方法，用于给这个参数进行求值。只要不是循环结构，一般只求值一次。
        // 然后再返回一个Node
        (ln, ls) -> {
          System.out.println("");
          return new ValueNode(100, ln);
        }
    );
    Lice.run(new File("test.lice"), sl);
  }
}
```

先给IntelliJ IDEA的Markdown编辑器点个赞，支持语言的高亮真是帮大忙了：

![highlighting languages](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/0/6.PNG)

首先你需要了解两个类和一个接口。

### Value类

这表示Lice语言执行过程中产生的“值”。它有一个Object属性和一个Class<*>属性。

Object属性是它的值，Class<*>属性是它的类型。

+ 为什么不直接从 obj.getClass() 获取类型？<br/>
为了使obj为null的时候依旧有效。

### Node接口

这是这个Lisp语法树的一个节点的抽象。

+ 它具有一个方法eval()：对这个节点进行求值。
+ 它还具有一个属性：在源码中的行号（用于报错）。

之所以它是接口，是因为它有两种可能的求值过程：

+ 一个表达式：函数调用+参数传递，求值结果是函数返回值
+ 一个字面量：就是这个字面量的值

### ValueNode类

它实现了Node接口。你需要传入一个值，随便一个值都可以，比如一个对象。这里以刚才给出的代码为例：

```java
(ln, ls) -> {
  return new ValueNode(100, ln);
}
```

在定义了这个方法之后，运行指定Lice脚本，Lice代码就可以拿到100这个对象了。

比如我们在Lice脚本里面写下如下代码：

```lisp
(print (java-api-invoking))
```

保存在上面Java代码所指定的test.lice中，运行上面的Java代码，将会得到如下输出：

```lisp
100

100 => java.lang.Integer

Process finished with exit code 0
```

也就是说，我们成功将对象传给了Lice。

类似的，我们还可以在方法里面添加一些可变的对象（我知道这很不fp，但是这起码算一种交互啊）

```java
import lice.Lice;
import lice.compiler.model.ValueNode;
import lice.compiler.util.SymbolList;
import java.io.File;

class TheJavaClass2 {
  public static void main(String[] args) {
    SymbolList sl = new SymbolList();
    final int[] a = new int[1];
    a[0] = 0;
    sl.defineFunction(
        "java-api-invoking2",
        (ln, ls) -> {
          System.out.println("Boy next door");
          a[0]++;
          return new ValueNode(a[0], ln);
        }
    );
    Lice.run(new File("test.lice"), sl);
  }
}
```

再在test.lice里面多调用几次这个函数：

```lisp
(print (java-api-invoking))
(print (java-api-invoking))
(print (java-api-invoking))
(print (java-api-invoking))
(print (java-api-invoking))
```

运行结果：

```lisp
1
2 => java.lang.Integer
3
4 => java.lang.Integer
5
6 => java.lang.Integer
7
8 => java.lang.Integer
9
10 => java.lang.Integer
```

好了本篇教程就到这里啦。
