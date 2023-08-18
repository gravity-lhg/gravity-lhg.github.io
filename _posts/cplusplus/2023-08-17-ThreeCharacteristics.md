---
layout: post
title: 'C++ 面相对象的三大特性'
data: 2023-08-17
author: 刘浩光
cover: 'assets/images/header/cplusplus.png'
tags: C++
---

## C++ 三大特性共同构建了面向对象编程的基石，使代码更加安全、可维护和扩展

> **封装 Encapsulation、继承 Inheritance、多态 Polymorphism**

#### 1. 封装（Encapsulate）

封装是一种将数据和操作封装在一个单元内部的机制，通过将数据成员和成员函数封装在类中，可以实现数据的隐藏与保护，防止外部直接访问会修改类的内部数据，外部只能通过 public 的接口来进行操作。封装可以提高代码的可维护性和安全性，同时也支持实现信息的隐藏和模块化设计。

类(class)是 C++ 中最常用于实现封装的方式，它具有三种主要的访问权限，如下：

```C++
class Space {
public:
    // 公共访问权限，类内部、类外部以及派生类中都可以访问
  
protected:
  	// 受保护访问权限，类外部无法访问，仅允许类内部和派生类中访问
  
private:
  	// 私有访问权限，仅允许在类内部访问，类外部和派生类中无法访问
};
// 注：类外部无法直接访问的成员可以通过实现公共接口进行访问
```

#### 2. 继承（Inheritance）

继承是一种通过从现有类派生新类的机制，派生类可以继承现有类的成员变量和成员函数，同时可以新增或重写自己的成员。继承可以实现代码的复用、扩展和层次化设计，是的类之间可以建立层次关系，派生类可以共享基类的特性。

在类的继承中，派生类可以继承基类的成员和访问权限，继承规则如下：

* pravite：成员在派生类中和类外部均不可以访问
    * 基类的公有成员 ==> 派生类的私有成员
    * 基类的保护成员 ==> 派生类的私有成员
    * 基类的私有成员：不会继承到派生类
* protented：成员在派生类中可以访问，但在类外部不可以访问
    * 基类的公有成员 ==> 派生类的保护成员
    * 基类的保护成员 ==> 派生类的保护成员
    * 基类的私有成员：不会继承到派生类
* public：成员在派生类中可以访问，也可以在类外部访问
    * 基类的公有成员 ==> 派生类的公有成员
    * 基类的保护成员 ==> 派生类的保护成员
    * 基类的私有成员：不会继承到派生类

**注：**无论如何的继承方式，基类的私有成员都不会被直接继承到派生类中，因为私有成员只能在基类内部访问，派生类无妨直接访问它们。

公有继承通常用于 "是一个" 关系，保护继承和私有继承则用于 "实现了一个" 或 "实现了一个接口" 的关系。

* “是一个” 关系：当一个类公有地继承另一个类时，这意味着派生类是基类的一种特殊类型。派生类拥有基类的所有公有接口和功能，可以被视为基类的一种扩展。例如，有一个基类 Animal，可以创建一个派生类 Dog，狗是一种动物，它继承了动物的特性和行为
* ”实现了一个“ 或 ”实现了一个接口“ 关系：当一个类保护或私有的继承了另一个类时，这通常表示派生类在实现中使用了基类的接口和功能，但并不打算向外界暴露这种关系。保护继承和私有继承更多的用于实现代码的复用和实现细节的隐藏。例如，有一个基类 Shape，可以使用保护和私有继承来创建一个派生类 Circle，以便在圆的实现中重用 Shape 类的功能，但并不希望直接将 Circle 视为 Shape 类的一种类型

#### 3. 多态（Polymorphism）

多态是一种允许不同对象以统一的方式进行操作的特性，它使得派生类对象可以通过基类的指针或引用来调用相同的方法，从而实现不同的行为。多态可以通过 **虚函数（动态多态性）**或 **函数重载 以及 泛型编程（静态多态性）**来实现，它可以提高代码的灵活性和扩展性，同时支持运行时的动态绑定。

首先来区分一下 **静多态** 和 **动多态**

1. 静多态

    静态多态性是在 **编译的时候** 就确定调用函数的类型，比如函数重载就是一个典型的静多态，函数重载的规则：

    * 函数名称相同
    * 参数的个数不同、参数的类型不同、参数的顺序不同
    * 返回类型不同 **不可以** 构成重载
    * 相同的 scope 

