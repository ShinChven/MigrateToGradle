# Migrate To Gradle 手动迁移ADT 的ANT结构工程至Gradle

## 前言
现在Android Studio 已经发布正式版，其带来的新的功能对于原来的eclipse 用户是非常友好的，只需要指定目录就能自动将代码导入并配置好相关的Gradle 脚本。
我们可以用正式版的AS中的可视化插件轻松的将旧工程升级到Gradle 构建系统下，而这个项目旨在向大家介绍Gradle 的Android 工程的结构和常用配置。

## 工具
1、Android Studio<br>
2、Intellij IDEA

## 了解Gradle
Gradle 以module 来管理project，在Gradle 构建的Gradle project中通常包含application module（com.android.application），与library module（com.android.library）两种module。<p>
在Gradle 的project 中需要使用，基本上全都使用.gradle 文件来配置，是一个脚本化的工程构建，而非原先ADT中那种eclipse 的或视化构建。

## 迁移工程
1、创建一个文件夹来放你的工程，比如Migrate to Gradle。

2、<a href="http://www.gradle.org/docs/current/userguide/gradle_wrapper.html">Wrapper</a>：你需要Gradle 的Wrapper 来下载和管理当前项目使用的Gradle 的版本，当你的环境中没有配置Gradle 时它可以自动下载Gradle 并配置到你的环境中去。<br>
如果你在天朝，那么配置Gradle 的时间可能会稍长，所以我一般都是直接从Android Studio 新建的工程中拷贝Wrapper 出来使用，以避免重复配置不同版本的Gradle。<br>
而如果你不想使用工具中的版，你还可以进行其它配置，见下一点。

3、在文件夹中建一个app （或者其它什么名字）文件夹来存放你的application module，请将你原先的工程文件拷贝到app 文件夹中去。然后你还需要一个build.gradle 和settings.gradle文件。看起来如下图：<br>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/gradle_1.png)<br>
两个文件的配置如下：<br>
build.gradle  --  根目录的build.gradle 文件一般用来配置整个工程

``` groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.14.2'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

// 如果你想配置你自己制定的Gradle 版本，加入以下配置，然后在导入工程时选择use customizable gradle wrapper
task wrapper(type: Wrapper) {
    gradleVersion = '2.1'
}
```

settings.gradle  --  根目录的settings.gradle 文件用来制定哪个文件夹为module，以“:”符号给目录分级

``` groovy
include ':app'  // 根目录下的一级目录
//include ':libs:module0' // 根目录下的二级目录，如果你需要这个module 的话
```
4、配置你的app module：在其中加入build.gradle，具体配置如下：
``` groovy
buildscript {
    repositories {
        jcenter() //你所使用的仓库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.14.2' // Gradle 的Android 插件版本
    }
}
apply plugin: 'com.android.application' // 导入Android Application 插件，将此module 配置成application module

repositories {
    jcenter() // 仓库
}

android {

    compileSdkVersion 21 // 使用SDK的版本，请配置你SDK中有的最新版本
    buildToolsVersion "21.1.1" // buildTools 版本，你SDK中有哪个版本配哪个版本，建议更新到最新的版本


    defaultConfig {
        applicationId "com.github.ShinChven.migratetogradle" // 原来的包名，现在叫applicationId
        minSdkVersion 9
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

    buildTypes { // 配置打包的版本
        release { // 发行版
            runProguard false // 是否混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt' // 默认混淆文件
            proguardFiles 'proguard-project.txt' // 自定义混淆文件
        }
        debug { // debug 版

        }
    }

	sourceSets { // 如果你的工程是从ANT 中迁移过来，可以使用sourceSets 来配置工程结构，如果你使用的是标准Gradle 结构，可以不需要配置。
        main {
            java.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
            jniLibs.srcDirs = ['libs'] // 配置此处才会打包jni 的.so 文件
			jni.srcDirs=['jni']
            manifest.srcFile 'AndroidManifest.xml'
        }
    }
}

/**
 * https://gradle.org/docs/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html
 */
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.0'
    compile 'com.android.support:support-v4:21.0.0'
    compile(project(':LibModule')) // 包含module
}
```
完成这些你的工程就会看起来像这样：<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/gradle_0.png)<p>

