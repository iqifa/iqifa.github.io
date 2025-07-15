---
title: Event系统
date: 2024-01-04 00:05:42
categories: [引擎,事件]
---
## 事件系统
### 原理
c++的 **function**可以理解为c中的函数指针，保存触发执行的函数，具体就是写一个委托(c++没有高贵的c#里的)

```c++
    using ListenerID = unsigned int;//保存事件对应的ID

    using CallBack = std::function<void(ArgsTypes...)>;//此处即是代表执行的函数
```
### 实现
在Event中通过unorder_map<key,function>保存事件ID和具体实现，一般使用lambda函数来编写函数实现。
```c++
template<class ...ArgsTypes>//模板类
	ListenerID Event<ArgsTypes...>::AddListenerID(CallBack p_call)
	{
		ListenerID listenerid = ListenerID_now++;//返回创建的监听ID
		m_Callbacks[listenerid] = p_call;//保存传入的函数
		return listenerid;
	}
然后用重载运算符+=
template<class ...ArgsTypes>
	ListenerID Event<ArgsTypes...>::operator+=(CallBack p_call)
	{
		return AddListenerID(p_call);
	}
```
至此，整个实现结束，剩下的就是调用和删除(不写了)
```c++
    template<class... ArgsTypes>
	void Event<ArgsTypes ...>::Invoke(ArgsTypes... p_args)
	{
		for (auto& [key, value] : m_Callbacks)
			value(p_args...);
	}
```
### 运行示例
```c++
        Eventing::Event<> Events;
		Events += []() {
			cout << "Add Event!!!" << endl;
		};
		Events.Invoke();

		Eventing::Event<int,float>Events2;

		Events2 += [](int a, float b) {
			cout << "Events2 a+b" << endl;
			cout << a + b << endl;
		};
        
		Events2 += [](int a, float b) {
			cout << "Events2 a-b" << endl;
			cout << a - b << endl;
		};
		Events2.Invoke(2, 3);
```
![Event Test Image](Event_Test_Image.png)