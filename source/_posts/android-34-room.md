---
title: 重拾android路(三十四) room
date: 2019-12-20 22:35:51
tags:
  - android
  - room
---

关于数据库的部分，之前有写过一篇博客，当时使用的数据库是realm数据库。但是在使用了一段时间之后，发现他的使用比较复杂，而且在我们执行数据库语言的时候，总是不够智能化(难道说我用的是一个假的数据库？)，所以经过深思熟虑，决定使用room数据库作为项目的主要数据库框架。

<!--more-->

关于room数据库，网上有很多这样那样的教程，这里不做过多叙述，关键是如何使用，已经如何在项目中以比较灵活多变的形式使用room

## room数据库的引入

引入比较简单，只需要在gradle文件中添加如下依赖即可
```
// room
    compile "android.arch.persistence.room:runtime:1.1.1"
    annotationProcessor "android.arch.persistence.room:compiler:1.1.1"
```
目前好像room数据库有针对androidX进行更新，不过目前项目中并没有用到，所以也没有过多深入，后面如果有时间，再去学习吧。

引入完成之后，我们这里创建一个类名为AppDataBase的文件。

代码如下：
```
package com.paulniu.iyingmusic.db;

import android.content.Context;

import androidx.annotation.NonNull;
import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;
import androidx.room.TypeConverters;
import androidx.room.migration.Migration;
import androidx.sqlite.db.SupportSQLiteDatabase;

import com.paulniu.iyingmusic.Constant;
import com.paulniu.iyingmusic.db.dao.MusicInfoDao;
import com.paulniu.iyingmusic.model.MusicInfo;

/**
 * Coder: niupuyue (牛谱乐)
 * Date: 2020-01-05
 * Time: 21:53
 * Desc: 数据库操作
 * Version:
 */
@Database(entities = {MusicInfo.class}, version = 1)
@TypeConverters({})
public abstract class AppDataBase extends RoomDatabase {

    private static AppDataBase INSTANCE = null;

    public abstract MusicInfoDao getMusicInfoDao();

    public static final Migration mirgration_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            // 删除本地歌曲表
            database.execSQL("DROP TABLE IF EXISTS `MusicInfo`");
            // 创建本地歌曲表
            database.execSQL("CREATE TABLE IF NOT EXISTS `MusicInfo` (`_id` INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, `songId` INTEGER NOT NULL, `albumId` INTEGER NOT NULL, `duration` INTEGER NOT NULL, `musicName` TEXT, `artist` TEXT, `data` TEXT, `folder` TEXT, `musicNameKey` TEXT, `artistKey` TEXT, `favorite` INTEGER NOT NULL)");
            // 创建本地歌曲索引
            database.execSQL("CREATE  INDEX `index_MusicInfo__id_songId_albumId` ON `${TABLE_NAME}` (`_id`, `songId`, `albumId`)");
        }
    };

    public static AppDataBase getInstance(Context context) {
        synchronized (AppDataBase.class) {
            if (null == INSTANCE) {
                INSTANCE = Room.databaseBuilder(context.getApplicationContext(), AppDataBase.class, Constant.DATABASE_NAME)
                        .setJournalMode(JournalMode.TRUNCATE)
                        .allowMainThreadQueries()
                        .addMigrations(mirgration_1_2)
                        .build();
            }
            return INSTANCE;
        }
    }

}
```
在这个方法中，首先我们需要将当前类是抽象类，让room数据库帮我们自动创建实现类，因为这个内容是自动创建的，代码不用过于看重，只需要能看懂即可

![Database实现类](/assets/room/room01.png)

我们需要使用注解(@Database)的方法来设置当前类是数据库操作引入类，其中在这里我们需要声明数据库的实体类entities = {MusicInfo.class}
我们通过注解的方式完成数据库中类型比较复杂的样式，比如，我们需要存储List集合，或者其他的自定义对象，我们就可以通过这样的的方法实现类型的转换，后面会对这一块进一步阐述，@TypeConverters({})
我们需要让当前类继承RoomDatabase，并且将该类声明成抽象类
为了防止多个数据库对象同时操作数据库，我们使用单例模式，生成当前类对象，这样就保证了唯一性
在单例模式中，我们创建数据库操作的实体对象