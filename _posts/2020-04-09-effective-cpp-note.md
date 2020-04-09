---                                                                                                                                                                                                                  
layout: post                                                                                                                                                                                                         
title: 'Effective C++阅读笔记'                                                                                                                                                                                              
date: 2020-04-09                                                                                                                                                                                                     
author: wyg1997                                                                                                                                                                                                      
color: rgb(255,210,32)                                                                                                                                                                                               
cover: '../assets/imgs/EffectiveCpp.png'                                                                                                                                                                                   
tags: cpp                                                                                                                                                                                                         
---

* any list
{:toc}

# Effective C++笔记

## 目录

{:toc}

## 条款

### 条款2：尽量以const、enum、inline替换#define

- 对于单纯常量，最好以`const`对象或`enums`填换`#define`。
- 对于形似函数的宏，最好改用`inline`函数替换`#define`。

### 条款3：尽可能使用const

- 使用`const`声明可以帮助编译器找到错误用法。

> const修饰指针时，如果在`*`的左侧，表示被指物是常量；如果出现在星号右边，说明指针自己是常量；如果在两边，说明被指物和指针自己都是常量。

> const方法有两个特性：
>
> 1. 调用此方法时不能改变类的数据成员的值。
> 2. 非const实例化对象不能调用const声明的方法。

### 条款4：确定对象被使用前已先被初始化

- 为内置型对象进行手工初始化，因为C++不能保证初始化他们。
- 构造函数最好使用**成员初值列**，而不是在构造函数内进行赋值（构造函数里进行的只是赋值操作，赋值前会对成员调用默认构造函数做初始化，导致资源浪费；而使用成员初值列的值是作为各成员构造函数的实参进行初始化）。且成员初值列的次序应该和他们在`class`中声明的次序相同。
- 如果多个编译单元（编译单元可以理解为单一目标文件加上他们的头文件）均有`non_local static`对象（对象是`global`或位于`namespace`域内或`class`内或`file`作用域内），假如某一个`non_local static`对象需要调用另一个`non_local static`对象时，那个对象不一样已经完成初始化，这时就可能会出现错误。这时使用类似**单例设计模式**的思想，把`static`对象初始化放在某方法内来被动初始化，就可以解决这个问题。

### 条款5：了解C++默默编写并调用哪些函数

#### 编译器自动生成的方法

1. 拷贝构造函数
2. 拷贝赋值运算符
3. 析构函数
4. 默认构造函数

> 当拷贝赋值运算是非法时，编译器会拒绝生成：
>
> 1. 引用成员数据。
> 2. 常量成员数据。
>
> 这样的类不能用于STL容器中，因为容器的元素必须能够拷贝构造和拷贝赋值。

### 条款6：若不想使用编译器自动生成的函数，就应该明确拒绝

- 为驳回编译器自动提供的机能，可将相应的成员函数声明为`private`并且不予实现。使用像`Uncopyable`这样的`base class`也是一种做法（多重继承可能会出现问题）。

> 注：如果只是不想使用默认的拷贝构造函数或赋值函数，可以使用C++11提供的`=delete`特性来删除此函数。

### 条款7：为多态基类声明virtual析构函数

- 带多态性质的基类应该声明一个`virtual`析构函数。如果这个类有虚函数，那么它就应该有一个虚析构函数。

  > 举个例子（错误写法）：
  >
  > ```cpp
  > #include<iostream>
  > using namespace std;
  > class ClxBase{
  > public:
  >     ClxBase() {};
  >     virtual ~ClxBase() {cout << "Output from the destructor of class ClxBase!" << endl;};
  >     virtual void DoSomething() { cout << "Do something in class ClxBase!" << endl; };
  > };
  > 
  > class ClxDerived : public ClxBase{
  > public:
  >     ClxDerived() {};
  >     ~ClxDerived() { cout << "Output from the destructor of class ClxDerived!" << endl; };
  >     void DoSomething() { cout << "Do something in class ClxDerived!" << endl; };
  > };
  > 
  > int main(){  
  >     ClxBase *p = new ClxDerived;
  >     p->DoSomething();
  >     delete p;
  >     return 0;
  > } 
  > ```
  >
  > 输出为：
  >
  > ```
  > Do something in class ClxBase!
  > Output from the destructor of class ClxBase!
  > ```
  >
  > 上面这段代码，如果用基类的指针去操作子类的成员，`DoSomething`实现的是多态的性质，会输出子类的函数。但是在`delete`析构时，因为父类的析构函数不为虚函数时，子类的析构函数就不会被先调用，就只会删除基类对象，子类对象就会内存泄漏。

