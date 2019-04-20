![图片发自简书App](http://upload-images.jianshu.io/upload_images/1518951-e8f3f75f534bade1.jpg)

前段时间，做马甲包相关的知识，把H5资源下载到本地，然后从本地解析js，渲染并加载css和图片等。再此过程中，遇到了URL字符串自动转义的问题，记录一下~

项目需要从本地加载的Url链接是
> /var/mobile/Containers/Data/Application/22438350-8530-4B0B-BFDD-FBCE7A9F873B/Documents/components/dist/main.html#/main

但是调用
>NSURL * URL = [NSURL fileURLWithPath:indexHtmlPath];

打印`URL.absoluteString`却变成了
>file://var/mobile/Containers/Data/Application/22438350-8530-4B0B-BFDD-FBCE7A9F873B/Documents/components/dist/main.html%23/main

什么鬼，我的 `#` 被`%23`吃了，快还我`#`。我找来了“谷歌”和“百度”两位大神来赶走`%23`，结果呐，铩羽而归~

原来，webView的Url链接中的特殊字符串在未经我允许的情况下，摇身一变，我不认识了。QAQ~ 

URL编码和ASCII码值间的转换：
```
     +    URL 中+号表示空格                      %2B
     空格 URL中的空格可以用+号或者编码              %20
     /   分隔目录和子目录                         %2F
     ?    分隔实际的URL和参数                     %3F
     %    指定特殊字符                           %25
     #    表示书签                              %23
     &    URL 中指定的参数间的分隔符               %26
     =    URL 中指定参数的值                      %3D

```
那既然这样，就想着解码吧，但是试了好几种方法，都无法阻挡Url转义，也是醉了~
做为一名打不死的小强，岂能就此放弃，解码走不通，能不能换一种方法呐？！结果，还真被我想出来了~

对比两个Url链接，除了`#` 被`%23`替换之外，链接还加了前缀`file:/`,我们能不能直接在初始化之前，自己拼接Url加上前缀，接着用我们常见的Url初始化方法，来初始化呐，答案是肯定de

> indexHtmlPath = [NSString stringWithFormat:@"file:/%@", indexHtmlPath];
> NSURL * URL = [NSURL URLWithString: indexHtmlPath];

如果你有更好的解决方法，欢迎留言~