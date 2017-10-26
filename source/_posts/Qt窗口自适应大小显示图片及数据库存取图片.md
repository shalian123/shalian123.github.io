---
title: QT窗口自适应大小显示图片及数据库存取图片
date: 2017-07-13 17:54:59
tags: [QT,自适应图片滚轮]
categories: QT

---
##### Qt窗口自适应大小显示图片
实现思路也挺简单的，使用QLabel显示图片，把这个QLabel放在一个ScrollArea上面，这样图片过大的时候会自动的添加滚动条，最后窗体使用水平布局，这样ScrollArea的大小会随着窗口的大小自动改变。下面上代码
```
private:
    Ui::MainWindow *ui;
    QLabel *label ;
```
首先声明一个QLabel用于图片（**ScrollArea在设计器上拖放到窗体上，并设置窗口的布局方式为水平布局**）。
设置QLabel的pixmap，并设置QLabel的大小和图片一致，最后将该QLabel添加到ScrollArea上。 
```
    QImage *image=new QImage("C:/Users/hasee/Desktop/nfc-02/nfc/pics/changshi.png");
    ui->label_42->setGeometry(0,0,500,500);
    ui->label_42->setPixmap(QPixmap::fromImage(*image));
        //label->setPixmap(QPixmap("E:/Qt_Program/Chrysanthemum.jpg"));
    ui->label_42->resize(QSize(image->width(),image->height()));//设置QLabel的大小和图片一致
    ui->label_42->show();
    ui->label_42->setScaledContents(true);//自动适应布满整个label
```
效果如图：
![](http://ols4zt49w.bkt.clouddn.com/1499940207%281%29.png)
##### 数据库sqlite中存取图片
一般来说图片不会直接存在数据库里，都是存在文件里，然后将文件目录存到数据库，读取的话也只需读取它的目录，然后再将它显示出来。
代码如下：
```
 // 创建一个数据库连接，使用“connection1”为连接名
    QSqlDatabase db1 = QSqlDatabase::addDatabase("QSQLITE", "connection1");//可以省略第二个参数即连接名，创建默认连接
    db1.setDatabaseName("my1.db");//数据库名
    if (!db1.open()) {
        QMessageBox::critical(0, "Cannot open database1",
                              "Unable to establish a database connection.", QMessageBox::Cancel);
        return false;
    }

    // 这里要指定连接
    QSqlQuery query1(db1);
    //query1.exec("SET NAMES 'Latin1'");//使数据库支持中文
    query1.exec("create table Measuring_point (id int primary key, "
                "name varchar(20),"
                "position varchar(20),"
                "pressure varchar(20),"
                "tempeture varchar(20),"
                "medium varchar(20),"
                "material varchar(20),"
                "diameter varchar(20),"
                "thickness varchar(20),"
                "image_path varchar(100))");

    query1.exec("insert into Measuring_point values(0,'sha','lian','150MPa','20','无','无','12mm','14mm','C:/Users/hasee/Desktop/nfc-03/nfc/pics/changshi.jpg')");

```
