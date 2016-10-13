# README

**《More C++ idioms》**翻译

---

**译者的话：**

很喜欢这本书，但是没有中文版。便有了翻译这本书的想法，时间，水平有限，任何疏漏，欢迎大家 pull request : )

你可以在[这里](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms)免费下载到这本书的PDF。

---

**目录**

1. 不使用 operator & 取地址
2. 代数抽象基类
3. 
4. 
5. 
6. 
7. 
8. 在对象构造的时候调用虚函数


#### **1 取址器**

##### 1.0.1 目的

在不使用取地址运算符的情况下，取得对象的地址。

##### 1.0.1 别名

##### 1.0.3 动机

C++允许重载取地址运算符，却没有规定取地址运算符的返回类型是对象的实际地址。毫无疑问，取地址运算符不返回地址是

非常奇怪的，但语言自身无疑允许这样做。取址器手法是不依赖取地址运算符获得对象实际地址的一种方法。

在下面的例子中，由于取地址运算符是类私有的，main 函数会编译失败。即使没有私有的访问控制，返回类型double转成指

针也是无意义的。

~~~
class nonaddressable{
public:
    typedef double useless_type;
private:
    useless_type operator&() const;
};

int main(){

    nonaddressable na;
    nonaddressable * naptr = &na;    // Compiler error here.
}
~~~

##### 1.0.4 解决方案和示例代码

取址器用一系列的类型转换获得一个对象的地址。

~~~
template <class T>
T * addressof(T& v){

    return reinterpret_cast<T*>(& const_cast<char&>(reinterpret_cast<const volatile char&>(v)));

}

int main(){

    nonaddressable na;
    nonaddressable * naptr = &na;
}
~~~

##### 1.0.5 已知的用法

* Boost 中的 addressof 

在C++11的 <memory> 中已经包含了这个函数。

##### 1.0.6 相关的做法

##### 1.0.7 引用

---

#### **2 代数抽象基类**

##### 2.0.1 目的

给关系密切的具体类(如实数和复数)提供一致的接口。

##### 2.0.1 别名

* 状态法

##### 2.0.3 动机

在像Smalltalk这样纯粹的面向对象语言中，变量像标签那样在运行时动态绑定到对象。变量名绑定对象就像是给对象贴

上标签.这些语言中的赋值好比是从一个对象身上把标签斯下来贴到另外的对象身上。然而，在C和C++中，变量不像对象

的标签而是实际的地址和偏移。赋值不是名字重新绑定到对象，赋值意味着用旧值覆盖新值。代数抽象基类使用委托和多

态来模拟变量名字跟对象的绑定。代数抽象基类使用了“信封手法”。这里抽象基类的动机，是能够写如下的代码:

~~~
Number n1 = Complex (1,2);  // n1是一个复数
Number n2 = Real (10);      // n2是一个实数
Number n3 = n1 + n2;    // n3是n1和n2的和
Number n2 = n3;     // "重新贴标签" 
~~~

##### 2.0.4 解决方案和示例代码

代数抽象基类的实现代码如下。

~~~
#include <iostream>
 
using namespace std;
 
struct BaseConstructor { BaseConstructor(int=0) {} };
 
class RealNumber;
class Complex;
class Number;
 
class Number
{
    friend class RealNumber;
    friend class Complex;
 
  public:
    Number ();
    Number & operator = (const Number &n);
    Number (const Number &n);
    virtual ~Number();
 
    virtual Number operator + (Number const &n) const;
    void swap (Number &n) throw ();
 
    static Number makeReal (double r);
    static Number makeComplex (double rpart, double ipart);
 
  protected:
    Number (BaseConstructor);
 
  private:
    void redefine (Number *n);
    virtual Number complexAdd (Number const &n) const;
    virtual Number realAdd (Number const &n) const;
 
    Number *rep;
    short referenceCount;
};
 
class Complex : public Number
{
    friend class RealNumber;
    friend class Number;

    Complex (double d, double e);
    Complex (const Complex &c);
    virtual ~Complex ();

    virtual Number operator + (Number const &n) const;
    virtual Number realAdd (Number const &n) const;
    virtual Number complexAdd (Number const &n) const;

    double rpart, ipart;
};
 
class RealNumber : public Number
{
    friend class Complex;
    friend class Number;

    RealNumber (double r);
    RealNumber (const RealNumber &r);
    virtual ~RealNumber ();

    virtual Number operator + (Number const &n) const;
    virtual Number realAdd (Number const &n) const;
    virtual Number complexAdd (Number const &n) const;

    double val;
};
 
/// Used only by the letters.
Number::Number (BaseConstructor) : rep (0), referenceCount (1) {}
 
