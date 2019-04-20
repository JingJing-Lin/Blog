![女神亦非](http://upload-images.jianshu.io/upload_images/1518951-500e29bb794cabaa.jpg)

关于支付这块，之前项目用的是Ping++支付，前些日子换成了官方SDK原生支付，为了使用方便，我封装了一下，现分享出来供iOS程序猿(媛)们参考和指导。
前言：关于支付必要的配置，官方文档说的很清楚了，这块就不细讲了(挺头疼的，坑多)。以下主要针对的代码封装：
- 声明一个单例类

```
static id _instance;
+(instancetype)sharedApi 
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [[MJPayApi alloc] init];
    });
 return _instance;
}
```
- 把支付返回的可能Code码枚举出来

```
typedef NS_ENUM(NSInteger, PayCode) {
    WXSUCESS            = 1001,   /**< 成功    */
    WXERROR             = 1002,   /**< 失败    */
    WXSCANCEL           = 1003,   /**< 取消    */
    WXERROR_NOTINSTALL  = 1004,   /**< 未安装微信   */
    WXERROR_UNSUPPORT   = 1005,   /**< 微信不支持    */
    WXERROR_PARAM       = 1006,   /**< 支付参数解析错误   */
    
    ALIPAYSUCESS        = 1101,   /**< 支付宝支付成功 */
    ALIPAYERROR         = 1102,   /**< 支付宝支付错误 */
    ALIPAYCANCEL        = 1103,   /**< 支付宝支付取消 */
};
```
#### 微信支付

- 发起微信支付

```
- (void)wxPayWithPayParam:(NSString *)pay_param
                  success:(void (^)(PayCode code))successBlock
                  failure:(void (^)(PayCode code))failBlock {
    self.PaySuccess = successBlock;
    self.PayError = failBlock;
    //Json解析
    NSData * data = [pay_param dataUsingEncoding:NSUTF8StringEncoding];
    NSError *error;
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:&error];
    if(error != nil) {
        failBlock(WXERROR_PARAM);
        return ;
    }
   //设置必要的参数
    NSString *appid = dic[@"appid"];
    NSString *partnerid = dic[@"partnerid"];
    NSString *prepayid = dic[@"prepayid"];
    NSString *package = @"Sign=WXPay";
    NSString *noncestr = dic[@"noncestr"];
    NSString *timestamp = dic[@"timestamp"];
    NSString *sign = dic[@"paySign"];
    
    [WXApi registerApp:appid];
    
    if(![WXApi isWXAppInstalled]) {
        failBlock(WXERROR_NOTINSTALL);
        return ;
    }
    if (![WXApi isWXAppSupportApi]) {
        failBlock(WXERROR_UNSUPPORT);
        return ;
    }
    
    //发起微信支付
    PayReq* req   = [[PayReq alloc] init];
    //微信分配的商户号
    req.partnerId = partnerid;
    //微信返回的支付交易会话ID
    req.prepayId  = prepayid;
    // 随机字符串，不长于32位
    req.nonceStr  = noncestr;
    // 时间戳
    req.timeStamp = timestamp.intValue;
    //暂填写固定值Sign=WXPay
    req.package   = package;
    //签名
    req.sign      = sign;
    [WXApi sendReq:req];
    
    //日志输出
 MJLog(@"appid=%@\npartid=%@\nprepayid=%@\nnoncestr=%@\ntimestamp=%ld\npackage=%@\nsign=%@",appid,req.partnerId,req.prepayId,req.nonceStr,(long)req.timeStamp,req.package,req.sign );
}
```
- 支付回调

```
// 微信终端返回给第三方的关于支付结果的结构体
- (void)onResp:(BaseResp *)resp
{
    if ([resp isKindOfClass:[PayResp class]])
    {
        switch (resp.errCode) {
            case WXSuccess:
                self.PaySuccess(WXSUCESS);
                break;
                
            case WXErrCodeUserCancel:   //用户点击取消并返回
                self.PayError(WXSCANCEL);
                break;
                
            default:        //剩余都是支付失败
                self.PayError(WXERROR);
                break;
        }
    }
}
```
- 单例类回调处理

```
-(BOOL) handleOpenURL:(NSURL *) url
{
  //([url.host isEqualToString:@"pay"]) //微信支付
    return [WXApi handleOpenURL:url delegate:self];
}
```
#### 支付宝支付

- 支付宝支付

```
-(void)aliPayWithPayParam:(NSString *)pay_param
                  success:(void (^)(PayCode code))successBlock
                  failure:(void (^)(PayCode code))failBlock
{
    self.PaySuccess = successBlock;
    self.PayError = failBlock;
    NSString * appScheme =  @"APP Scheme";
//注：若公司服务器返回的json串可以直接使用，就不用下面的json解析了
    NSData *jsonData = [pay_param dataUsingEncoding:NSUTF8StringEncoding];
    NSError *err;
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingMutableContainers error:&err];
    if(err) {
        NSLog(@"json解析失败：%@",err);
    }
    NSString * orderSS = [NSString stringWithFormat:@"app_id=%@&biz_content=%@&charset=%@&method=%@&sign_type=%@&timestamp=%@&version=%@&format=%@&notify_url=%@",dic[@"app_id"],dic[@"biz_content"],dic[@"charset"],dic[@"method"],dic[@"sign_type"],dic[@"timestamp"],dic[@"version"],dic[@"format"],dic[@"notify_url"]];
    NSString * signedStr = [self urlEncodedString:dic[@"sign"]];
    NSString * orderString = [NSString stringWithFormat:@"%@&sign=%@",orderSS, signedStr];
//    MJLog(@"===%@",orderSS);
    
    [[AlipaySDK defaultService] payOrder:orderString fromScheme:appScheme callback:^(NSDictionary *resultDic) {
        MJLog(@"----- %@",resultDic);
        NSInteger resultCode = [resultDic[@"resultStatus"] integerValue];
        switch (resultCode) {
            case 9000:     //支付成功
                successBlock(ALIPAYSUCESS);
                break;
            case 6001:     //支付取消
                failBlock(ALIPAYCANCEL);
                break;
            default:        //支付失败
                failBlock(ALIPAYERROR);
                break;
        }
    }];
}

//url 编码
- (NSString*)urlEncodedString:(NSString *)string
{
    NSString * encodedString = (__bridge_transfer  NSString*) CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault, (__bridge CFStringRef)string, NULL, (__bridge CFStringRef)@"!*'();:@&=+$,/?%#[]", kCFStringEncodingUTF8 );
    return encodedString;
}


```
- 单例类回调处理

```
- (BOOL) handleOpenURL:(NSURL *) url
{
    if ([url.host isEqualToString:@"safepay"])
    {
        // 支付跳转支付宝钱包进行支付，处理支付结果
        [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
             //【由于在跳转支付宝客户端支付的过程中，商户app在后台很可能被系统kill了，所以pay接口的callback就会失效，请商户对standbyCallback返回的回调结果进行处理,就是在这个方法里面处理跟callback一样的逻辑】
            NSLog(@"result = %@",resultDic);
            NSInteger resultCode = [resultDic[@"resultStatus"] integerValue];
            switch (resultCode) {
                case 9000:     //支付成功
                    self.PaySuccess(ALIPAYSUCESS);
                    break;
                case 6001:     //支付取消
                    self.PaySuccess(ALIPAYCANCEL);
                    break;
                default:        //支付失败
                    self.PaySuccess(ALIPAYERROR);
                    break;
            }
        }];
        
        // 授权跳转支付宝钱包进行支付，处理支付结果
        [[AlipaySDK defaultService] processAuth_V2Result:url standbyCallback:^(NSDictionary *resultDic) {
            MJLog(@"result = %@",resultDic);
            // 解析 auth code
            NSString *result = resultDic[@"result"];
            NSString *authCode = nil;
            if (result.length>0) {
                NSArray *resultArr = [result componentsSeparatedByString:@"&"];
                for (NSString *subResult in resultArr) {
                    if (subResult.length > 10 && [subResult hasPrefix:@"auth_code="]) {
                        authCode = [subResult substringFromIndex:10];
                        break;
                    }
                }
            }
            MJLog(@"授权结果 authCode = %@", authCode?:@"");
        }];
        return YES;
    } 
}
```
#### 支付调用    

好了，哪里用到写哪里
- 微信

```
 [[MJPayApi sharedApi]wxPayWithPayParam:data success:^(PayCode code)
             {
                 [self payStatus:code Co_nbr:co_nbr];
             } failure:^(PayCode code) {
                 [self payStatus:code Co_nbr:co_nbr];
             }];
```
- 支付宝

```
[[MJPayApi sharedApi]aliPayWithPayParam:data success:^(PayCode code)
             {
                 [self payStatus:code Co_nbr:co_nbr];
             } failure:^(PayCode code) {
                 [self payStatus:code Co_nbr:co_nbr];
             }];
```
是不是很简单那~~~

#### 重要：

封装好之后，收到了好多简友的留言和私信，所以抽个时间把项目传到了github，如果能给正在蹲坑的码友们一丝帮助，请标个※星※。

Demo地址：[喜欢，就标个星吧](https://github.com/JingJing-Lin/MJPayApi)