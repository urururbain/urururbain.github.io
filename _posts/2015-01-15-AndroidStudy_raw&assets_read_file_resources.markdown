---
author: Urbain
layout: post
title:  "Android学习笔记：工程目录/res/raw和/assets的异同及用法"
date: 2015-01-15 13:40:55
categories: 学习笔记
comments: true
tags: Android
thread: 8
---

在Android工程的目录下，/assets是应用系统内部需要使用到的诸如音乐、视频类文件，而/res是应用的资源文件，/res内有一个/raw目录，这个目录和/assets有类似，这里记录一下两者的异同和用法。

**/res/raw和/assets的相同点**

1.  两者目录下的文件在打包后会直接保存在apk包中而不被编译成二进制。

**/res/raw和/assets的不同点**

1.  /res/raw中的文件会被映射到R文件中，访问的时候直接使用资源ID即R.raw.*,而/assets文件夹下的文件不会被映射到R.java中，访问的时候需要使用AssetManager类；  

2.  /res/raw不可以有目录结构，而/assets则可以有目录结构，也就是/assets目录下可以再建立文件夹。

**读取文件资源**

1.  读取/res/raw下的文件资源，通过以下方式获取输入流来进行写操作:  
	InputStream ins = getResources().openRawResource(R.raw.filename);

2.  读取assets下的文件资源，通过以下方式获取输入流来进行写操作:  
	AssetManager am = null;  
	am = getAssets();  
	InputStream ins = am.open("filename");  

**下面是一个读取/res/raw下文件并写入sd卡的示例方法**

	void makeFile() {
		Toast.makeText(MainActivity.this, "开始创建文件", Toast.LENGTH_SHORT).show();
			if (Environment.getExternalStorageState().equals(
				Environment.MEDIA_MOUNTED)) {
			String dirPath = Environment.getExternalStorageDirectory()
					.getPath() + "/WeiPics"; // 要保存的路径
			String fileName = "notes1.png"; // 文件名

			try {
				File dir = new File(dirPath);
				if (!dir.exists()) {// 如果目录不存在，创建目录
					dir.mkdirs();
				}

				File file = new File(dirPath + "/" + fileName);
				if (!file.exists()) {// 如果文件不存在，创建文件
					InputStream ins = getResources().openRawResource(
							R.raw.notes1);
					FileOutputStream fos = new FileOutputStream(file);
					byte[] buffer = new byte[8192];
					int count = 0;
					while ((count = ins.read(buffer)) > 0) {
						fos.write(buffer, 0, count);
					}
					fos.close();
					ins.close();
					Toast.makeText(MainActivity.this, "创建文件成功",
							Toast.LENGTH_SHORT).show();
				}
			} catch (Exception e) {

			}
		}
	}

另外，在Manifest中记得加入相关权限

	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

**读取/assets下文件并写入sd卡的示例方法**

和上面的方法几乎一致，只是把：

	InputStream ins = getResources().openRawResource(
							R.raw.notes1);

改为了：

	AssetManager assetManager = getAssets();
	InputStream ins = assetManager.open("Notes1.png");