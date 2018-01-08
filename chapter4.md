# 代理客户

## 4.0.1 目的

控制对类实现细节的访问粒度。 

## 4.0.2 作用 

C++中的友元声明提供对类内部完全的访问权限。友元声明是让人不满的，因为友元声明打破了精心设计的封装。即使是类中的私有成员，C++中的友元特性也不提供任何有选择性的权限控制。C++中的友元是一种要么有，要么没有的选择。比如下边的例子中，Foo声明了Bar类是它的友元，因此，Bar类可以访问任何Foo类中的私有成员。通常这不是我们想要的，因为这样增加了耦合性，这样的耦合意味着Bar类永远要跟Foo类一起发布。

~~~
class Foo{

private:
    void A(int a);
    void B(float b);
    void C(double c);
    friend class Bar;
}

class Bar{

// 这个类只需要访问Foo中的A和B函数，但是C++中的友元声明给了Bar类Foo类中所有私有成员的访问权限。
}
~~~

提供有选择性的权限控制给类的部分成员是我们想要的(比如让私有成员不能被其他类的对象改变，即使是友元类)，这样我们的私有成员可以在需要的时候改变接口。这样帮助减小了类之间的耦合。代理客户技巧可以让一个类精确地控制暴露给它友元类的访问权限。

##### 4.0.3 解决方案和示例代码

代理客户手法通过增加额外的一层间接性工作。想要控制其内部实现细节的可访问性的客户类，指定一个代理类并把代理类声明为友元。代理类被巧妙地设计以作为客户类的代理，不像通常的代理类，代理类仅仅重复了客户类私有接口的子集。比如，考虑Foo类想要控制其他类对其内部实现细节的访问粒度，为了更清晰得说明我们把这个类命名为Client。Client类想要它的代理类只提供对Client::A和Client::B成员的访问。

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

   // Bar类现在通过代理类只有对Client::A和Client::B的访问权限
}；
~~~

代理类限制了对一组成员函数子集的访问，代理类的成员函数都是内联的静态成员函数，其中一个参数是Client类的引用，转发函数调用到Client类对象内部的成员函数。代理类的实现有一些约定俗成的规矩，代理类的所有的实现都是private的，避免了其他类获取到了Client类内部实现的访问权限。代理类决定了其他类，成员函数或者自由函数可以访问(修改)到Client类内部的哪些成员。代理类把外部的类声明为友元，以便外部类能够访问到代理类内部的成员函数，最终调用到Client对象的函数。没有代理类，Client类的友元类将对Client类内部实现具有完全的修改权限。


用多个代理类提供对Client类实现内部不同组成员的访问是可能的，比如AttorneyC类只提供对Client::C成员函数的访问。这个有意思的例子表现出代理类的作用就像中介，给不同的类暴露出不同部分的内部实现。C++中的友元是不能够被继承的，但是如果基类中私有的虚函数可以被调用，派生类中对应的虚函数也是能够被调用的，我们就可以想到类似于这样的设计。在下面的例子中，Base类和main函数使用了代理客户的技巧。派生类中的Func函数被通过多态调用。为了访问派生类的实现细节，使用了相同的技巧。

~~~
#include <cstdio>

class Base {
private:
    virtual void Func(int x) = 0;
    friend class Attorney;
public:
    virtual ~Base() {}
};

class Derived : public Base {
private:
    virtual void Func(int x) {
        printf("Derived::Func\n"); // 即使main函数不是派生类的友元函数，这个函数还是能够被调用
    }
public:
    ~Derived() {}
};

class Attorney {
private:
    static void callFunc(Base & b, int x) {
        return b.Func(x);
    }
    friend int main (void);
};

int main(void) {
    Derived d;
    Attorney::callFunc(d, 10);
}

~~~
#### 4.0.4 已知的使用

* Boost中的迭代器库(Boost.Iterators Library)
* Boost.Serialization: class boost::serialization::access;

---
