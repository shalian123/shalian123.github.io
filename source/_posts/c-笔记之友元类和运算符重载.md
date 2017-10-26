---
title: c++笔记之友元类和运算符重载
date: 2017-06-14 18:59:00
tags: [友元类，运算符重载]
categories: c/c++

---
之前在看设计模式时遇到了一些问题，一直没有时间来整理，现在终于研一课程全部结束了，因此回过头来整理一下。

1.friend ostream& operator <<（ostream& out，const Money& x）是什么意思？
答：对"<<"运算符的重载。
**一般我们用的"<<"只能输出整型、实型等普通类型**。
要**想输出类的类型，则必须对"<<"进行重载**，**其中一个参数为类类型对象**。
为了**方便对对象内部数据的操**作，设置为friend友元函数。
为了能达到cout<<对象<<对象<<endl;的连续输出对象的效果，**设置返回类型为引用**。

**参数**：第一个为输出流对象。第二个为要输出的对象（**为了防止产生临时对象、提高程序的效率，将参数设置为引用类型，但引用类型又不想让它改变实参的值，所以设置为const**）。
例子：
friend ostream& operator<<(ostream& o,const Person& p)（**输出运算符重载**）
{
 o<<”名字”<<p.name<<”年龄”<<p.age<<”身高”<<p.height<<”体重”<<p.weight<<”.\n”;
 return o;
}

friend istream& operator<<(istream& i, Person& p)（**输入运算符重载**）
{
i>>p.name>>p.age>>p.height>>p.weight;
 return i;
}

2.friend关键字
c++中有个friend关键字，它能让被修饰的对象冲破本class的封装特性，从而能够访问本class的私有对象。
简单来讲，就是：
•	如果你在A类中，申明了函数func()是你的friend，那么func()就可以使用A类的所有成员变量，无论它在什么地方定义的。
•	如果你在A类中，申明了B类是你的friend，那么B类中的方法就可以访问A类的所有成员变量。


```
#include<iostream>
using namespace std;

class A {
public:
    A() {
        password = 1111;
        birthday = 808;
    }

    ~A() { }

    friend int func(A a); // 向c++表示，int func(A a)是我的朋友，所以它可以使用我的所有东西。
    friend class B;       // 向c++表示，class B是我的朋友，所以它可以使用我的所有东西。

private:
    int password;
    int birthday;
};

int func(A a) {
    cout << a.password << " and " << a.birthday << endl; //可以访问
    a.password = 1;                                      //甚至可以修改
    cout << a.password << endl;
    return 0;
}

class B {
public:
    B() { }

    ~B() { }
    // 因为在A类中已经声明了B类是它的朋友，所以B类中方法就可以访问A类的私有变量了
    void show(A a) {
        cout << "your account is " << a.account << " and with pass: " << a.password << endl;
    }

private:
};

int main() {

    A a;
    func(a);
    B b;
    b.show(a);
    system("pause");
}

```
加上运算符重载：
```
#include<iostream>
using namespace std;

class A {
public:
    A() {
        password = 1111;
        account = 808;
    }

    ~A() { }

    friend ostream& operator << (ostream &os , A a);

private:
    int password;
    int account;
};

ostream& operator << (ostream &os , A a) {
    os << "your account is " << a.account << " and with pass: " << a.password << endl;
    return os;
}

int main() {

    A a;

    cout << a;

    system("pause");
}

```