- 如果类的设计目的并不是为了作为基类使用，或不是为了具备多态性，就不要声明`virtual`析构函数。

### 条款8：别让异常逃离析构函数

- 析构函数决不要吐出异常。如果一个被析构函数调用的函数可能抛出异常时，析构函数要把异常捕捉，要么吞掉，要么结束程序。
- 如果客户需要对某个操作函数运行时抛出的异常做出反应，那么`class`应该提供一个普通函数而不是在析构函数中操作。

### 条款9：绝不在构造和析构过程中调用virtual函数

- 构造函数和析构函数调用时调用虚函数，因为子类还未构造或已析构，这些调用并不会下降至子类，完全没有虚函数的性质。

### 条款11：在operator=中处理“自我赋值”

- 确保对象在自我赋值时有良好的行为。
- 确定任何函数操作一个以上的对象时，如果其中多个对象是同一对象时，其行为仍然正确。

### 条款13：以对象管理资源

- 为防止资源泄漏，使用RAII对象，它们在构造函数中获得资源并在析构函数中释放。
- 两个常用的RAII对象分别是`tr1::shared_ptr`和`auto_ptr`。使用`tr1::shared_ptr`是更推荐的选择，因为其`copy`行为比较直观。

### 条款14：在资源管理类中小心coping行为

- 设计RAII对象必须一并复杂它管理的资源，所以资源的coping行为决定RAII对象的coping行为。
- 普遍的RAII类的coping行为是：
  1. 抑制coping（删除拷贝构造函数和拷贝赋值函数）。
  2. 使用引用计数法（`shared_ptr`）。

### 条款16：成对使用new和delete时要采用相同形式

- 如果在`new`表达式中使用`[]`，必须在相应的`delete`表达式中也使用`[]`。否则都不使用。
- 尽量不要对数组做`typedef`操作，否则在`new`和`delete`操作时就容易出错。

### 条款17：以独立的语句将new对象置入智能指针

- 使用独立的语句来把`new`出来的对象置入智能指针内。

> 例如：
>
> ```cpp
> int priority();
> void processWidget(std::tr1::shared_ptr<Widget> pw, int priorty);
> 
> processWidget(new Widget, priority());
> ```
>
> 如果第4行这么调用的话，程序要做以下3件事：
>
> ```
> new Widget
> priority()
> std::tr1::shared_ptr()构造函数
> ```
>
> 而执行顺序并不能确定（和JAVA、C#不同）。
>
> 这时如果先执行了`new Widget`，再执行`priority()`时出现异常，这时前面new申请的内存就泄漏了。
>
> 所以上述代码应该改为：
>
> ```cpp
> std::tr1::shared_ptr<Widget> pw(new Widget);
> processWidget(pw, priority());
> ```

### 条款20：宁以pass-by-reference替换pass-by-value

- 尽量以`pass-by-reference-to-const`替换`pass-by-value`，前者通常比较高效，不用复制一遍成员数据以及执行父类的构造函数等。而且可以避免切割问题（指使用多态时，如果传递的是值，函数的参数是父类而传入的参数是子类，则执行的将会是父类的方法）。
- 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。因为这些对象本身并不占用很大空间，传引用可能反而更占资源。

### 条款25：考虑写出一个不抛异常的swap函数

