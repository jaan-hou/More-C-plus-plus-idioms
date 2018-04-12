# 清理和最小化
## 目的
清理一个容器并最小化该容器容量。
## 别名
它有时被称作临时交换。
## 作用
标准库容器分配的内存经常比它里面的实际元素的数量多，这样的策略使得一个容器的大小增长时，能节省一些内存分配。另一方面，当容器大小减小时，经常会有被遗留的容量。容器中剩下的容量通常是对内存资源不必要的浪费，清理和最小化方法通常被用来清理一个容器并使额外资源减少到0，以此来节省内存资源。
## 解决方案和示例代码
清理和最小化方法就和下面的例子一样简单：
```
std::vector <int> v; 
//... Lots of push_backs and then lots of remove on v. 
std::vector<int>().swap (v); 
```
前面一半代码，std::vector<int>() 创建了一个整型的临时容器,它保证分配了一个大小为0的或者实现上的最小内存。代码的第二部分使用非丢弃的交换方法将v和临时容器交换，这是效率很高的。交换过后，由编译器创造的临时容器超出了范围，而由v持有的内存块则被释放掉了。
## C++14中的解决方案
C++11之后，一些容器声明shrink_to_fit()函数，例如vector, deque, basic_string。shrink_to_fit()是一个非绑定请求，用来将capacity()减少到 size()，因此，clear()和shrink_to_fit()是清理和最小化的非绑定请求。
## 已知的使用
## 相关的技巧
- Shrink-to-ﬁt<sup>1</sup>
- Non-throwing swap<sup>2</sup>
## 参考
- Programming Languages — C++<sup>3</sup> draft standard.
***
1.Chapter 64.0.6 on page 217  
2.Chapter 47.0.7 on page 163  
3.http://open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2800.pdf