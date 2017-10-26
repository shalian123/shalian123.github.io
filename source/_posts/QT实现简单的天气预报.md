---
title: QT实现简单的天气预报
date: 2017-05-05 16:16:26
tags: [访问网络,json,API]
categories: QT

---
Qt 进行网络访问的类是QNetworkAccessManager,QNetworkAccessManager类允许应用程序发送网络请求以及接受服务器的响应。事实上，Qt 的整个访问网络 API 都是围绕着这个类进行的。QNetworkAccessManager保存发送的请求的最基本的配置信息，包含了代理和缓存的设置。最好的是，这个 API 本身就是异步设计，这意味着我们不需要自己为其开启线程，以防止界面被锁死。**（补充：Qt 的界面活动是在一个主线程中进行。网络访问是一个相当耗时的操作，如果整个网络访问的过程以同步的形式在主线程进行，则当网络访问没有返回时，主线程会被阻塞，界面就会被锁死，不能执行任何响应，甚至包括一个代表响应进度的滚动条都会被卡死在那里。）**QNetworkAccessManager是使用信号槽来达到这一目的的。
在掌握了访问网络和json解析等，我在此简易做了一个天气预报，既添加了学习趣味，也算是一种检验吧。
#### 一、API
通过天气api获取当前天气，api网上有很多，我在此注册了一个 http://www.haoservice.com/ 账号，可以免费申请数据，由于是体验用户，可以享受免费请求50次（调试的时候注意，如果调试访问超过50次就得花钱了）。
需要自己申请apikey,同时获取到的天气api数据是Jason格式的。
![](http://ols4zt49w.bkt.clouddn.com/WX%7D@H%250$OFOPFI%5D$0%25AE901.png)



![](http://ols4zt49w.bkt.clouddn.com/~@%28GSJZI@NMK%7BV~Y$YBOJUA.png)

**API发送请求**
```
//发送请求
void Widget::on_pushButton_clicked()
{
    manage = new QNetworkAccessManager(this);
    QNetworkRequest network_request;


    /*设置发送数据*/

   //QString cityName = "杭州";

   network_request.setUrl(QUrl("http://apis.haoservice.com/weather?cityname=%E6%9D%AD%E5%B7%9E&key=4c08a10447eb435e98080000c50f3f77&paybyvas=false"));


   /* 注意：当url里有中文或空格之类的，需要用一些在线转码工具进行url编码，这里纠结了很久*/
   // network_request.setUrl(QUrl(QString(" http://apis.haoservice.com/weather?cityname=1%&key=4c08a10447eb435e98080000c50f3f77&paybyvas=false").arg(cityName)));



    /*发送get网络请求*/
    manage->get(network_request);

    /*建立connect，replyFinished为自定义槽函数，获取api发回来的天气信息（Jason）格式*/
    connect(manage,SIGNAL(finished(QNetworkReply *)),this,SLOT( getReplyFinished(QNetworkReply *)) );

}

```
在这里我一开始遇到了一个问题，很容易忽视，因为我的url中有中文的存在，每次访问都会失败，但是将网址输入浏览器却能有响应，这是因为浏览器会自动解析编码url，但是当你在代码中想访问请求时必须得编码，在此可以用在线编码软件，如下图：
![](http://ols4zt49w.bkt.clouddn.com/url%E7%BC%96%E7%A0%81.png)

**获取API传回得数据**
```
void Widget::getReplyFinished(QNetworkReply *reply)
{
    if(reply->error()!= QNetworkReply::NoError)
       qDebug() << QStringLiteral("响应失败");

    QByteArray bytes = reply->readAll();
    QJsonParseError jsonError;
    QJsonDocument document = QJsonDocument::fromJson(bytes,&jsonError);
    if (jsonError.error != QJsonParseError::NoError) {
            qDebug() << QStringLiteral("解析Json失败");//此处可以验证是否获取到天气信息（json格式）
            qDebug() << bytes;//如果API响应的不是json格式的数据，则不用解析，直接输出
            return;
        }
    QJsonObject data = document.object();
    qDebug() << "天气Jason：" << data;

    getTodayWeatherInfo(data);//获取当日天气信息
}
```
```
//获取当日天气信息，即解析json
void Widget::getTodayWeatherInfo(QJsonObject data)
{
    QJsonObject today = data.value("result").toObject().value("today").toObject();
    WeatherInfo todayInfo;
    todayInfo.city = today.value("city").toString();//北京
    todayInfo.date = today.value("date_y").toString();//2017年05月04日
    todayInfo.week = today.value("week").toString();//星期
    todayInfo.type = today.value("weather").toString();//阵雨
    todayInfo.curTemp = today.value("temperature").toString();//温度
    todayInfo.fengli = today.value("wind").toString();//风


    qDebug() << "getTodayWeatherInfo:\n" <<todayInfo.city<< todayInfo.date << todayInfo.week << todayInfo.type << todayInfo.curTemp
            <<todayInfo.fengli ;

    QStringList todayInfoList;//在文本框里显示
    todayInfoList <<todayInfo.city << todayInfo.date + todayInfo.week << todayInfo.type << todayInfo.curTemp
                   << todayInfo.fengli ;
    for(int i=0; i < todayInfoList.size(); i++)
    {
        ui->textBrowser->append(todayInfoList.at(i));
    }
}
```
结果如下：
![](http://ols4zt49w.bkt.clouddn.com/8%7DPG7BQ76JQ%29R20%5DE%608%7D0B2.png)
返回的json数据如下：
```
{
    "error_code": 0,
    "reason": "成功",
    "result": {
        "sk": {
            "temp": "25",
            "wind_direction": "北风",
            "wind_strength": "4级",
            "humidity": "57",
            "time": "16:34"
        },
        "today": {
            "city": "杭州",
            "date_y": "2017年05月05日",
            "week": "星期五",
            "temperature": "17~27",
            "weather": "多云",
            "fa": "01",
            "fb": "01",
            "wind": "西北风 3-4 级",
            "dressing_index": "舒适",
            "dressing_advice": "建议着长袖T恤、衬衫加单裤等服装。年老体弱者宜着针织长袖衬衫、马甲和长裤。",
            "uv_index": "弱",
            "comfort_index": "--",
            "wash_index": "较适宜",
            "travel_index": "较适宜",
            "exercise_index": "较适宜",
            "drying_index": "--"
        },
        "future": [{
            "temperature": "15~23",
            "weather": "阴",
            "fa": "02",
            "fb": "01",
            "wind": "东北风 微风",
            "week": "星期六",
            "date": "20170506"
        },
        {
            "temperature": "16~25",
            "weather": "阴",
            "fa": "02",
            "fb": "03",
            "wind": "东风 微风",
            "week": "星期日",
            "date": "20170507"
        },
        {
            "temperature": "17~22",
            "weather": "阵雨",
            "fa": "03",
            "fb": "02",
            "wind": "东南风 微风",
            "week": "星期一",
            "date": "20170508"
        },
        {
            "temperature": "17~24",
            "weather": "多云",
            "fa": "01",
            "fb": "01",
            "wind": "南风 微风",
            "week": "星期二",
            "date": "20170509"
        },
        {
            "temperature": "19~32",
            "weather": "晴",
            "fa": "00",
            "fb": "00",
            "wind": "西南风 微风",
            "week": "星期三",
            "date": "20170510"
        },
        {
            "temperature": "21~31",
            "weather": "多云",
            "fa": "01",
            "fb": "04",
            "wind": "西北风 微风",
            "week": "星期四",
            "date": "20170511"
        },
        {
            "temperature": "20~28",
            "weather": "雷阵雨",
            "fa": "04",
            "fb": "01",
            "wind": "西北风 微风",
            "week": "星期五",
            "date": "20170512"
        },
        {
            "temperature": "20~29",
            "weather": "多云",
            "fa": "01",
            "fb": "00",
            "wind": "西南风 微风",
            "week": "星期六",
            "date": "20170513"
        },
        {
            "temperature": "19~29",
            "weather": "多云",
            "fa": "01",
            "fb": "01",
            "wind": "西南风 微风",
            "week": "星期日",
            "date": "20170514"
        }]
    }
}
```
这里我只解析了显示了today的一部分，并用文本框textbrowser里显示
#### 二、换肤
这里我主要设置了背景图片，并且想要达到点击换肤的效果。
widget设置显示背景图片我这里尝试了三种方法：
方法一：如果widget是顶层窗口（无父类的窗口),设置背景图片
```
/* 如果widget是顶层窗口（无父类的窗口),设置背景图片*/
           //方法一：
     /* QImage _image;
      _image.load(":/image/images/UI1.png");
      setAutoFillBackground(true);   // 这个属性一定要设置
      QPalette pal(palette());
      pal.setBrush(QPalette::Window, QBrush(_image.scaled(size(), Qt::IgnoreAspectRatio,
                          Qt::SmoothTransformation)));
      setPalette(pal);
     */
```
方法二：
```
        // 方法二：
   /*QPalette palette;
   //palette.setColor(QPalette::Background, QColor(192,253,123));//设置背景颜色
     palette.setBrush(QPalette::Background, QBrush(QPixmap(":/image/images/UI1.png")));

         //上面一句相当于下面两句
   //pixmap.load(":/image/images/UI1.png");
   //palette.setBrush(QPalette::Background, QBrush(pixmap));

     setPalette(palette);
   */
```
方法三：利用QPainter的drawPixmap函数，这种方法只能用在paintEvent()函数中
```
   //方法三：利用QPainter的drawPixmap函数，这种方法只能用在paintEvent()函数中
void Widget::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
   // QPixmap pixmap(":/image/images/UI.png");
    painter.drawPixmap(0, 0, pixmap);//绘制UI
}

```
这里为了达到换肤的效果，我采用了第三种方法。同时初始背景图片
> pixmap.load(":/image/images/UI0"); //初始背景图片
> // resize(pixmap.size());//保留图片大小
> uid=0;

下面是简单的换肤程序：
```
//简单换肤
void Widget::on_updateButton_clicked()
{
    //选择UI的id
    if(uid == 3)
        uid = 0;
    else
        uid = uid + 1;
    //拼凑成UI路径
    QString UIpath = tr(":/image/images/UI%1").arg(uid);
    //qDebug()<<UIpath;
    //加载UI
    pixmap.load(UIpath);
    //产生paintEvent重绘UI
    update();
}

```
效果如下：
![](http://ols4zt49w.bkt.clouddn.com/%E5%A4%A9%E6%B0%94%E9%A2%84%E6%8A%A5.png)
 此处是由于在初始化时没有注释掉  `// resize(pixmap.size());//保留图片大小`
 在注释掉后，界面明显有了改善：
 ![](http://ols4zt49w.bkt.clouddn.com/8$B$VNGGW%28%7DW~%29%259E@W%25X%7BR.png)
 ![](http://ols4zt49w.bkt.clouddn.com/G2@B9OIGUOBUCB%60_A%5BG0WWP.png)
#### 三、展望
到此，我想实现的效果已经基本实现，所学的知识也有了检验，但是偶然在网上看到一个帖子，里面有一个更加完善的demo,在此我把它贴出，代码量不是太多，功能和我的差不多，但是他的布局和界面优化显示明显花了更大的功夫。详细代码参考这里：[github](https://github.com/ghsxl/QT5-Weather)
最终的效果：
![](http://ols4zt49w.bkt.clouddn.com/@%6037TE2%7B05ZAYR5%29VAQ%7B45O.png)
