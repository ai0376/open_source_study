# [JThread][code]
[code]:https://github.com/mjrao/JThread
## 介绍
在许多项目中，我们会经常使用到线程．为了在unix和windows平台下使用一样的代码，我决定在这些平台上将已经存在的线程函数封装成一个简单的类．

JThread 包非常的简单：目前，它只包含三个类：分别是JThread, JMutex和JMutexAutoLock,从它们的名称上可以想到：JThread表示一个线程，JMutex表示一个互斥体，线程类仅仅包含很基本的函数，例如：开始或结束一个线程．

## 使用
下面是JThread，JMutex和JMutexAutoLock类的描述．注意函数的返回类型int总是返回０及大于０的值表示成功，负值表示出现了问题．

### JMutex
下面是JMutex类的定义．在你使用这种类型的实例之前，你必须首先调用`init`函数.你可以通过`IsInitialized`的返回值检测互斥体是否已经初始化．初始化之后，通过调用Lock和Unlock函数可以使互斥体锁定和解锁．
```
class JMutex
{
public :
	JMutex();
	~JMutex() ;
	int Init();
	int Lock();
	int UnLock();
	bool IsInitialized();
};
```
### JMutexAutoLock
下面是JMutexAutoLock类的定义．它的目的是更容易的实现线程安全函数，不用担心什么时候位互斥体解锁．
```
class JMutexAutoLock
{
public:
	JMutexAutoLock(JMutex &m);
	~JMutexAutoLock();
};
```
下面的代码演示了这个类的使用:
```
void MyClass::MyFunction()
{
	JMutexAutoLock autoLock(m_myMutex);
    // 在这做互斥保护
}
```
当autolock变量被创建时，它自动锁定结构体中指定的互斥体m_myMutex．autoLock变量的析构函数确保lock被释放.

### JThread
为了创建你自己的线程，你不得不从JThread继承一个类，下面是描述．在你的继承类里，你必须实现一个`Thread`成员函数，这将要在新的线程里执行．在你的自己的线程实线中应当调用`ThreadStarted`.

为了启动你的线程，你只要简单的调用`Start`函数．你需要在你自己的`Thread`函数中调用`ThreadStarted`．这样，当你的`Start`函数完成滞后，你才能确保你的Thread能真正的立即运行起来．

你可以通过调用`IsRunning`函数来检测你的线程是否正在运行．如果线程已经完成，通过调用`GetReturnValue`检测到返回值，最后通过`Kill`函数结束它．

你要小心的使用`Kill`函数：当调用kill时，一个互斥体正在工作(例如一个网络互斥体)，这个互斥体可能在锁状态，这将导致其他线程阻塞．当你确保线程在一个循环中不能被结束的时候，才可以使用`Kill`函数
```
class JThread
{
public:
	JThread();
	virtual ~JThread();
	int Start();
	int Kill();
	virtual void * Thread() = 0;
	bool IsRunning();
	void * GetReturnValue();
protected:
	void ThreadStarted();
};
```
___
*[原文文档](http://research.edm.uhasselt.be/jori/jthread/manual.pdf)，翻译出入的地方，欢迎提交pull request或　mjrao@foxmail.com　联系我！*