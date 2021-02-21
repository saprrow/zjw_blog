---
title: tqdm:让你的进度条不再单调
date: 2021-02-04 11:29:05
tags: "trciks"
categories: "python"
---

  在看PVN3D的代码时,看到了tqdm这个库,当时不知道用来干嘛的,查了一下后,
发现有点好玩.
  其实就是一个修饰进度条的库.
  比如你想让程序等待1秒,用tqdm后效果就会炫酷得多.
  <!--more-->

  下面可以看到用sleep延时2秒的一个例程,加了tqdm后会舒服很多

  ![只用sleep](/img/tqdm.png)

  ![加tqdm后](/img/tqdm1.png)

  tqdm还有一个方法是trange,这个相当于tqdm(range())的一个实例,如图:
  ![加tqdm后](/img/tqdm2.png)



