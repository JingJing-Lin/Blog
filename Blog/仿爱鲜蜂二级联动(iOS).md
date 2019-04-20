下雨天，在家敲代码很爽哦~~~  
![亦菲女神](http://upload-images.jianshu.io/upload_images/1518951-286331ddd449fcb6.jpg)  

闲话不多说，放效果图：  
![BeeQuick.gif](http://upload-images.jianshu.io/upload_images/1518951-b2a3865e34deaf52.gif?imageMogr2/auto-orient/strip)



### 正文

#### 一、准备

1.第三方库：本Demo用到的第三方库有：MJRefresh、Masonry、MJExtension、SDWebImage ，都是比较常见的，使用也比较多的。
2.分类：UIImage、UIImageView

 - 颜色转换为图片
```
+(UIImage *)createImageWithColor:(UIColor *)color {
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return theImage;
}
```
 - 由于上次被SDWebImage更新所坑过，所以对其简单的封装了一下
```
- (void)setImageWithURLStr:(NSString *)url placeholderImage:(UIImage *)placeholderImage
{
    [self sd_setImageWithURL:[NSURL URLWithString:url] placeholderImage:placeholderImage];
}
```
3.资源：数据库及图片

#### 二、绘制视图View

 - 一级表（左）
```
self.categoriesTableView = ({
        UITableView *tabView = [[UITableView alloc]initWithFrame:CGRectZero style:UITableViewStylePlain];
        tabView.delegate = self;
        tabView.dataSource = self;
        tabView.separatorStyle = UITableViewCellSeparatorStyleNone;
        tabView.backgroundColor = COLOR_BG;
        tabView.showsVerticalScrollIndicator = NO;
        tabView;
    });
    [self.view addSubview:self.categoriesTableView];
    [self.categoriesTableView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.view).offset(64);
        make.leading.equalTo(self.view);
        make.bottom.equalTo(self.view);
        make.width.mas_equalTo(self.view).multipliedBy(0.25);
    }];
```
 - 二级表（右）:创建一个子控制器，在子控制器中绘制右表
```
self.productsController = [[ProductsController alloc]init];
[self addChildViewController:self.productsController];
 [self.view addSubview:self.productsController.view];
```
 - 绘制tableViewCell
```
+ (instancetype)cellWithTable:(UITableView *)tableView {
    static NSString *cellId = @"ProductsCellID";
    ProductsCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
    if (cell == nil) {
        cell = [[ProductsCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
    }
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
    return cell;
}
```

#### 三、左右表二级联动

在第一个控制器中，声明一个代理协议方法；
```
@protocol CategoryTableViewDelagate <NSObject>
- (void)didTableView:(UITableView *)tableView clickedAtIndexPath:(NSIndexPath*)indexPath;
@end
@property (nonatomic,weak) id<CategoryTableViewDelagate>delegate;
```
```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    if ([self.delegate respondsToSelector:@selector(didTableView:clickedAtIndexPath:)]) {
        [self.delegate didTableView:self.categoriesTableView clickedAtIndexPath:indexPath];
    }
}
```
然后，由第二个控制器来调用该协议方法，实现左边表一点击列表分区，右侧表二相应的分区头视图调到列表头部。
```
- (void)didTableView:(UITableView *)tableView clickedAtIndexPath:(NSIndexPath*)indexPath;
{
    [self.productsTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:0 inSection:indexPath.row] animated:YES scrollPosition:UITableViewScrollPositionTop];
}
```
同样，右边控制左边，也是通过协议的方式来实现。当表二分区从上往下滑动时，即将展示分区头视图时，选中该分区对应的表一分区。当表二分区从下往上滑动时，即将结束分区头视图时，选中该分区对应的下一个表一分区；
```
@protocol ProductsDelegate<NSObject>
- (void)willDislayHeaderView:(NSInteger)section;
- (void)didEndDislayHeaderView:(NSInteger)section;
@end
@property (nonatomic,weak) id<ProductsDelegate>delegate;
```
```
-(void)tableView:(UITableView *)tableView willDisplayHeaderView:(UIView *)view forSection:(NSInteger)section
{
    if (!self.isScrollDown && [self.delegate respondsToSelector:@selector(willDislayHeaderView:)])
    {
        [self.delegate willDislayHeaderView:section];
    }
}
-(void)tableView:(UITableView *)tableView didEndDisplayingHeaderView:(UIView *)view forSection:(NSInteger)section
{
    if (self.isScrollDown && [self.delegate respondsToSelector:@selector(didEndDislayHeaderView:)])
    {
        [self.delegate didEndDislayHeaderView:section];
    }
}
```
由第一个控制器来实现该方法。
```
- (void)willDislayHeaderView:(NSInteger)section
{
    [self.categoriesTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:section inSection:0] animated:YES scrollPosition:UITableViewScrollPositionMiddle];
}
- (void)didEndDislayHeaderView:(NSInteger)section
{
    [self.categoriesTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:section+1 inSection:0] animated:YES scrollPosition:UITableViewScrollPositionMiddle];
}
```
#### 四、获得表数据

由于Demo需要，采用的是本地数据库数据，通过MJExtension动态加载属性。
```
NSString *path = [[NSBundle mainBundle] pathForResource:@"supermarket" ofType:nil];
    NSData *data = [NSData dataWithContentsOfFile:path];
    NSDictionary *json = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:nil];
    SuperMarketSource *source = [SuperMarketSource mj_objectWithKeyValues:json];
    SuperMarketData *superMarketData = source.data;
    for (NSInteger i = 0; i < superMarketData.categories.count; i++) {
        ProductCategory *catgeory = superMarketData.categories[i];
        NSArray *productsArr = superMarketData.products[catgeory.id];
        catgeory.products = [Goods mj_objectArrayWithKeyValuesArray:productsArr];
    }
    complete(superMarketData,nil);
```

#### 五、刷新表

自定义继承自MJRefreshGifHeader的一个类，通过实现父类的方法，来完成下拉刷新功能。
```
- (void)prepare {
    [super prepare];
    [self setImages:@[[UIImage imageNamed:@"v2_pullRefresh1"]] forState:MJRefreshStateIdle];
    [self setImages:@[[UIImage imageNamed:@"v2_pullRefresh2"]] forState:MJRefreshStatePulling];
    [self setImages:@[[UIImage imageNamed:@"v2_pullRefresh1"],[UIImage imageNamed:@"v2_pullRefresh2"]] forState:MJRefreshStateRefreshing];
    
    [self setTitle:@"下拉刷新" forState:MJRefreshStateIdle];
    [self setTitle:@"松开刷新" forState:MJRefreshStatePulling];
    [self setTitle:@"正在刷新" forState:MJRefreshStateRefreshing];
}
```
就这样，是不是很简单就完成了~~~
由于程序员文采不好，文中有不理解的或者有错误的地方，欢迎提问和指教。
最重要的来了，本Demo下载链接，[【GitHub】](https://github.com/JingJing-Lin/BeeQuick_One)   如果您觉得本Demo对您有用，欢迎点star ，谢谢 
注：技术无罪，本demo数据来自网络，如侵权，请联系我下架。