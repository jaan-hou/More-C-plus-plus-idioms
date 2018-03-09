# 功能查询
## 目的
在运行时检测某个对象是否支持某个接口。
## 别名
## 作用
将接口和实现分离是面向对象编程的一个好的实践。在C++中，接口类<sup>1</sup>被用来将接口和实现分离，并应用运行时的多态性调用任何抽象的公共方法。在接口用法扩展技巧中，一个实体类可以实现多个接口，正如下面所展示的。
```
class Shape { // An interface class 
   public: 
     virtual ~Shape(); 
     virtual void draw() const = 0; 
     //...
}; 
class Rollable { // One more interface class 
   public: 
     virtual ~Rollable(); 
     virtual void roll() = 0; 
}; 
class Circle : public Shape, public Rollable { // circles roll - concrete class 
     //... 
     void draw() const; 
     void roll(); 
     //... 
}; 
class Square : public Shape { 
    // squares don't roll - concrete class 
    //... 
    void draw() const; 
    //...
}
```
现在我们被给予了一个包含多个指向抽象的Rollable类的指针的容器，正如在接口类技巧中提到的那样，我们可以简单地调用每个指针上的roll函数。
```
std::vector<Rollable *> rollables; 
// Fill up rollables vector somehow. 
for (vector<Rollable *>::iterator iter (rollables.begin()); 
     iter != rollables.end(); 
     ++iter)
     {
     iter->roll();
     } 
```
有时，不能事先知道某个类是否实现了某个特定的接口，这样的情况通常在一个类继承了多个接口的情况下出现。为了在运行时发现接口的存在或缺失，可以使用功能查询。
## 解决方案和示例代码
在C++中，功能查询通常被表示为不想关类型之间的dynamic_cast。
```
Shape *s = getSomeShape(); 
if (Rollable *roller = dynamic_cast<Rollable *>(s)) 
   roller->roll(); 
```
dynamic_cast的使用通常被称为cross-cast，因为它尝试在层次结构中进行交叉的转换，而不是顺着或逆着层次结构。在我们举例的shapes和rollables层次结构中，Rollable的dynamic_cast仅用于Circle而不用于Square，因为后者不继承Rollable接口。  
过度使用功能查询是糟糕的面向对象设计的象征。
## 别名
无环的访问者模式<sup>2</sup>——Robert C.Martin
## 相关技巧
- 接口类<sup>3</sup>
- 内部类<sup>4</sup>
## 参考
Capability Queries - C++ Common Knowledge by Stephen C. Dewhurst 
***
1.见35.0.7章  
2.http://www.objectmentor.com/resources/articles/acv.pdf   
3.见35.0.7章  
4.https://en.wikibooks.org/wiki/More%20C%252B%252B%20Idioms%2FInner%20Class  
5.https://en.wikibooks.org/wiki/Category%3A

