![goddess.JPG](https://upload-images.jianshu.io/upload_images/1518951-bbb8c945ba279be3.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 前言
>单元测试简单来说，就是为了方便测试一些功能是否正常运行，调试接口是否能正常使用，用代码去检测代码是否正确的一种手段。例如：你为了测试某一个网络接口，每次都重新启动，经过很多操作之后，才测试到那个网络接口。如果使用了单元测试，就可以直接测试那个方法，相对方便很多。单元测试不仅没有降低我们Coding的效率，也能保证在之后的改动中及时发现可能出现的错误。

学习单元测试之前，让我们先来看看一些常用第三方所选用的测试框架：

![图1.jpg](https://upload-images.jianshu.io/upload_images/1518951-d6bdfefd826319eb.jpg?imageMogr2/auto-orient/strip)

从图中得知，苹果官方的测试框架XCTest 还是很受欢迎的哈 ~

并不是所有的方法都需要测试，一般而言，私有方法不需要测试，只有暴露在 .h 中的方法需要测试。那到底测试用例的覆盖率是多少才合适呐？其实一个软件覆盖度在50%以上就可以称为一个健壮的软件了，要达到70，80这些已经是非常难了，但“**史莱克从来不缺少天才**”，例如：AFNNetWorking的覆盖率高达88%,SDWebImage的覆盖率也达到75%。

![图2.png](https://upload-images.jianshu.io/upload_images/1518951-ac9eae71fc795c52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图3.png](https://upload-images.jianshu.io/upload_images/1518951-f827b4c03f90dab0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 一、集成

1. 创建工程的时候，直接勾选 Include Unit Tests

![屏幕快照_2018-08-01_下午6_33_49.png](https://upload-images.jianshu.io/upload_images/1518951-91f0722941124c3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.如果已有项目未勾选，则通过以下方式再创建一个
File-->new-->Target-->iOS-->iOS Unit Testing Bundle

![图4.png](https://upload-images.jianshu.io/upload_images/1518951-96ae992981e225d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.工程创建好之后，找到系统单元测试Tests 文件夹，在 .m文件中就可以写我们的测试用例了，是不是很简单呐~

![图5.png](https://upload-images.jianshu.io/upload_images/1518951-7f5b6fb1e75d50fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.一般我们会新建不同的测试用例类与代码类一一对应，可以通过新建 Unit Test Case Class 来实现

![图6.png](https://upload-images.jianshu.io/upload_images/1518951-ea2b39b76733d5b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 二、方法

测试用例类 .m 文件中，会有几个默认方法，我们来看下这几个方法是什么时候调用和他们的作用：

```
- (void)setUp {
    [super setUp];
    //初始化，在测试方法调用之前调用
}

- (void)tearDown {
    // 释放测试用例的资源代码，这个方法会每个测试用例执行后调用
    [super tearDown];
}

- (void)testExample {
    // 测试用例的例子，注意测试用例一定要test开头
}

- (void)testPerformanceExample {
    [self measureBlock:^{
        // 需要测试性能的代码
    }];
}
```
>**注意：**测试用例必须以Test开头，且不能有参数，不然不会被识别。


#### 三、使用

1. 快捷键 Command + U 会运行全部单元测试；
2. 鼠标放在方法右边，会出现播放按钮，点击后开始单个方法的测试；

![图7.png](https://upload-images.jianshu.io/upload_images/1518951-88b9862fa96d4cf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 鼠标放在方法左边，会出现播放按钮，点击后开始单个方法的测试；
  ![图8.png](https://upload-images.jianshu.io/upload_images/1518951-0339965d69e5a36d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 如测试通过，会有“Test Succeeded”提示,且函数左边菱形图标展示为绿色；如测试不通过，会有“Test Failed”提示,且函数左边菱形图标展示为红色。

![图9.png](https://upload-images.jianshu.io/upload_images/1518951-f029db6a8217a3e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 四、测试

#####1. 基本**断言**的逻辑测试，关于断言会在文末说明；
**例1：**有一个函数目的是生成在[base, end]之间的随机数，我们来检测一下会不会出现越界的情况：
```
// 生成在[base, end]之间的随机数
- (int)randomNumberFrom: (int)base End: (int)top{
    if (base >= top) {
        return base;
    }
    return (arc4random() % (top - base + 1)) + base;
}
```
```
- (void)testRandom{
    int base = 3;
    int top = 80;
    
    for (int i=0; i<100; i++) {
        int temp = [self randomNumberFrom:base End:top];
        if (temp < base || temp > top) {
            XCTFail(@"invalid num = %d",temp);
        }
    }
}
```
**例2：**在ViewController中写一个简单的方法
```
- (int)getNum{
    return 100;
}
```
在测试的文件中导入ViewController.h，并且定义一个vc属性
```
#import <XCTest/XCTest.h>
#import "ViewController.h"

@interface MJViewControllerTest : XCTestCase
@property (nonatomic, strong) ViewController *VC;
@end

@implementation MJViewControllerTest
```
测试用例的实现
```
- (void)setUp {
    [super setUp];
    self.VC = [[ViewController alloc]init];
}

- (void)tearDown {
    self.VC = nil;
    [super tearDown];
}

- (void)testGetNum{
    int result = [self.VC getNum];
    XCTAssertEqual(result, 100, @"不相等，测试不通过");
}
```
运行测试用例，可以看到测试通过，菱形图标显示绿色。
如果这时我们改下断言，把100随便改成一个数，则测试不通过，如下：

![图10.png](https://upload-images.jianshu.io/upload_images/1518951-e781a1dfafb02ab1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2. 异步测试
代码中会有很多异步的场景需要验证，例如网络请求callback中执行的操作，由于测试方法主线程执行完就会结束，所以需要在方法结束前设置等待，调回回来的时候再让它继续执行，如果超时或者是遇到断言的失败，该用例会失败。

>**注意：**使用pod的项目中，在XC测试框架中测试内容包括第三方包时，需要手动去设置Header Search Paths才能找到头文件

1. `expectationForNotification` 方法 ,该方法监听一个通知,如果在规定时间内正确收到通知则测试通过。

```
#define WAIT do {\
[self expectationForNotification:@"MJUnitTest" object:nil handler:nil];\
[self waitForExpectationsWithTimeout:30 handler:nil];\
} while (0);
// waitForExpectationsWithTimeout是等待时间，超过了就不再等待往下执行。
#define NOTIFY \
[[NSNotificationCenter defaultCenter]postNotificationName:@"MJUnitTest" object:nil];

```

```
- (void)testRequest{
    
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/html"];
    NSString *urlStr = @"http://www.weather.com.cn/data/cityinfo/101190401.html";
    [manager GET:urlStr parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        NSLog(@"responseObject:%@",responseObject);
        XCTAssertNotNil(responseObject, @"返回出错");
        NOTIFY //继续执行
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        NSLog(@"error:%@",error);
        XCTAssertNil(error, @"请求出错");
        NOTIFY //继续执行
        
    }];
    WAIT   //暂停
}
```
2.`expectationWithDescription ` 来进行异步是否完成期望的测试。
```
- (void)testRequest2{
    
    XCTestExpectation *exp = [self expectationWithDescription:@"接口请求失败。。。"];
    
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/html"];
    NSString *urlStr = @"http://www.weather.com.cn/data/cityinfo/101190401.html";
    [manager GET:urlStr parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        NSLog(@"responseObject2:%@",responseObject);
        XCTAssertNotNil(responseObject, @"返回出错");
        //如果断言没问题，就调用fulfill宣布测试满足
        [exp fulfill];
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        NSLog(@"error:%@",error);
        XCTAssertNil(error, @"请求出错");
        
        [exp fulfill];
        
    }];
    
    //设置延迟多少秒后，如果没有满足测试条件就报错
    [self waitForExpectationsWithTimeout:15 handler:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"Timeout Error: %@", error);
        }
    }];
}
```
3.`expectationForPredicate `测试方法，代码来自于AFNetworking，用于测试backgroundImageForState方法
```
- (void)testThatBackgroundImageChanges {
    XCTAssertNil([self.button backgroundImageForState:UIControlStateNormal]);
    NSPredicate *predicate = [NSPredicate predicateWithBlock:^BOOL(UIButton  * _Nonnull button, NSDictionary<NSString *,id> * _Nullable bindings) {
            return [button backgroundImageForState:UIControlStateNormal] != nil;
    }];

    [self expectationForPredicate:predicate
              evaluatedWithObject:self.button
                          handler:nil];
    [self waitForExpectationsWithTimeout:20 handler:nil];
}
```
利用谓词计算，button是否正确的获得了backgroundImage，如果正确20秒内正确获得则通过测试，否则失败。

##### 3.性能测试
将要测量执行时间的代码放到testPerformanceExample方法内部的block中:
```
- (void)testPerformanceExample {
    
    [self measureBlock:^{
        
        NSMutableArray * mutArray = [[NSMutableArray alloc] init];
        for (int i = 0; i < 10000; i++) {
            NSObject * object = [[NSObject alloc] init];
            [mutArray addObject:object];
        }
    }];
}
```
在block中写一个for循环执行10000次，然后点击方法左边的菱形图标,得到：average: 0.003sec

![图11.png](https://upload-images.jianshu.io/upload_images/1518951-16671bf5908334e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以从控制台打印信息获取程序运行10次的时间，取一个平均运行时间值：
```
measured [Time, seconds] average: 0.003, relative standard deviation: 9.329%,   
values: [0.002840, 0.002487, 0.003074, 0.002515, 0.002386, 0.002313, 0.002351, 0.002362, 0.002455, 0.002741], 
```

####五、断言
```
XCTFail(format…) 生成一个失败的测试； 
XCTAssertNil(a1, format...)为空判断，a1为空时通过，反之不通过；
XCTAssertNotNil(a1, format…)不为空判断，a1不为空时通过，反之不通过；
XCTAssert(expression, format...)当expression求值为TRUE时通过；
XCTAssertTrue(expression, format...)当expression求值为TRUE时通过；
XCTAssertFalse(expression, format...)当expression求值为False时通过；
XCTAssertEqualObjects(a1, a2, format...)判断相等，[a1 isEqual:a2]值为TRUE时通过，其中一个不为空时，不通过；
XCTAssertNotEqualObjects(a1, a2, format...)判断不等，[a1 isEqual:a2]值为False时通过；
XCTAssertEqual(a1, a2, format...)判断相等（当a1和a2是 C语言标量、结构体或联合体时使用, 判断的是变量的地址，如果地址相同则返回TRUE，否则返回NO）；
XCTAssertNotEqual(a1, a2, format...)判断不等（当a1和a2是 C语言标量、结构体或联合体时使用）；
XCTAssertEqualWithAccuracy(a1, a2, accuracy, format...)判断相等，（double或float类型）提供一个误差范围，当在误差范围（+/-accuracy）以内相等时通过测试；
XCTAssertNotEqualWithAccuracy(a1, a2, accuracy, format...) 判断不等，（double或float类型）提供一个误差范围，当在误差范围以内不等时通过测试；
XCTAssertThrows(expression, format...)异常测试，当expression发生异常时通过；反之不通过；（很变态） XCTAssertThrowsSpecific(expression, specificException, format...) 异常测试，当expression发生specificException异常时通过；反之发生其他异常或不发生异常均不通过；
XCTAssertThrowsSpecificNamed(expression, specificException, exception_name, format...)异常测试，当expression发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertNoThrow(expression, format…)异常测试，当expression没有发生异常时通过测试；
XCTAssertNoThrowSpecific(expression, specificException, format...)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertNoThrowSpecificNamed(expression, specificException, exception_name, format...)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过

```

文章附件：[Demo](https://github.com/JingJing-Lin/MJUnitTestDemo)
>附录：本来写好了，去开了个需求会议，回来打开页面，内容丢了一半，(╥╯^╰╥)，历史版本也没保存，不得不重新写了一遍，下次一定要备份，看在辛苦的份上，喜欢的点个赞吧~

参考文章：
[iOS单元测试(作用及入门提升)](https://www.jianshu.com/p/8bbec078cabe)
[浅谈iOS单元测试](https://www.jianshu.com/p/1e8d6885d517)
[iOS单元测试初探以及OCMock使用入门](https://www.jianshu.com/p/c37bde847682)
[iOS-使用Xcode自带单元测试UnitTest](https://www.jianshu.com/p/009844a0b9ed)
[iOS - UnitTests 单元测试](https://www.cnblogs.com/QianChia/p/6379191.html)
[iOS单元测试](https://www.jianshu.com/p/81b10745044b)