2. 动多态

    动态多态性是 **在运行的时候** 才确定调用哪个函数，动态绑定。实质是派生类重新定义了基类的虚函数，基类的指针根据赋给它的不同派生类的指针，动态的调用属于派生类的该函数，这样的调用在编译期间是无法确定的。此处引入一个概念：**覆盖**。覆盖（override）实质派生类中重新定义基类中的函数，其规则如下：

    * 函数名称相同
    * 函数的参数列表相同
    * 基类函数必须有 virtual 关键字
    * override 关键字并不必须，但使用会增强代码的可读性和安全性
    * 隐藏
        * 如果派生类的函数与基类的函数同名，但参数列表不同，此时无论是否有 virtual 关键字，基类的函数都将被隐藏（注意与 重载 区分）
        * 如果派生类的函数与基类的函数同名，并且参数也相同，但基类的该函数并没有 virtual 关键字，基类的函数被隐藏（注意与 覆盖 区分）

在 C++ 的类中，采用虚函数的实现多态。即在基类的函数前面加上 vistual 关键字，在派生类中重写该函数。其格式如下所示：

```C++
class className {
public:
  	virtual func(patameter list) {
      	// 函数体
    }
};
class childName : public className {
public:
  	func(patameter list) override {		// override 非必须
      	// 派生类重写函数体
    }
};
```

下述时一些虚函数相关的规则：

* 在基类中用 virtual 声明成员函数为虚函数，在类外实现虚函数时，就不必再加 virtual 了
* 派生类中重新定义基类的虚函数称为覆写，要求函数名、返回类型、参数列表全部匹配。只有函数体需要重新定义
* 当一个成员函数被声明为虚函数后，其派生类中完全相同的函数也为虚函数。可以在其前面加上 vitrual 关键字，但也是非必须的
* 派生类中覆写的函数，可以为任意访问类型，依派生类需求决定
* 只有类中的成员函数才能声明为虚函数，因为虚函数仅适用于与有继承关系的类对象，所以普通函数（包括友元函数）不能声明为虚函数
* 静态成员函数不能是虚函数，因为静态成员函数不受限于某个对象
* 内联函数不能是虚函数，因为内联函数不能在运行中动态确定位置。但 `inline virtual func()` 在某些编译器是可以通过的，因为 inline 并不是强制型的关键字，而是建议型的
* 构造函数不能是虚函数，析构函数可以是虚函数，而且通常声明为虚函数（析构函数比较特殊，因为派生类的析构函数跟基类的析构函数名称是不一样的，但是构成覆盖，这里编译器做了特殊处理）
    * 构造函数为什么不能是虚函数？
    * **析构函数可以是虚函数，通常声明为虚函数？**当我们将对象创建在栈上时，作用域结束后程序就会自动调用析构函数，无须我们手动的释放内存。当将对象动态的创建在堆上时（new），作用域结束后程序并不会自动释放对象的内存，这需要我们手动进行内存释放回收（delete），确保内存得到正确的回收。在继承关系中，如果派生类对象创建在栈上，则无需手动释放内存，在作用域结束后，程序会自外向内的（派生类到基类）调用各自的析构函数释放内存。如果派生类对象创建在堆上，作用域结束后程序不会自动释放内存，需要手动释放。这时，当派生类对象通过基类指针或引用进行释放时，如果不将析构函数设置为虚函数，它就只调用基类的析构函数，并不会调用派生类的析构，造成内存泄漏。这种现象应该是多态特性造成的 (注：delete 派生类对象时就不会出现这样的问题)

