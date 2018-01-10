# 初始化附着
## 目的  
在程序开始执行前，将用户自定义的对象附着到框架。
## 别名  
带有构造器的静态对象（Static-object-with-constructor）。
## 作用  
&#8195;特定的应用程序框架，例如GUI框架（例.Microsoft MFC）和对象请求代理（例.一些corba实现）使用它们自己的内部消息循环（又名事件循环）
来控制整个应用。应用编程人员或许有也或许没有编写应用级主函数的自由。通常情况下，主函数被深埋在应用框架中（例.MFC的AfxWinMain）。无法访问主函数，使得程序员无法在主循环开始前编写特定的初始化代码。初始化附着是一种在主循环开始执行前执行特定于该应用程序的代码的方式。
## 解决方案和示例代码  
&#8195;在C++中，全局对象和全局名空间中的静态对象在主函数开始执行前被初始化。这些对象是拥有静态存储周期的对象。如果编程者不被允许编写自己的主函数，可以使用拥有静态存储周期的对象来将一个对象附着到系统。例如以下使用MFC的（尽可能小的）示例。
```
///// File = Hello.h
class HelloApp: public CWinApp 
{ 
public: virtual BOOL InitInstance (); 
};
```
```
///// File = Hello.cpp
#include <afxwin.h> 
#include "Hello.h" 
HelloApp myApp; // Global "application" object 
BOOL HelloApp::InitInstance () 
{
m_pMainWnd = new CFrameWnd();
m_pMainWnd->Create(0,"Hello, World!!");
m_pMainWnd->ShowWindow(SW_SHOW); 
return TRUE;
} 
```
&#8195;上面的例子仅仅创建了一个名为“Hello,World!”的窗口。此处需要重点关注的是HelloApp的全局对象myApp。myApp在主函数执行前被默认初始化了。作为初始化的侧面影响，CWinApp的构造函数也被调用了。CWinApp类是框架中的一部分，并且调用了框架中其他几个类的构造函数。在这些构造函数的执行过程中，这个全局对象就被附着到了框架。该对象后续会被AfxWinMain检索，AfxWinMain相当于MFC的常规main函数。成员函数HelloApp::InitInstance只是为了完整而显示，并不是这个技巧的组成部分，该函数在AfxWinMain开始执行后被调用。  
&#8195;全局和静态对象能以多种方式初始化：默认构造函数、带参构造函数、通过函数返回值初始化、动态初始化等等。  
> 警告  
在C++中，同一个编译单元中的对象根据定义顺序创建。然而，位于不同编译单元中的静态存储周期对象的初始化顺序没有被很好地定义。某一名空间中的对象在该名空间咋中的任一函数/变量被访问到之前创建。这可能会也可能不会在主函数开始前。析构顺序和初始化顺序正相反，然而初始化顺序本身并没有被标准化。由于这个未定义的行为，当一个静态对象的构造函数使用另一个没有被初始化的静态对象时，会出现静态初始化顺序问题。这个技巧依赖于拥有静态存储周期的对象，因此很容易进入这个陷阱。
## 已知的使用
MFC
## 相关的技巧
## 参考
Proposed C++ language extension to improve portability of the Attach by Initialization idiom1 
