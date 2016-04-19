# JThread

## 介绍
在许多项目中，我们会经常使用到线程．为了在unix和windows平台下使用一样的代码，我决定在这些平台上将已经存在的线程函数封装成一个简单的类．

JThread 包非常的简单：目前，它只包含三个类：分别是JThread, JMutex和JMutexAutoLock,从它们的名称上可以想到：JThread表示一个线程，JMutex表示一个互斥体，线程类仅仅包含很基本的函数，例如：开始或结束一个线程．

## 使用
下面是JThread，JMutex和JMutexAutoLock类的描述．注意函数的返回类型int总是返回０及大于０的值表示成功，负值表示出现了问题．

### JMutex

### JMutexAutoLock

### JThread

___
*[原文文档](http://research.edm.uhasselt.be/jori/jthread/manual.pdf)，翻译出入的地方，欢迎提交pull request或　mjrao@foxmail.com　联系我！