最近在学Swift 3.0，更新慢了点，勿怪哦~~
![亦菲女神](http://upload-images.jianshu.io/upload_images/1518951-35d3906ffdf37d89.jpg)
先放效果图哦：
![BeeQuick_2.gif](http://upload-images.jianshu.io/upload_images/1518951-bde999ecdbba92ff.gif?imageMogr2/auto-orient/strip)
###正文
这版时间基本上花在了用Masonry布局UICollectionView上，都是些比较基本的，不同的是代码格式更清晰，易于阅读。
下面说些本项目用到的一些可以复用的知识点。
- 字符串转颜色的分类
```
+(UIColor *) hexStringToColor: (NSString *) stringToConvert
{
    NSString *cString = [[stringToConvert stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]] uppercaseString];
    // String should be 6 or 8 characters
    
    if ([cString length] < 6) return [UIColor blackColor];
    // strip 0X if it appears
    if ([cString hasPrefix:@"0X"]) cString = [cString substringFromIndex:2];
    if ([cString hasPrefix:@"#"]) cString = [cString substringFromIndex:1];
    if ([cString length] != 6) return [UIColor blackColor];
    
    // Separate into r, g, b substrings
    
    NSRange range;
    range.location = 0;
    range.length = 2;
    NSString *rString = [cString substringWithRange:range];
    range.location = 2;
    NSString *gString = [cString substringWithRange:range];
    range.location = 4;
    NSString *bString = [cString substringWithRange:range];
    // Scan values
    unsigned int r, g, b;
    
    [[NSScanner scannerWithString:rString] scanHexInt:&r];
    [[NSScanner scannerWithString:gString] scanHexInt:&g];
    [[NSScanner scannerWithString:bString] scanHexInt:&b];
    
    return [UIColor colorWithRed:((float) r / 255.0f)
                           green:((float) g / 255.0f)
                            blue:((float) b / 255.0f)
                           alpha:1.0f];
}
```
- 仿安卓提示框[Toast](https://github.com/scalessec/Toast)分类，使用起来也比较简单。
```
 [self.view makeToast:[NSString stringWithFormat:@"  %ld ",tag] duration:1 position:CSToastPositionCenter];
```
- 导航条滚动渐隐渐现[HYNavBarHidden](https://github.com/newyeliang/HYNavBarHidden.git),封装的也很不错，用的时候根据实际情况来修改一下就好。（顺便解决个别bug）
```
[self setKeyScrollView:self.collectionView scrolOffsetY: 200 options:HYHidenControlOptionTitle];
[self setNavBarBackgroundImage:[UIImage createImageWithColor:COLOR_YELLOW]];
```


其它的可以看下[仿“爱鲜蜂”二级联动(iOS)](http://www.jianshu.com/p/f7dd1aa9c737)，最新版本已经上传至github，并解决上次遗留的bug，这里就不在重复讲解了。具体请看我的[【DEMO】](https://github.com/JingJing-Lin/BeeQuick_Two)吧
相互学习，提高iOS开发技术~~
如果您觉得本Demo对您有用，欢迎点star ，谢谢 