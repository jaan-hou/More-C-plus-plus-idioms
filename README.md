# README

**《More C++ idioms》**翻译

---

**译者的话：**

很喜欢这本书，但是没有中文版。便有了翻译这本书的想法，时间，水平有限，有任何疏漏，欢迎大家 pull request : )

---

**目录**

1. 不使用 operator & 取地址
2. 

---

#### **1 取址器**

##### 1.0.1 目的

在不使用取地址运算符的情况下，取得对象的地址。

##### 1.0.1 别名

##### 1.0.3 动机

C++允许重载取地址运算符，取地址运算符的返回类型却不一定是对象的实际地址。重载取地址运算符的做法是非常有争议的，

但语言无疑允许这样做。取址器是不使用取地址运算符获得对象实际地址的一种方法。

在下面的例子中，由于取地址运算符是类私有的，main 函数会编译失败。即使没有私有的访问控制，返回类型double也不能够自动转成指针类型。

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

取址器用类型转换获得一个对象的地址。


