ImmigrateToGradle
=================

[android]手动将 ADT的 ANT工程 迁移到Android Studio/ Intellij IDEA 的Gradle工程<p>

首先，新版本的<a href="http://developer.android.com/intl/zh-cn/sdk/installing/migrate.html">eclipse 支持直接将工程导出到Android Studio</a>，
但是，手动将一个eclipse 工程配置成一个Gradle 工程，将让你熟练的掌握Gradle 的许多常用配置。<p>

## 准备
下载一份Gradle，并配置进你的系统环境

## 整理目录
打开你的工程副本，查看工程结构<p>
这是一些由开发工具生成的文件，包含着开发工具的对该工程的配置和生成的编译文件，Gradle 用不上这些文件，建议直接删除掉，同时也建议不要把这些文件添加到版本控制中去。
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_1.png)

## jar包去重
如果使用Gradle 构建工程，建议将libs 文件夹中的jar 包都转换成gradle dependencies（依赖），而不是直接存放在文件夹中。两者不可同时存在，否则因为包名、文件重复则报错。<p>
例如，android-support-v4.jar 你可以在build.Gradle 中这样配置 <p>
``` groovy
    dependencies { // 依赖配置
        compile 'com.android.support:support-v4:20.0.0'
    }
```
