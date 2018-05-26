[返回首页](/index.html)

本系列，是参照“奔跑中的IT男”的“runtime从入门到精通”系列写就的。本系列的脉络和原系列一致，只是在语言描述和例子上有所不同。点击[这里](https://blog.csdn.net/coyote1994/article/details/52355026)查看原文。

#### 什么是runtime

runtime是一套纯C语言的API，是OC的底层。所有的OC代码都要由编译器转换为runtime的C语言代码，所以runtime算是OC的幕后工作者。例如：

```
// OC
[[NSString alloc] init];

// runtime
objc_msgSend(objc_msgSend("NSString", "alloc"), "init");
```

#### runtime可以用来做什么

1. 在程序运行过程中，动态创建一个类（比如KVO的底层实现）。
2. 在程序运行过程中，动态地为某个类添加属性和方法。可以用于封装框架，这是使用runtime机制的主要运用方向。
3. 遍历一个类中所有的成员变量、属性和方法。（比如字典转换为模型，利用runtime遍历模型对象的所有属性，根据属性名从字典中取出对应的值，设置在模型的属性上。）

#### runtime相关头文件和函数

头文件有：
```
<objc/runtime.h>
<objc/message.h>
```

相关的函数的举例（还有很多）：
```
objc\_msgSend: 给对象发送消息
class\_copyMethodList: 遍历某个类所有的方法
class\_copyIvarList: 遍历某个类所有的成员变量
```

#### 必备知识

- Ivar：成员变量
- Method：成员方法

在原系列中，runtime官方文档的翻译是作为一部分存在的。本系列没有这章，可直接点击[这里](https://blog.csdn.net/coyote1994/article/details/52441513)查看。建议没有阅读过的先阅读然后再看本系列后面的内容。
