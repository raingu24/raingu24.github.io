[返回首页](/index.html)

本系列，是参照“奔跑中的IT男”的“runtime从入门到精通”系列写就的。本系列的脉络和原系列一致，只是在语言描述和例子上有所不同。点击[这里](https://blog.csdn.net/coyote1994/article/details/52452346)查看原文。

这里所谓的术语，也就是数据类型。本文是介绍runtime中的一些重要的数据类型的结构的。

#### SEL

```
// 它是selector在objc中的表示。它的声明是：
typedef struct objc_selector *SEL;
```

#### id

```
// id是指向某个类实例的指针, 声明如下：
typedef struct objc_object *id;
struct objc_object {
	Class isa;
};
```

#### Class

```
typedef struct objc_class *Class;

struct objc_class {
	Class isa					OBJC_ISA_AVALIABILITY;
#if !__OBJC2__
	Class super_class			OBJC2_UNAVAILABLE;
	const char *name			OBJC2_UNAVAILABLE;
	long version				OBJC2_UNAVAILABLE;
	long info					OBJC2_UNAVAILABLE;
	long instance_size			OBJC2_UNAVAILABLE;
	struct objc_ivar_list *ivars			OBJC2_UNAVAILABLE;
	struct objc_method_list **methodLists	OBJC2_UNAVAILABLE;
	struct objc_cache *cache				OBJC2_UNAVAILABLE;
	struct objc_protocol_list *protocols	OBJC2_UNAVAILABLE;
#endif
}OBJC2_UNAVAILABLE;
```

OBJC2\_UNAVAILABLE的意思是在OBJC2.0之后就不能使用了，从源码的预编译指令if中可以看出这点。源码的注释中，明确表示，在objc2开始的版本，要使用Class来代替objc\_class的使用。

但也可以从中看出，一个运行时类关联了它的父指针、类名、成员变量列表、方法列表、附属的协议列表以及缓存等内容。其中成员变量列表和方法列表的数据结构如下：

```
// 成员变量列表
struct ojbc_ivar_list {
	int ivar_count					OBJC2_UNAVAILABLE;
#ifdef	__LP64__
	int space						OBJC2_UNAVAILABLE;
#endif
	/*可变长度结构*/
	struct objc_ivar ivar_list[1]	OBJC2_UNAVAILABLE;
}

// 方法列表
struct objc_method_list {
	struct objc_method_list *obsolete		OBJC2_UNAVAILABLE;
	int method_count						OBJC2_UNAVAILABLE;
#ifdef __LP64__			// 64位操作系统
	int space								OBJC2_UNAVAILABLE;
#endif

	/*可变长度结构*/ 
	struct objc_method method_list[1]		OBJC2_UNAVAILABLE;
}
```

从上面两段源码可以看出，我们可以动态修改\*methodList的值来添加成员方法。这也是Category实现的原理，同样解释了Category不能添加属性的原因。这可以参考美团技术团队的文章：[深入理解Objecive-C：Catetory](https://tech.meituan.com/DiveIntoCategory.html)。

#### 帮助了解的内容

这是我在另一份资料上看到的内容，经过总结后写的例子和注释。可以清楚的看到Class、id以及isa指针的详细解读。

```
/*****************重点 ！！！*****************
         
         1. 编译器在解析OC源码的类时，会生成运行时系统的Class数据结构。Class数据结构会指向带objc_class标识符的不透明数据类型的指针。
         2. 编译器在解析OC源码的对象时，会生成运行时系统的数据结构，该结构指向带objc_object标识符的不同命数据类型的指针。
         3. 编译器解析对象消息时，会生成调用运行时系统库中的objc_megSend()代码。
         4. 从上可以看出，OC的源码在编译的时候都会首先被转化为运行时系统的函数或结构体。这些函数和结构体的实际实现代码，都会在运行程序时最终决定。
         
         *****************************************/
        
        // 为测试类创建几个实例并显示其数据
        TestClass1 *tc1A = [[TestClass1 alloc] init];
        tc1A->myInt = 0xa5a5a5a5;   // 为实例变量赋值
        TestClass1 *tc1B = [[TestClass1 alloc] init];
        tc1B->myInt = 0xc3c3c3c3;
        long tc1Size = class_getInstanceSize([TestClass1 class]);   // 运行时系统库函数，获取类实例的尺寸。
        NSData *obj1Data = [NSData dataWithBytes:(__bridge const void *)(tc1A) length:tc1Size];
        NSData *obj2Data = [NSData dataWithBytes:(__bridge const void *)(tc1B) length:tc1Size];
        NSLog(@"TestClass object tc1A contains %@", obj1Data);
        NSLog(@"TestClass object tc1B contains %@", obj2Data);
        NSLog(@"TestClass memory address = %p", [TestClass1 class]);
        
        /*结果分析
         TestClass object tc1A contains <d1110000 01801d00 a5a5a5a5 00000000>
         TestClass object tc1B contains <d1110000 01801d00 c3c3c3c3 00000000>
         TestClass memory address = 0x1000011d0
         
         d1110000 01801d00 是isa的值，指向对象的类，所以两个对象的该值是一样的，它们都是TestClass1类的。
         a5a5a5a5 00000000 和 c3c3c3c3 00000000是这两个对象的值。是我们设定的。
         TestClass1这个类的内存地址是0x1000011d0，它其实和d1110000 01801d00是一致的，只不过它是逆序书写地址的缘故，这是Mac的规则问题。
         */
        
        // 获取并显示 TestClass1 类实例的数据
        id testClz = objc_getClass("TestClass1"); //得到类，其实就是TestClass1类，类也是可以被看作是一个对象。
        long tcSize = class_getInstanceSize([testClz class]);
        NSData *tcData = [NSData dataWithBytes:(__bridge const void  *)(testClz) length:tcSize];
        NSLog(@"TestClass1 class contains: %@", tcData);
        NSLog(@"TestClass1 superclass memory address = %p", [testClz superclass]);
        
        /*结果分析
          TestClass1 class contains: <f9110000 01801d00 40c110a5 ff7f0000>
          TestClass1 superclass memory address = 0x7fffa510c140
         
         f9110000 01801d00 是类的isa指针，它指向的是本类的元类，所以和本类对象的isa的数值不同。
         40c110a5 ff7f0000 是超类的指针
         */
        
        /************** 重点！！！！ ****************
          关于isa指针的详解：
         
         类的结构：
         struct objc_class
         {
             struct objc_class *isa;             // 指向metaClass
             struct objc_class *super_class;     // 指向其父类
             const char *name;                   // 类名
             long version;                       // 类的版本信息，默认0
             long instance_size;                 // 类的实例变量大小
         long info;             // 一些标识信息（用掩码表示），区分普通类还是元类
             struct objc_ivar_list *ivars;       // 所有成员变量的地址列表
             struct objc_method_list *methodLists; // 所有方法的地址列表
             struct objc_cache *cache;           // 指向最近使用的方法的指针，提高效率
             sturct objc_protocol_list *protocols;  // 该类遵守的协议的列表
         }
         
         对象id的结构：
         typedef struct objc_class *Class
         typedef sturct objc_object
         {
             Class isa;
         } *id;
         
         从以上结构体观察得知，无论是类还是对象，都有一个objc_class指针，isa。所以本质上类也是对象。类的对象成为类对象，类的实例的对象称为实例对象。
         
         那么isa和super_class指针分别都指向谁？
         
         1. 类的实例对象的isa指向该类；该类的isa指向该类的元类。
         2. 类的super_class指向其父类，如果该类为根类，则super_class的值为null。
         3. 元类的isa指向根元类；如果该元类是根类，则指向自身。
         4. 元类的super_class指向父元类，如果该元类是根元类，则指向该根元类对应的根类。
         
         那么类（class）和元类（meta class）有什么区别？
         
         1. class存储的是普通成员变量和普通成员方法（-号开头）；meta class存储的是类成员变量和类成员方法（+号开头）。
         2. 在响应者链中，它们会沿着各自的继承关系逆向寻找变量和方法。
         
         
         *********************************************/
        
        // 获取并显示元类数据
        id metaClass = objc_getMetaClass("TestClass1");
        long mclzSize = class_getInstanceSize([metaClass class]);
        NSData *mclzData = [NSData dataWithBytes:(__bridge const void *)(metaClass) length:mclzSize];   //  这里使用__bridge桥接，是因为ARC禁止从OC对象到(void *)类型的直接转换。
        NSLog(@"TestClass1 metaclass contains %@", mclzData);
        class_isMetaClass(metaClass) ? NSLog(@"Class %s is a metaclass", class_getName(metaClass)) : NSLog(@"Class %s is not a metaclass", class_getName(metaClass));
        
        /*结果分析
         TestClass1 metaclass contains <f1c010a5 ffff1d00 f0c010a5 ff7f0000 00007000 01000000 07000000 01000000 e0ad1802 01000000>
         Runspector[30713:1052772] Class TestClass1 is a metaclass
         
         元类包含了isa指针，superclass指针和附加信息。因为这个类没有自定义的类方法，所以isa指向的是根类，也就是NSObject类。所以superclass指针和isa指针的值相同。
         */

```

#### Method

```
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name                           OBJC2_UNAVAILABLE;
    char *method_types                        OBJC2_UNAVAILABLE;
    IMP method_imp                            OBJC2_UNAVAILABLE;
}
```

- 方法名类型为SEL，这点很有意思，表示直接用方法名表示了方法选择器，而不是一个字符串。
- 方法类型，是一个字符串，存储方法的参数类型和返回值类型。
- method\_imp指向了方法的实现，本质是一个函数指针。

#### Ivar

Ivar表示成员变量。

```
typedef struct objc_ivar *Ivar;
struct objc_ivar {
    char *ivar_name                                          OBJC2_UNAVAILABLE;
    char *ivar_type                                          OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}
```
其中的ivar\_offset是基地址偏移字节。

#### IMP

它是一个函数指针，指向的是函数的实现方法。一个确定的方法也只有唯一的一组id和SEL参数。

```
typedef id (*IMP)(id, SEL, ...);
```

#### Cache

Cache为方法调用的性能进行优化。每当实例对象接收到一个消息时，它不会直接在 isa 指针指向的类的方法列表中遍历查找能够响应的方法，因为每次都要查找效率太低了，而是优先在 Cache 中查找。

Runtime 系统会把被调用的方法存到 Cache 中，如果一个方法被调用，那么它有可能今后还会被调用，下次查找的时候就会效率更高。就像计算机组成原理中 CPU 绕过主存先访问 Cache 一样。

```
typedef struct objc_cache *Cache

struct objc_cache {
	unsigned int mask /*total = mask + 1*/		OBJC2_UNAVALIABLE;
	unsigned int occupied						OBJC2_UNAVALIABLE;
	Method buckets[i]							OBJC2_UNAVALIABLE;
}
```
