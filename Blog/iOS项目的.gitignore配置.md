![image](http://upload-images.jianshu.io/upload_images/1518951-2c68f87c835e8561.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最近使用 .gitignore 的时候，遇到了一个坑，解决之后，重新复习了一下关于iOS的.gitignore，再次总结了一下~ ﻿

####前言﻿

iOS 项目多人协作开发的时候，一言不合就有冲突。好好分析一下，这里面有坑，而`.gitignore就是一个避免冲突的利器。﻿

我们在使用 git 的过程当中，不是任何文件都需要 commit 到本地或者远程仓库的，比如 Mac 的.DS_Store，一些第三方框架类库文件，用户个人元数据等等，都没有必要提交。所以，我们就用到了.gitignore，接下来我们来介绍一下这一利器~﻿

####教程﻿

#####1.创建gitignore文件﻿

打开terminal (终端)﻿
输入指令:  `cd '项目目录'`﻿
输入指令:  `vim .gitignore`﻿
把 附文代码（下文中有） copy 到终端﻿
按 esc 键 ，输入指令 :wq (指令意思:保存并返回上一层)﻿

#####2.查看gitignore文件,方法1或2都可以﻿

* 1>输入指令 `sudo ls` 查看目录下是否存在gitignore，有则表示成功﻿

* 2>输入指令，在 Finder 窗口中显示隐藏的文件和文件夹`defaults write com.apple.finder AppleShowAllFiles -boolean true ; killall Finder`﻿
  隐藏原本的隐藏文件和文件夹`defaults write com.apple.finder AppleShowAllFiles -boolean false ; killall Finder`﻿

#####3.提交项目到远端﻿

如果之前没有提交过项目到远端，则按照正常情况，commit一下再push到远端就完成了﻿
如果之前提交过得话，则按下面指令提交﻿
```
# 注意有个点“.” 清除缓存﻿
git rm -r --cached .﻿
git add .﻿
git commit -m "update .gitignore"﻿
git push origin master﻿
```

#####4.效果检验﻿

重新clone一份这个项目到本地﻿

```
git clone<版本库的网址>  ﻿
git clone <版本库的网址><本地目录名>﻿
```

你会发现文件夹下 .gitignore忽略的内容都没有，项目不可以运行，这时使用终端， cd 到项目目录 ,执行﻿
```
pod install --verbose --no-repo-update﻿
```
下载完成，就OK了~﻿

##### 5.gitignore 代码附文：﻿

* 1.我项目中提取出来的﻿
```
# Xcode﻿

.DS_Store﻿

## Build generated﻿

build/﻿

DerivedData/﻿

## Various settings﻿

*.pbxuser﻿

!default.pbxuser﻿

*.mode1v3﻿

!default.mode1v3﻿

*.mode2v3﻿

!default.mode2v3﻿

*.perspectivev3﻿

!default.perspectivev3﻿

xcuserdata/﻿

## Other﻿

*.moved-aside﻿

*.xccheckout﻿

*.xcworkspace﻿

!default.xcworkspace﻿

## Obj-C/Swift specific﻿

*.hmap﻿

*.ipa﻿

*.dSYM.zip﻿

*.dSYM﻿

# CocoaPods﻿

Pods﻿

!Podfile﻿

!Podfile.lock﻿

# Carthage﻿

Carthage/Build﻿

# fastlane﻿

fastlane/report.xml﻿

fastlane/Preview.html﻿

fastlane/screenshots﻿

fastlane/test_output﻿

# Code Injection﻿

iOSInjectionProject/﻿

```
* 2.到gitignore.io去选择自定义配置﻿

在 [gitignore.io](https://www.gitignore.io) 输入你需要配置的语言，会帮助你自动生成一份配置。比如，输入Objective-C和Swift会帮助你生成下面的配置。﻿

* 3.去github下载﻿

从github下载，[gitignore](https://github.com/github/gitignore)，选取`Objective-C.gitignore`或者`Swift-C.gitignore`﻿

#### gitignore中，屏蔽整个目录Pods，但是保留Pods下的子目录B,这时应该怎么处理呐？﻿

1.看取了这篇文章 [gitignore中，如何屏蔽整个目录Pods，但是保留Pods下的子目录B](http://blog.csdn.net/madongchunqiu/article/details/45600729)﻿

试了一下，无法解决，QAQ~﻿

2.找到这个方法，也证明了方法1是错误的。但是Pods里面那么多库，按照此方法岂不是很麻烦~﻿

>若test下有test1，test2，test3文件。要track test3，则.gitignore文件为：﻿
>test/test1﻿
>test/test2 ﻿
>!test/test3﻿
>若为：﻿
>test/﻿
>!test/test3 ﻿
>则不能track test3。﻿

3.最后，看了关于 .gitignore 的一些简介之后，终于让我找到了方法：﻿

```
Pods/*﻿
!Pods/kerkee/ ﻿
!Pods/LBXScan/﻿
```

Amazing,感觉整个人都好了~﻿

参考：﻿

[https://www.jianshu.com/p/4ed175f13e97](https://www.jianshu.com/p/4ed175f13e97)﻿

[https://bingozb.github.io/37.html](https://bingozb.github.io/37.html)﻿

[https://www.cnblogs.com/haiq/archive/2012/12/26/2833746.html](https://www.cnblogs.com/haiq/archive/2012/12/26/2833746.html)