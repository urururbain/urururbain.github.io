---
author: Urbain
layout: post
title:  "Android整理：简单实现再按一次退出"
date: 2015-01-11 13:21:45
categories: 学习笔记
comments: true
tags: Android
thread: 5
---

前言：上个月与同学一起做了一个简单的Android应用，这段时间正好没有很多事情所以趁热整理一下学习到的知识，刚开始学习Android还有很多不懂的地方，继续努力吧！

---

作业中需要用到“再按一次退出程序”，于是百度搜到了很多种方法，但大多实现起来比较麻烦，下面是一种比较简单的实现方法，通过重写主Activity类的onKeyUp方法达到效果：

	private long firstTime = 0;

	//监测返回键，再按一次退出程序
	@Override
	public boolean onKeyUp(int keyCode, KeyEvent event) {
			if (keyCode == KeyEvent.KEYCODE_BACK) {
				long secondTime = System.currentTimeMillis();
				if (secondTime - firstTime > 2000) { //如果两次按键时间间隔大于2秒，则不退出
					Toast.makeText(getApplicationContext(), "再按一次退出问日记",
							Toast.LENGTH_LONG).show();  //提示消息
					firstTime = secondTime;// 更新firstTime
					return true;
				} else { // 两次按键小于2秒时，退出应用
					System.exit(0);
				}
			}
			return super.onKeyUp(keyCode, event);
		}

