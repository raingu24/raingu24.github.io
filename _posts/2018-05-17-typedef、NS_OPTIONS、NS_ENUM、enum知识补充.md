#### 1. typedef
  - 1.1 typedef的作用：用于类型定义。如果是一个已经存在的类，可以给这个类起别名。还用于声明结构体、枚举等。

  ```
  // 给NSString起别名
  typedef NSString MyString;

  // 定义一个结构体
  struct Person {
      char *name;
  };
  
  // 定义Person的别名
  typedef struct Person MyPerson;

  // 定义一个枚举，并直接定好别名
  typedef enum Gender {
      Man,
      Woman
  }MyGender;

  // 定义一个函数，并用typedef定义一个类型，囊括这种类型的函数
  int add (int a, int b) {
      retur a + b;
  }
  typedef int(*MyFunction) (int a, int b);


  /////////// 应用上面的声明 ////////////

    NSLog(@"%@", str1);  // found
    MyPerson person1;
    person1.name = "name";
    NSLog(@"person1 name: %@", [NSString stringWithUTF8String:person1.name]);			// person1 name: name
    struct Person person2;
    person2.name = "name2";
    NSLog(@"person2 name: %@", [NSString stringWithUTF8String:person2.name]);			// person2 name: name2
    NSLog(@"gender man's value: %d", Man);	// gender man's vlaue: 0 这个要特别说明一下，不同的枚举的成员名不能相同。如果再声明一个Gender1的枚举，也有一个Man成员，就会报错。所以一般在声明枚举的时候为了避免这种情况，会把枚举类型名作为前缀，例如GenderMan。

    // 使用自定义的函数类型
    MyFunction m = add;
    NSLog(@"add result = %d", m(2,3));

  ```

#### 2. NS\_OPTIONS、NS\_ENUM和enme的区别
  - 2.1 NS\_OPTIONS：小括号中的第一个NSUInteger是固定值，第二个为枚举类型名。大括号中枚举必须全部包含小括号的枚举类型，枚举项后面再跟上作为区别的标记。
   ```
  typedef NS_OPTIONS(NSUInteger, RGEncodingType) {
      RGEncodingTypeMask      = 0xFF, // 类型掩码的值
      RGEncodingTypeUnknown   = 0, // unknown
      RGEncodingTypeVoid      = 1, // void
      
      RGEncodingTypeQualifierMask     = 0xFF00, // 限定符掩码
      RGEncodingTypeQualifierConst    = 1 << 8, // const
      RGEncodingTypeQualifierIn       = 1 << 9, // in
  };  
  ```

  - 2.2 NS\_ENUM：小括号中第一个NSInteger为固定值，第二个为枚举类型名。大括号中枚举必须全部包含小括号的枚举类型，枚举项后面再跟上作为区别的标记。
  ```
  typedef NS_ENUM(NSInteger, NSWritingDirection) {
      NSWritingDirectionNatural       = -1,
      NSWritingDirectionLeftToRight   =  0,
      NSWritingDirectionRightToLeft   =  1
  }
  ```

  - 2.3 enum：声明C的枚举类型。当然OC也可以用。

  - 2.4 在使用的时候，发现三者好像没有太多的不同。但是还是有一定的区别：
    - 2.4.1 enum是原始的C的枚举类型。苹果官方推荐使用另两种声明枚举的方式。
    - 2.4.2 另外两种在普通情况下没有什么区别。但是NS\_OPTIONS在C++编译的时候提供更好的支持。所以在使用的时候，如果使用位移的方式来设定枚举值，那么最好采用NS\_OPTIONS。虽然在编译的时候NS\_ENUM定义位移的枚举值也不会出错，但是如果使用C++编译器的时候就会出现问题。具体解释参见[这里](https://www.jianshu.com/p/7abeef86d94f)。

 
