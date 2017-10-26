---
title: QT之http网络编程，json访问应答
date: 2017-04-23 11:02:46
tags: [QT,HTTP]
categories: QT

---
结合前面两篇文章《浅谈http之post与get的区别》和《QT之json》，本文重点尝试了如何在qt中通过http协议json格式请求和响应服务器的json格式，并解析。
本文的重点是客户端的编写，服务器用的是一个简易的http本地服务器。
服务器端的响应程序如下：
```
void HttpServer::onRequest(HttpRequest* req, HttpResponse* resp)
{
	//实现你的响应代码
	QString reply  = tr("<html><head><title>Rokh Server Test</title></head><body><h1>hello!</h1></body></html>");
    resp->setHeader("Content-Type", "text/html");
    resp->setHeader("Content-Type","application/json");
	resp->setStatus(200);

    resp->end(QString("{"
                      "\"name\":\"QT\","
                      "\"version\":5,"
                      "\"windows\":true"
                      "}")

);
    //resp->end(QString("Test ok!"));
	return;

}
```

客户端的程序：在mainwindow.cpp中
```
 {
    QNetworkRequest req;//构造一个请求
    req.setUrl(QUrl("http://localhost:58890/post"));
    req.setHeader(QNetworkRequest::ContentTypeHeader, QVariant("application/json"));
    QNetworkAccessManager *manager = new QNetworkAccessManager(this);
    /***json数据**********/
    QJsonObject json;
    json.insert("name", "Qt");
    json.insert("version", 7);
    json.insert("windows", true);

    QJsonDocument document;
    document.setObject(json);
    QByteArray byte_array = document.toJson(QJsonDocument::Compact);

    //QNetworkReply *reply = manager.post(req, QByteArray("{}"));
    //QNetworkReply *reply = manager->post(req,byte_array);//发送请求，并会获得一个reply的响应对象
    manager->post(req,byte_array);//发送请求，使用 json 参数直接传递


    //开启一个局部的事件循环，等待响应结束，
    //退出因为请求的过程是异步的，所以此时使用 QEventLoop 开启一个局部的事件循环，等待响应结束，事件循环退出。
    //注意：开启事件循环的同时，程序界面将不会响应用户操作（界面被阻塞）。
   // QEventLoop ev;
   //连接槽信号
   // QObject::connect(manager, &QNetworkAccessManager::finished, &ev, &QEventLoop::quit);
    connect(manager, SIGNAL(finished(QNetworkReply *)), this, SLOT(replyFinished(QNetworkReply *)));

   // ev.exec();
   //connect(reply, SIGNAL(finished()), &ev, SLOT(quit()));
   // ev.exec(QEventLoop::ExcludeUserInputEvents);

}

MainWindow::~MainWindow()
{
    delete ui;
}
//槽函数，获取响应的内容，调用 readAll()，由于上述的 POST 请求返回的数据为 Json 格式，将响应结果先转化为 Json，然后再对其解析
void MainWindow::replyFinished(QNetworkReply *reply)
{
    if(reply->error()!= QNetworkReply::NoError)
         qDebug() << QStringLiteral("响应失败");

    // 获取响应信息，响应的内容可以是 HTML 页面，也可以是字符串、Json、XML等
    QByteArray bytes = reply->readAll();

    QJsonParseError jsonError;
    QJsonDocument doucment = QJsonDocument::fromJson(bytes, &jsonError);// 转化为 JSON文档
    if (jsonError.error != QJsonParseError::NoError) {
        qDebug() << QStringLiteral("解析Json失败");
        qDebug() << bytes;//如果服务器里响应的不是json格式的数据，则不用解析，直接输出
        return;
    }

    // 解析Json
    if (doucment.isObject()) {              // JSON 文档为对象
        QJsonObject obj = doucment.object();// 转化为对象
        QJsonValue value;
        if (obj.contains("name")) {
            value = obj.take("name");      //Removes key from the object.
            if (value.isString()) {         //判断 value 是否为字符串

                QString data = value.toString();// 将 value 转化为字符串

                qDebug() << endl << data;
            }
        }
        if (obj.contains("version")) {
            value = obj.take("version");//Removes key from the object.
            if(value.isDouble())
            {
                int data = value.toVariant().toInt();
                qDebug() << data;
            }
        }
        if (obj.contains("windows")) {
            value = obj.take("windows");//Removes key from the object.
            if(value.isBool())
            {
                bool flag = value.toBool();
                qDebug() << flag ;
            }
        }
    }
    else
    {
        qDebug() << "error, shoud json object";
    }

    //检测响应状态码,200 OK，表示请求已成功，请求所希望的响应头或数据体将随此响应返回
    QVariant variant = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute);
    if (variant.isValid())
        qDebug() << variant.toInt();

}
```
程序中请求是以json对象格式请求的，所以需要先使用 tojson()将QJsonDocument类文件转换成文本文件
> QByteArray byte_array = document.toJson(QJsonDocument::Compact);//转换成了字节数组

然后发送请求`manager->post(req,byte_array);`//发送请求

槽函数，获取响应的内容，调用 readAll()，由于上述的 POST 请求返回的数据为 Json 格式，将响应结果先转化为Json（fromjson函数），然后再对其解析。

结果解析如下：
![](http://ols4zt49w.bkt.clouddn.com/1492952805%281%29.png)
服务器端响应的json格式数据被解析并输出。

