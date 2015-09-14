---
layout: post
title: "Comparison method violates its general contract"
date: 2015-09-14 16:19:03 +0800
comments: true
categories: java sort exception
---

排序是算法学习中最重要的算法之一，不过使用Java等面向对象的语言之后很少再自己实现排序了，只有在面试的时候才用的上了，作为对基础知识的考查。曾经在面试百度的时候被要求立即写一个归并排序并编译通过运行正确，还好顺利过关。

Java中直接使用Collections.sort(List<T> list)方法或Arrays.sort(Object[] array)来实现排序。对于数组中的元素如何比较，有两种方式：一种是在类中重写compareTo()方法，一种是在调用sort方法的时候传入一个Comparator<T>的对象。若没有实现，会抛出ClassCastException错误。比较基础，这里不展开了。

Java中的排序并非使用基础算法中的某一种，因为每一种都有退化的危险或排序算法非稳定（详细请自行搜索或查阅算法书），Java的排序算法是稳定的。Java中是用TimSort算法进行排序。TimSort 是一个归并排序做了大量优化的版本。对归并排序排在已经反向排好序的输入时表现O(n^2)的特点做了特别优化。对已经正向排好序的输入减少回溯。对两种情况混合（一会升序，一会降序）的输入处理比较好。对于算法的详细介绍，可以参考[百度百科](http://baike.baidu.com/view/7780831.htm)。

最近遇到一个排序的问题，在某些情况下，排序方法会抛出llegalArgumentException：Comparison method violates its general contract。因为代码比较简单，而且不容易复现，起初没有受到重视。后来具体研究了一下，才明白是什么一回事：
* 首先，这是JDK6到JDK7的一个兼容性问题，在JDK6中没有问题，但是在JDK7中，排序算法做了修改，必须要满足比较的[约束条件](http://docs.oracle.com/javase/7/docs/api/java/util/Comparator.html#compare(T,%20T\))，不然则会抛出此异常。
* 在我的代码中，某种极端情况下会使A，B，C三个元素满足A>B，A=C，B=C的条件，这违反了约束条件。另一个会违反约束条件的例子为 return x > y ? 1 : -1;相信您可以看明白。
所以结论是，当抛出这个异常的时候，仔细检查Comparator中的代码，保证不能违反约束条件。
