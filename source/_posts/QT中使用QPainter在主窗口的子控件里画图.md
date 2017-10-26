---
title: QT中使用QPainter在主窗口的子控件里画图
date: 2017-07-19 18:35:58
tags: [Qt,折线图]
categories: QT

---
怎么在QT主窗口的一个控件里面画图,在这里我讲两种方法，其实是一种，只不过一种是有ui的，还有一种是代码new出来的控件。
#### QT中使用QPainter在ui子控件中绘图
在使用QT中的QPainter绘制图片时发现其只能够在当前类中执行绘制操作。本文介绍一下怎么实现在ui的子控件中用QPainter实现绘图。以QLabel为例：
1.在QT工程中新建一个类PaintLabel，继承自QLabel
```
paintlabel.h文件
#ifndef PAINTLABEL_H
#define PAINTLABEL_H
#include class PaintLabel:public QLabel
{
   Q_OBJECT
public: 
   explicit PaintLabel(QWidget *parent = 0);
   void paintEvent(QPaintEvent *event);
};
#endif // PAINTLABEL_H
```
```
//paintlabel.c文件
#include 'paintlabel.h'

#include 
#include 
#include 
#include 
#include 
#include 
PaintLabel::PaintLabel(QWidget *parent):QLabel(parent)
{

}
void PaintLabel::paintEvent(QPaintEvent *event)
{
QPainter painter(this);
global_var::Cap_Image = global_var::Cap_Image.scaled(this->width(),this->height(),Qt::IgnoreAspectRatio);
painter.drawImage(QPoint(0,0), global_var::Cap_Image);


//global_var::time_End = (double)clock();
//qDebug()<<(global_var::time_End-global_var::time_Start)/1000;
//global_var::time_Start = global_var::time_End;
}
```


2.**在界面设计文件mainwindow.ui中拖入一个QLabel控件，右键->提升为->选择基类QLabel->名称为PaintLabel->输入h文件paintlabel.h->选中->提升。**

3.原程序中的功能是载入QImage类型的global_var::CapImage图片。读者可以修改代码

painter.drawImage(QPoint(0,0), global_var::Cap_Image);

将其修改为载入一幅图片进行实验。
#### 纯代码在子控件里显示折线图
写一个类A继承QWidget类还是什么的，然后重写类A的paintEvent，在这个里面用QPainter绘图，只是**在显示的时候将这个类new到你的目标窗口里，然后show**
我在这里兴建了一个类PaintLine,继承基类QWidget,最终想在主窗口widget的子窗体widget_3中显示图
代码如下：
```
  paintLine.h文件
#ifndef PAINTLINE_H
#define PAINTLINE_H

#include <QWidget>
#include <QPainter>
#include <QPainterPath>
#include <QTimer>
class PaintLine : public QWidget
{
    Q_OBJECT
public:
    explicit PaintLine(QWidget *parent = 0);
    QPoint Point;   /* 线条绘制点的坐标 */
    void paintEvent(QPaintEvent *event); /* 重绘事件函数 */

signals:

private slots:
    void timerUpDate();     /* 定时器槽函数 */
private:
    unsigned int t; /* 用来保存时间 */
    int p; /* 用来调整坐标原点 */
    QPainterPath *path;
    QTimer *timer;
};

#endif // PAINTLINE_H

```
```
    paintLine.c文件
#include "paintline.h"

PaintLine::PaintLine(QWidget *parent) :
    QWidget(parent)
{
    p = t = 0;
    Point.setX(0);  /* 初始化起始点的纵坐标为0 */
    Point.setY(0);  /* 初始化起始点的横坐标为0 */
    path = new QPainterPath();
    timer = new QTimer(this);

    connect(timer,SIGNAL(timeout()),this,SLOT(timerUpDate()));  //关联定时器计满信号和相应的槽函数
    timer->start(500);
}
void PaintLine::paintEvent(QPaintEvent *event)
{

    //Point.setX(100);
    //Point.setY(100);
    //path->lineTo(Point);
    QPainter painter(this);
    painter.setPen(QPen(Qt::red, 2)); //设置画笔颜色和大小
    painter.translate(p,0);     //调整坐标原点
    painter.drawPath(*path);    //绘制路径
    //painter.drawPoint(Point);
    //update();
    //QWidget::paintEvent(event);
}
void PaintLine::timerUpDate()//定时时间到
{
    t += 10;
    Point.setX(t);       /* 时间加二秒 */
    Point.setY(qrand() % 100);    /* 设置纵坐标值 */
    path->lineTo(Point);    /* */

    if(t > width()) /* 当时间值 T大于窗口的宽度时需调整坐标原点  */
        p -= 10;    /* 调整坐标原点 */

    update();
}

```
**在主窗体的程序代码中这样显示**
```
//画线
void Widget::on_pushButton_9_clicked()
{
     pp = new PaintLine(ui->widget_3)；//将这个类new到你的目标窗口里
     pp->show();
     //update();

}

```
**这样写会发现有一个问题**，就是显示出现问题，它老是在widget_3的上面一部分显示，无法布满整个子控件。（原因可能是因为pp的默认高度比较小）
解决如下：
**new一个水平或垂直布局layout，然后将pp放到布局里，然后设置widget_3的布局为这个layout**

```
//画线
void Widget::on_pushButton_9_clicked()
{
    pp = new PaintLine();
    layout = new QHBoxLayout();
    layout->addWidget(pp);
    ui->widget_3->setLayout(layout);
    //pp->show();
    //update();

}
```