ImmigrateToGradle
=================

[android]手动将 adt的 ANT工程 迁移到Android Studio/ Intellij IDEA 的Gradle工程<p>

首先，新版本的<a href="http://developer.android.com/intl/zh-cn/sdk/installing/migrate.html">eclipse 支持直接将工程导出到Android Studio</a>，
但是，手动将一个eclipse 工程配置成一个Gradle 工程，将让你熟练的掌握Gradle 的许多常用配置。<p>

# 操作

## 拷贝一份ADT使用ANT 构建的工程到新目录

## 整理目录
Gradle 用不上这些文件，建议直接删除掉。同时，请把libs 文件夹中android-support-v4.jar 等你可能会要迁移到build.gradle 中的jar 包也删除掉，否则打包时会出现文件冲突。
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_1.png)
