---
title: C++ 高阶函数
date: 2016-07-21 23:39:52
categories:
- C++
tags:
- C++
- 高阶函数
---

>函数式编程有很多优点，详情可以参考我的博文卡马克：《用C++ 进行函数式编程》。在C++中，我们可以借助于函数对象或者函数指针来实现高阶函数。

<!-- more -->

#### 高阶函数是至少满足下列一个条件的函数:
- 接受一个或多个函数作为输入
- 输出一个函数

### 三种求和方式

![示例](http://i.imgur.com/7S95DNV.png)

``` C++
//公式（1）
int sumInt( int a, int b ){
    int result { 0 };
    for ( int i = a; i <= b; ++i ) {
        result += i;
    }
    return result;
}

//公式（2）
int sumCube( int a, int b ){
    int result { 0 };
    for ( int i = a; i <= b; ++i ) {
        result += i * i * i;
    }
    return result;
}

//公式（3）
double sumPi( int a, int b ){
    double result = 0;
    for ( int i = 1; i <= b; i += 4 ) {
        result += 1 / ( (double)(i) * (double)( i + 2 ) );
    }
    return result;
}
```
    
上述示例中，三个程序表面不同，但是程序包含的逻辑（对于不同序列进行求和）是相同的。对于计算机程序，这种类似就意味着抽象，进行更高层次的抽象，以较少重复劳动，减小出现错误的风险。

上述代码可以抽象为：

``` C++
//advanced abstraction
template < class T, class F, class G >
T sumGeneric( T a, T b, F func, G next ){
    T result( 0 );
    for ( T i = a; i <= b; next( i ) ) {
        result += func( i ); 
    }
    return result;
}
```

上述代码，允许用户将循环体内的过程func和nex以参数的形式传入。只要它们能以函数的形式调用即可，在C++中，我们可以使用函数对象做到这一点。

**实现如下：**
``` C++
template < class T >
class Self{
public:
    T operator()( T x ){ return x;  }
};

template < class T >
class Cube{
public:
    T operator()( T x ){ return x*x*x; }
};

template< class T >
class MyFunc{
public:
    T operator()( T x ){
        return 1 / ( x * ( x + 2 ) );
    }
};

template < class T >
class Inc{
public:
    void operator()( T& x ){ ++x; }
};

template < class T >
class Inc4{
public:
    void operator()( T& x ){ x += 4; }
};
```

**测试结果：**
``` C++
int main(){
    cout << "Normal Cal:" << endl;
    cout << "sumInt( 1, 50 ):" << sumInt( 1, 50 ) << " sumCube( 1, 50 ):" << sumCube( 1, 50 ) << " sumPi( 1, 50 ):" << sumPi( 1, 50 );

    cout << endl << "High Order Function:" << endl;
    cout << "sumInt( 1, 50 ):" << sumGeneric( 1, 50, Self<int>(), Inc<int>() ) 
        << " sumCube( 1, 50 ):" << sumGeneric( 1, 50, Cube<int>(), Inc<int>() )
        << " sumPi( 1, 50 ):" << sumGeneric( (double)1, (double)50, MyFunc<double>(), Inc4<double>() );

    system( "pause" );
}
```
![结果](http://i.imgur.com/nvvFPya.png)

### lambda表达式实现高阶函数

> C++11在语言中加入了lambda表达式，我们可以借助与lambda表达式实现高阶函数。

参看我的博文C++ lambda表达式

代码下载地址：
https://github.com/bjut-hz/High-Order-Function