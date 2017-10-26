---
title: 商品促销-策略模式(c++实现)
date: 2017-06-14 19:36:07
tags: [c++,设计模式]
categories: 大话设计模式

---
策略模式是一种定义一系列算法的方法，从概念上来看，所有这些算法完成的都是相同的工作，只是实现不同，它(策略模式)可以以相同的方式调用所有的算法，**减少**了各种算法类与使用算法类的之间的**耦合**。
策略模式就是用来封装算法的，但在实践中，可以用它来封装几乎任何类型的规则，只要在分析过程中听到需要在不同的时间应用不同的业务规则，就可以考虑使用策略模式处理这种变化的可能性。
![](http://ols4zt49w.bkt.clouddn.com/1497440990%281%29.png)
补充：UML类图：

类图一般分三层，第一层显示类的名称，如果是抽象类，则用斜体表示。第二层是类的特性，通常是字段和属性。第三层是类的操作，通常是方法或行为。
注意前面的符号，‘+’表示public，‘-’表示private，‘#’表示protected。
**继承**用空心箭头+实线表示
**实现接口**用空心箭头+虚线表示
**关联**关系用实线箭头表示
**聚合**表示一种弱的‘拥有’关系，体现的是A对象可以包含B对象，但B对象不是A对象的一部分，聚合用空心的菱形+实线箭头表示。
**合成**（组合）是一种强的‘拥有’关系，体现了严格的部分和整体的关系。合成关系用实心的菱形+实线箭头表示。
**依赖**关系，用虚线箭头表示。

以商场促销活动为例，可以有多种促销方式，如打折、返现、积分等。这些活动都对应着不同的算法，也就是不同的策略来计算应付的金额。如果不对这些策略进行封装，每次活动的改变都要修改程序中的很多地方，不够灵活，可维护性差。将所有的策略封装成相互独立的类，由一个统一的抽象类Strategy类指明具体用到的策略类。还需要一个上下文策略类来配置这些策略即Context类。
```
#include <iostream>
using namespace std;

/*付款金额计算的抽象类*/
class CashSuper
{
public:
	/*计算应付金额*/
	virtual double AcceptCash(double money) = 0;//纯虚函数
};

/*正常收费策略*/
class CashNormal : public CashSuper
{
public:
	virtual double AcceptCash(double money)
	{
		return money;
	}
};

/*打折收费策略*/
class CashRebate : public CashSuper
{
private:
	double discount;
public:
	//构造函数
	CashRebate(double dis) : discount(dis)//构造函数初始化列表以一个冒号开始，接着是以逗号分隔的数据成员列表，每个数据成员后面跟一个放在括号中的初始化式。(显性初始化类成员)
	{
	}

	virtual double AcceptCash(double money)
	{
		return money * discount;
	}
};

/*返现收费策略*/
class CashReturn : public CashSuper
{
private:
	double moneyCondition;
	double moneyReturn;
public:
	CashReturn(double condition, double retu) : moneyCondition(condition), moneyReturn(retu)
	{}


	virtual double AcceptCash(double money)
	{
		double result = money;

		if (money > moneyCondition)
		{
			result = money - (static_cast<int>(money / moneyCondition)) * moneyReturn;
		}

		return result;
	}
};

/* 收费策略 */
class CashContext
{
private:
	CashSuper* cs; /* 声明付款金额的收费策略 */

public:
	/*初始化时，传入具体的策略对象(正常、打折、返现)*/
	CashContext(int type)
	{
		switch (type)
		{
		case 1:
			cs = new CashNormal();
			break;
		case 2:
			cs = new CashRebate(0.8);
			break;
		case 3:
			cs = new CashReturn(300, 100);
			break;
		default:
			throw exception("没有这种促销活动");
		}
	}

	~CashContext()
	{
		delete cs;
	}

	/*得到现金计算结果(利用了多态机制，不同的策略行为导致不同的结果)*/
	double GetResult(double money)
	{
		return cs->AcceptCash(money);
	}
};


int main()
{
	double price;
	double number;
	int type;  /* 活动类型 */

	cout << "请输入单价: ";
	cin >> price;

	cout << "请输入数量: ";
	cin >> number;

	cout << "请选择促销活动：" << endl
		<< "1 : 正常收费" << endl
		<< "2 : 打8折" << endl
		<< "3 : 满 300 返 100" << endl;
	cin >> type;

	try
	{
		CashContext cashcontext(type);

		cout << "总计：" << price * number << endl;

		cout << "实收现金：";
		cout << cashcontext.GetResult(price * number) << endl;
	}
	catch (exception& e)
	{
		cout << e.what() << endl;
	}

	return 0;
}
```
如果有新的促销活动需要加入，只需再写一个对应的策略类，并在CashContext类中修改部分代码即可实现。