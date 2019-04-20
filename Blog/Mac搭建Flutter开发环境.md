![图片发自简书App](http://upload-images.jianshu.io/upload_images/1518951-7c625c9d97ebb541.jpg)

>Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。  

Flutter在18年很火的样子，要赶超 React Native的感觉，所以19年春节刚过，我就开始入门了，目前正在学习阶段，路过的大牛多多指导呦。学习新的开发语言，第一步，肯定是要搭建开发环境喽，通过两次的搭建经历，遇到了一些问题，用文章记录一下，方便以后翻看，也可以给新手提供一些帮助。

### 环境搭建

##### 1.下载Flutter

推荐去官网下载，可以下载最新的SDK，网址：
[https://flutter.io/setup-macos/](https://flutter.io/setup-macos/)

##### 2.配置环境变量

先把刚才下载的zip包，解压缩到你想要配置的文件夹下，然后使用命令行配置环境变量。

```
vim ~/.bash_profile
```
增加一行

```
export PATH=/app/flutter/bin:$PATH
```

`/app/flutter/bin ` 为解压文件路径，如笔者的:
```
export PATH=/Users/MinJing_Lin/iOS/flutter/bin:$PATH
```

保存(:wq)完毕之后运行命令:

```
source ~/.bash_profile
```

这个时候就可以运行Flutter 命令行了,会展示Flutter命令帮助。

```
flutter -h
```

##### 3.检查环境

```
flutter doctor
```

![图1.png](https://upload-images.jianshu.io/upload_images/1518951-190ee78f3d3577e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果有 ✗ 标志，表示本行检测不通过，需要做一些设置或者安装一些软件。

如果Android Studio没有安装，可以先装下（Mac版）。Android Studio下载地址:[http://www.android-studio.org/](http://www.android-studio.org/)

##### 4.安装安卓环境

```
flutter doctor --android-licenses
```
这里界面会要求输入Y/N,一路输入Y就行了。

打开Android Studio，打开plugin，输入flutter搜索，点击install，安装Flutter插件时，会自动下载Dart语言，安装完成后重启Android Studio。

![图2.png](https://upload-images.jianshu.io/upload_images/1518951-daf4ea8e9476c1e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 5.安装iOS环境

```
brew install --HEAD libimobiledevice
brew install ideviceinstaller
brew install ios-deploy
brew install cocoapods
pod setup
```

如果资源brew update 太慢，没动静，可以替换一下镜像，使用中科大的镜像
第一步，替换brew.git
```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
```
第二步：替换homebrew-core.git
```
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

##### 6.配置环境变量

如果在国内，你懂的，还需要设置一下pub源，不然就不能愉快的使用别人写的库。

运行
```
vim ~/.bash_profile
```
增加

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

保存退出，然后运行

```
source ~/.bash_profile
```

至此，我们的环境就搭建完毕了，可以愉快的开发了。

### 开启flutter“Hello world”之旅

打开 Android Studio ，选择“Start a new Flutter project”
![图3](https://upload-images.jianshu.io/upload_images/1518951-c5aec282dfa1e948.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##### 参考链接

https://flutterchina.club
https://segmentfault.com/a/1190000014845833
https://www.jianshu.com/p/9617bd923159
https://blog.csdn.net/JerryWu145/article/details/86214908