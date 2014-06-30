---
author: Urbain
layout: post
title:  "使用Qt5编写桌面时钟挂件"
date: 2014-06-30 15:34:39
categories: 学习笔记
comments: true
tags: Qt 软件
thread: 3
---

前言：前段时间断断续续的学习了一点Qt，准备这个暑假深入学习，昨天空闲时间做了一个桌面时钟挂件，这个挂件几乎包含了目前为止学习Qt的所有内容，虽很简单，但是一个脚印，分享以共勉，谨望假期学业有成！  
Ps：本挂件灵感源自[江涛](http://blog.1994.mobi/)学长的 [Pure Clock](http://blog.1994.mobi/lab/Clock)，向学长致敬！

---

挂件化名"叮叮时钟挂件"，源代码大致可分为更新图案，更新时期，右键托盘菜单三部分。

先上图：  
![clock_img1](\images\2014-06-30-images\clock_img1.png)
![clock_img2](\images\2014-06-30-images\clock_img2.png)


更新图案：

    void MainWindow::paintEvent(QPaintEvent *event)
    {
        Q_UNUSED(event);
        //绘制路径，外围圆环
	    QPainterPath path(QPointF(95, 14));
	    if(second == 0)
	        second = 60;
	    path.arcTo(QRectF(0, 0, 190, 190), 90, -6*second);
	    path.arcTo(QRectF(14, 14, 162, 162), 90-6*second, 6*second);
	    if(second == 60)
	        second = 0;
	    path.moveTo(95, 17);
	    path.arcTo(QRectF(15, 15, 160, 160), 90, -6*(minute+second/60.0));
	    path.arcTo(QRectF(17, 17, 156, 156), 90-6*(minute+second/60.0), 6*(minute+second/60.0));
	    path.moveTo(95, 20);
	    path.arcTo(QRectF(18, 18, 154, 154), 90, -30*(hour%12 + minute/60.0 ));
	    path.arcTo(QRectF(20, 20, 150, 150), 90-30*(hour%12 + minute/60.0 ), 30*(hour%12 + minute/60.0 ));

	    //添加锥形渐变
	    QConicalGradient conicalGradient(QPointF(95, 95), 90);
	    conicalGradient.setColorAt(0.0, Qt::red);
	    conicalGradient.setColorAt(1.0, Qt::white);
	    //开始绘制圆环
	    QPainter painter(this);
	    painter.setRenderHint(QPainter::Antialiasing);//添加反走样
	    painter.setPen(QColor(0, 0, 0, 0));//删除边框，即颜色置为全透明
	    painter.setBrush(conicalGradient);
	    painter.drawPath(path);
	}


更新时期：

	void MainWindow::timeUpdate()
	{
	    current_DateTime = QDateTime::currentDateTime();
	    hour   = current_DateTime.time().hour();
	    minute = current_DateTime.time().minute();
	    second = current_DateTime.time().second();
	    year = current_DateTime.date().year();
	    month = current_DateTime.date().month();
	    day = current_DateTime.date().day();
	    week = current_DateTime.date().dayOfWeek();

	    labelDate1.setText("<font color=white>"+current_DateTime.toString("dddd")+"</font>");
	    labelTime.setText("<font color=white>"+current_DateTime.toString("hh:mm:ss")+"</font>");
	    labelDate2.setText("<font color=white>"+current_DateTime.toString("yyyy年MM月dd日")+"</font>");

	    update();
	}


右键托盘菜单：

	void MainWindow::createTrayIcon()
	{ // 创建托盘图标及右键菜单
	    trayIcon = new QSystemTrayIcon(this);
	    trayIcon->setIcon(QIcon(":/ico/ICON"));
	    trayIcon->setToolTip("叮叮时钟挂件");
	    //设置右键菜单：开机自启动，最小化，还原，退出
	    trayIconMenu = new QMenu(this);
	    atstartAction = new QAction("开机自启动", this);
	    atstartAction->setCheckable(true);
	    if(is_auto_start == "1")
	        atstartAction->setChecked(true);
	    connect(atstartAction, &QAction::triggered, this, &MainWindow::AutoStart);
	    minimizeAction = new QAction("最小化", this);
	    connect(minimizeAction, &QAction::triggered, this, &MainWindow::hide);
	    restoreAction = new QAction("还原", this);
	    connect(restoreAction, &QAction::triggered, this, &MainWindow::showNormal);
	    quitAction = new QAction("退出", this);
	    connect(quitAction, &QAction::triggered, this, &QApplication::quit);
	    //添加右键菜单
	    trayIconMenu->addAction(atstartAction);
	    trayIconMenu->addAction(minimizeAction);
	    trayIconMenu->addAction(restoreAction);
	    trayIconMenu->addSeparator();//分割线
	    trayIconMenu->addAction(quitAction);

	    trayIcon->setContextMenu(trayIconMenu);
	    connect(trayIcon, &QSystemTrayIcon::activated, this, &MainWindow::showNormal);
	}

	void MainWindow::AutoStart()
	{// 设置开机自启动
	    if(atstartAction->isChecked())
	        is_auto_start = "1";
	    else
	        is_auto_start = "0";

	    QString application_name = QApplication::applicationName();
	    QSettings *settings = new QSettings(REG_RUN, QSettings::NativeFormat);
	    if(is_auto_start == "1")
	    {
	        QString application_path = QApplication::applicationFilePath();
	        settings->setValue(application_name, application_path.replace("/", "\\"));
	    }
	    else
	    {
	        settings->remove(application_name);
	    }

	    if(!f->open(QIODevice::WriteOnly | QIODevice::Text))
	        ;
	    QTextStream inf(f);
	    inf << is_auto_start << endl;
	    f->close();
	}

以上部分代码为主要代码，完整Qt工程给出链接：[叮叮时钟挂件源代码](\download\2014-06-30\叮叮时钟挂件源代码.zip)  
另外，也单独封装了一个windows下安装包供体验：[叮叮时钟挂件安装程序](\download\2014-06-30\叮叮时钟挂件安装程序.exe)

以上代码均为个人想法，其中多有不妥之处，敬请指出，共同进步。