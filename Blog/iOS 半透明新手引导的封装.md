![Liu](https://upload-images.jianshu.io/upload_images/1518951-53913cb1c580e340.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 经过一个月的加班加点，由我负责的公司分项目终于完成了阶段性的开发，在改bug的空余，总结了一下工程中用到的一些东西，现把我封装的“半透明新手引导功能”分享出来，供大家交流学习，有不足，还望提出，共同进步~   

[hahha](https://www.jianshu.com/p/bbcab990a2da#2.代码区)
##### 1.功能图
一般的新手引导页，只需要出现一次，一次显示出全部需要提示的功能就可以了，像这样：

![MJGuide0.png](https://upload-images.jianshu.io/upload_images/1518951-5f5844438d9111b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但在我们项目中，不仅仅这样，还要通过点击一次次的显示不同的功能提示，一次可以显示一张，也可以显示多张，像这样：

![MJGuide1.gif](https://upload-images.jianshu.io/upload_images/1518951-92394f2238f91683.gif?imageMogr2/auto-orient/strip)

![MJGuide2.gif](https://upload-images.jianshu.io/upload_images/1518951-8a28a17e9d438511.gif?imageMogr2/auto-orient/strip)

##### 2.代码区
万变不离其宗，如果你的项目只需要简单的一个新功能引导，并不需要那么多引导视图，那么我们可以用以下代码来实现，我也是根据此代码来封装多视图的，参考 [黑黑的小土豆](https://www.jianshu.com/p/00d4fe5a3c1a) ：
1. 创建半透明视图加载在window上,并添加点击手势
```
CGRect frame = [UIScreen mainScreen].bounds;
UIView * bgView = [[UIView alloc]initWithFrame:frame];
bgView.backgroundColor = RGBHEXA(0x323232, 0.8);

UITapGestureRecognizer * tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(sureTapClick:)];
[bgView addGestureRecognizer:tap];
[[UIApplication sharedApplication].keyWindow addSubview:bgView];
```
2. 获取目标视图在window坐标系的frame
```
CGRect rect = [self.view convertRect:self.wobBtn.frame toView:[UIApplication sharedApplication].keyWindow];
```
3. UIBezierPath 绘制镂空路径
```
UIBezierPath *path = [UIBezierPath bezierPathWithRect:frame];
[path appendPath:[[UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:5] bezierPathByReversingPath]];
// bezierPathByReversingPath 的意思就是反转 就是将你设置的路径以外的部分加入到路径中这样 你设置的路径就不在绘制的范围之内。在setMaskLayer的时候 你设置的那部分路径就没有了 就成了镂空的。 clockwise 这个值得意思也是这样。
```
4. 渲染路径
```
CAShapeLayer *shapeLayer = [CAShapeLayer layer];
shapeLayer.path = path.CGPath;
shapeLayer.strokeColor = [UIColor blueColor].CGColor;
[bgView.layer setMask:shapeLayer];
```
5. 把UI设计的功能引导图加到半透明视图上
```
UIImageView * imageView = [[UIImageView alloc]initWithFrame:CGRectMake(frame.size.width -225,CGRectGetMaxY(rect),200, 100)];
imageView.image = [UIImage imageNamed:@"img_guidehelp_bookshelf_purchasehistory"];
[bgView addSubview:imageView];
```
6. 手势事件
```
- (void)sureTapClick:(UITapGestureRecognizer *)tap{
    UIView * view = tap.view;
    [view removeFromSuperview];
    [view.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
    [view removeGestureRecognizer:tap];
}
```

##### 3.封装区
`MXRGuideMaskView` 想要展示多个新功能引导视图，肯定需要接受外界传递参数：图片组、图片位置组、镂空区位置组、展示顺序
```
/**
 初始化数据
 
 @param images 图片字符串
 @param imageframeArr 图片位置 @[[NSValue valueWithCGRect:CGRectMake(20, 205, 170, 50)]]
 @param rectArr 矩形透明区位置
 @param orderArr 顺序 例：@[@2,@1,@3]
 */
- (void)addImages:(NSArray *)images imageFrame:(NSArray *)imageframeArr TransparentRect:(NSArray *)rectArr orderArr:(NSArray *)orderArr;
```
然后，在 .m 方法中，判断具体的逻辑：
1. 判断顺序数组总数是否等于图片数组,避免不必要的 crash
```
NSInteger numCount = 0;
for (NSNumber * num in orderArr) {
    NSInteger order = [num integerValue];
    numCount += order;
}
if (numCount != images.count) {
    return;
}
```
2. 准备数据
```
self.orderArr = [orderArr mutableCopy];
[images enumerateObjectsUsingBlock:^(NSString *obj, NSUInteger idx, BOOL * _Nonnull stop) {
    UIImage * image = [UIImage imageNamed:obj];
    [self.imageArr addObject:image];
}];
self.frameArr = [imageframeArr mutableCopy];

for (NSInteger i=0; i<rectArr.count; i++) {
    UIBezierPath *transparentPath = [UIBezierPath bezierPathWithRoundedRect:[rectArr[i] CGRectValue] cornerRadius:5];
    [self.transparentPaths addObject:transparentPath];
}
```
3. 控制多个显示逻辑
```
for (NSInteger i=0; i<[orderArr[0] integerValue]; i++) {
    [self addImage:_imageArr[i] withFrame:[_frameArr[i] CGRectValue]];
    [self addTransparentPath:_transparentPaths[i]];
}
- (void)addImage:(UIImage*)image withFrame:(CGRect)frame{
    
    UIImageView * imageView   = [[UIImageView alloc]initWithFrame:frame];
    imageView.backgroundColor = [UIColor clearColor];
    imageView.image           = image;
    [self addSubview:imageView];
}

- (void)addTransparentPath:(UIBezierPath *)transparentPath {
    [self.overlayPath appendPath:transparentPath];
    self.fillLayer.path = self.overlayPath.CGPath;
}
```
4.点击事件
```
- (void)tapClickedMaskView{
    
    _index++;
    if (_index < _orderArr.count) {
        
        [self refreshMask];
        [self.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
        
        // 控制多个显示逻辑
        NSInteger baseNum = [_orderArr[_index-1] integerValue];
        countNum = countNum + baseNum;
        NSInteger endNum = [_orderArr[_index] integerValue]+countNum;
        for (NSInteger i=countNum; i<endNum; i++) {
            
            [self addTransparentPath:_transparentPaths[i]];
            [self addImage:_imageArr[i] withFrame:[_frameArr[i] CGRectValue]];
        }
    }else{
        countNum = 0;
        [self dismissMaskView];
    }
}
- (void)refreshMask {
    
    UIBezierPath *overlayPath = [self generateOverlayPath];
    self.overlayPath = overlayPath;
    
}

- (UIBezierPath *)generateOverlayPath {
    
    UIBezierPath *overlayPath = [UIBezierPath bezierPathWithRect:self.bounds];
    [overlayPath setUsesEvenOddFillRule:YES];
    
    return overlayPath;
}
```
#### 4.使用方法
使用起来也很方便，只需要初始化`MXRGuideMaskView`视图，然后传递参数，展示出来，就OK啦~
```
- (void)setupGuideView{
    MXRGuideMaskView *maskView = [MXRGuideMaskView new];
    [maskView addImages:imageArr imageFrame:imgFrameArr TransparentRect:transparentRectArr orderArr:orderArr];
    [maskView showMaskViewInView:self.view];
}
```
是不是很简单那，嘤嘤嘤 ~ 
整理了一个[【Demo】](https://github.com/JingJing-Lin/MXRGuideMaskView)出来，喜欢就点赞哦 ~

![star.gif](https://upload-images.jianshu.io/upload_images/1518951-84484f7dfeee3e9c.gif?imageMogr2/auto-orient/strip)