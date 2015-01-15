---
author: Urbain
layout: post
title:  "Android整理：SQlite数据库的使用以及通过listView显示数据"
date: 2015-01-11 15:45:09
categories: 学习笔记
comments: true
tags: Android
thread: 7
---

前言：上个月与同学一起做了一个简单的Android应用，这段时间正好没有很多事情所以趁热整理一下学习到的知识，刚开始学习Android还有很多不懂的地方，继续努力吧！

---

作业中需要用到数据库，当然首选Android集成了的SQLite数据库。

直接使用数据库可以这样：

	//打开或创建test.db数据库
    SQLiteDatabase db = openOrCreateDatabase("test.db", Context.MODE_PRIVATE, null);  
    
    //创建person表
    db.execSQL("CREATE TABLE IF NOT EXISTS person([id] INTEGER PRIMARY KEY, [name] TEXT)");

    //查询
    Cursor cursor = db.rawQuery("select * from person where name = " + "'urbain'", null);
    if(cursor.moveToNext()) {
    	id = Integer.parseInt( cursor.getString(0) );
    	name = cursor.getString(1);
    }

    //更新删除操作均可用db.execSQL(string)来操作

利用listView显示数据库中的数据要加一个adapter，步骤：  
1.将Sqlite数据库的内容查询出来并放入数组列表中，形成ListView的数据源；  
2.adapter适配器绑定数据源，显示在ListView item中。

自己写类DatabaseHelper继承SQLiteOpenHelper并实现部分数据库操作：

	package com.zkurbain.wenriji;

	import android.content.Context;
	import android.database.Cursor;
	import android.database.sqlite.SQLiteDatabase;
	import android.database.sqlite.SQLiteDatabase.CursorFactory;
	import android.database.sqlite.SQLiteOpenHelper;

	public class DatabaseHelper extends SQLiteOpenHelper{

		private static final int VERSION = 1;  
		public DatabaseHelper(Context context, String name, CursorFactory factory,
				int version) {
			super(context, name, factory, version);
		}
		
		public DatabaseHelper(Context context, String name){  
				this(context, name, VERSION);  
		}
		
		public DatabaseHelper(Context context, String name, int version){
			this(context, name, null, version);
		}

		@Override
		public void onCreate(SQLiteDatabase db) {
			
			//创建表
			String crtWenTable = "CREATE TABLE IF NOT EXISTS [wenti]([wentiHash] INTENGER PRIMARY KEY, [wentiName] TEXT)" ;
			db.execSQL(crtWenTable);

			//如果数据库为空，则预置一些数据
			Cursor cursor = db.rawQuery("select * from wenti", null);
			if( !cursor.moveToNext()){
				String[] swen = {"我今天读书了吗？",	"我明天打算做什么？",	 "今天吃啥好吃的？"};
				
				db.beginTransaction(); //开始事务处理 
				for (String s : swen) {
					db.execSQL("insert into wenti values(" + s.hashCode() + ", '"+ s +"')");
				}
				db.setTransactionSuccessful();  //设置事务标志为成功，当结束事务时就会提交事务 
				db.endTransaction(); //结束事务
			}
		}

		@Override
		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			
		}
	}

在主类中，借助SimpleAdapter使数据显示在listView：

	package com.zkurbain.wenriji;

	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;

	import android.app.Activity;
	import android.database.Cursor;
	import android.database.sqlite.SQLiteDatabase;
	import android.os.Bundle;
	import android.view.Menu;
	import android.view.MenuItem;
	import android.widget.ListView;
	import android.widget.SimpleAdapter;

	public class ChooseWen extends Activity {

		private ListView listView = null;
		private List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
		private SimpleAdapter simpleAdapter = null;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_choose_wen);
			
			listView = (ListView) findViewById(android.R.id.list);

			simpleAdapter = new SimpleAdapter(this, getdata(), R.layout.list,
					new String[] { "wentiName" }, new int[] { R.id.wenti });

			//listView绑定adapter
			listView.setAdapter(simpleAdapter);
		}

		private List<Map<String, Object>> getdata() {
			DatabaseHelper dbHelper = new DatabaseHelper(ChooseWen.this,
					"wenriji.db");
			SQLiteDatabase db = dbHelper.getReadableDatabase();
			Cursor cursor = db.rawQuery("select * from wenti", null);

			// 清空list
			list.clear();
			// 查询到的数据添加到list集合
			while (cursor.moveToNext()) {
				String wenti = cursor.getString(1);
				Map<String, Object> map = new HashMap<String, Object>();
				map.put("wentiName", wenti); // 获取name
				list.add(map);
			}

			return list;
		}

		@Override
		public boolean onCreateOptionsMenu(Menu menu) {
			getMenuInflater().inflate(R.menu.choose_wen, menu);
			return true;
		}

		@Override
		public boolean onOptionsItemSelected(MenuItem item) {
			return super.onOptionsItemSelected(item);
		}
	}

下面是XML文件内容:

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:paddingBottom="@dimen/activity_vertical_margin"
	    android:paddingLeft="@dimen/activity_horizontal_margin"
	    android:paddingRight="@dimen/activity_horizontal_margin"
	    android:paddingTop="@dimen/activity_vertical_margin"
	    tools:context="com.wenriji.wenriji.ChooseWen"
	    android:id="@+id/activity_choose_wen" >

	    <ListView
	        android:id="@android:id/list"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:layout_alignParentLeft="true" >
			<requestFocus />
	    </ListView>

	</RelativeLayout>

