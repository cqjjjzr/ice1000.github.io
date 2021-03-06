---
layout: post
title: 使用内存池消除可能出现的野指针delete操作
category: JNI
tags: Java
keywords: Java,JNI,C++
description: JNI Tutorial
---

## 为什么我想到要干这个事情

最近又一次构建了下项目[algo4j](https://github.com/ice1000/algo4j)，遇到一个比较麻烦的问题，就是之前写的那个矩阵乘法加速斐波那契数列求值的JNI函数老是让JVM崩溃，而且这个bug不能复现。

附一张面向运气编程时测试通过的爽图：

![爽](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/0.png)

再附一张测试通不过的爽图：

![爽](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/2.png)

这薛定谔程序我也是醉了。

我非常苦恼，而且看那个Process finished with exit code -多少多少多少，就感觉它多半又是delete了不该delete的东西——野指针。

## 不是很重要的前提知识

+ [知乎：矩阵乘法加速斐波那契数列求值原理](https://www.zhihu.com/question/23582123)
+ [博客园：矩阵乘法加速斐波那契数列求值原理](http://www.cnblogs.com/xudong-bupt/archive/2013/03/19/2966954.html)

## 内存池是啥

内存池就是一个内存管理器，负责分配内存（就是通过它来管理矩阵对象的new和delete）。

## 内存池怎么解决

这样就不需要在一些函数中间delete（由于你可以把一个矩阵对象赋值给多个指针，delete其中一个之后，
剩下的都变成了野指针，你再delete一次就GG了），而是在整个流程结束之后一次性完全delete，解决了内存分配的问题。

## 内存池如何设计

我第一反应是：

```cpp
class Matrix22Pool {
public:
  Matrix22Pool() {
    v = *new std::vector<Matrix22>;
  }

  ~Matrix22Pool() {
    delete v;
  }

  auto create(jlong a, jlong b, jlong c, jlong d) -> jsize {
    return store(*new Matrix22(a, b, c, d));
  }

  auto store(Matrix22& m) -> jsize {
    v.push_back(ret);
    return v.size() - 1;
  }

  auto get(jsize n) -> Matrix22 {
    return v[n];
  }
private:
  std::vector<Matrix22> v;
}
```

然后我发现这样实在是太垃圾了！

原因：

0. 使用了STL，作为一个拿MinGW编译C++端的垃圾，实在是受不了MinGW编译STL之后生成文件的大小。
0. 万一分配了大量没用到的内存呢
0. 万一size()的实现是O(n)的呢（我问了指针，他说没有（因为STL要求O(1)），于是这条可以无视）
0. ~~其实没那么多万一，反正垃圾MinGW，clang才是正义，不是很懂~~
0. ~~就是不想用STL就是不想用STL~~

**数组才是正义！**

## 拿数组怎么搞

数组的大小是在创建的时候确定的，所以我们需要在一开始就确定我们大概需要多少个矩阵。我使用了一个奇技淫巧来查看矩阵的数量。

由于这只是临时性的一个测试，我们就可以随便整点糟糕的东西，比如使用全局变量。我们先整个全局变量，并且每生成一个Matrix22对象就让它自增：

```cpp
int num = 0;

algo4j_matrix::Matrix22::~Matrix22() {
}

algo4j_matrix::Matrix22::Matrix22() {
  ++num;
}
```

然后在矩阵斐波那契的函数里面加上num的初始化和输出：

```cpp
auto algo4j_matrix::fib_matrix(jlong n, jlong mod) -> jlong {
  num = 0;
  auto base = *new Matrix22(1, 1, 1, 0);
  auto ans = pow(base, n - 1, mod);
  auto ret = ans.a[0][0];
  delete &base;
  printf("%d\n", num);
  return ret;
}
```

别忘了临时加上这个：

```cpp
#include "stdio.h"
```

然后编译一下，嗯没毛病。然后跑测试。我使用的是Run test with Coverage，使用的IntelliJ的控制台：

```
1200 test cases: test passed
3
4
4
5
5
6
5
6
6
... 此处省略巨量输出
15
16
15
16
16
17
15
16
16
17
16
17
17
18
15
16

Process finished with exit code 0
```

我擦嘞？好像它没按顺序输出，多半是控制台IO的问题（IntelliJ的控制台有些时候有点奇怪），我试试把测试输出到文件：

```cpp

auto algo4j_matrix::fib_matrix(jlong n, jlong mod) -> jlong {
  num = 0;
  auto base = *new Matrix22(1, 1, 1, 0);
  auto ans = pow(base, n - 1, mod);
  auto ret = ans.a[0][0];
  delete &base;
  auto fp = fopen("out.txt", "a");
  fprintf(fp, "%lli:%i\n", n, num);
  fclose(fp);
  return ret;
}
```

然后发现似乎不是IO的问题：

```
3:3
4:4
5:4
6:5
7:5
8:6
9:5
10:6
11:6
.... 此处省略
653:14
654:15
655:15
656:16
657:13
658:14
659:14
660:15
661:14
662:15
663:15
664:16
665:14
666:15
667:15
668:16
669:15
670:16
671:16
672:17
673:13
674:14
675:14
```

~~啊！~~这真是蠢爆了，才十几个矩阵对象！

于是我们来使用内存池吧！

使用内存池才是正道啊（雾）。

好，根据观察得出，所需矩阵的数量大约和斐波那契数项数成对数关系。我觉得这个关系可以作为深度学习探究的题材（滑稽）

那么具体是多少呢？可以明显地看到数据有不稳定的波动，那么为了确保安全，应该是先求n的对数，然后加上一个与n有关的常数。至于参数是多少呢？

我也不知道。。。于是我随手撸了个工具来生成这个东西对应的函数图像（话说我才发现垃圾Windows的\r\n问题是多么坑，辣鸡Windows）。

### 矩阵乘法斐波那契产生的矩阵数量，和项数的关系

先上这个工具的代码，看不懂Kotlin的可以跳过（突然有种自己在做大数据研究的感觉）：

```kotlin
fun main(args: Array<String>) {
  val txt = File("out.txt")
      .readText()
      .split("\r\n")
      .map { it.split(":").last() }
  JFrame("Fib Matrix Viewer").run {
    setSize(700, 500)
    layout = BorderLayout()
    defaultCloseOperation = WindowConstants.EXIT_ON_CLOSE
    add(object : JPanel() {
      override fun paintComponent(g: Graphics?) {
        super.paintComponent(g)
        txt.forEachIndexed { x, y ->
          try {
            g?.drawRect(x, this@run.height - y.toInt() * 20, 1, 1)
          } catch (e: Exception) { }
        }
      }
    }, BorderLayout.CENTER)
    isVisible = true
  }
}

```

我解释一下，就是读取文件里面的每个数对，然后以前一个数为横坐标（最左边是0，往右是正半轴），后一个数为纵坐标（最下面是0，往上是正半轴）

然后显示出来是这个效果：

![Fib Viewer](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/1.png)

It's beautiful, it's beautiful, it's true~

讲道理我也不知道为什么它会长成这个神奇的样子，这或许是一个新的关于斐波那契数列的奥秘吧。

注意，这里为了视觉效果，将纵坐标按比例放大了20倍。

于是我们可以再绘制一个对数函数的派生函数的图像。我们先造一个这样的函数：

```kotlin
fun log(x: Int) = (Math.log(x.toDouble()) * 10).toInt()
```

然后把上面的绘图代码改成这样：

```kotlin
fun main(args: Array<String>) {
  val txt = File("out.txt")
      .readText()
      .split("\r\n")
      .map { it.split(":").last() }
  JFrame("Fib Matrix Viewer").run a@ {
    setSize(700, 500)
    layout = BorderLayout()
    defaultCloseOperation = WindowConstants.EXIT_ON_CLOSE
    add(object : JPanel() {
      override fun paintComponent(g: Graphics?) {
        super.paintComponent(g)
        txt.forEachIndexed { x, y ->
          try {
            g?.run {
              color = Color.BLACK
              drawRect(x, this@a.height - y.toInt() * 20, 1, 1)
              color = Color.BLUE
              drawRect(x, this@a.height - log(x), 1, 1)
            }
          } catch (e: Exception) {
          }
        }
      }
    }, BorderLayout.CENTER)
    isVisible = true
  }
}
```

看图，不忍直视：

![不忍直视](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/3.png)

于是我们可以再改改fun函数。我就不贴中间过程了，最终结果是（中间为了视觉效果，我对放大倍数进行多次调整）：

![比较好看](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/4.png)

这时的log函数：

```kotlin
fun log(x: Int) = (Math.log(x.toDouble() + 1) * 3).toInt() + 4
```

这时，绘制代码我已经改成了（为了防止肉眼不可见的边界情况）：

```kotlin
color = Color.BLACK
val _1_ = y.toInt() * 5
val _2_ = log(x) * 5
drawRect(x, this@a.height - _1_, 1, 1)
color = Color.BLUE
drawRect(x, this@a.height - _2_, 1, 1)
if (_1_ > _2_) println("Fail at $x")
```

命令行什么也没有输出：

```

Process finished with exit code 0
```

完美！于是我们可以把这个公式用进刚才的内存池里了！

首先写一个内存池，这里就不贴代码了，直接在[algo4j的GitHub仓库](https://github.com/ice1000/algo4j/blob/master/jni/global/matrix.cpp)就可以看到啦（链接给的是cpp文件，请参阅h文件哦）。

来看看这次编译之后跑出来的结果吧（我把之前的1200改成5000了）：

![All Hail Memory Pool](https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/4/5.png)

All Hail Memory Pool!

注意看运行时间！缩短了很多很多很多很多！

以上，是今天晚上我自嗨的全过程。

