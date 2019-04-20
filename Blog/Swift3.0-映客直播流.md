最近“你的名字”很火哦，我和小伙伴也去电影院看了一下，新海城笔下的景色好美哦。  
![" 你的名字 "可以做手机壁纸哦](http://upload-images.jianshu.io/upload_images/1518951-5a7447a8fd99d7ee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



>重要的人，不能忘记的人，不想忘记的人。你，是谁？



好了，言归正传，学习Swift3.0也有一段时间了，一直在“慕课网”学习语法，最近看到“小波说雨燕”的Swift3.0实现“映客”App的仿写，就跟着学习了。项目由StoryBoard和代码混合开发，播放流采用的是“ ijkplayer”框架。看图：  
![Calendar2.gif](http://upload-images.jianshu.io/upload_images/1518951-855b5143aa6f9850.gif?imageMogr2/auto-orient/strip)

**1.准备**
1.[Kingfisher](https://github.com/onevcat/Kingfisher) 类似OC语言中的SDWebImage  
2.[Just](https://github.com/JustHTTP/Just) 类似OC语言中的AFNetworking  
3.[ijkplayer](https://github.com/Bilibili/ijkplayer)  视频直播的框架,支持 Android 和 iOS  参考文章[iOS中集成ijkplayer视频直播框架](http://www.jianshu.com/p/1f06b27b3ac0)  
4.[JSONExport](https://github.com/Ahmed-Ali/JSONExport) Json转Model工具  

**2.解析数据**

```
Just.post(liveListUrl) { (r) in
        guard let json = r.json as? NSDictionary else{
                return
        }
        let lives = MJStreamModel(fromDictionary: json).lives!
        self.list = lives.map({ (live) -> YKCell in
            return YKCell(portrait: live.creator.portrait, cover: live.creator.nick, location: live.city, viewers: live.onlineUsers, url: live.streamAddr)
            })
```
**guard语句：**判断其后的表达式布尔值为false时，才会执行之后代码块里的代码，如果为true，则跳过整个guard语句

**as操作符：**

- 用来把某个实例转型为另外的类型，由于实例转型可能失败，故选项形式as?  如果转换不成功的时候便会返回一个 nil 对象。成功的话返回可选类型值（optional），需要我们拆包使用。
- as!是强制类型转换，如果转换失败会报 runtime 运行错误，所以对于如果能确保100%会成功的转换则可使用 as!

**3.视图模糊**

```
//  创建需要的毛玻璃特效类型   iOS8.0之后
let blurEffect = UIBlurEffect(style: .light)
//  毛玻璃view 视图
let effectView = UIVisualEffectView(effect: blurEffect)
effectView.frame = imgBack.frame
imgBack.addSubview(effectView)
```
之前因为项目要兼容iOS7，用的是[FXBlurView](https://github.com/nicklockwood/FXBlurView)

**4.视频流播放**
```
  func setPlayView() {
        self.playerView = UIView(frame: view.bounds)
        view.addSubview(self.playerView)
        
        ijkPlayer = IJKFFMoviePlayerController(contentURLString: live.url, with: nil)
        ijkPlayer.scalingMode = .aspectFill
        let pv = ijkPlayer.view!
        pv.frame = playerView.bounds
        pv.autoresizingMask = [.flexibleWidth,.flexibleHeight]
        playerView.insertSubview(pv, at: 1)
    }
```
这里因为ijkPlayer是用OC语言写的，所以需要建立桥接头文件。创建方法，可以自行百度，也可以参考[Swift中桥接头文件建立的两种方法](http://www.jianshu.com/p/7c941d274f5a)

**5.烟花效果**

```
 // 配置emitter
    emitter.emitterPosition = pos //发射源位置
    emitter.renderMode = kCAEmitterLayerBackToFront  //渲染模式
    
    // CAEmitterCell用来表示一个个的粒子, 它有一系列的参数用于设置效果.
    let rocket = CAEmitterCell()
    rocket.emissionLongitude = CGFloat(M_PI_2)
    rocket.emissionLatitude = 0
    rocket.lifetime = 1.6               //生命周期，必需大于1.0
    rocket.birthRate = 1                //每秒生成粒子个数，默认1.0
    rocket.velocity = 40                //粒子运动的速度均值
    rocket.velocityRange = 100          //速度范围
    rocket.yAcceleration = -250         //粒子y方向的 加速度分量
    rocket.emissionRange = CGFloat(M_PI_4)  //周围发射角度
    rocket.color = UIColor(colorLiteralRed: 0.6, green: 0.6, blue: 0.6, alpha: 1.0).cgColor   //粒子的颜色
    rocket.redRange = 1.0   //一个粒子的颜色green 能改变的范围
    rocket.greenRange = 1.0
    rocket.blueRange = 1.0
    rocket.name = "rocket"  //Name the cell so that it can be animated later using keypath
```
具体参考本项目Demo

**小问题**

- 模拟器播放视频流声音不一致，换成真机测试就好了；
- `navigationController?.popViewController(animated: true)` 视图控制器pop返回，报如下警告
   `Expression of type uiviewcontroller? is unused` 
   改成：` _ = navigationController?.popViewController(animated: true)`  参考：[warning](http://www.jianshu.com/p/2fe13d368959)



刚开始用Swift3.0写项目，写的不好，还请大神们多多指导，最后奉上项目地址:[【YKStream_swift】](https://github.com/JingJing-Lin/YKStream)  

注：** 由于ijkplayer太大，所以没放在项目中，将你集成好的ijkplayer放入Frameworks中，就可运行。  
Frameworks: 如果文件夹不存在, 点击classes选择Show in Finder, 新建一个即可, 将你打包的或者下载的framework拖入其中并拉进项目中. 你也可以自己建一个文件夹, 把这个Frameworks直接delete即可。