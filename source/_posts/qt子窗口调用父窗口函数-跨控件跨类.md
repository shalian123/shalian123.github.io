---
title: qt子窗口调用父窗口函数(跨控件跨类)
date: 2017-07-20 17:27:57
tags: [窗体间传递,qt]
categories: QT

---
在qt中有时需要窗体间传递数据，在这里我写了一个demo,主要的效果是：主窗体点击跳出一个子窗体，然后由子窗体控制在主窗体显示画图等。简要来说就是，**子窗口中的一个按钮被点击了，父窗口上的一个slot函数响应这个点击。**
思路：
1）在子窗口里面增加一个signal，在父窗口里面增加一个响应slot用于接收这个信号。
2）子窗口的按钮slot函数中emit这个signal。
3）在父窗口中把子窗口的这个signal连到自己的响应slot。
下面是子窗体的代码：
```
  //dialog_mea.h
#ifndef DIALOG_MEA_H
#define DIALOG_MEA_H

#include <QDialog>

namespace Ui {
class Dialog_mea;
}

class Dialog_mea : public QDialog
{
    Q_OBJECT

public:
    explicit Dialog_mea(QWidget *parent = 0);
    ~Dialog_mea();


signals:

    void Dialog_meaEvent();//子窗体添加的信号

protected slots:

    void Button_clicked();

private:
    Ui::Dialog_mea *ui;
};

#endif // DIALOG_MEA_H

```
```
      //dialog_mea.c
#include "dialog_mea.h"
#include "ui_dialog_mea.h"

Dialog_mea::Dialog_mea(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Dialog_mea)
{
    ui->setupUi(this);
    connect(ui->pushButton,SIGNAL(clicked()),this,SLOT(Button_clicked()));
}

Dialog_mea::~Dialog_mea()
{
    delete ui;
}
void Dialog_mea::Button_clicked()
{
    emit Dialog_meaEvent();//发射信号
    this->close();
}

```
下面是主窗体的代码：
```
  //widget.h文件
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include "dialog_mea.h"
#include "paintline.h"
#include <QHBoxLayout>
namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();
protected slots:
    void SonWindowEventClicked() ;
private slots:

    void on_pushButton_clicked();

private:
    Ui::Widget *ui;
    Dialog_mea dialog2;
    paintline *pp;
    QHBoxLayout  *layout;
    Dialog_mea *ww;
};

#endif // WIDGET_H

```
```
 //widget.c文件
#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_pushButton_clicked()
{
    dialog2.show();
    connect(&dialog2,SIGNAL(Dialog_meaEvent()),this,SLOT(SonWindowEventClicked()));
}
void Widget::SonWindowEventClicked()//主窗体响应子窗体信号
{
    pp = new paintline();
    layout = new QHBoxLayout();
    layout->addWidget(pp);
    ui->widget->setLayout(layout);
    pp->show();

}

```
关于上面代码中涉及到的paintline类等，参考我上一篇的文章。[QT中使用QPainter在主窗口的子控件里画图](http://shalian.site/2017/07/19/QT%E4%B8%AD%E4%BD%BF%E7%94%A8QPainter%E5%9C%A8%E4%B8%BB%E7%AA%97%E5%8F%A3%E7%9A%84%E5%AD%90%E6%8E%A7%E4%BB%B6%E9%87%8C%E7%94%BB%E5%9B%BE/)
最后附上几篇讲的不错的帖子：
[【Qt】窗体间传递数据（跨控件跨类），三种情况与处理方法 ](http://blog.csdn.net/shihoongbo/article/details/48681979)
[qt子窗口调用父窗口的函数](http://blog.csdn.net/guo503604087/article/details/48826025)
https://zhidao.baidu.com/question/2078605278997086108.html