/// Used by user and static factory functions.
Number::Number () : rep (0), referenceCount (0) {}
 
/// Used by user and static factory functions.
Number::Number (const Number &n) : rep (n.rep), referenceCount (0)
{
    cout << "Constructing a Number using Number::Number\n";
    if (n.rep)
        n.rep->referenceCount++;
}
 
Number Number::makeReal (double r)
{
    Number n;
    n.redefine (new RealNumber (r));
    return n;
}
 
Number Number::makeComplex (double rpart, double ipart)
{
    Number n;
    n.redefine (new Complex (rpart, ipart));
    return n;
}
 
Number::~Number()
{
    if (rep && --rep->referenceCount == 0)
        delete rep;
}
 
Number & Number::operator = (const Number &n)
{
    cout << "Assigning a Number using Number::operator=\n";
    Number temp (n);
    this->swap (temp);
    return *this;
}
 
void Number::swap (Number &n) throw ()
{
    std::swap (this->rep, n.rep);
}
 
Number Number::operator + (Number const &n) const
{
    return rep->operator + (n);
}
 
Number Number::complexAdd (Number const &n) const 
{
    return rep->complexAdd (n);
}
 
Number Number::realAdd (Number const &n) const
{
    return rep->realAdd (n);
}
 
void Number::redefine (Number *n)
{
    if (rep && --rep->referenceCount == 0)
        delete rep;
    rep = n;
}
 
Complex::Complex (double d, double e) : Number (BaseConstructor()), rpart (d), ipart (e)
{
    cout << "Constructing a Complex\n";
}
 
Complex::Complex (const Complex &c) : Number (BaseConstructor()), rpart (c.rpart), ipart (c.ipart)
{
    cout << "Constructing a Complex using Complex::Complex\n";
}
 
Complex::~Complex()
{
    cout << "Inside Complex::~Complex()\n";
}
 
Number Complex::operator + (Number const &n) const
{ 
    return n.complexAdd (*this); 
}
 
Number Complex::realAdd (Number const &n) const
{
      cout << "Complex::realAdd\n";
      RealNumber const *rn = dynamic_cast <RealNumber const *> (&n);
      return Number::makeComplex (this->rpart + rn->val, this->ipart);
}
 
Number Complex::complexAdd (Number const &n) const
{
    cout << "Complex::complexAdd\n";
    Complex const *cn = dynamic_cast <Complex const *> (&n);
    return Number::makeComplex (this->rpart + cn->rpart, this->ipart + cn->ipart);
}
 
RealNumber::RealNumber (double r) : Number (BaseConstructor()), val (r)
{
    cout << "Constructing a RealNumber\n";
}
 
RealNumber::RealNumber (const RealNumber &r)
  : Number (BaseConstructor()),
    val (r.val)
{
    cout << "Constructing a RealNumber using RealNumber::RealNumber\n";
}
 
RealNumber::~RealNumber()
{
    cout << "Inside RealNumber::~RealNumber()\n";
}
 
Number RealNumber::operator + (Number const &n) const
{ 
    return n.realAdd (*this); 
}
 
Number RealNumber::realAdd (Number const &n) const
{
    cout << "RealNumber::realAdd\n";
    RealNumber const *rn = dynamic_cast <RealNumber const *> (&n);
    return Number::makeReal (this->val + rn->val);
}
 
Number RealNumber::complexAdd (Number const &n) const
{
    cout << "RealNumber::complexAdd\n";
    Complex const *cn = dynamic_cast <Complex const *> (&n);
    return Number::makeComplex (this->val + cn->rpart, cn->ipart);
}
namespace std
{
    template <>
    void swap (Number & n1, Number & n2)
    {
        n1.swap (n2);
    }
}
int main (void)
{
    Number n1 = Number::makeComplex (1, 2);
    Number n2 = Number::makeReal (10);
    Number n3 = n1 + n2;
    cout << "Finished\n";

    return 0;
}
~~~

##### 2.0.5 已知的用处

##### 2.0.6 相关的手法

* 桥接模式

* 信封手法

##### 2.0.7 引用

---
#### **3 初始化附着**

##### 3.0.1 目的

在程序开始执行之前，产生用户定义的对象，以在框架中使用。

##### 3.0.1 别名

有构造函数的静态对象

##### 3.0.3 动机

一些应用程序框架，比如图形用户界面框架（如MFC），和一些代理对象（如一些代理对象结构的实现）使用他们自己的消

息循环（也叫做事件循环）来控制整个应用程序。程序员可能没有写应用程序级别的main函数的自由。通常来说，main函数

