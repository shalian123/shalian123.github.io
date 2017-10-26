---
title: QT小结(1)
date: 2017-06-28 19:32:50
tags: [qt,标签,新窗体,标题栏]
categories: QT

---
最近在帮导师公司做一个项目，需要用到QT开发，在此先做个小结吧，也算一天的收获吧。
##### 标签点击信号
标签有时可能需要“点击”信号，但没有QLabel发出的“点击”信号。 通过**设置“flat”属性**，可以通过像一个标签一样使用**QPushButton**来轻松解决这个问题。(这个方法不错)
但是，如果您需要QLabel对象的其他属性，则可以使用自定义QLabel的代码段来发出一个信号：“clicked”。 换句话说，一个可点击的QLabel！
###### 1给label添加点击事件
Qt中原本的label是没有点击事件的，如果想添加点击事件的话，可以继承QLabel类并重载鼠标事件（比如mousePressedEvent），然后在鼠标事件中发送一个信号，具体如下：
```
// clickedlabel.h

#ifndef CLICKEDLABEL_H
#define CLICKEDLABEL_H

#include <QWidget>
#include <QLabel>

class ClickedLabel : public QLabel
{
    Q_OBJECT
public:
    ClickedLabel(QWidget *parent=0): QLabel(parent){}
    ~ClickedLabel() {}
signals:
    void clicked(ClickedLabel* click); // 点击信号
protected:
    void mouseReleaseEvent(QMouseEvent*); // 重载了鼠标释放事件

};

#endif // CLICKEDLABEL_H
```
```
// clickedlabel.c

#include "clickedlabel.h"
void ClickedLabel::mouseReleaseEvent(QMouseEvent *)
{
    emit clicked(this); // 在点击事件中发送信号
}
```
这里是最简单的例子，在label上获取鼠标点击事件，然后发送'clicked'信号，可以当鼠标释放的时候发送信号，这由开发这定义。
http://wiki.qt.io/Clickable_QLabel
具体还可参考以下博客：
Qt:添加点击事件的Label并显示图片 http://www.cnblogs.com/whlook/p/6810346.html
http://blog.chinaunix.net/uid-25808509-id-1748991.html等。

##### 多窗口跳转
1.在主窗体设置一个按钮；
2.添加登入对话框；
  **往项目中添加新文件**，这里可以在编辑模式左侧的项目目录上右击，然后选择添加新文件菜单，如下图所示。当然也可以在文件菜单中进行添加。**模板选择Qt分类中的Qt设计师界面类**，然后界面模板选择Dialog withoutButtons，单击下一步进入类信息界面，这里将类名更改为LoginDlg（注意类名首字母一般大写）。如下图所示，下面的相关文件会自动改名。
  **当完成后会自动跳转到设计模式**，可以对新添加的对话框进行设计。我们向界面上拖入一个Push Button（按钮），可以**右击转到槽**，选择clicked（）信号，接着在对应的槽里编写关闭这个新窗口的命令。
```
  dialog_set.c文件
#include "dialog_set.h"
#include "ui_dialog_set.h"

Dialog_set::Dialog_set(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Dialog_set)
{
    ui->setupUi(this);
    setWindowFlags(Qt::FramelessWindowHint);//去掉窗体的标题栏
    //setAttribute(Qt::WA_TranslucentBackground);//背景透明

}

Dialog_set::~Dialog_set()
{
    delete ui;
}

void Dialog_set::on_pushButton_clicked()
{
    this->close();
}

```
3.在主窗体界面上设置的按钮的槽里显示新窗口
```
在widget.cpp里
void Widget::on_pushButton_5_clicked()
{
    dialog1.show();
    dialog1.exec();
    this->show();
}

```
```
在widget.h里
private:
 Dialog_set dialog1;//新对话框
```
##### 标题栏隐藏
` setWindowFlags(Qt::FramelessWindowHint);`//去掉窗体的标题栏
` //setAttribute(Qt::WA_TranslucentBackground);`//背景透明 
`//setWindowFlags(Qt::FramelessWindowHint | Qt::WindowSystemMenuHint);`//去掉边框，保留任务栏菜单
最后附上一天的成果图：
![](http://ols4zt49w.bkt.clouddn.com/1498652235.png)