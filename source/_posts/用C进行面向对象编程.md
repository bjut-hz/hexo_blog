---
title: 用C进行面向对象编程
date: 2018-01-10 17:34:11
categories:
- C
tags:
- 面向对象
- OO
---

> 为了解决编程的复杂性，自计算机发明以来，产生了三种编程范式：结构化编程、面向对象编程及函数式编程。长久以来，C语言都是被当作结构化编程典型，但是语言只是工具，编程范式更多提供给我们规划及编写代码的思路。本篇博客介绍了使用"结构化编程语言"：C语言，实现面向对象编程。
<!-- more -->

## 面向对象编程

### 定义

长久以来，面向对象被认为是数据与函数的结合，然而这个定义是十分荒谬的，你能说`o.f()`与`f(o)`是不同的么，很明显两者表述的是同一个事物。目前我没发现一个适合的定义，但是面向对象编程必有三个特性：封装，继承及多态。

### OO语言三要素

#### 封装

一种将抽象性函数接口的实现细节部分包装、隐藏起来的方法。可以防止外界调用端去访问对象内部实现。

#### 继承

继承可以使得子类别具有父类别的各种属性和方法，而不需要再次编写相同的代码。大大提高了代码的可重用性。

#### 多态

由继承而产生的相关的不同的类，其对象对同一消息会做出不同的响应。

## 用C进行面向对象编程

### 封装

封装提高了代码的安全性，可以对外屏蔽具体的实现，在C语言中，代码分为头文件及对应的实现文件，利用此机制，可以做到封装：

``` C
// person.h
struct Person;

struct Person* newPerson(int age);
void delPerson(struct Person* this);
int getAge(struct Person* this);
void setAge(struct Person* this, int age);
```
在`person.h`中，声明了Person结构体及对应的函数，`person.h`是数据与函数的结合，我们可以在`person.c`中实现对应的功能：

```C
struct Person {
	int age;
};

struct Person* newPerson(int age){
	struct Person* p = malloc(sizeof(struct Person));
	p->age = age;
	return p;
}

void delPerson(struct Person* this){
	free(this);
}

int getAge(struct Person* this){
	return 
	this->age;
}
void setAge(struct Person* this, int age){
	this->age = age;
}
```

相比较C++这类OO语言来说，C语言版本的封装可以认为是更加完美的封装，因为在`person.h`头文件里，完全没有暴露`struct Person`的相关定义，但是在C++这类OO语言中，必须要这么定义：

``` C++
class Person{
private:
	int age;
public:
	Person(int age);
	~Person();

	int getAge();
	void setAge(int age);
}
```
暴露了`Person`的内部成员。

### 继承

继承是子类自动包含父类的所有成员，可以认为子类是父类的超集。在C语言中两个结构体如果内存布局*“相似”*，如：

``` C
struct Person {
	int age;
};

struct Student{
	int age;
	int cls;
};
```
`Person`与`Student`两个结构体，`Student`是`Person`的超集，两者的数据成员顺序相同，内存布局相似，基于此技巧，借助于强制类型转换，我们可以使用C语言实现继承：

``` C
// person.h
struct Person;

struct Person* newPerson(int age);
void delPerson(struct Person* this);
int getAge(struct Person* this);
void setAge(struct Person* this, int age);

// student.h
struct Student;

struct Student* newStudent(int age, int cls);
void setClass(struct Student* this, int cls);
int getClass(struct Student* this);
void delStudent(struct Student* this);

// main.c
int main(void){
	struct Student* stu = newStudent(10, 4);
	printf("Age: %d, Class: %d\n", getAge((struct Person*)stu), getClass(stu));
	// inheritance, Student get method from Person
	setAge((struct Person*) stu, 15);
	setClass(stu, 9);

	printf("Age: %d, Class: %d\n", getAge((struct Person*)stu), getClass(stu));
	delStudent(stu);
}
```

### 多态

我们直接看个程序：

``` C
// file.h
struct File {
	int (*read)(char* name, int size);
	void (*write)(char* name, int size);
};
```
我们给出了`File`类型，其中包含了两个函数指针，则即给定了函数声明，这不就是OO语言中的接口么？

``` C
// console.c
static int read(char* name, int size) {
	printf("console read function, filename: %s, size: %d\n", name, size);
	return size;
}

static void write(char* name, int size) {
	printf("console write function, filename: %s, size: %d\n", name, size);
}

struct File console = {read, write};

// socket.c
static int read(char* name, int size) {
	printf("socket read function, filename: %s, size: %d\n", name, size);
	return size;
}

static void write(char* name, int size) {
	printf("socket write function, filename: %s, size: %d\n", name, size);
}

struct File socket = {read, write};

// STDIN.c
extern struct File console;
struct FILE* STDIN = (struct FILE*)&console;

// main.c
int main(void) {
	extern struct File* STDIN;
	STDIN->read("test.txt", 10);
	STDIN->write("test.txt", 100);

	extern struct File socket;
	STDIN = &socket;
	STDIN->read("socket", 1000);
	STDIN->write("socket", 10);
}
```
可以看出通过`STDIN`指针指向不同的实现，就可以做出不同的响应，即多态。在C++中，多态的底层实现是通过虚表虚指针来实现的，与本实例同理。


## 总结

面向对象编程是把这一些列的指针操作(最易出错的地方)隐藏在了底层，程序员接触不到。从语言层级提高了程序员的抽象能力。但是编程范式的发展限制了程序员可用的具体操作，如：OO编程范式把一系列的指针操作隐藏在底层。结构化程序设计使得我们只能用三种流程控制，远离goto人人有责。而函数式编程则禁止使用变量赋值。

代码地址：https://github.com/bjut-hz/C_OO

### 引用

[《Clean Architecture: A Craftsman's Guide to Software Structure and Design》](https://github.com/bjut-hz/E-Books#software-engineering)