被深埋在框架里边（如MFC中的AfxWinMain)，缺乏对main函数的控制使得程序员很难编写在程序主消息循环开始之前的初始

化代码。用静态对象的初始化附着手法是在事件循环开始之前执行应用相关初始化代码的方法。

##### 3.0.4 解决方案和示例代码

在C++中，全局作用域中的全局和静态对象在main函数开始执行之前初始化。换句话说这些对象具有静态存储周期。具有静

态生命周期对象初始化的特性，在程序员不能修改框架main函数的时候，可以用来附着对象到框架上。比如下边使用MFC的

这个例子：

~~~
// file : hello.h
class HelloApp : public CWinApp{
public:
    virtual BOOL InitInstance();
}

// file : hello.cpp

#include <afxwin.h>
#include "hello.h"
HelloApp myApp;     // 全局的应用对象

BOOL HelloApp::InitInstance(){

    m_pMainWnd = new CFrameWnd();
    m_pMainWnd->Create(0,"Hello, World!!");
    m_pMainWnd->ShowWindow(SW_SHOW);
    return TRUE;
}
~~~

上边的例子建立了一个标题是“Hello，World"的窗口，这里关键的地方是类型为HelloApp的全局对象myApp，myApp在main

函数开始之前默认初始化。初始化这个对象的另外一个作用，也执行了CWinApp的构造函数。CWinApp类是MFC框架中的，

调用框架中其他几个类的构造函数。在这些构造函数执行的过程中，全局对象附着到了框架。这个对象可以在AfxWinMain

中使用，MFC中的AfxWinMain相当于通常的main函数。这里的HelloApp::InitInstance成员函数只是为了完整性，并不是这

个技巧所必须的。

全局和静态的对象可以以以下几种方式初始化：默认构造函数，带参数的构造函数，函数的返回值初始化，动态初始化等

等。
##### 3.0.5 已知的用处

* 微软MFC中

##### 3.0.6 相关的做法

---

#### **4 代理客户**

##### 4.0.1 目的

控制对一个类的实现的可访问粒度 

##### 4.0.2 动机

C++中的友元声明提供对类内部完全的访问权限。友元声明打破了精心设计的封装。C++中的友元不给私有成员提供任何可

以选择的权限控制。C++中的友元是一种0或全部的地位。比如，下边的例子中，Foo声明了类Bar是它的友元，因此，Bar

可以访问任何Foo中的私有成员。通常这不是想要的，因为这样增加了耦合性，类Bar永远要跟类Foo一起分发。

~~~
class Foo{

private:
    void A(int a);
    void B(float b);
    void C(double c);
    friend class Bar;
}

class Bar{

// 这个类只需要访问Foo中的A和B函数，但是C++中的友元声明给了类Bar类Foo中所有私有成员的访问权限。
}
~~~

提供有选择性访问权限给类的部分成员是我们想要的，因为这样剩余的私有成员可以在需要的时候改变接口

。减小了类之间的耦合性。代理客户手法允许精确地控制一个类给予它友元的访问权限。

##### 4.0.3 解决方案和示例代码

代理客户手法通过增加额外的一层间接性工作。客户类想要控制其内部实现细节的可访问性，指定一个代理

并把代理声明为友元。代理类被巧妙地设计，不像通常地代理类，这里的代理只复制了客户私有接口的子集。

比如，Foo想要控制对自己内部实现的访问粒度，为了更清晰我们还是把这个类命名为Client。Client类想要

它的代理只提供对Client::A 和 Client::B的访问。

~~~
class Client{

private:
    void A(int a);
    void B(float b);
    void c(double c);
    friend class Attorney;
};

class Attorney{

private:
    static void callA(Client& c,int a){
        c.A(a);
    }
    static void callB(Client& c,float b){
        c.B(b);
    }
    friend class Bar;
};

class Bar{

   /// Bar 类现在通过代理类只有对 Client::A 和 Client::B 的访问权限
}；
~~~

代理类提供了对一组(private)成员函数子集的可访问性的限制，代理类有内联的静态成员函数，一个参数是

Client的引用，转交调用Client内部的成员函数。代理类的实现是符合习惯的。所有的成员都是private的，

阻止了其他类直接访问到Client的内部实现。代理类决定哪个类，那个成员函数或者自由函数可以访问Client

内部。代理类把外部的类声明为友元，以便外部类能够访问到代理类内部的成员函数，最终访问Client。没有

代理类，Client声明为友元的类将可以毫无约束地访问Client内部所有成员。

##### 4.0.4 已知的用处

* Boost中的迭代器库(Boost.Iterators Library)
* Boost.Serialization: class boost::serialization::access;

