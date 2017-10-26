---
title: http协议的补充
date: 2017-08-04 14:51:16
tags: [http]
categories: 网络编程

---

http://www.cnblogs.com/ranyonsue/p/5984001.html
在最近的项目中需要将本地文本中的json数据发送到服务器，在这里我采取了http的post方式，它的**请求头部**和**请求数据**是分开的。**思路**：我先将文档的数据读取出来，转化成字符串，然后拼接到http包的body中，最后提交数据请求。

一、首先我们了解一下我采取如此方案的原因：

**POST提交**：把提交的数据放置在是HTTP包的包体中。
**GET提交**，请求的数据会附在URL之后（就是把数据放置在HTTP协议头中），以?分割URL和传输数据，多个参数用&连接；例 如：login.action?name=hyddd&password=idontknow&verify=%E4%BD%A0 %E5%A5%BD。如果数据是英文字母/数字，原样发送，如果是空格，转换为+，如果是中文/其他字符，则直接把字符串用BASE64加密，得出如： %E4%BD%A0%E5%A5%BD，其中％XX中的XX为该符号以16进制表示的ASCII。
**因此，GET提交的数据会在地址栏中显示出来，而POST提交，地址栏不会改变。**

**我们看看GET和POST的区别**
1.GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456. POST方法是把提交的数据放在HTTP包的Body中.
**2.GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制**.
3.GET方式需要使用Request.QueryString来取得变量的值，而POST方式通过Request.Form来获取变量的值。
4.GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码。

二、下面是部分代码
```
void Widget::on_pushButton_3_clicked()
{

    QFile file("C:/Users/hasee/Desktop/shujv-ceshi/build-untitled-Desktop_QT_5_3-Debug/test.txt");
    file.open(QIODevice::ReadWrite);
    //int file_len =file.size();
    //QByteArray fdata=file.readAll();
    QString data =file.readAll();
    qDebug()<<data;
    QNetworkRequest request;

    request.setUrl(QUrl("http://172.16.10.24:8081/shsh/nfc/nfc-thickness!uploadThicknessRecordString.action"));
   // request.setUrl(QUrl("http://172.16.10.24:8081/shsh/nfc/nfc-thickness!checkUser.action"));
    request.setHeader(QNetworkRequest::ContentTypeHeader,"application/x-www-form-urlencoded");
    request.setRawHeader("Accept","text/html, application/xhtml+xml, */*");
   // request.setRawHeader("Referer","http://localhost:8888/login");
    request.setRawHeader("Accept-Language","zh-CN");
    request.setRawHeader("X-Requested-With","XMLHttpRequest");
    request.setRawHeader("User-Agent","Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)");
    request.setRawHeader("Content-Type","application/x-www-form-urlencoded");//窗体数据被编码为名称/值对。这是标准的编码格式
    //request.setRawHeader("Content-Type", "application/json");//如果是json格式访问则加,但是这里我以字符串来发送，将文本里的内容读取发送
    request.setRawHeader("Accept-Encoding","gzip,deflate");
    request.setRawHeader("Connection","Keep-Alive");
    request.setRawHeader("Cache-Control","no-cache");

    QString postData;
   // postData.append("pid=1332&username=superuser&password=admin");
   // postData.append("token=7eb935fb518e4234b83d3cae22d3873d&recordFileString={thicknessrecord:'sha',a:12.21}");
    postData.append("token=7eb935fb518e4234b83d3cae22d3873d&recordFileString=");
    postData+=data;//将文件中的字符串拼接上
    QByteArray postData1=postData.toLatin1();//很重要，要将请求数据编码,toUtf8()也行
    request.setRawHeader("Content-Length",QString::number(postData1.length()).toLatin1());//在请求头部设置请求数据的长度
    _uploadManager->post(request,postData1);//postData1是请求数据
          qDebug()<<postData1;
    connect( _uploadManager,SIGNAL(finished(QNetworkReply*)),SLOT(replyFinished(QNetworkReply*)));

}

```