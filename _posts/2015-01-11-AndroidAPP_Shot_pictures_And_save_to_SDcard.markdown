---
author: Urbain
layout: post
title:  "Android整理：调用系统截图并保存"
date: 2015-01-11 13:41:09
categories: 学习笔记
comments: true
tags: Android
thread: 6
---

前言：上个月与同学一起做了一个简单的Android应用，这段时间正好没有很多事情所以趁热整理一下学习到的知识，刚开始学习Android还有很多不懂的地方，继续努力吧！

---

作业中需要用到“调用系统截图并保存”，通过分写为两个方法myshot()和saveToSD()先截图后保存：

	// 调用系统截图，返回所截图
	public Bitmap myShot(Activity activity) {
		// 获取windows中最顶层的view
		View view = activity.getWindow().getDecorView();
		view.buildDrawingCache();

		// 获取状态栏高度
		Rect rect = new Rect();
		view.getWindowVisibleDisplayFrame(rect);
		int statusBarHeights = rect.top;

		// 获取屏幕宽和高
		DisplayMetrics dm = new DisplayMetrics();
		getWindowManager().getDefaultDisplay().getMetrics(dm);
		int widths = dm.widthPixels;
		int heights = dm.heightPixels;
		int actionBarHeight = getActionBar().getHeight(); //actionBar的高度

		// 允许当前窗口保存缓存信息
		view.setDrawingCacheEnabled(true);

		// 截图时去掉状态栏和actionBar
		Bitmap bmp = Bitmap.createBitmap(view.getDrawingCache(), 0,
				statusBarHeights + actionBarHeight, widths,
				heights - statusBarHeights - actionBarHeight);

		// 销毁缓存信息
		view.destroyDrawingCache();

		return bmp;
	}

	// 将图片写入sd卡中保存
	// 参数为（需要保存的图片，目录名，文件名）
	private void saveToSD(Bitmap bmp, String dirName, String fileName)
			throws IOException {
		// 判断sd卡是否存在
		if (Environment.getExternalStorageState().equals(
				Environment.MEDIA_MOUNTED)) {
			File dir = new File(dirName);
			// 判断文件夹是否存在，不存在则创建
			if (!dir.exists()) {
				dir.mkdirs();
				//注意这里用的是mkdirs()，它可以同时创建多级目录再进一步创建文件；
				//另外一个方法mkdir(),只可以创建文件，如创建目录则会出错。
			}

			File file = new File(dirName + fileName);
			// 判断文件是否存在，不存在则创建
			if (!file.exists()) {
				file.createNewFile();
			}

			FileOutputStream fos = null;
			try {
				fos = new FileOutputStream(file);
				if (fos != null) {
					// 第一参数是图片格式，第二个是图片质量，第三个是输出流
					bmp.compress(Bitmap.CompressFormat.PNG, 100, fos);
					// 用完关闭
					fos.flush();
					fos.close();
				}
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

下面是调用截图并保存的几步：

	//截图
	Bitmap shotPicture = myShot(MainActivity.this);
	//指定保存的路径名
	String shotPicDirName = Environment.getExternalStorageDirectory()
					.getPath() + "/Wenriji/ShotPictures/";
	//指定保存的文件名，利用“日期_哈希值”的方式避免文件重名
	String shotPicFileName = date + "_" + shotPicture.hashCode()
					+ ".png";
	//保存到SD卡
	try {
		saveToSD(shotPicture, shotPicDirName, shotPicFileName);
	} catch (IOException e) {
		e.printStackTrace();
	}

同时，要记得在Manifest中加入写入SD卡的权限，刚开始一直出问题原来是忘了加权限：

	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

