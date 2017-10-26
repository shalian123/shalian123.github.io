---
title: Qt数据库操作
date: 2017-07-30 17:17:33
tags: [Qt,数据库,sqlite,oracle]
categories: QT

---


最近在公司的项目涉及到用QT来操作数据库，在qt里自带了一个小型本地型的数据库sqlite，一般在移动设备或嵌入式设备中经常使用，但是公司服务器端用的是**oracle数据库**，这就需要实现两种数据库的交互。在此我采用的方案是，服务器端提供相关**接口**，然后在软件中通过**http**协议**获取一些json数据**，通过**解析**再将它们**写入**软件自带的**sqlite数据库**。

##### 数据库的建立
```
#ifndef COLLECTION_H
#define COLLECTION_H
#include <QMessageBox>
#include <QSqlDatabase>
#include <QSqlQuery>

static bool createConnection()
{
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
                "pointpart varchar(100),"
                "pressure number(10,3),"
                "tempeture number(10,3),"
                "medium varchar(100),"
                "material varchar(100),"
                "originthickness number(10),"
                "outerdiameter number(10),"
                "deviceid number(19),"
                "devicename varchar(100),"
                "orgseq varchar(30),"
                "equipid number(19),"
                "equipcode varchar(100),"
                "equipname varchar(100),"
                "partual varchar(200))");

   // query1.exec("insert into Measuring_point values(0,'sha','lian','150MPa','20','无','无','12mm','14mm','C:/Users/hasee/Desktop/nfc-04/nfc/pics/changshi.png')");


    QSqlQuery query2(db1);
    //query1.exec("SET NAMES 'Latin1'");//使数据库支持中文
    query2.exec("create table ThicknessRecord (pointid int primary key, "
                "checkdate date(100),"
                "value number(10,2))");


    return true;
}




#endif // COLLECTION_H
```
##### http访问
```
//发送请求
void Widget::on_pushButton_clicked()
{
    manage = new QNetworkAccessManager(this);
    manage1= new QNetworkAccessManager(this);
    QNetworkRequest network_request;
    QNetworkRequest network_request1;

    /*设置发送数据*/

   //QString cityName = "杭州";

  // network_request.setUrl(QUrl("http://apis.haoservice.com/weather?cityname=%E6%9D%AD%E5%B7%9E&key=4c08a10447eb435e98080000c50f3f77&paybyvas=false"));

   /* 注意：当url里有中文或空格之类的，需要用一些在线转码工具进行url编码，这里纠结了很久*/

  //验证用户
 // network_request.setUrl(QUrl("http://172.16.10.24:8081/dreamfoil/nfc/nfc-thickness!checkUser.action?pid=1349&username=superuser&password=admin"));

 //下载测点信息
    network_request.setUrl(QUrl("http://172.16.10.24:8081/dreamfoil/nfc/nfc-thickness!downloadMeasurePoint.action?token=306d848a21164e23a4ce33a051b6dde7&pnum=6"));

    //下载测厚记录
  //  network_request1.setUrl(QUrl("http://172.16.10.24:8081/dreamfoil/nfc/nfc-thickness!downloadThicknessRecord.action?token=306d848a21164e23a4ce33a051b6dde7"));
    /*发送get网络请求*/
    manage->get(network_request);
   // manage1->get(network_request1);
    /*建立connect，replyFinished为自定义槽函数，获取api发回来的天气信息（Jason）格式*/
    connect(manage,SIGNAL(finished(QNetworkReply *)),this,SLOT( getReplyFinished(QNetworkReply *)) );
  //  connect(manage1,SIGNAL(finished(QNetworkReply *)),this,SLOT( getReplyFinished(QNetworkReply *)) );
}
```
##### json解析及插入数据库
```
//获取传回的数据
void Widget::getReplyFinished(QNetworkReply *reply)
{
    if(reply->error()!= QNetworkReply::NoError)
       qDebug() << QStringLiteral("响应失败");

    QByteArray bytes = reply->readAll();
    QJsonParseError jsonError;
    QJsonDocument document = QJsonDocument::fromJson(bytes,&jsonError);//转化为json文档
    if (jsonError.error != QJsonParseError::NoError) {
            qDebug() << QStringLiteral("解析Json失败");//此处可以验证是否获取到天气信息（json格式）
            //qDebug() << bytes;//如果API响应的不是json格式的数据，则不用解析，直接输出
            return;
        }
    QJsonObject data = document.object();//转化为对象，传回的整体是一个对象
    //QJsonArray data = document.array();  // 转化为数组
    qDebug() << "测点信息：" << data;
    qDebug()<<"result:";
    getMeasurePointInfo(data);//获取测点信息
   // getMeasureRecordInfo(data);
}

//获取测点result信息，进一步解析json
void Widget::getMeasurePointInfo(QJsonObject data)
{
    if (data.contains("result"))
    {
     QJsonValue value = data.value("result");
     if(value.isArray()){    //result是数组
        QJsonArray array = value.toArray();
        int nSize = array.size();  // 获取数组大小
        MeasurePointInfo pointInfo;
        //QStringList todayInfoList;//在文本框里显示
        QSqlDatabase db1 = QSqlDatabase::database("connection1");//database（）静态函数，通过指定链接名来获取相应的数据库连接
        QSqlQuery query1(db1);
        for (int i = 0; i < nSize; ++i) {  // 遍历数组
               if(array.at(i).isObject()){  //数组里面又是对象
                   QJsonObject object =array.at(i).toObject();
                   pointInfo.orgseq = object.value("orgseq").toString();//装置序列
                   pointInfo.pointpart = object.value("pointpart").toString();//测点部位
                   pointInfo.equipname = object.value("equipname").toString();//设备名称
                   pointInfo.deviceid = object.value("deviceid").toString();//装置id
                   pointInfo.originthickness = object.value("originthickness").toDouble();//公称壁厚
                   pointInfo.equipcode = object.value("equipcode").toString();//设备编号
                   pointInfo.equipid = object.value("equipid").toInt();       //设备id
                   pointInfo.parturl = object.value("parturl").toString();  //部位图
                   pointInfo.id = object.value("id").toInt();                 //测点id
                   qDebug() << "getMeasurePointInfo:\n" <<pointInfo.orgseq<< pointInfo.pointpart << pointInfo.equipname << pointInfo.deviceid << pointInfo.originthickness
                           <<pointInfo.equipcode<<pointInfo.equipid<<pointInfo.parturl<<pointInfo.id;
                 }
               //插入数据
               query1.prepare("insert into Measuring_point values (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)");//位置绑定
               query1.bindValue(0,pointInfo.id);
               query1.bindValue(1,pointInfo.pointpart);
               query1.bindValue(10,pointInfo.orgseq);
               query1.bindValue(13,pointInfo.equipname);
               query1.bindValue(8,pointInfo.deviceid);
               query1.bindValue(6,pointInfo.originthickness);
               query1.bindValue(12,pointInfo.equipcode);
               query1.bindValue(11,pointInfo.equipid);
               query1.bindValue(14,pointInfo.parturl);
               query1.exec();

               //更新数据

               /*query1.clear();
               query1.prepare(QString("update pointpart=?,orgseq=?,"
                                      "equipname=?,deviceid=?,"
                                      "originthickness=?,equipcode=?,"
                                      "equipid=?,partural=? where id=%1").arg(pointInfo.id));
               query1.bindValue(1,pointInfo.pointpart);
               query1.bindValue(10,pointInfo.orgseq);
               query1.bindValue(13,pointInfo.equipname);
               query1.bindValue(8,pointInfo.deviceid);
               query1.bindValue(6,pointInfo.originthickness);
               query1.bindValue(12,pointInfo.equipcode);
               query1.bindValue(11,pointInfo.equipid);
               query1.bindValue(14,pointInfo.partural);
               query1.exec();
              */
            }


    }
 }
}
```
##### 将数据库的数据在tablewidget显示
```
 if (createConnection()) //连接成功
 {
    // 使用QSqlQuery查询连接1的整张表，先要使用连接名获取该连接
    QSqlDatabase db1 = QSqlDatabase::database("connection1");//database（）静态函数，通过指定链接名来获取相应的数据库连接
    QSqlQuery query1(db1);
    qDebug() << "connection1:";
    //query1.exec("select * from Measuring_point");
    query1.exec("select id,pointpart,originthickness from Measuring_point");
    query1.last();
    int nRow=query1.at()+1;
    query1.first();
    //tableWidget->setColumnCount(nRow);
    //while(query1.next())//这边不能再有了，因为会影响到for循环的query1.next()
    //{
        //qDebug() << "ok";
       // qDebug() << query1.value(0).toInt() << query1.value(1).toString();
    //}

    tableWidget = new QTableWidget(this);
    QStringList header;
    header<<"设备管理"<<"测点部位" <<"公称壁厚(mm)" <<"测量壁厚(mm)"
         <<"测量时间"<<"测量提示"<<"关联状态"<<"更多";
    tableWidget->setRowCount(nRow);//一定要有行和列
    tableWidget->setColumnCount(8);
    tableWidget->setHorizontalHeaderLabels(header);
    /*QPalette pll = tableWidget->palette();
    pll.setBrush(QPalette::Base,QBrush(QColor(255,255,255,0)));
    tableWidget->setPalette(pll);  //和QTextEdit一样，都可以使用样式表QPalette来修改它的背景颜色和背景图片，这里我们把刷子设置为全透明的，就可以是透明的
   */
    ui->manageLayout->addWidget(tableWidget);
   // tableWidget->show();

              /************第一行的数据******************/
    tableWidget->setItem(0,0,new QTableWidgetItem(query1.value(0).toString()));
    tableWidget->setItem(0,1,new QTableWidgetItem(query1.value(1).toString()));
    tableWidget->setItem(0,2,new QTableWidgetItem(query1.value(2).toString()));

    /********************将数据在table widget显示************************/
    for(int i=1;query1.next();i++){

        tableWidget->setItem(i,0,new QTableWidgetItem(query1.value(0).toString()));
        tableWidget->setItem(i,1,new QTableWidgetItem(query1.value(1).toString()));
        tableWidget->setItem(i,2,new QTableWidgetItem(query1.value(2).toString()));

        QCheckBox* checkBoxSeclect = new QCheckBox();
        QCheckBox* checkBoxInsert = new QCheckBox();
        QCheckBox* checkBoxUpdate = new QCheckBox();
        QCheckBox* checkBoxDelect = new QCheckBox();
        tableWidget->setCellWidget(i,3,checkBoxSeclect);
        tableWidget->setCellWidget(i,4,checkBoxInsert);
        tableWidget->setCellWidget(i,5,checkBoxUpdate);
        tableWidget->setCellWidget(i,6,checkBoxDelect);
        qDebug() << "ok";
     }

    tableWidget->show();

 }
```