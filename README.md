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
5. ---
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

在下面的例子中，由于取地址运算符是类私有的，main 函数会编译失败。即使没有私有的访问控制，返回类型double也不能

够自动转成指针。

~~~
class nonaddressable{
public:
    typedef double useless_type;
private:
    useless_type operator&() const;
};

int main(){

    noaddressable na;
    noaddressable * naptr = &na;    // Compiler error here.
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

    noaddressable na;
    noaddressable * naptr = &na;
}
~~~

##### 1.0.5 已知的用法

* Boost 中的 addressof 

在C++11的 <memory> 头文件中已经定义了这个函数。

##### 1.0.6 相关的做法

##### 1.0.7 引用

---

#### **2 代数抽象基类**

##### 2.0.1 目的

为了隐藏单个范型抽象

##### 2.0.1 别名

* 状态法

##### 2.0.3 动机

在纯的像 Smalltalk 面向对象语言中，变量像标签那样在运行时绑定到对象。变量名绑定对象就像是给对象贴上标签.这些

语言中的赋值就像是从一个对象上把标签扯下来贴到另一个对象上。然而，在 C 和 C++ 中，变量不是对象的标签而是实际

的地址或者偏移。赋值不是重新变量名重新绑定对象，赋值意味着用旧值覆盖新值。代数抽象基类是在C++中利用委托和多态

来模拟变量跟对象的绑定。代数抽象基类在实现中使用了“信封手法”，代数抽象基类的目的，就可以够写出如下的代码:

~~~
Number n1 = Complex (1,2);  // n1是一个复数
Number n2 = Real (10);      // n2是一个实数
Number n3 = n1 + n2;    // n3是n1和n2的和
Number n2 = n3;     // "重新贴标签" 
~~~

##### 2.0.4 解决方案和示例代码

代数抽象基类的实现代码如下。

~~~
template <class T>
T * addressof(T& v){

    return reinterpret_cast<T*>(& const_cast<char&>(reinterpret_cast<const volatile char&>(v)));
}

int main(){

    noaddressable na;
    noaddressable * naptr = &na;
}
~~~

##### 2.0.5 已知的用处

##### 2.0.6 相关的做法

* 

* 信封手法

##### 2.0.7 引用

---
#### **3 取址器**

##### 3.0.1 目的

##### 3.0.1 别名

##### 3.0.3 动机

##### 3.0.4 解决方案和示例代码

在C++中，全局作用域中的全局和静态对象在main函数开始执行之前初始化。换句话说这些对象具有静态存储周期。

##### 3.0.5 已知的用处

* 微软MFC中

##### 3.0.6 相关的做法

---

#### **4 代理客户**

##### 4.0.1 目的

控制对一个类的实现的可访问粒度 

##### 4.0.1 别名

##### 4.0.3 动机

##### 4.0.4 解决方案和示例代码

##### 4.0.5 已知的用处

---

#### **5 取址器**

##### 5.0.1 目的

##### 5.0.1 别名

##### 5.0.3 动机

##### 5.0.4 解决方案和示例代码

##### 5.0.5 已知的用处

##### 5.0.6 相关的做法

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
