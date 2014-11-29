ImmigrateToGradle
=================
[android]手动将 ADT的 ANT工程 迁移到Android Studio/ Intellij IDEA 的Gradle工程
=================


##更新：<p>
2014/11/29：<p>
1、此项目已经过时，在更新了Android 5.0的完整SDK之后，使用Intellij IDEA 14/ Android Stuio 创建gradle 项目会自动下载和配置gradle 2.1，而自己手动下载的2.1似乎无法正确导入Android 工程。<p>
2、若要将此项目使用最新的SDK和工具打开，请先新建一个Gradle:android 工程，然后将其中的gradle 文件夹复制到此项目中，以让IDEA 来正确读取gradle wraper及其配置，这样方能使用工具自动下载的gradle 2.1或者更新版本。<p>
3、请自己根据需要或者依照最新的新建工程里面的默认配置修改本项目的build.gradle 文件，以适应新的开发环境，如果日后有空会来更新此项目的。<p>
4、关于Android 5.0的SDK，如果阁下翻墙无力，可以到<a href="http://pan.baidu.com/s/1pJwmySF">这里</a>下载，但这也不是最新的版本，只是更新到了10月初的版本，buildtools和support 包 在之后升级过几次了，ADB有点小bug，上传量大不作长期维护。可以用Android 手机挂fqrouter2 在局域网中供sdk manager更新，网速飞快。


正文
====
首先，新版本的<a href="http://developer.android.com/intl/zh-cn/sdk/installing/migrate.html">ADT eclipse 支持直接将工程导出到Android Studio</a>，
但是，手动将一个eclipse 工程配置成一个Gradle 工程，能帮助你熟练的掌握Gradle 的许多常用配置。<p>

## 准备
（注：以下准备可能已经不适用，请查看2014/11/29更新的内容，以正确使用Gradle 2.1。）<p>
<a href="https://dl.google.com/android/studio/install/0.8.6/android-studio-bundle-135.1339820-windows.exe">Android studio 下载</a>，下载完之后直接安装。<p>
<a href="https://services.gradle.org/distributions/gradle-1.10-all.zip">gradle 1.10 下载</a>，下载完之后解压，并将其路径配置到系统环境变量“GRADLE_HOME”中。<p>
<a href="http://www.gradle.org/downloads">其它版本gradle 的下载地址</a>

## 整理目录
打开你的工程副本，查看工程结构<p>
这是一些由开发工具生成的文件，包含着开发工具的对该工程的配置和生成的编译文件，Gradle 用不上这些文件，建议直接删除掉，同时也建议不要把这些文件添加到版本控制中去。<p>
（使用IDEA 14自动下载和配置gradle 2.1，请从IDEA的新建工程中拷贝一份gradle 文件夹出来放在这个工程里面，具体请看更新说明。）<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_1.png)

## jar包去重
如果使用Gradle 构建工程，建议将libs 文件夹中的jar 包都转换成gradle dependencies（依赖），而不是直接存放在文件夹中。两者不可同时存在，否则因为包名、文件重复则报错。<p>
例如，android-support-v4.jar 你可以在build.Gradle 中这样配置 <p>
``` groovy
    dependencies { // 依赖配置
        compile 'com.android.support:support-v4:20.0.0'
    }
```
<p>
## 去除冲突的依赖
``` groovy
    compile ('com.github.xxx:1.0'){
        exclude module: 'appcompat-v7'
    }
```

## 向工程添加Gradle 脚本文件
创建两个Gradle 的脚本文件（build.gradle、settings.gradle）到项目根目录<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_2.png)

## 使用Android Studio 导入工程 
1、open project <p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_3.png) 
<p>
2、选中build.gradle 文件 <p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_4.png) 
<p>
3、使用gradle-1.10 进行构建（目前的版本，Android 只支持gradle 1.10版） <p>
（更新：使用IDEA 14自动下载和配置gradle 2.1，请从IDEA的新建工程中拷贝一份gradle 文件夹出来放在这个工程里面，然后选择use default gradle wrapper。在没有gradle 文件夹和wrapper 相关文件的时候，此选项不可用。）<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_5.png) 
<p>
4、如果你是第一次运行Android Studio ，那么请先感谢郭嘉，然后等待配置完成 <p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_6.png) 
<p>

## 简洁有效的使用版本控制
你的版本控制中不需要这些文件和文件夹，这些都是开发工具的配置文件和编译生成的临时文件，你可以随时将这些文件删除，下次运行gradle 的时候它们又会生成。<p>
如果将它们放入版本控制中的话，很可能会造成版本控制的管理混乱。<p>
删除build文件夹可以有效清理缓存<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_7.png) <p>

## 同步工程
每次更改了build.gradle 文件时，你需要进行工程同步，否则新的配置不会生效<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_8.png) <p>

## build.Gradle 文件示例
``` groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies { // 加入android build tools，使用0.+ 为版本号表示自动向maven 获取0.版本下最新的版本。
        classpath 'com.android.tools.build:gradle:0.+'
    }
}
apply plugin: 'android'

repositories { // 配置仓库
    mavenCentral()
}

android { // android 工程配置
    compileSdkVersion 20
    buildToolsVersion "20.0.0"

    signingConfigs { // 签名，你可以配置多个签名，然后再在buildTypes 进行指定。
        release {
            storeFile file("keystore\\key.jks") // 签名文件存放路径
            storePassword "your.password"
            keyAlias "your.alias"
            keyPassword "password"
        }
    }

    packagingOptions { // 打包配置
        exclude 'META-INF/LICENSE' // 排除一些文件
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }

    lintOptions { // 打包时，也不因为出错中断。
        checkReleaseBuilds false
        abortOnError false
    }

    defaultConfig { // SDK 和版本号配置
        minSdkVersion 8
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes { // 打包版本配置
        release {
            signingConfig signingConfigs.release // 指定签名配置
            runProguard false // 是否进行Proguard
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        }
        debug {
            //  signingConfig signingConfigs.release // 如果你的debug版本也需要签名，请将这一行配置解开注释
        }
    }

    sourceSets { // 工程结构配置
        main {
            java.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
            jniLibs.srcDirs = ['libs'] // 配置此处才会打包jni 的.so 文件
            manifest.srcFile 'AndroidManifest.xml'
        }
    }
}

dependencies { // 依赖配置
    compile fileTree(dir: 'libs', include: ['*.jar']) // 整合所有libs 文件夹中的jar 包
    compile 'com.android.support:support-v4:20.0.0'
    compile 'com.android.support:gridlayout-v7:20.0.0'
    compile 'com.android.support:appcompat-v7:20.0.0'
    compile 'com.loopj.android:android-async-http:1.4.5' // 1.4.4 不可升级至1.4.5
    compile('com.fasterxml.jackson.core:jackson-databind:2.4.1.3') {
        // 需要配置 packagingOptions  'META-INF/LICENSE' NOTICE
        transitive = true
    }
}

tasks.withType(Compile) {
    options.encoding = "UTF-8"
}

apply plugin: 'idea'

idea {
    module {
        downloadJavadoc = true // 下载文档
        downloadSources = true // 下载源码
    }
}



```
