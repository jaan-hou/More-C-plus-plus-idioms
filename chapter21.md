# 奇特的递归模板模式
## 目的
使用派生函数作为模板参数来特化基类。
## 别名
- CRTP(Curiously Recurring Template Pattern)
- Mixin-from-above
## 作用
提取出基类中独立于类型的，但类型可定制的功能，将该接口/属性的行为混合入派生类，为派生类定制。
## 解决方案和示例代码
在CRTP技巧中，类T继承了一个专用于类T的模板。
```
class T : public X<T> {…}; 
```
这当且仅当X<T>的大小与T无关时合法。特别的，基类模板将利用成员函数体（定义）直到被声明时才会实例化的事实，并将会通过static_cast在它自己的成员函数中使用派生类的成员。例：
```
template <class Derived> 
struct base 
{ 
    void interface() 
    { 
        // ... 
        static_cast<Derived*>(this)->implementation(); 
        // ... 
    } 
    
    static void static_interface() 
    { 
        // ... 
        Derived::static_implementation(); 
        // ...
    } 
    // The default implementation may be (if exists) or should be (otherwise) 
    // overridden by inheriting in derived classes (see below) 
    void implementation(); 
    static void static_implementation();
}; 

```
```
// The Curiously Recurring Template Pattern (CRTP) 
struct derived_1 : base<derived_1> 
{ 
    // This class uses base variant of implementation 
    //void implementation(); 
    
    // ... and overrides static_implementation 
    static void static_implementation(); 
}; 

struct derived_2 : base<derived_2> 
{ 
    // This class overrides implementation 
    void implementation(); 
    
    // ... and uses base variant of static_implementation 
    //static void static_implementation(); 
};
```
## 已知的使用
Barton-Nackman技巧(见第5章）
## 相关的技巧
- 参数化的基类技巧(见第52章)
- Barton-Nackman技巧
## 参考
维基百科上的奇特的递归模板模式  
http://en.wikipedia.org/wiki/Curiously_Recurring_Template_Pattern

