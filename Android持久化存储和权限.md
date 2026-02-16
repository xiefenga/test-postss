---
title: Android持久化存储和权限
created: 2025-12-16T06:38:41.580Z
updated: 2025-12-16T06:38:41.580Z
slug: Android持久化存储和权限
tags: []
description: ""
---

---
title: Android持久化存储和权限
created: 2023-8-24 07:29:04
updated: 2023-11-17 07:35:53
---

## 持久化存储

### 存储位置

Android 提供两类物理存储位置：内部存储空间和外部存储空间

> 原先内部存储和外部存储是分开的(多发生在 Android4.4 及以前)，渐渐的 Android 都做成了一体机甚至将内部存储和外部存储都集成在了一起，只是在逻辑上区分了内部存储和外部存储。

- 从物理角度来说，手机自带的存储空间就是内部存储，外置存储比如说 SD 卡就是外部存储
- 从逻辑上来说，`data` 目录就是内部存储，而 `mnt` 或 `storage` 下的`sdcard`目录就是外部存储

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a35a43cfec5a6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

**私有目录** 

私有目录是指仅某个应用自己可管理访问的文件目录，即该目录归属于该应用

应用私有的目录主要分布在内部存储和外部存储两个地方，应用卸载后自动清理

- `/data/data/packagename` 
  - 其他应用无法访问
  - 空间通常比较小
  - 通过 `getFilesDir` 获取路径
- `/storage/emulated/0/Android/data/packagename` 
  - 应用无需请求任何与存储空间相关的权限即可访问外部存储空间中的应用专属目录
  - 此目录更适合存储一些比较大的私有的媒体文件，比如说 音乐，图片
  - 通过 `getExternalFilesDir` 获取路径
  - 也是一个私有存储目录，其他程序无权访问

Android 应用管理界面的 CLEAR DATA 和 CLEAR CATCH 分别会清除私有目录下的文件

- CLEAR DATA：清除私有目录，即 data/data/程序包名 和 mnt/sdcard/Android/程序包名 
- CLEAR CACHE: 清除私有目录的 cach 目录，即 data/data/程序包名/catch 和 mnt/sdcard/Android/程序包名/catch 

**公有目录** 

公有目录即非应用私有目录，所有应用均可访问的目录

- 应用卸载后不会清理
- 通过 `Environment` 类访问，例如 `Environment.getExternalStoragePublicDirectory` 
- 需要有读写 SD卡的权限

### 存储方式

Android 系统中主要提供了3种方式用于简单地实现数据持久化功能

- 文件存储
-  SharedPreferences 
- SQLite 数据库存储

#### 文件存储

文件存储就是使用 java 的流直接读写内部/外部存储中的文件

- `getFilesDir` 获取 data/data/程序包名/files 目录
- `getExternalFilesDir` 获取 mnt/sdcard/Android/程序包名/files/指定类型目录

#### SharedPreferences

Sharedpreferences 是 Android 平台上一个轻量级的存储类

其本质是一个以“键-值”对的方式保存数据的xml文件，位于 /data/data/程序包名/shared_prefs/xx.xml

通过 `getSharedPreferences` 可以获取 SharedPreferences 对象

```java
SharedPreferences p = getSharedPreferences("data", Context.MODE_PRIVATE);
```

有几种模式

- `Context.MODE_PRIVATE` 指定该SharedPreferences数据只能被本应用程序读写
- `Context.MODE_WORLD_READABLE` 指定该SharedPreferences数据能被其他应用程序读，但不能写
- `Context.MODE_WORLD_WRITEABLE` 指定该SharedPreferences数据能被其他应用程序写
- `Context.MODE_APPEND` 该模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件

```java
//写入数据
SharedPreferences.Editor editor = sharedPreferences.edit();
editor.putString("name", "Tom");
editor.putInt("age", 28);
editor.apply();

// 读取数据
String name = sharedPreferences.getString("name", "");

// 删除指定数据
editor.remove("name");
editor.apply();

// 清空数据
editor.clear();
editor.apply();
```

`name` 参数会作为 xml 文件的文件名，其存储的格式为

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="name">Tom</string>
    <int name="age" value="28" />
</map>
```

#### SQLite

##### SQLiteOpenHelper

Android 专门提供了一个 `SQLiteOpenHelper` 类，借助这个类可以非常简单地对数据库进行创建和升级

`SQLiteOpenHelper` 是一个抽象类，使用时需要创建自己的类去继承它

##### Room

谷歌 2018 发布的 Room 持久性库在 SQLite 上提供了一个抽象层，能让开发者更好地操作 SQLite

- 针对 SQL 查询的编译时验证
- 可最大限度减少重复和容易出错的样板代码的方便注解
- 简化了数据库迁移路径

使用 Room 需要在 app **build.gradle** 中添加依赖

```groovy
// 添加依赖
dependencies {
    def room_version = "2.2.5"
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version" 
  	// Kotlin 使用 kapt 替代 annotationProcessor
}

// 配置编译器选项
android {
  defaultConfig {
    javaCompileOptions {
      annotationProcessorOptions {
        arguments += [
          "room.schemaLocation":"$projectDir/schemas".toString(),
          "room.incremental":"true",
          "room.expandProjection":"true"
        ]
      }
    }
  }
}
```

**Room 包含三个主要组件** 

- 数据库类
  - 用于保存数据库并作为应用持久性数据底层连接的主要访问点
  - `@Database` 
  - 抽象类

- 数据实体
  - 用于表示应用的数据库中的表
  - `@Entity` 
  - pojo 类

- 数据访问对象
  - 提供可用于查询、更新、插入和删除数据库中的数据的方法
  -  `@Dao` 
  - 接口
  - 成员方法使用类似于 mybatis 的几个注解

## 权限

### 权限分类

- 安装时权限：应用安装时自动授予应用相应权限，包括普通权限和签名权限
- 运行时权限：也称为危险权限，需要在应用中执行权限申请相关代码，系统弹出授权框提示框，需要用户点击才可以授权
- 特殊权限：需要用户在应用设置界面中才能开启权限

Android 6.0 以下不管是普通权限还是危险权限，只需在 **AndroidManifest.xml** 清单文件中进行申明即可

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

android 6.0 以上安装时只会给普通权限授权，其他权限需要特殊操作进行申请。

- 危险权限：需要同时在清单文件中申明和在代码中动态申请，系统会弹出对应的权限提示框，用户点击同意才可以授权。

- 特殊权限：不仅要在清单文件中申明权限，还需要在用户在应用设置界面进行授权。

### 动态申请权限

1. `ContextCompat.checkSelfPermission` 检查权限

2. 权限申请 `ActivityCompat.requestPermissions` / `registerForActivityResult().launch`

3. 重写 `onRequestPermissionsResult` 处理用户响应系统权限对话框后行为

一些常量

- 通过 `Manifest.permission.xx` 访问权限列表

- `PackageManager.PERMISSION_GRANTED` 已授权

- `PackageManager.PERMISSION_DENIED` 已拒绝