- 当`std::swap`对你的类型效率不高时（如果只需交换对象成员的指针指向，使用`std::swap`两个对象则会创建新的对象，效率就会较低），提供一个`public swap`成员函数，且这个函数不抛出异常（一般这个函数只对基本类型做操作，也不会抛出异常）。

  >例如：
  >
  >```cpp
  >class WidgetTmpl
  >{
  >private:
  >    int a,b,c;
  >    std::vector<double> v;
  >};
  >
  >class Widget
  >{
  >public:
  >    Widget(const Widget& rhs);
  >    Widget& operator=(const Widget& rhs)
  >    {
  >        ...
  >        *pImpl = *(rhs.pImpl);
  >        ...
  >    }
  >private:
  >    WidgetImpl* pImpl;
  >};
  >```
  >
  >如果我们想交换`Widget`对象，只需要交换它的`pImpl`指针就行了。如果使用`std::swap`函数交换，会复制3个`Widget`对象，还会复制3个`WidgetImpl`对象，效率非常低下。
  >
  >所以我们在`Widget`类中加入`swap`函数，就会方便的多。代码如下：
  >
  >```cpp
  >class Widget
  >{
  >public:
  >    void swap(Widget& other)
  >    {
  >        using std::swap;  // 选择最佳的swap版本，必要
  >        swap(pImpl, other.pImpl);
  >    }
  >    ...
  >};
  >```
  >
  >再提供一个`non-member swap`来调用：
  >
  >```cpp
  >void swap(Widget &a, Widget &b)
  >{
  >    a.swap(b);
  >}
  >```

- 调用`swap`时应使用`using std::swap`，然后调用不带任何命名空间修饰的`swap`（见上面代码），这样就可以为要交换的对象选择最佳的`swap`形式。

- 如果`class`是模板类型，则特化`non-member swap`。

  > ```cpp
  > Template<typename T>
  > void swap(Widget<T> &a, Widget<T> &b)
  > {
  >     a.swap(b);
  > }
  > ```

### 条款26：尽可能延后变量定义式的出现时间

- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序的效率。

### 条款27：尽量少做转型动作

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免`dynamic_casts`。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需要转型放进他们自己的代码内。
- 宁可使用C++新式转型（`const_cast`、`dynamic_cast`、`reinterpret_cast`、`static_cast`），不要使用旧式转型。

> 优质的C++代码很少使用强制类型转换。

### 条款28：避免返回handles指向对象内部成分

- 避免返回handles（包括`引用`、`指针`、`迭代器`）指向对象内部。遵守这个条款可增加封装性。

### 条款30：透彻了解inlining的里里外外

- 将大多数`inlining`限制在小型、被频繁调用的函数身上。
- 不要只因为`function template`出现在头文件，就将它们声明为`inline`。

> `inline`函数在调用的时候编译器会让它展开，表现出的并不是函数调用的关系。

### 条款32：确定你的public继承塑模出is-a关系

- `public`继承意味`is-a`。适用于基类的每一件事（类的方法）也适用于派生类，因为每一个派生类对象也都是一个基类对象。

> 这里举个例子：例如`Bird`类和`Penguin`（企鹅）类。如果`Bird`类有`fly`的方法，这时就不能随便使用`public`继承，会造成语言的不严谨。
>
> 如果`Bird`类有`fly`方法，则说明此时定义的是一个会飞的鸟类，使用企鹅来继承就不合适。

### 条款36：绝不重新定义继承而来的non-virtual函数

- 绝不重新定义继承而来的非虚函数。

  > 例如：
  >
  > ```cpp
  > class B
  > {
  > 	void mf();
  > };
  > class D: public B
  > {
  >     void mf();
  > };
  > 
  > D x;
  > B* pB = &x;
  > D* pD = &x;
  > // 下面两行将会可能产生不一样的结果
  > pB->mf();
  > pD->mf();
  > ```

### 条款37：绝不重新定义继承而来的缺省参数值

- 绝不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而virtual函数却是动态绑定。

  > 如果需要在虚函数使用默认参数的话，可以使用替代方法：令基类内的一个`public non-virtual`函数调用`private virtual`函数，后者可以被派生类重新定义，这里我们让`public non-virtual`指定缺省参数。

### 条款42：了解typename的双重意义

- 声明`template`参数时，前缀关键字`class`和`typename`可以互换。
- 使用关键字`typename`标识嵌套从属类型的名称，但不得在**基类列**和**成员初值列**内以它作为`base class `修饰符。

> ```cpp
> template<typename C>
> void f(const C& container,  // 不允许使用typename
>        typename C::iterator iter)  // 一定要使用typename
> ```