---

#### **5 Barton-Nackman技巧**

##### 5.0.1 目的

在不依赖命名空间或者函数模板重载决议的条件下重载运算符。

##### 5.0.1 别名

这个技巧的发明者称之为限定的模板展开，但这个技巧没有被广泛的使用过。

##### 5.0.3 动机

John Barton 和 Lee Nackman 在1994年提出了这个用法，为了解决当时C++实现的限制。尽管从其最开始的

目的来看，这个技巧已经不再需要，现在的C++标准任然保留对它的支持。

在 Barton 和 Nackman 发明这个技巧的时候，C++不支持函数模板的重载，许多实现甚至都不支持命名空间

。在为类模板重载运算符的时候就遇到了麻烦。考虑如下的代码：

~~~
template<typename T>
class List{
 
    // ……
};
~~~

定义相等运算符最自然的方式是作为非成员函数，在这个类所在的命名空间里。那时的编译器还不支持命名

空间，所以在全局的命名空间。将相等运算符作为非成员函数意味着两个参数要具有相同类型。如果作为成

员函数的话，一个参数就是this指针，这两个参数就不可能有相同类型了。相等运算符可能是下边这样：

~~~
template<typename T>
bool operator==(List<T> const& lft,List<T> const& rgt){
 
    // ……
}
~~~

然而，函数模板那时候不能被重载，将函数放在自己的命名空间里边在有些平台行不通。这意味着只有一个

类可以有这样的相等运算符。给另一个类提供相等运算符会导致歧义性。

##### 5.0.4 解决方案和示例代码

解决方案是将运算符声明为这个类的友元函数。

~~~
template<typename T>
class List{
public:
    friend operator==(const List<T>& lft,const List<T>& rgt){
    
        // ……
    }
};
~~~

模板具现化后就是全局命名空间种的参数是具体类型的非模板函数了。不是模板的函数可以参与重载决议。

这个做法可以被泛化，把友元函数声明在基类中，通过怪异的循环模板模式继承。

~~~
template<typename T>
class EqualityComparable{

public:
    friend bool operator==(const T& lft,const T& rgt){

        return lft.equalTo(rgt);
    }
    friend bool operator!=(const T& lft,const T& rgt){

        return !lft.equalTo(rgt);
    }
};

class ValueType : private EqualityComparable<ValueType>{

public:
    bool equalTo(const ValueType & other) const;
};
~~~

##### 5.0.5 已知的用处

* Boost中的运算符库

##### 5.0.6 相关的技巧

* Barton-Nackman技巧
---

#### **6 从派生类成员构造基类**

##### 6.0.1 目的

用派生类的数据成员初始化基类。

##### 6.0.2 别名

##### 6.0.3 动机

在C++中，基类部分总是在派生类其他成员初始化之前初始化。这样做的原因是派生类的成员可能使用基类部分，因此，所有

基类部分必须在派生类其他成员初始化之前初始化。然而，有时需要从派生类的数据成员去初始化基类。听起来好像跟C++的

规则矛盾，因为传到基类构造函数的参数（派生类的某个成员）必须已被完全初始化。这样导致了环状初始化的问题（无穷

地递归）。

下边从boost库中获得的代码展示了这个问题；

~~~

~~~

##### 6.0.4 解决方案和示例代码

##### 6.0.5 已知的用处

##### 6.0.6 相关的做法

---
#### **8 在对象构造的时期调虚函数**

##### 8.0.1 目的

模仿在对象的构造时期调用虚函数。

##### 8.0.1 别名

构造期间动态绑定。

##### 8.0.3 动机

有时，在派生类对象还在构造的期间，我们想调虚函数。语言的规则明确禁止，因为在派生类被完全初始化之前调用其成

员函数是非常危险的。如果虚函数不去访问派生类对象还没有构造好的成员，是安全的，但没有通用的方法来保证这一点。

~~~
class Base{

public:
    Base();
    ...
    virtual void foo(int n) const; // 通常是纯虚函数
    virtual double bar() const; // 通常是纯虚函数
}; 

Base::Base(){
 
    ...
    foo(42);
    ...
    bar();
}

class Derived: public Base{
    ... 
    virtual void foo(int n) const;
    virtual double bar() const;
};
~~~~

##### 8.0.4 解决方案和示例代码

有许多种方法实现构造函数中动态绑定的效果。每种方法都有优点和局限。这些方法大体上可以分为两类，一类使用两阶

段构造，另一类是一阶段构造。两阶段构造把对象初始化和构造的状态分开了。这样的分离并不总是可行的。对象状态的

