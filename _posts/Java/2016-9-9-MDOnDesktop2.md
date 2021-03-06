---
layout: post
title: 在JavaFX中进行Material Design：第二章
category: Java
tags: Java
keywords: Java,JavaFX,MaterialDesign
description: Use Material Design on desktop 2
---

我现在还记得，上一篇是我在暑假前写的，当时我以为暑假会填坑。。。。没错，现在我来填坑了！

现在就让我们开心地在JavaFX中使用Material Design控件吧！

## 依赖

上一篇已经讲过了。

## 准备

1. 阅读上一篇教程。

## 开始

首先打开之前我们创建好的项目。由于我已经删了所以让我重新弄一个吧。（喂）首先按照上一次教程的内容创建JavaFX工程，然后使用Scene Builder打开fxml文件。配置好 jfoenix.jar 。

拖一个按钮上去。我们要开始了。

## 对上次内容的一些补充

我上次写的博客中有一个问题没有mention到，就是fxml在IDEA里面报错的问题（<? impoort com.jfoenix.controls.* ?>）。这时你需要进行一次这样的配置：

<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/2.png" align="center">

然后就没问题辣。

## 让它变得好看

+ 首先确保你掌握CSS的语法。我认为，学习只需要一分钟，甚至根本不需要学习时间。然后你需要知道rgb是什么意思。

选中按钮，看到右边的属性编辑器。这是一个GUI的属性编辑器，零基础学习。比如，先把按钮设置为Raised。等会再告诉你们是什么意思。

<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/0.png" align="center">

然后我们继续编辑，设置一些奇怪的属性。下面是一些属性的介绍，别的读者可以自行研究。这几个是常用的。

属性|作用
:---|---:
Button Type|按钮类型
Rippler Fill|按下去的时候波纹特效的波纹颜色
Text|按钮文本
Font|按钮文本字体
Text Fill|文本颜色
Text Alignment|文本对齐方式
UnderLine|是否有下划线

<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/1.png" align="center">

然后我们保存，跑一下程序。可以看到，按钮的样式改变了。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/3.png" align="center">

这里特别要提一下这两个属性，非常好玩非常常用的。

Specific栏下面的Default Button，如果勾选，可以看到按钮颜色变深了。并且在选中这个属性之后，按钮将会变成默认按钮，也就是说当GUI窗口没有任何操作，而用户按下回车时，这个按钮将被按下。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/4.png" align="center">

然后是这个属性，Node栏下面的Effect，顾名思义就是特效。像我这种hentai就选了个反射（reflection），可以看到，按钮有了倒影。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/5.png" align="center">

地板真光滑，不禁脑补了下很多MMD里面的极其光滑的地板。咳咳。运行之后可以看到更牛逼的动画，按下按钮之后下面的倒影会往下滑一些。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/6.png" align="center">

然后我们可以开始写CSS了。先在工程里面创建一个CSS文件，写个设置背景颜色和旋转角度的CSS（这个不能在可视化编辑器里面设置，简直坑死我了）：

```css
.button {
    -fx-background-color: rgb(43,43,43);
    -fx-rotate: 5;
}
```

然后找到这里，JavaFX CSS项下面的Style Sheet，找到CSS文件的路径。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/7.png" align="center">

选中刚才编辑好的CSS，你可以看到它发生了变化。和我们之前想的一模一样。跑一下试试。
<img src="https://coding.net/u/ice1000/p/Images/git/raw/master/blog-img/old/java/javafx2/8.png" align="center">

还有很多属性可以设置，具体请使用快捷键**Ctrl+6**查看CSS Analyzer，里面列出了所有可以设置的属性。

好了，这里是关于颜值的设定。下一篇讲事件处理。





