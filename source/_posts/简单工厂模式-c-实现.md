---
title: 简单工厂模式(c++实现)
date: 2017-06-14 19:20:45
tags: [c++,设计模式]
categories: 大话设计模式

---
面向对象设计有封装、继承与多态三大特性。
多态的实现可以采用：虚函数，抽象类，模板等来实现。
用简单工厂模式设计一个计算器：
这里**我采用的是虚函数的重载**，也可以用抽象类来实现的：让计算数据和过程与显示结果分开体现了封装的特性；为了程序的可扩展性，比如以后要加入开根号的运算而不去重写整个类，**可以将每个操作符单独作为一个类**，他们都继承共同的父类------**抽象类：Operation类**，这里体现了**继承**和**多态**的特性。
如何让计算器知道该用哪个操作符的类呢，这里用到了“简单工厂模式”，也就是，**到底要实例化哪个操作符类**，应该考虑**用一个单独的类来做这个创造实例的过程，这就是工厂。**

```
#include<iostream>
#include<exception>
using namespace std;

//Operation运算类  
class Operation
{
private:
	double _numberA;
	double _numberB;

public:
	Operation(){}//构造函数，可以不用自己定义
	Operation(double numA, double numb) :_numberA(numA), _numberB(numb)
	{
	}
	virtual double GetResult()
	{
		double result = 0;
		return result;
	}
	void set_numberA(double numA)
	{
		_numberA = numA;
	}
	void set_numberB(double numB)
	{
		_numberB = numB;
	}
	double get_numberA()
	{
		return _numberA;
	}
	double get_numberB()
	{
		return _numberB;
	}

};

//加减乘除类  
class OperationAdd :public Operation      //默认的是私有继承
{
public:
	OperationAdd(){}
	OperationAdd(double numA, double numB) :Operation(numA, numB){}
	double GetResult()
	{
		double result = 0;
		result = get_numberA() + get_numberB();
		return result;
	}

};

class OperationSub :public Operation
{
public:
	OperationSub(){}
	OperationSub(double numA, double numB) :Operation(numA, numB){}
	double GetResult()
	{
		double result = 0;
		result = get_numberA() - get_numberB();
		return result;
	}
};

class OperationMul : public Operation
{
public:
	OperationMul(){}
	OperationMul(double numA, double numB) :Operation(numA, numB){}
	double GetResult()
	{
		double result = 0;
		result = get_numberA() * get_numberB();
		return result;
	}
};

class OperationDiv :public Operation
{
public:
	OperationDiv(){}
	OperationDiv(double numA, double numB) :Operation(numA, numB){}
	double GetResult()
	{
		double result = 0;

		if (get_numberB() == 0){
			throw new exception("除数不能为零");
		}
		result = get_numberA() / get_numberB();
		return result;
	}
};

//简单运算工厂类  
class OperationFactory
{
public:
	Operation *createOperate(char operate)
	{
		Operation *oper = NULL;
		switch (operate){
		case '+':
			oper = new OperationAdd();
			break;
		case '-':
			oper = new OperationSub();
			break;
		case '*':
			oper = new OperationMul();
			break;
		case '/':
			oper = new OperationDiv();
			break;
		}
		return oper;
	}
};

int main()
{
	Operation *oper=NULL;
	OperationFactory operFactory;
	double numA, numB;
	char operate;
	cout << "请输入第一个数字" << endl;
	cin >> numA;
	cout << "请输入第二个数字" << endl;
	cin >> numB;
	cout << "请输入操作符" << endl;
	cin >> operate;
	oper = operFactory.createOperate(operate);
	oper->set_numberA(numA);
	oper->set_numberB(numB);
	double result = oper->GetResult();
	delete oper;
	cout << "result=" << result << endl;
}

```