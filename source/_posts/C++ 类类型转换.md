---
title: C++类类型转换
date: 2016-05-06 22:32:28
categories:
- C++
tags:
- C++
- 隐式转换
---



### 隐式类类型转换
>在c++中，可以调用单参数构造函数，把指定类型转换为该类类型。这样的构造函数有时也被称为转换构造函数(converting constructors)。也就是说类的单参数构造函数，提供了把该形参类型转换为该类类型的一种方式。

<!-- more -->
**最常见的例子：**

string name = "Bill";
在该例子中，由于string类提供了构造函数string (const char * s);因此存在字符串常量"Bill"到string类的隐式转换。

**自定义类型示例：**

    class Sales_data {
    public:
        Sales_data() = default;
        Sales_data( const std::string &s ) : bookNo( s ) { }
        Sales_data( const std::string &s, unsigned n, double p ) :
            bookNo( s ), units_sold( n ), revenue( p*n ) { }
        Sales_data( std::istream & );
    
        std::string isbn() const { return bookNo; }
        Sales_data& combine( const Sales_data& );
    
    private:
        std::string     bookNo;
        unsigned         units_sold;
        double             revenue;
    };
在上述例子中，由于`Sales_data( const std::string &s )；`构造函数的存在，因此存在`string`类型到`Sales_data`的转换，因此在需要`Sales_data`对象的时候，我们可以使用`string`类型替代。

    Sales_data item;
    string null_book = "9-999-99999-9";
    item.combine( null_book );
**PS：**

只有一次的隐式类类型转换是可行的，item.combine( "9-999-99999-9" );是错误的，因为在该语句中，存在着两次隐式转换，一次是字符串常量"9-999-99999-9"到string的转换，另一次是string到Sales_data的转换。

**explicit constructors：**

在你不想隐式转换，以防用户误操作怎么办？

C++提供了一种抑制构造函数隐式转换的办法，就是在构造函数前面加explicit关键字，你试试就知道，那时你再希望隐式转换就会导致编译失败，但是，要说明的是，显式转换还是可以进行。

---

### 类型转换函数

类型转换函数(type conversion function)的作用是将一个类的对象转换成另一类型的数据

我们经常下述代码风格：

    while( cin >> num ){
    }
输入操作符 `>>` 是二元操作符，返回做操作数作为其表达式结果，因此`cin >> num`返回`cin`,然而`cin`是输入流`istream`的对象，该对象能出现在条件表达式中，是因为在`istream`中定义了类型转换函数 `operator bool();`。

**示例：**

    class Sales_data {
    public:
            return true;
        }    
    };
类`Sales_data`定义了从该类对象到`bool`类型的转换，因此，在需要`bool`类型的表达式中可以使用该类对象代替：

    Sales_data item;
    
    if ( item ) {
        cout << "true";
    }