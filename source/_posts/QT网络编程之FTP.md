---
title: QT网络编程之FTP
date: 2017-05-15 14:33:15
tags: [网络编程,qt,ftp]
categories: QT

---
FTP(文件传输协议)是一个主要用来**浏览远程目录**和**传输文件**的协议。FTP使用两个网络连接，一个用来发送命令，一个用来输出数据。FTP协议有一个状态，并且需要客户端在传输文件之前发送一些命令。FTP客户端建立一个连接，并且在整个会话期间**一直保持打开**。在每个会话期间可以发生多个传输。

**注意：**在QT5之前，为了方便网络编程，Qt 提供了 Network 模块。**该模块包含了许多类**，例如：**QFtp类** - **能够更加轻松使用 FTP 协议进行网络编程**。但是，从 Qt5.x 之后，Qt Network 发生了很大的变化，不再导出 QFtp 和 QUrlInfo 类，改用了 QNetworkAccessManager。

在QT5中，使用 QNetworkAccessManager类实现FTP与前面的http应用十分相似，只需要在QUrl对象中设置好主机的地址，用户名，密码等，然后使用get()、put()等函数完成文件的获取和上传。**注意：因为Qt5不再支持QFtp类,需要的话得重现编译.所以只能在根目录上传下载。如果想要实现其它功能，如：cd()改变服务器的工作目录到指定目录；list()列出ftp服务器上指定目录的内容，默认列出当前目录的内容；等，就只能用QFtp。**

虽然在QT5中QFtp类被废除了，在QT5中不能直接使用该类，但是在**QT5的兼容性扩展模块QtFtp中包含了支持Qt5的QFtp类。**
QFtp 类提供了一个 FTP 协议的客户端实现，具有以下特点：
1.**非阻塞行为，QFtp异步工作**。如果无法立即执行操作，函数仍将立即返回，并且该操作将被调度以供以后执行。调度操作的结果通过信号报告，这种方法依赖于事件循环操作。

2.命令ID。可以调度的操作（也被称为“命令”）有：connectToHost()、login()、close()、list()、cd()、get()、put()、remove()、mkdir()、rmdir()、rename() 和 rawCommand()。**当每一个命令被执行时，QFtp都会使用该命令ID发射commandStarted()和commandFinished()信号。**

3.数据传输进度指示。当**有数据传输时**，QFtp会发射信号( **QFtp::dataTransferProgress()**,QNetworkReply::downloadProgress()和QNetworkReply::uploadProgress() ),可以关联这些信号到QProgressBar::setProgress()或者QProgressDialog::setProgresss()上。

4.QIODevice支持。该类支持对QIODevice进行上传和下载。

这里我主要采用第二种方法，即采用QFtp类。我挑重点讲：
**连接**
在此处可以直接在ui界面里初始化，然后添加信号槽。ui界面如图：
![](http://ols4zt49w.bkt.clouddn.com/34$LEQ9JRY3J5S~%5DCD08L14.png)
```
// 连接按钮
void MainWindow::on_connectButton_clicked()
{
    ui->fileList->clear();
    currentPath.clear();
    isDirectory.clear();
    ui->progressBar->setValue(0);
    ui->progressBar_2->setValue(0);
    ftp = new QFtp(this);
    //每当一个命令被执行时，QFtp都会使用该命令ID发射commandStartedcommunity()和commandFinished()信号
    connect(ftp, &QFtp::commandStarted, this,
            &MainWindow::ftpCommandStarted);
    connect(ftp, &QFtp::commandFinished,
            this, &MainWindow::ftpCommandFinished);

    //listInfo()信号在执行完list()命令时发射
    connect(ftp, &QFtp::listInfo, this, &MainWindow::addToList);

    //当有数据传输时，QFTP会发射信号（QFtp::dataTransferProgress(),QNetworkReply::downloadProgress()和QNetworkReply::uploadProgress()）
   /* connect(ftp, &QFtp::dataTransferProgress,
            this, &MainWindow::updateDataTransferProgress);
    connect(ftp, &QFtp::dataTransferProgress,
            this, &MainWindow::updateDataTransferProgress_2);*/

    QString ftpServer = ui->ftpServerLineEdit->text();
    QString userName = ui->userNameLineEdit->text();
    QString passWord = ui->passWordLineEdit->text();
    ftp->connectToHost(ftpServer, 21);
    ftp->login(userName, passWord);
}
```
当然，也可以请求连接服务器，代码如下：
```
    ftp = new QFtp(this);

    ftp->connectToHost("wscloudy.com");//指定端口连接到FTP服务器主机，默认端口号21
    ftp->login("user", "8677");//指定用户名和密码登陆到FTP服务器
    ftp->get("sha.txt");//从服务器下载指定文件
    ftp->close();
    connect(ftp, &QFtp::commandStarted,
            this, &MainWindow::ftpCommandStarted);
    connect(ftp, &QFtp::commandFinished,
            this, &MainWindow::ftpCommandFinished);
```

**下载**
```
void MainWindow::on_downloadButton_clicked()
{
    connect(ftp, &QFtp::dataTransferProgress,
            this, &MainWindow::updateDataTransferProgress);
    // 注意：因为文件名称可能是中文，所以在使用get()函数前需要进行编码转换
    QString fileName = ui->fileList->currentItem()->text(0);
    QString name = QLatin1String(fileName.toUtf8());
    file = new QFile(fileName);//根据要下载的文件的名称创建本地文件
    if (!file->open(QIODevice::WriteOnly)) {
        delete file;
        return;
    }
    ui->downloadButton->setEnabled(false);
    ftp->get(name, file);//文件下载
}
//下载进度更新
void MainWindow::updateDataTransferProgress(qint64 readBytes,qint64 totalBytes)
{
    ui->progressBar->setMaximum(totalBytes);
    ui->progressBar->setValue(readBytes);
   // ui->progressBar_2->setValue(0);
}

```
**上传**
```
//上传按钮
void MainWindow::on_uploadButton_clicked()
{
    connect(ftp, &QFtp::dataTransferProgress,
            this, &MainWindow::updateDataTransferProgress_2);

    file = new QFile ("E:/ftpshare/shalian.txt");
    ftp->put(file,"shalian.txt"); //将我电脑E盘下的文件上传至服务器
    ui->uploadButton->setEnabled(false);

}
//上传进度更新
void MainWindow::updateDataTransferProgress_2(qint64 readBytes,qint64 totalBytes)
{
    ui->progressBar_2->setMaximum(totalBytes);
    ui->progressBar_2->setValue(readBytes);
    //ui->progressBar->setValue(0);
}

```


**注意**：一开始我在学校用自己的电脑配置搭建了一个ftp服务器，但是由于是校园局域网，外网进来需要路由允许穿越。(当然我可没法要求校园网络管理员实现)，因此借助了公司同事服务器上的ftp服务器，在他的路由上设置了允许我的穿进。
本文参考了：
http://blog.csdn.net/liang19890820/article/details/53318906
http://blog.csdn.net/liang19890820/article/details/53188182
http://blog.csdn.net/sinat_29173167/article/details/54601643
欢迎一起讨论交流！