```C++
#include <iostream>
#include <string>

using namespace std;

class People {
public:
    People(string name, int age) : name(name), age(age) {
        cout << ">> People constructor, initList" << endl;
    }
  	// 将基类析构定义为虚函数，便于基类指针或引用释放派生类对象
    virtual ~People() {
        cout << ">> People destructor" << endl;
    }
    string getName() const { return name; }
    int getAge() const { return age; }
    virtual void introduce() const {
        cout << "This is a people introduce" << endl;
    }
private:
    string name;
    int age;
};

class Student : public People {
public:
    Student(string name, int age, int studentID, float score = 0.0) 
        : People(name, age), studentID(studentID), score(score) {
        cout << ">> Student constructor" << endl;
    }
    ~Student() override {
        cout << ">> Student destructor" << endl;
    }
    int getStudentID() const { return studentID; }
    float getScore() const { return score; }
    void introduce() const override {
        cout << "== Student Introduction ==" << endl;
        cout << "I'm a student, my name is " << getName() << ", my age is "
        << getAge() << ", my studentID is " << getStudentID() << ", my score is "
        << getScore() << endl;
    }
private:
    int studentID;
    float score;
};

class Worker : public People {
public:
    Worker(string name, int age, int workerID, float salary = 0.0)
        : People(name, age), WorkerID(workerID), salary(salary) {
        cout << ">> Worker constructor" << endl;
        }
    ~Worker() override {
        cout << ">> Worker destructor" << endl;
    }
    int getWorkerID() const { return WorkerID; }
    float getSalary() const { return salary; }
    void introduce() const override {
        cout << "== Worker Introduction ==" << endl;
        cout << "I'm a worker, my name is " << getName() << ", my age is "
        << getAge() << ", my workerID is " << getWorkerID() << ", my salary is "
        << getSalary() << endl;
    }
private:
    int WorkerID;
    float salary;
};

// base class reference
void someoneIntroduce(const People& people) {		// 基类引用接受派生类对象，实现多态
    people.introduce();
}

int main() {
  	// 栈上创建派生类 Student 对象
    Student s1("Bob", 20, 12345, 98.7);
    cout << "####### STUDENT INFO #######" << endl;
    cout << "Name: " << s1.getName() << endl;
    cout << "Age: " << s1.getAge() << endl;
    cout << "Student ID: " << s1.getStudentID() << endl;
    cout << "Score: " << s1.getScore() << endl;
    cout << "########### END ###########" << endl;

  	// 堆上动态创建派生类 Student 对象
    Student* s2 = new Student("Bob", 20, 12345, 98.7);
    cout << "####### STUDENT INFO #######" << endl;
    cout << "Name: " << s2->getName() << endl;
    cout << "Age: " << s2->getAge() << endl;
    cout << "Student ID: " << s2->getStudentID() << endl;
    cout << "Score: " << s2->getScore() << endl;
    cout << "########### END ###########" << endl;
		
  	// 栈上创建派生类 Worker 对象
    Worker w1("Tom", 38, 10024, 8000.74);
    cout << "####### WORKER INFO #######" << endl;
    cout << "Name: " << w1.getName() << endl;
    cout << "Age: " << w1.getAge() << endl;
    cout << "Worker ID: " << w1.getWorkerID() << endl;
    cout << "Salary: " << w1.getSalary() << endl;
    cout << "########### END ###########" << endl;
		
  	// 堆上动态创建派生类 Worker 对象
    Worker* w2 = new Worker("Tom", 38, 10024, 8000.74);
    cout << "####### WORKER INFO #######" << endl;
    cout << "Name: " << w2->getName() << endl;
    cout << "Age: " << w2->getAge() << endl;
    cout << "Worker ID: " << w2->getWorkerID() << endl;
    cout << "Salary: " << w2->getSalary() << endl;
    cout << "########### END ###########" << endl;

    // test base class reference
    cout << "####### BASE CLASS REFERENCE #######" << endl;
    someoneIntroduce(s1);
    someoneIntroduce(w1);

    // test base class pointer
    cout << "####### BASE CLASS POINTER #######" << endl;
    People* perplePtr = s2;
    perplePtr->introduce();
    delete perplePtr;		// 基类指针释放派生类对象

    perplePtr = w2;
    perplePtr->introduce();
    delete perplePtr;
    return 0;
}
```

纯虚函数：一个只有声明，没有定义，且被 “初始化” 为 0 的虚函数被称为纯虚函数。含有纯虚函数的类称为抽象基类，不可实例化。即不能创建对象，存在的意义就是被继承，提供族类的公共接口。如果一个类中声明了纯虚函数，而派生类中没有对该函数的定义，则该函数在派生类中仍然为纯虚函数，派生类仍然为纯虚基类。

```C++
class className {
  	virtual returnType func(parameters List) = 0;		// 虚函数的声明
};
```