初始化被放进了一个独立的函数，既可以是成员函数也可以是普通函数。

~~~
class Base{

public:
    void init(); // 可以是虚函数
    ...
    virtual void foo(int n) const; // 通常是纯虚函数
    virtual double bar() const; // 通常是纯虚函数
};

void Base::init(){
    ...
    foo(42);
    ...
    bar();
}

class Derived : public Base{

public:
    Derived(const char*);
    virtual void foo(itn n) const;
    virtual double bar() const;
};

//使用非成员函数：

template <class Derived,class Parameter p>
std::auto_ptr<Base> factory(Parameter p){
    
    std::auto_ptr<Base> ptr (new D(p));
    ptr -> init();
    return ptr;
}

//成员函数，非模板的版本的实现如下,工厂函数可以可以放进基类，但必须是静态的。

class Base{

public:
    template<class D,class Parameter>
        static std::auto_ptr<Base> Create(Parameter p){

            std::auto_ptr<Base> ptr(new D(p));
            ptr -> init();
            return ptr;
        }
};

int main(){

    std::auto_ptr<Base> b = Base::Create<Derived>("parametr");
}

~~~

派生类的构造函数应该声明为私有的，以防止用户不小心使用了他们。接口应当容易被正确的使用,还记得吗？工厂函数应

该作为派生类的友元函数，除了成员工厂函数，基类可以成为派生类的友元。

* 不使用两阶段构造

怪异的循环模版模式在这种情形下是有用的。

~~~

class Base{

};

template <class D>
class InitTimeCaller : public Base{

protected:
    InitTimeCaller(){
        D::foo();
        D::bar();
    }
};

class Derived : public InitTimeCaller <Derived>{

public:
    Derived() : InitTimeCaller<Derived>(){

        cout<<"Derived::Derived()\n";
    }
    static void foo(){
        cout<<"Derived::foo()\n";
    }
    static void bar(){
        cout<<"Derived::bar()\n";
    }
};
~~~

##### 8.0.5 已知的用处

##### 8.0.6 相关的做法

---

#### **22 空基类优化**

##### 5.0.1 目的

##### 5.0.1 别名

##### 5.0.3 动机

##### 5.0.4 解决方案和示例代码

##### 5.0.5 已知的用处

##### 5.0.6 相关的做法

---

#### **62 安全布尔**

##### 62.0.1 目的

为了对类提供布尔转换，防止表达式中出现不想要的隐式的布尔类型转换. 

##### 62.0.1 别名 

##### 62.0.3 动机

提供布尔转换函数会比好处带来更多的坏处，因为类型转换会在表达式中隐式发生，通常你不希望这样。如果定义了简单

的转换操作符使两种或更多种的对象可以相互比较，类型是安全的，比如：

~~~
sturct Testable{

    operator bool() const{

        return false;
    }
};

struct AnotherTestable{

    operator bool() const{

        return true;
    }
};

int main(){

    Testable a;
    AnotherTestable b;
    if(a == b){
    ...
    }           // 这里的比较完全不是有意的，但是编译器开心地编译了它们。
    if(a<0){
    ...
    }
    return 0;
}
~~~

##### 62.0.4 解决方案和示例代码

~~~
class Testable{
    bool ok_;
    typedef void (Testable::*bool_type)() const;
    void this_type_does_not_support_comparisons() const;
public:
    explicit Testable(bool b=true) : ok_(b){}

    operator bool_type() const{

        return ok_? &Testable::this_type_does_not_support_comparisons : 0;
    }
};

template <typename T>
bool operator !=(const Testable& lhs,const T&){
    lhs.this_type_does_not_support_comparisons();
    return false;
}
template <typename T>
bool operator ==(const Teatable& lhs,const T&){
    lhs.this_type_does_not_support_comparisons();
    return false;
}

class AnotherTestable...
{};

int main(void){
    
    Testable t1;
    AnotherTestable t2;
    if(t1){
    // 可以正常工作
    }
    if(t2 == t1){
    // 编译失败
    }
    if(t1<0){
    // 编译失败
    }

    return 0;
}

~~~

##### 62.0.5 已知的用处

##### 62.0.6 相关的做法

---

#### **66 智能指针**

##### 66.0.1 目的

##### 66.0.1 别名

##### 5.0.3 动机

##### 5.0.4 解决方案和示例代码

##### 5.0.5 已知的用处

##### 5.0.6 相关的做法

---

#### **74 虚构造函数**

##### 5.0.1 目的

##### 5.0.1 别名

##### 5.0.3 动机

##### 5.0.4 解决方案和示例代码

##### 5.0.5 已知的用处

##### 5.0.6 相关的做法

---
