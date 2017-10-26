---
title: QT数据库操作2
date: 2017-08-23 18:58:55
tags: [QT,sqlite]
categories: QT

---
在前面我写了[Qt数据库操作](http://shalian.site/2017/07/30/Qt%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C/)，在那实现了从服务器端获取数据并存储，在这我讲解一种从文件中上传数据的方法，具体参考我之前的文章[http协议的补充](http://shalian.site/2017/08/04/http%E5%8D%8F%E8%AE%AE%E7%9A%84%E8%A1%A5%E5%85%85/)。
注意：文件中存储的数据是json形式，读取数据是json形式的字符串，以字符串形式上传。