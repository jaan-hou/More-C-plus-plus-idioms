# 取址器
## 作用
取得重载了取地址运算符(operator &)的类的对象的地址。
## 原因 
C++允许重载取地址运算符，因此取地址运算符的返回值不一定是对象的地址。尽管这样的做法极具争议，但语言自身并没有禁止这样做。取址器技巧是在重载了取地址运算符,或取地址运算符不可访问的情况下取得对象实际地址的一种方法。

在下面的例子中，由于取地址运算符是私有的，main函数会编译失败,即使取地址运算符可以访问，但从它的返回类型double转换到指针没有任何意义。

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

## 解决方案和示例代码
取址器用一系列的类型转换获得一个对象的地址。

~~~
template <class T>
T * addressof(T& v){

    return reinterpret_cast<T*>(&const_cast<char&>(reinterpret_cast<const volatile char&>(v)));

}

int main(){

    nonaddressable na;
    nonaddressable * naptr = &na;
}
~~~

## 用到的地方 

* Boost 中的 addressof 

  在C++11的 <memory> 中已经包含了这个函数。