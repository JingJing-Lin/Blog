Xcode的代码片段(Code Snippets)创建自定义的代码片段，当你重用这些代码片段时，会给你带来很大的方便。
###  常用的：
1.strong：
`@property (nonatomic, strong) <#Class#> *<#object#>;  `
2.weak：
`@property (nonatomic, weak) <#Class#> *<#object#>;  `
3.copy：
`@property (nonatomic, copy) NSString *<#string#>;  `
4.assign：
`@property (nonatomic, assign) <#Class#> <#property#>;  `
5.delegate：
`@property (nonatomic, weak) id<<#protocol#>> <#delegate#>;  `
6.block：
`@property (nonatomic, copy) <#returnType#>(^<#blockName#>)(<#arguments#>); `
7.mark：
`#pragma mark - <#mark#>  `
8.ReUseCell：
```
static NSString *rid=<#rid#>;  
  
 <#Class#> *cell=[tableView dequeueReusableCellWithIdentifier:rid];  
  
 if(cell==nil){  
  
 cell=[[<#Class#> alloc] initWithStyle:UITableViewCellStyleDefault      reuseIdentifier:rid];  
  
 }  
  
 return cell;  
```

9.MainGCD：
```
dispatch_async(dispatch_get_main_queue(), ^{  
<#code#>  
  });  
```
10.AfterGCD：
```
 dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(<#delayInSeconds#> * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{  
<#code to be executed after a specified delay#>  
});  
```
11.单例（OnceGCD）
```
static <#class#> *singleClass = nil;

+ (instancetype)shareInstance{
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        <#code to be executed once#>
        
    });
    return <#expression#>
}
}
```
12.TableView
```
- (UITableView *)tableView {
    
    if(!<#tableview#>) {
        
        <#tableview#> = [[UITableView alloc]initWithFrame:self.view.bounds style:UITableViewStylePlain];
        
        <#tableview#>.delegate =self;
        
        <#tableview#>.dataSource =self;
        
    }
    return<#tableview#>;
}

#pragma mark - tableView delegate

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    
    return <#expression#>
    
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    
    return <#expression#>
    
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    static NSString *identify =@"cellIdentify";
    
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identify];
    
    if(!cell) {
        
        cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleValue1 reuseIdentifier:identify];
        
    }
    
    cell.textLabel.text =self.arrayTitle[indexPath.row];
    
    return cell;
    
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    
}
```

##自定义代码片段：Xcode10之前
以Strong为例：
1.在书写@property属性的地方写下如下语句：
`@property (nonatomic,strong) <#Class#> *<#object#>;  `

2.选中上述语句，用鼠标左键拖到 下图中指示的代码片段在Xcode中的区域里，就新建了一个代码片段

![屏幕快照 2016-09-28 上午11.01.52.png](http://upload-images.jianshu.io/upload_images/1518951-56d8edf5f660204b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.松开鼠标左键的同时，会弹出代码片段编辑窗口，如下图所示：

![屏幕快照 2016-09-28 上午10.56.47.png](http://upload-images.jianshu.io/upload_images/1518951-394fdb2756a05d90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图中从上到下的含义依次是：
①Title
代码片段的标题
②Summary
代码片段的描述文字
③Platform
可以使用代码片段的平台，有IOS/OS X/All三个选项
④Language
可以在哪些语言中使用该代码片段
⑤Completion Shortcut
代码片段的快捷方式，例：copy
⑥Completion Scopes
可以在哪些文件中使用当前代码片段，比如全部位置，头文件中等，当然可以添加多个支持的位置。
最后的一个大得空白区域是对代码片段的效果预览。
一切设置完成以后，点击该菜单右下角的Done按钮，新建工作就结束了。

##Xcode10添加代码块：
![Xcode10.png](https://upload-images.jianshu.io/upload_images/1518951-27579c70e918e691.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##代码片段备份：
Xcode中的代码片段默认放在下面的目录中：

`~/Library/Developer/Xcode/UserData/CodeSnippets   `

我们可以将目录中的代码片段备份，也可以将其直接拷出来放在不同的电脑上使用。

欢迎关注我的博客，我会不定时更新