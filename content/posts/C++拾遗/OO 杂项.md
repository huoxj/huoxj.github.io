---
title: "OO 杂项"
date: 2024-11-20T10:34:25-08:00
categories: 
- "C++拾遗"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/universal/background1.jpg
summary: "```cpp class T {}; class T { T(); // 构造函数 T(const T&); // 拷贝构造 ~T(); // 析构函数 T& operator=(const T&);..."
---

## 空类默认提供的方法

```cpp
class T {};  
class T {  
	T();                            // 构造函数
	T(const T&);                    // 拷贝构造
	~T();                           // 析构函数
	T& operator=(const T&);         // 拷贝赋值
	T *operator &();                // 取地址
	const T* operator &() const;    // 取地址 const 重载版
};
```

最后两个函数比较怪。一个是返回非 const 的指针，另一个返回 const 的。在调用时，编译器如何决定调用哪个呢？

注意函数后的 const 修饰符。这两个函数实际应该长这样：

```cpp
T *operator &(T *this);
const T* operator &(const T *this);
```

是 this 指针的功劳。调用方是哪种就执行哪个。