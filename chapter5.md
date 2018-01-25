# Barton-Nackman技巧
## 用法
支持不依赖于名空间或函数模板重载解决方案来重载操作符。
## 别名
发明者最初将其称为被限制的模板扩展，尽管这个术语并没有被广泛使用。
## 原因
John Barton和Lee Nackman在1994年最先发表了这个术语，当时是为了解决可用的C++实现的局限。
尽管现在的标准不再需要它最初的目的，但它仍然支持它。在Barton和Nackman最初创造这个术语的时候，
C++并不支持函数模板的重载，很多实现也不支持名空间。这导致定义类模板的重载操作符时出现了问题。
考虑下面的类：
```
template<typename T>
class List {
// ...
};
```
定义等号最自然的方法是作为名空间作用域中的非成员函数（既然那时的编译器不支持名空间，因此在全局作用域中）。
将==操作符定义成非成员函数意味着两个参数是对称的，并非其中一个是指向该对象的this指针。这样的等于操作符可能会是这样的：
```
template<typename T>
bool operator==(List<T> const & lft, List<T> const & rgt) {
//...
}
```
然而，既然函数模板在那时无法被重载，并且将函数放入它的名空间中无法在所有平台上工作，这意味着只有一个类能拥有这样的等于操作符。对第二种类型做相同的事情将会造成歧义。
## 解决方案和示例代码
解决方案是将类中的操作符重载定义成友元函数。
```
template<typename T> 
class List { 
    public: 
        friend bool operator==(const List<T> & lft, 
                                   const List<T> & rgt){
             // ... 
        } 
};
```
实例化模板会导致一个非模板函数被注入到全局范围内，参数类型也会变成具体的、固定的类型。这个函数和其他非模板函数一样，能够通过函数重载解决方案而被选择。  
这个实现可以通过将友元函数作为基类的一部分提供来实现一般化，该基类是经由一个奇特的递归模板模式（见第21章）继承而来的。
```
template<typename T> 
class EqualityComparable { 
    public: 
        friend bool operator==(const T & lft, const T & rgt) { return 
    lft.equalTo(rgt); } 
        friend bool operator!=(const T & lft, const T & rgt) { return 
    !lft.equalTo(rgt); } 
    };
```
```   
class ValueType : 
    private EqualityComparable<ValueType> { 
public: 
    bool equalTo(const ValueType & other) const; 
};
```
## 已知的使用
Boost.Operators library   
http://www.boost.org/doc/libs/1_50_0/libs/utility/operators.htm 
## 相关的技巧
奇怪的递归模板模式(详见第21章)。
## 参考
维基百科上的Barton-Nackman技巧  
http://en.wikipedia.org/wiki/Barton%E2%80%93Nackman_trick