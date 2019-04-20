![依然是女神镇楼](http://upload-images.jianshu.io/upload_images/1518951-ef1bfd49a9a56830.jpg)

闲话不多说，先看效果

![MJTabbar-Swift.gif](http://upload-images.jianshu.io/upload_images/1518951-eff608aaedc66c15.gif?imageMogr2/auto-orient/strip)

自学Swift3.0中,看到了一种新方式来自定义Tabbar，觉得很有必要研究一下，记录下来，以备后来使用和参考。

**主题词：反射机制**  
Java语言中是这样定义的：反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

苹果开发语言中，系统Foundation框架为我们提供了一些方法反射的API，我们可以通过这些API执行将字符串转为类等操作。由于OC语言的动态性，这些操作都是发生在运行时的。
```
NSClassFromString
NSSelectorFromString
NSProtocolFromString
```
通过这些方法，我们可以在运行时选择创建那个实例，并动态选择调用哪个方法。这些操作甚至可以由**服务器**传回来的参数来控制，我们可以将服务器传回来的类名和方法名，实例为我们的对象。
```
//OC方法
Class class = NSClassFromString(@"ViewController");
ViewController *vc = [[class alloc] init];
SEL selector =NSSelectorFromString(@"getDataList");
[vc performSelector:selector];
```
**常用判断方法 **

```
// 当前对象是否这个类或其子类的实例
- (BOOL)isKindOfClass:(Class)aClass;
// 当前对象是否是这个类的实例
- (BOOL)isMemberOfClass:(Class)aClass;
// 当前对象是否遵守这个协议
- (BOOL)conformsToProtocol:(Protocol *)aProtocol;
// 当前对象是否实现这个方法
- (BOOL)respondsToSelector:(SEL)aSelector;
```
**代码区**
- 设置子视图控制器
```
func setUpChildController() {
        //[clsName ,title ,imageName]
        let array = [
          ["clsName":"MJHomeViewController","title":"首页","imageName":"home"],
          ["clsName":"MJMessageViewController","title":"消息","imageName":"message_center"],
          ["clsName":"UIViewcontroller"],
          ["clsName":"MJDisCoverViewController","title":"发现","imageName":"discover"],
      ["clsName":"MJPrefileViewController","title":"我","imageName":"profile"],
        ]
        var arrM = [UIViewController]()
        for dict in array {
           arrM.append(controller(dict: dict))
        }
        viewControllers = arrM  
   }
```
```
 /// 使用字典创建控制器 反射
    /// - parameter dict:信息字典[clsName ,title ,imageName]
    /// - returns : 子控制器
    private func controller(dict:[String:String]) -> UIViewController {
        guard let clsName = dict["clsName"],
            let title = dict["title"],
            let imageName = dict["imageName"],
            let cls = NSClassFromString(Bundle.main.namespace + "." + clsName) as? UIViewController.Type
            else {
                return UIViewController()
        }
        let vc = cls.init()
        vc.title = title
        
        //设置图像
        vc.tabBarItem.image = UIImage(named: "tabbar_" + imageName)
        vc.tabBarItem.selectedImage = UIImage(named:"tabbar_" + imageName + "_selected" )?.withRenderingMode(.alwaysOriginal)
    
        //设置字体颜色（大小）
        vc.tabBarItem.setTitleTextAttributes([NSForegroundColorAttributeName:UIColor.orange], for: .highlighted)
        vc.tabBarItem.setTitleTextAttributes([NSFontAttributeName:UIFont.systemFont(ofSize: 13)], for: UIControlState(rawValue: 0))     //默认为 ：12
        let nav = MJNavigationController(rootViewController: vc)
        return nav
    }
```
Bundle 分类：添加一个计算属性作为“命名空间”， 默认 “项目名称”（最好不要有数字和特殊符号）
计算型属性：类似于函数，没有参数，有返回值
```
    var namespace: String {
 
        return infoDictionary?["CFBundleName"] as? String ?? ""
    }
```
- 设置中间撰写按钮
```
 func setUpcomposeButton()  {
        tabBar.addSubview(composeBtn)
        
        let count = CGFloat(childViewControllers.count)
        //将向内缩进的宽度减小，能够让按钮的宽度变大，盖住容错点
        let w = tabBar.bounds.width / count - 1
        //CGRectInset 整数向内缩进，负数向外扩展
        composeBtn.frame = tabBar.bounds.insetBy(dx: 2 * w, dy: 0)
        composeBtn.addTarget(self, action:#selector(composeStatus), for: .touchUpInside)
    }
```
好了，激动人心的时刻到了，奉上[【demo】](https://github.com/JingJing-Lin/MJTabbar-Swift)源代码，你就可以愉快的使用了。拿去，不谢，内有其它福利哦。若喜欢，请点赞。
老司机，带带我~~~~~~

