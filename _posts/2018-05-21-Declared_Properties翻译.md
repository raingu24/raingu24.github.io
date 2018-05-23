[返回首页](/index.html)

## 声明的属性

当编译器遇到属性声明的时候（参见The Objective-C Programming Language中的Declared Properties），它产生与封闭类、类别或协议相关的描述性元数据。你可以使用支持通过类或协议的名字来查找属性的函数来访问该元数据，获取@encode()字符串形式的属性类型，并且以C字符串数组的形式拷贝属性的属性列表。每个类和协议都有声明的属性列表。

### 属性类型和函数

Property结构体为属性描述符定义了一个不透明的句柄。

```
typedef struct objc_property *Property;
```

你可以使用函数class\_copyPropertyList和protocol\_copyPropertyList来检索与类和协议各自相关的属性数组：

```
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```

例如，给定下面的类声明：

```
@interface Lender: NSObject {
    float alone;
}
@property float alone;
@end
```

你可以使用下面的方式得到属性列表：

```
id LenderClass = objc_getClass("Lender");
unsigned int outCount;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
```

你可以使用property\_getName函数来发现属性的名字：

```
const char *property_getName(objc_property_t property);
```

你可以使用函数class\_getProperty 和 protocol\_getProperty结合类和属性的各自给定的名字来得到属性的引用：
```
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```

你可以使用property\_getAttributes函数来发现属性的名字和@encode类型字符串。更详细的关于编码类型字符串的内容，参阅Type Encodings;该字符串的更详细内容，参阅Property Type String和Property Attribute Description Examples。

```
const char *property_getAttributes(objc_property_t property)
```

把这些放在一起，你可以使用下面的代码来打印和一个类相关的所有属性列表：
```
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n",property_getName(property), property_getAttributes(property));
}
```

### 属性类型字符串

你可以使用property\_getAttributes函数来发现名字、属性的@encode类型字符串、以及该属性的其它属性。

该字符串从T开始，后面跟着@encode类型和都好，用V结束，后面跟着实例变量的名字。在它们之间，属性（attributes）用以下描述符制定，用逗号隔开：

**Table 7-1** 声明的属性类型编码

|Code|Meaning|
|-|-|
|R|只读属性（readonly）|
|C|该属性是上次分配的值的副本（copy）|
|&|该属性是上次分配的值的引用（retain）|
|N|该属性是非原子（nonatomic）|
|G<Name>|该属性定义一个自定义的getter选择器名字。该名字跟在G后面（例如，GcustmGetter）。|
|S<Name>|该属性定义一个自定义的setter选择器名字。该名字跟在S后面（例如，ScustomSetter）。|
|D|动态属性（@dynamic）|
|W|弱引用属性（\__weak）|
|P|该属性有资格进行垃圾收集|
|t<encoding>|使用旧式编码制定类型|

例子，可参阅下面。

### Property Attribute描述例子

给定这些定义：

```
enum FooManChu {
    FOO,
    Man,
    CHU
};

struct YorkshireTeaStruct {
    int pot;
    char lady;
};

typedef struct YorkshireTeaStruct YorkshireTeaStructType;
union MoneyUnion {
    float alone;
    double down;
};
```

下面的表格显示了示例属性声明和通过property\_getAttributes返回的相应字符串：

[详见](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html)