## 运行工程
1. open project
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/Image_3.png)
2. 选中build.gradle 文件<br>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/gradle_4.png)
3. 导入：<br>
project 中包含Wrapper 选Use default gradle wrpper<br>
在project 的task 配置来自动配置gradle 选Use customizable gradle wrapper<br>
Use local Gradle distribution 为使用系统变量中配置的gradle <br>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/gradle_3.png)
4. 如果你是第一次运行Android Studio ，那么请先感谢郭嘉，然后等待配置完成

## jar包去重
如果你运气不好遇到了下图，说明你的工程中包含的v4 包或者其它什么包在你的dependencies 配置中出现了重复引用的冲突，你需要去重：<p>
![Screenshot](https://raw.githubusercontent.com/ShinChven/ImmigrateToGradle/master/screenshots/gradle_2.png)<p>

如果使用Gradle 构建工程，建议将libs 文件夹中的jar 包都转换成gradle dependencies（依赖），而不是直接存放在文件夹中。两者不可同时存在，否则因为包名、文件重复则报错。<p>
例如，android-support-v4.jar 你可以在build.Gradle 中这样配置 <p>
``` groovy
    dependencies { // 依赖配置
        compile 'com.android.support:support-v4:21.0.0'
    }
```

## 去除冲突的依赖
你所添加的那些依赖（dependencies）中的项目的可能会引用同一个项目，却是不同版本，在build 的时候可能会出错，如果要去除冲突，配置如下
``` groovy
    compile ('com.github.xxx:1.0'){
        exclude module: 'appcompat-v7'
    }
```

## build.Gradle 另外的一些配置，Good luck.
配置中的版本号，请根据最新sdk 中的版本进行配置 
``` groovy
bBuildscript { // 这段配置如果放入project 级的build.gradle 中则可以在module 中省掉。
    repositories {
        jcenter()
    }
    dependencies { // 加入android build tools
        classpath 'com.android.tools.build:gradle:1.1.1'
    }
}
apply plugin: 'com.android.application'

repositories { // 配置仓库
    jcenter()
}

android { // android 工程配置
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

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
        targetSdkVersion 21
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
    compile 'com.android.support:support-v4:21.0.0'
    compile 'com.android.support:gridlayout-v7:21.0.0'
    compile 'com.android.support:appcompat-v7:20.0.0'
    compile 'com.loopj.android:android-async-http:1.4.5' // Maven 中存在的项目
    compile('com.fasterxml.jackson.core:jackson-databind:2.4.1.3') {
        // 需要配置 packagingOptions  'META-INF/LICENSE' NOTICE
        transitive = true
    }
}

apply plugin: 'idea' // 加载idea 插件

idea {
    module {
        downloadJavadoc = true // 下载依赖项目的文档
        downloadSources = true // 下载依赖项目的源码
    }
}
```
## library modue build.gradle 文件示例
请注意，以下的配置的某些版本号已经过时，请根据自己的sd配置最新的版本号
``` groovy

apply plugin: 'com.android.library' // 配置为library

repositories {
    jcenter()
}

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.1"

    defaultConfig {
        minSdkVersion 9
        targetSdkVersion 21
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

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.0'
    compile 'com.android.support:support-v4:21.0.0'
}
```

## local.properties文件
在OS X 系统中，你的工程需要这个文件来告诉工具的gradle 插件，你的sdk在哪。如果不进行配置的话将不能使用gradle 插件中的clean build 等方便的功能。
``` bash
echo sdk.dir=$ANDROID_HOME >local.properties
```

