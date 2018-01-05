---
title: C++ lambda表达式
date: 2016-06-20 14:36:53
categories:
- C++
tags:
- C++
- lambda
---


> C++ 11在语言中加入了lambda表达式，lambda表达式可以方便地构造匿名函数。当我们定义lambda表达式时，C++编译器会创建一个匿名的与lambda表达式有关的类类型。

<!-- more -->
### 使用lambda表达式进行函数式编程

我们知道，在函数式编程语言中，比如scheme，函数作为一等公民，与普通的数据类型相同，可以作为函数的参数以及返回值，可以很方便的实现高阶函数等。 在C++中，我们可以借助于函数对象以及函数指针实现。 在C ++11中，借助于lambda表达式，也可以实现高阶函数。

### 将lambda表达式用作返回值
定义在头文件中的std::function是多态的函数对象包装，类似函数指针。它可以绑定至任何可以被调用的对象(仿函数、成员函数指针、函数指针和lambda表达式)，只要参数和返回类型符合包装的类型即可。返回一个double、接受两个整数参数的函数包装定义如下：
    
    function< double(int, int) > my_wrapper;
    
通过std::function，可以从函数中返回lambda表达式，示例如下：

    function<int(void)> multiplyBy2Lambda(int x)  
    {  
        return [=]()->int{ return 2 * x; };  
    }
    
这个函数的主体部分创建了一个lambda表达式，这个lambda表达式通过值捕捉所在作用域的变量，并返回一个整数，这个返回的整数是传给multiplyBy2Lambda()的值的两倍。这个multiplyBy2Lambda()函数的返回值类型为 function，即一个不接受参数并返回一个整数的函数。函数主体中定义的lambda表达式正好匹配这个原型。变量x通过值捕捉，因此，在lambda表达式从函数返回之前，x值的一份副本绑定至lambda表达式中的x。
可以通过下述方式调用该函数：

    auto fun = multiplyBy2Lambda( 5 );
    cout << fun(); //输出为10

### 将lambda表达式用作参数
可以编写lambda表达式作为参数的函数。例如，可通过这种方式实现回调函数。下面的代码实现了一个testCallback()函数，这个函数接受一个整数vector和一个回调函数作为参数。这个实现迭代给定vector中的所有元素，并对每个元素调用回调函数，回调函数接受vector中每个元素作为int参数，并返回一个布尔值。如果回调函数返回false，那么停止迭代。

    //注意参数类型，第二个参数如果使用pass by reference，则必须加const修饰，否则编译错误
    //如果采用值传递，无影响
    void testCallback(const vector<int>& vec, const function<bool(int)>& callback)  
    {  
        for (auto i : vec)  
        {  
            if (!callback(i))  
                break;  
            cout << i << " ";  
        }  
        cout << endl;  
    }

**测试：**

    auto callback = []( int i ) -> bool { return i < 6; };
    vector<int> vec( 10 );
    int index = 0;
    generate( vec.begin(), vec.end(), [&index] {return ++index; } );
    
    testCallback( vec, callback );

**结果：**

![结果](http://i.imgur.com/OHyYteu.png)

### lambda表达式实现高阶函数

有关C++ 高阶函数可以参看我的博文：C++ 高阶函数

    int sumInt( int a, int b ){
        int result { 0 };
        for ( int i = a; i <= b; ++i ) {
            result += i;
        }
        return result;
    }
    
    int sumCube( int a, int b ){
        int result { 0 };
        for ( int i = a; i <= b; ++i ) {
            result += i * i * i;
        }
        return result;
    }
    
    //advanced abstraction
    template < class T >
    T sumGeneric( T a, T b, const function< int( int ) >& func,  const function< void( int& ) >& next ){
        T result( 0 );
        for ( T i = a; i <= b; next( i ) ) {
            result += func( i );
        }
        return result;
    }

> 上述示例中，前两个求和函数有着相同的逻辑，可以进行抽象`sumGeneric`。

**测试：**

    auto self = []( int i ) -> int { return i; };
    auto inc = []( int& i ) -> void {  ++i; };
    auto cube = []( int i ) -> int { return i * i * i; };
    
    cout << "Normal Cal:" << endl;
    cout << "sumInt( 1, 50 ):" << sumInt( 1, 50 ) << " sumCube( 1, 50 ):" << sumCube( 1, 50 );
    
    cout << endl << "High Order Function:" << endl;
    cout << "sumInt( 1, 50 ):" << sumGeneric( 1, 50, self, inc )
        << " sumCube( 1, 50 ):" << sumGeneric( 1, 50, cube, inc );

**结果：**

![结果](http://i.imgur.com/fyweW6U.png)

代码下载地址：
https://github.com/bjut-hz/High-Order-Function
