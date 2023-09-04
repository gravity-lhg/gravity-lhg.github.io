---
layout: post
title: '设计模式之单例模式 (Singleton)'
data: 2023-08-31
author: 刘浩光
cover: 'assets/images/header/designPatterns.png'
tags: C++ DesignPattern
---

## 单例模式在实际应用中常用于管理全局资源、配置信息、xx池实现、日志记录等

> 单例模式（Singleton）

单例模式是一种创建型设计模式，它确保一个类只有一个实例，并提供一个全局访问点以访问这个实例。这样的设计可以在整个应用程序中共享一个共享资源或状态，以及避免重复创建相同的对象。单例模式在实际应用中常用于管理全局资源、配置信息、xx池实现、日志记录等。

一般的，单例模式的设计应该注意以下几点：

* 将构造函数、析构函数设置为 private，删除默认的复制构造和赋值操作符重载（可选）
* 在 public 下提供外部可以访问的静态接口，**静态** 是因为可以通过类名来进行访问

单例模式通常的实现方式有：饿汉式、懒汉式 和 局部静态变量

### 1. 饿汉式（Eager Inirialization）

在程序启动或类被加载的时候就创建一个单例实例，即在实例化类的静态成员时直接初始化单例对象。这种方式简单直接，但可能会造成不必要的资源浪费，因为即使程序可能没有使用到该单例对象，它也会被提前创建。

```C++
class Singleton {
public:
    // 饿汉式, 类加载的时候创建实例，整个程序生命周期都在
    // 程序结束时自动释放, 无需显式提供销毁方法
    static Singleton* getInstance() {
        return instance;
    }

private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;

    static Singleton* instance;
};

Singleton* Singleton::instance = new Singleton();
```

### 2. 懒汉式（Lazy Initialization）

只有在实际使用时才创建单例对象。懒汉式在需要的时候才创建实例，避免了不必要的资源浪费，**但需要考虑多线程环境下的线程安全性**。

```C++
class Singleton {
public:
    // 懒汉式，需要的时候才创建，可能存在线程安全问题
    static Singleton* getInstance() {
        if(!instance) {
            instance = new Singleton();		// 可能导致线程安全的位置
        }
        return instance;
    }
    // 提供销毁实例的方法，便于能够正确手动的释放资源
    static void destoryInstance() {
        if(!instance) {
            delete instance;
            instance = nullptr;
        }
    }

private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;

    static Singleton* instance;
};

Singleton* Singleton::instance = nullptr;
```

在使用如下的多线程测试，显现出了安全性的问题，即创建了多个实例

```C++
void threadFunc() {
    Singleton& instance = Singleton::getInstance();
    cout << "Thread ID: " << std::this_thread::get_id() << ", "
            << "Singleton address: " << &instance << endl;
}

int main() {
    // thread test for lazy impl
    thread t1 = thread(threadFunc);
    thread t2 = thread(threadFunc);
    t1.join();
    t2.join();
    return 0;
}

result:
Thread ID: 0x16ff13000, Singleton address: 0x100404090
Thread ID: 0x16fe87000, Singleton address: 0x100504080
```

以下的代码展示了通过对创建过程加互斥锁来解决线程安全的问题：

```C++
#include <mutex>

class Singleton {
public:
    // 懒汉式，需要的时候才创建，可能存在线程安全问题
    static Singleton& getInstance() {
        std::lock_guard<std::mutex> lock(mtx);	// 对下面的操作加互斥锁
        if(!instance) {
            instance = new Singleton();
        }
        return *instance;
    }
    
    // 提供销毁实例的方法，便于能够正确的释放资源
    static void destoryInstance() {
        if(!instance) {
            delete instance;
            instance = nullptr;
        }
    }

private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;

    static Singleton* instance;
    static std::mutex mtx;
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;
```

### 3. 局部静态变量（Local Static Varibale）

C++11 及以后的标准保证了局部静态变量初始化的线程安全性。**局部静态变量 **是指在函数内部定义的静态变量，其具有以下特点：

* **生命周期延长：**局部静态变量的生命周期不仅限于函数的执行过程，在整个程序运行期间都存在，直到程序结束才会销毁。这使得局部静态变量可以在多次函数调用之间保持其值
* **初始化：**局部静态变量在首次执行到该变量声明的代码行时进行初始化，而且只会进行一次。之后每次函数调用都会使用上一次函数调用结束时的值
* **线程安全性：**在C++11及以后的标准中，局部静态变量的初始化在多线程环境下是线程安全的。编译器会保证初始化只会发生一次，即使在多线程环境下也不会出现竞争条件
* **作用域：**局部静态变量的作用域仍然是限于其所在的函数内部，只能在声明它的函数中访问

采用局部变量实现单例模式，代码如下：

```C++
class Singleton {
public:
    // 局部静态变量, 在第一次调用时被初始化一次, C++11之后是线程安全的
    static Singleton& getInstance() {
        static Singleton instance;	// 局部静态变量
        return instance;
    }

private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;
};
```

**注：**在设计单例模式的时候，在很多情况下其实是可以不用去考虑到线程安全的问题的，不用进行繁琐的加锁等操作，可以在工程实现层面解决，如：在主线程中创建实例，子线程中调用实例方法，这样可以规避多线程创建实例安全性的问题

### 4. 采用模版类来实现单例模式

```C++
template<typename T>
class Singleton {
public:
    static T* getInstance() {
        if (m_instance == nullptr) {
            m_instance = new T();		// 懒汉式实现
        }
        return m_instance;
    }
private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton<T>&) = delete;
    Singleton<T>& operator = (const Singleton<T>&) = delete;

    static T* m_instance;
};

template<typename T>
T* Singleton<T>::m_instance = nullptr;

class A {
    friend class singleton::Singleton<A>;		// 友员类，用于调用构造函数创建对象
private:
    A() {}
    ~A() {}
    A(const A&) = delete;
    A& operator = (const A&) = delete;
};

int main() {
  	Singleton<A>::getInstance();
		return 0;
}
```

**注：**在类的外部是无法访问 private 下的成员与函数，在此将 `singleton<A>` 声明为类 A 的友员类，在不破坏单例模式的前提下构造实例。