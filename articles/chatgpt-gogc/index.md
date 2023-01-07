---
title: "我和ChatGPT掰扯GoGC这件事"
date: "2022-12-08"
categories:
  - Golang

toc: true
---

最近`ChatGPT`很火，我也在尝鲜，探求它能解决那些问题，以及它的局限性。

我发现它在小段特定功能代码的生成是很出色的，但是它不能预测，不能对复杂问题有效把握重点。甚至有时候像小学生一样嘴犟。


<!--more-->


以下是我们的对话：

![在这里插入图片描述](https://img-blog.csdnimg.cn/c9939ac859de4104b5cde66409524e53.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/87eb9aa3ce184666bf6aa7ad9f3dea36.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6434add08e0543eb9d2dc6db8a54d09d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/478770d13c03494aa671b98e79f8639f.png)


事实上我也搞错了，go在1.5版本就使用三色标记法了，而不是1.15版本。按理说它学习的东西不至于如此过时，况且现在全网都是说三色标记法。

你去纠正它，我理解它不屈服，防止被带坏，但是它仍然坚持自己的观点，是不是有些过于自信了呢？

最后说一下，现在go的GC是使用三色标记法+混合写屏障。
