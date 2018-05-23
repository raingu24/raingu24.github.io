[返回首页](/index.html)

## 类型编码

为了访问运行时系统，编译器把每个方法的返回值和参数类型编码为字符串，并把该字符串和方法选择器（method selector）联系起来。它所使用的编码方案在别的上下文中也是可用的，所以使用@encode()编译器指令公开使用。当给定类型规范时，@encode()返回该类型的字符串编码。该类型可以是基本类型，诸如int、指针、结构体或者联合体，或者是任何类型的类名。事实上，它可以被用作C的sizeof()操作符的参数。

```
char *buf1 = @encode(int **);
char *buf2 = @encode(struct key);
char *buf3 = @encode(Rectangle);
```

下面的表格列出了类型编码。注意，它们中很多和你在编码对象的时候的代码重叠，这样做的目的是为了归档或分发。但是这里列出了你不能在编写编码器时使用的代码，以及当编写不能被@encode()生成的编码器的时候使用的代码。（参阅Foundation Framework reference 中的NSCoder class specification来获取更多的关于为归档和发布编码对象的信息。）

**Table 6-1** Objcetive-C类型编码

|Code|Meaning|
|:-:|:-:|
|c|一个char|
|i|一个int|
|s|一个short|
|l|一个long。在64位程序中，l被处理为32位。|
|q|一个long long|
|C|一个unsigned char|
|I|一个unsigned int|
|S|一个unsigned short|
|L|一个unsigned long|
|Q|一个unsigned long long|
|f|一个float|
|d|一个double|
|B|一个C++ bool或者一个C99 \_Bool|
|v|一个void|
|\*|一个字符串（char \*)|
|@|一个对象（无论是静态类型还是id类型）|
|#|一个类对象（Class）|
|:|一个方法选择器（SEL）|
|[array type]|一个数组（array）|
|{name=type...}|一个结构体（structure）|
|(name=type...)|一个联合体（union）|
|bnum|一个num位的位域（a bit field of num bits）|
|^type|一个指针类型|
|?|一个unknown类型（除其它情况外，该代码用于函数指针）|

> **重要：**OC不支持long double类型。@encode(long double)返回d，和double的编码一样。

数组（array）的类型被方括号包裹；数组中的数字元素在括号被打开的时候被立即指定，在数组类型之前。例如，一个有12个指向float类型的指针的数组，会被编码为 [12^f]。

结构体（structures）使用花括号来指定，联合体（union）使用小括号指定。首先列出结构标签，然后是等号和顺序的结构字段的代码。例如，结构体：

```
typedef struct example {
    id anObject;
    char *aString;
    int anInt;
}Example;
```

编码后是这个样子的：

```
{example=@*i}
```
 
无论是使用定义的名字（Example）还是结构体标记（example）在通过@encode()编码后，都会有相同的编码结果。

```
^{example=@*i}
```

但是，另一个间接的级别会删除内部类型规格：

```
^^{example}
```

对象的处理和结构体一样。例如，把NSObject类名传递给@encode()会得到编码：

```
{NSObject=#}
```

该NSObject类声明的只是Class类型的一个实例变量（isa）。

注意，尽管@encode()指令不返回它们，当它们在协议中被用于声明方法，运行时系统使用Table 6-2中列明的用于类型限定符的额外编码。

**Table 6-2** Objective-C method encodings

|Code|Meaning|
|:-:|:-:|
|r|const|
|n|in|
|N|inout|
|o|out|
|O|bycopy|
|R|byref|
|V|oneway|

