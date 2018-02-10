# Boost<sup>1</sup>的变形
## 目的
在不物理重组或复制数据项的情况下，交换一对普通的数据类型。
## 别名
## 作用
这个技巧在使用Boost.Bimap<sup>2</sup>数据结构时最被需要。Boost.Bimap是C++的双向映射库。在Bimap<X,Y>中，
X和Y的值都可以用作键值，这种数据结构的实现可以用Boost的变形技巧进行优化。
## 解决方案和示例代码
Boost的变形技巧使用了reinterpret_cast，并且在很大程度上基于这种假设，即两个结构不同，但具有相同数据成员（类型和顺序上）的内存布局是可交换的。尽管C++标准不保证这个假设，但事实上所有的编译器都满足它。并且，如果在使用POD类型的情况下，这个变形技巧是标准的<sup>3</sup>。下面的例子显示了该技巧是怎样工作的。
```
template <class Pair> 
struct Reverse 
{ 
    typedef typename Pair::first_type second_type; 
    typedef typename Pair::second_type first_type;
    second_type second; 
    first_type first; 
}; 

template <class Pair> 
Reverse<Pair> & mutate(Pair & p) 
{ 
    return reinterpret_cast<Reverse<Pair> &>(p);
} 

int main(void) 
{ 
    std::pair<double, int> p(1.34, 5);
    std::cout << "p.first = " << p.first << ", p.second = " << p.second << 
    std::endl; 
    std::cout << "mutate(p).first = " << mutate(p).first << ", mutate(p).second = 
    " << mutate(p).second << std::endl; 
} 
```
给定一个仅包POD数据类型的std::pair<X,Y>对象，Reverse<std::pair<X,Y>>的布局和大多数编译器上的布局相同，Reverse模板在没有逆转数据的情况下逆转了数据成员的名称。一个辅助的mutate函数用来轻松地构建一个Reverse<Pair>的引用，这可以被看做原pair对象的一个视图。上述程序的输出结果证实了可以在不重新组织数据的情况下输出反向视图：p.first=1.34,p.second=5,mutate(p).first=5,mutate(p).second=1.34。
## 已知的使用
Boost.Bimap
## 相关的技巧
## 参考
Boost.Bimap library, Matias Capeletto<sup>4</sup>
***
1.Boost是一个可移植C++库  
2.http://www.boost.org/doc/libs/1_43_0/libs/bimap/doc/html/index.html 
3.http://beta.boost.org/doc/libs/1_43_0/libs/bimap/test/test_mutant.cpp  
4.https://en.wikibooks.org/wiki/Category%3A



