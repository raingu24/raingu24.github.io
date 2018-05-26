[返回首页](/index.html)

本系列，是参照“奔跑中的IT男”的“runtime从入门到精通”系列写就的。本系列的脉络和原系列一致，只是在语言描述和例子上有所不同。点击[这里](https://blog.csdn.net/coyote1994/article/details/52451309)查看原文。

#### 温故

**RunTime，就是系统在运行的时候的一些机制，其中最主要的是消息机制。**
**RunTime基本是用C和汇编写的，从而有了动态系统的高效。**

#### OC和C在函数调用上的对比

##### C语言
1. 函数的调用在编译的时候就决定调用哪个函数，编译完成之后直接顺序执行，无任何二义性。
2. C语言在编译阶段调用未实现的函数就会报错。

##### OC
1. 函数的调用成为消息发送，属于动态调用过程。
2. 在编译的时候并不能决定真正调用哪个函数（事实证明，在编译阶段，OC可以调用任何函数，即使这个函数并未实现，只要声明过就不会报错）。

#### 三种层面上与runtime系统进行交互

##### 1. 通过OC源码

多数情况我们只需要编写OC代码即可。runtime系统自动在幕后搞定一切。编译器会将OC代码转换成运行时代码，在运行时确定数据结构和函数。

##### 2. 通过Foundation框架的NSObject类定义的方法

有一些NSObject的方法可以从runtime系统中获取信息，允许对象进行自我检查，例如：
- class方法返回对象的类；
- isKindOfClass:和isMemberOfClass:方法检查对象是否存在于指定的类的继承体系中；
- respondsToSelector:检查对象能否响应指定的消息；
- conformsToProtocol:检查对象是否实现了指定协议类的方法；
- methodForSelector:返回指定方法实现的地址。

##### 3. 通过runtime库函数的直接调用

只要我们引入头文件objc/runtime.h，就可以直接使用rungtime库的公共接口。下面介绍一些常用的runtime接口。

**3.1 获取类**
```
Class PersonClass = objc_getClass([Person class]);
```

**3.2 SEL是selector在objc中的表示**
```
SEL oriSEL = @selector(test);
```

**3.3 获取类方法**
```
Method class_getClassMethod(Class cls, SEL name);
```

**3.4 获取实例方法**
```
Method class_getInstanceMethod(Class cls, SEL name); 
```

**3.5 添加方法**
```
BOOL addSucc = class_addMethod(Class cls, SEL sel, method_getImplemetation(Method), meethod_getTypeEncoding(Method));
```

**3.6 替换原方法实现**
```
class_replaceMethod(toolClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
```

**3.7 交换两个方法**
```
method_exchangeImplemetation(oriMethod, cusMethod);
```

**3.8 获取一个类的属性列表(返回值是一个数组)**
```
objc_property_t *propertyList = class_copyPropertyList([self class], &count);
```

**3.9 获取一个类的方法列表**
```
Method *methodList = class_copyMethodList([self class], &count);
```

**3.10 获取一个类的成员变量列表**
```
Ivar *ivarList = class_copyIvarList([self class], &count);
```

**3.11 获取成员变量的名字**
```
const char *ivar_getName(Ivar v)
```

**3.12 获取成员变量的类型**
```
const char *ivar_getTypeEncoding(Ivar v)
```

**3.13 获取一个类的协议列表**
```
__unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
```

**3.14 set方法**
```
// 作用：将值value和对象objcet关联起来（将值value存储到对象object中），实际就是给object增加属性。
// 参数object：给哪个对象设置属性
// 参数key：属性的名字
// 参数value：属性的值
// 参数policy：存储策略（assign、copy、retain、strong）
void objc_setAsscciatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)

```

**3.15 利用参数key将对象objcet中存储的对应值取出来**
```
id objc_getAssociatedObject(id object, const void *key)
```


