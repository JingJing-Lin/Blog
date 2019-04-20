#### 副标题：图片下载

最近做一个首页悬浮按钮功能，我们的逻辑是根据接口来获取图片链接及是否显示,类似于美团这种功能~

![美团.jpg](https://upload-images.jianshu.io/upload_images/1518951-b5d313e0281993a2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

尴尬的是：SDWebImage下载的PNG图片却有白底

```
[[SDWebImageManager sharedManager] downloadImageWithURL:iconUrl options:SDWebImageAvoidAutoSetImage progress:^(NSInteger receivedSize, NSInteger expectedSize) {
        } completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
            if (image) {
                  // 刷新UI
            }
  }];
```
以前也没遇到这种情况呀，捉急上线，就没细细研究，于是另辟蹊径，采用异步执行下载图片方式 ~

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            NSData *data = [NSData dataWithContentsOfURL:iconUrl];
            if (data) {
                UIImage *image = [UIImage imageWithData:data];
                dispatch_async(dispatch_get_main_queue(), ^{
                    if (image) {
                        // 刷新UI
                    }
                });
            }
  });
```
但是测试反馈，悬浮按钮显示过慢，有时候很久才显示出来 ~
也对，`dataWithContentsOfURL ` 确实是耗时操作~
于是，想着第一次下载成功后，保存key值及图片数据到沙盒中，下次直接从里面获取，不就解决了嘛，oh，yeah！
```
NSString * diskCachePath = [NSFileManager mar_getCacheDirectoryForFile:@"imageCache"];
DLOG(@"--paths:%@",diskCachePath);
//如果目录imageCache不存在，创建目录
if (![[NSFileManager defaultManager] fileExistsAtPath:diskCachePath]) {
        NSError *error=nil;
        [[NSFileManager defaultManager] createDirectoryAtPath:diskCachePath withIntermediateDirectories:YES attributes:nil error:&error];
}
NSString *txtPath = [diskCachePath stringByAppendingPathComponent:@"imagekey.txt"];
NSString *imgPath = [diskCachePath stringByAppendingPathComponent:@"capsuletoys"];
        
NSString *resultStr = [NSString stringWithContentsOfFile:txtPath encoding:NSUTF8StringEncoding error:nil];
if ([resultStr isEqualToString:autoString(model.icon)]) {
        NSData *resultData = [NSData dataWithContentsOfFile:imgPath];
        UIImage *resultImage = [UIImage imageWithData:resultData];
            
         // 刷新UI
}else{
        [self downloadCapsuleToysImageWithIcon:autoString(model.icon) Textpath:txtPath ImagePath:imgPath];
 }

```
```
/**
 第一次加载图片并保存key值及图片data到沙盒中
 */
- (void)downloadCapsuleToysImageWithIcon:(NSString*)urlStr Textpath:(NSString*)txtPath ImagePath:(NSString *)imgPath {
    
    NSURL *iconUrl = [NSURL URLWithString:[urlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]]];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSData *data = [NSData dataWithContentsOfURL:iconUrl];
        if (data) {
            UIImage *image = [UIImage imageWithData:data];
            dispatch_async(dispatch_get_main_queue(), ^{
                if (image) {
                    
                    [urlStr writeToFile:txtPath atomically:YES encoding:NSUTF8StringEncoding error:nil];
                    NSData * imageData = UIImagePNGRepresentation(image);
                    [imageData writeToFile:imgPath atomically:YES];
                    
                     // 刷新UI
                }
            });
        }
    });
}
```
这样做，很好的解决了，但是，不是很完美，能不能再继续优化呐，和同事讨论后，采用SDWebImage的实现逻辑方式，把图片的data数据命名为key，(iconUrl的MD5加密后的值)，这样的代码看着才舒服嘛~ 

```
NSString * diskCachePath = [NSFileManager mar_getCacheDirectoryForFile:@"mxrimageCache"];
//如果目录imageCache不存在，创建目录
if (![[NSFileManager defaultManager] fileExistsAtPath:diskCachePath]) {
    NSError *error=nil;
    [[NSFileManager defaultManager] createDirectoryAtPath:diskCachePath withIntermediateDirectories:YES attributes:nil error:&error];
}
DLOG(@"--paths:%@",diskCachePath);

NSString *url = autoString(model.icon);
NSString *key = [url mar_md5String];
NSString *imagePath = [diskCachePath stringByAppendingPathComponent:key];

BOOL isDirectory = NO;
BOOL isExist = [[NSFileManager defaultManager] fileExistsAtPath:imagePath isDirectory:&isDirectory];
if (isExist && !isDirectory) {
    UIImage *resultImage = [UIImage imageWithContentsOfFile:imagePath];
     // 刷新UI
}
else{
    [self downloadCapsuleToysImageWithIcon:autoString(model.icon) ImagePath:imagePath];
}
```

```
/**
 第一次加载图片并保存key值到沙盒中
 */
- (void)downloadCapsuleToysImageWithIcon:(NSString*)urlStr ImagePath:(NSString*)imagePath
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSURL *iconUrl = [NSURL URLWithString:[urlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]]];
        NSData *data = [NSData dataWithContentsOfURL:iconUrl];
        UIImage *image = [UIImage imageWithData:data];
        if (image) {
            NSData * imageData = UIImagePNGRepresentation(image);
            [imageData writeToFile:imagePath atomically:YES];
            dispatch_async(dispatch_get_main_queue(), ^{
                [self setupCapsuleToysWithImage:image SkipUrl:ServiceURL_CapsuleToys_WEB_URL];
            });
        }
    });
}
```

功能完成后，抽个时间，查看SDWebImage的源码，发现是image压缩的时候，调用了
`
NSData *data = UIImageJPEGRepresentation(self, 1.0);
`
导致了下载透明图片显示白底的问题，在仔细看，不对劲啊，这个压缩方法不是SDWebImage里面的啊，原来是以前的同事由于某个需求加进去了，这就尴尬了，纯属乌龙事件啊，真是冤枉SDWebImage了，在此向[her](https://github.com/rs/SDWebImage)道歉了~

