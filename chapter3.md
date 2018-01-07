# 初始化附着
## 目的  
在程序开始执行前，将用户自定义的对象附着到框架  
## 别名  
带有构造器的静态对象(Static-object-with-constructor)  
## 作用  
&#8195;特定的应用程序框架，例如GUI框架（例.Microsoft MFC)和对象请求代理（例.一些corba实现）使用它们自己的内部消息循环(又名事件循环)
来控制整个应用。应用编程人员或许有也或许没有编写应用级主函数的自由。通常情况下，主函数被深埋在应用框架中（例.MFC的AfxWinMain）。无法访问主函数，使得程序员无法在主循环开始前编写特定的初始化代码。初始化附着是一种在主循环开始执行前执行特定于该应用程序的代码的方式。
## 解决方案和示例代码
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
## 已知的使用  
MFC
## 相关的习语
## 参考
