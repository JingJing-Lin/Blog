**Spritekit**是iOS 7之后苹果官方推出的2D游戏开发框架，最近利用业余时间认真学习了这方面的知识，并利用网上资源及教程用Swift语言仿写了一个以前比较火的小游戏**FlappyBird** 。

![bird.gif](http://upload-images.jianshu.io/upload_images/1518951-845b3e1bdb1df620.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.准备

新建一个Project项目，模板选择**Game**，语言选择**Swift**，开发库选择**SpriteKit**

![3.41.51.png](http://upload-images.jianshu.io/upload_images/1518951-d8c75b64f5845922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![3.43.14.png](http://upload-images.jianshu.io/upload_images/1518951-81abc457cf968913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

删除xcode自动创建的实例文件**GameScene.sks**和**Actions.sks**，把GameViewController.swift中**viewDidLoad**方法中的代码替换为下面的代码
```
 if let view = self.view as! SKView? {
            
            let scene = GameScene(size:view.bounds.size)
                scene.scaleMode = .aspectFill
                view.presentScene(scene)
            view.ignoresSiblingOrder = true
            
            view.showsFPS = true
            view.showsNodeCount = true
        }
```
删除**GameScene.swift**中的代码，留下**didMove**和**update**方法，在**didMove**方法中添加背景颜色，如下
```
override func didMove(to view: SKView) {
        self.backgroundColor =  SKColor(red: 81.0/255.0, green: 192.0/255.0, blue: 201.0/255.0, alpha: 1.0)
    }
```
这时候就可以运行代码了

![16.23.45.png](http://upload-images.jianshu.io/upload_images/1518951-c99c74f4373ae6e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

导入资源文件
新建一个**.atlas**为后缀的文件，放入小鸟的图片，然后将这个文件夹拖到工程中。然后把其他图片放在Asserts.xcasserts里就可以了。

![4.27.57.png](http://upload-images.jianshu.io/upload_images/1518951-623fd8b2315debb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注：因为当你把一类相关的贴图图片素材放在一个.atlas文件夹里，编译程序的时候Xcode会把这个文件夹里的图片都导入“纹理图集”里，相对于只用独立的图片文件而言，使用纹理图集会非常显著地提升游戏的渲染性能。
部分xcode版本，加载.atlas中的图片时，提示 SKTexture: Error loading image resource: "bird-01.png"
是因为： Xcode在直接Add Files to Project 图片文件的时候,没有自动将其添加到编译资源文件中,需要去项目中(Build Phases)手动添加资源文件(copy Bundle Resource)。

到此，准备工作结束~

#### 2.布置场景

切换到GameScene.swift类中，添加变量
```
/// 小鸟精灵
var bird :SKSpriteNode!
````
然后再在didMove()方法里添加下面的代码
```
 // 地面
 let groundTexture = SKTexture(imageNamed: "land")
 groundTexture.filteringMode = .nearest
 for i in 0..<2 + Int(self.frame.size.width / (groundTexture.size().width * 2)) {
       let i = CGFloat(i)
       let sprite = SKSpriteNode(texture: groundTexture)
       sprite.setScale(2.0)
       // SKSpriteNode的默认锚点为(0.5,0.5)即它的中心点。
       sprite.anchorPoint = CGPoint(x: 0, y: 0)
       sprite.position = CGPoint(x: i * sprite.size.width, y: 0)
       self.addChild(sprite)
}
```
```
// 天空
let skyTexture = SKTexture(imageNamed: "sky")
    skyTexture.filteringMode = .nearest
    for i in 0..<2 + Int(self.frame.size.width / (skyTexture.size().width * 2)) {
        let i = CGFloat(i)
        let sprite = SKSpriteNode(texture: skyTexture)
        sprite.setScale(2.0)
        sprite.zPosition = -20
        sprite.anchorPoint = CGPoint(x: 0, y:0)
        sprite.position = CGPoint(x: i * sprite.size.width, y:groundTexture.size().height * 2.0)
        moving.addChild(sprite)
    }
```
```
//小鸟
bird = SKSpriteNode(imageNamed: "bird-01")
    bird.setScale(1.5)
    bird.position = CGPoint(x: self.frame.size.width * 0.35, y: self.frame.size.height * 0.6)
    addChild(bird)
```
运行代码，如下图所示：

![18.06.29.png](http://upload-images.jianshu.io/upload_images/1518951-9d5327508e37b95a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.精灵动起来

由于地面移动和天空移动类似，都是两个图片交替出现，只是速度不同，所以我们封装一个方法，分别传入对应的精灵及时间。如下：
```
//陆地及天空移动动画
func moveGround(sprite:SKSpriteNode,timer:CGFloat) {
    let moveGroupSprite = SKAction.moveBy(x: -sprite.size.width, y: 0, duration: TimeInterval(timer * sprite.size.width))
    let resetGroupSprite = SKAction.moveBy(x: sprite.size.width, y: 0, duration: 0.0)
    //永远移动 组动作
    let moveGroundSpritesForever = SKAction.repeatForever(SKAction.sequence([moveGroupSprite,resetGroupSprite]))
    sprite.run(moveGroundSpritesForever)
}
```
然后在上面代码中，添加精灵之前，就是在 self.addChild(sprite)之前，分别加入精灵动作
```
// 地面
self.moveGround(sprite: sprite, timer: 0.02)
```
```
//天空
self.moveGround(sprite: sprite, timer: 0.1)
```
之后是我们的主要对象小鸟精灵的活灵活现~
```
///  小鸟飞的动画
func birdStartFly()  {
    let birdTexture1 = SKTexture(imageNamed: "bird-01")
    birdTexture1.filteringMode = .nearest
    let birdTexture2 = SKTexture(imageNamed: "bird-02")
    birdTexture2.filteringMode = .nearest
    let birdTexture3 = SKTexture(imageNamed: "bird-03")
    birdTexture3.filteringMode = .nearest
    let anim = SKAction.animate(with: [birdTexture1,birdTexture2,birdTexture3], timePerFrame: 0.2)
    bird.run(SKAction.repeatForever(anim), withKey: "fly")
}
 ///  小鸟停止飞动画
 func birdStopFly()  {
     bird.removeAction(forKey: "fly")
 }
```
运行效果图如下，看小鸟是不是真的飞起来了，哈哈：

![birdFly.gif](http://upload-images.jianshu.io/upload_images/1518951-9d1b9c714399d2c1.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.随机创建管道

首先添加变量
```
/// 竖直管缺口
let verticalPipeGap = 150.0;
/// 向上管纹理
var pipeTextureUp:SKTexture!
/// 向下管纹理
var pipeTextureDown:SKTexture!
/// 储存所有上下管道
var pipes:SKNode!
````
实现上下管道的随机创建及消失，需要下面四个方法：
- 方法creatSpawnPipes()    创建具体某一次某一对水管的方法
- 方法startCreateRandomPipes()    开始随机重复创建水管的动作方法
- 方法stopCreateRandomPipes()     停止创建水管的动作方法 
- 方法removeAllPipesNode()  移除所有正在场景里的水管

```
///创建一对水管
func creatSpawnPipes() {
    // 管道纹理
    pipeTextureUp = SKTexture(imageNamed: "PipeUp")
    pipeTextureUp.filteringMode = .nearest
    pipeTextureDown = SKTexture(imageNamed: "PipeDown")
    pipeTextureDown.filteringMode = .nearest
    
    let pipePair = SKNode()
    pipePair.position = CGPoint(x: self.frame.size.width + pipeTextureUp.size().width * 2, y: 0)
    // z值的节点(用于排序)。负z是”进入“屏幕,正面z是“出去”屏幕。
    pipePair.zPosition = -10;
    
    // 随机的Y值
    let height = UInt32(self.frame.size.height / 5)
    let y = Double(arc4random_uniform(height) + height)
    
    let pipeDown = SKSpriteNode(texture: pipeTextureDown)
    pipeDown.setScale(2.0)
    pipeDown.position = CGPoint(x: 0.0, y: y + Double(pipeDown.size.height)+verticalPipeGap)
    pipePair.addChild(pipeDown)
    
    let pipeUp = SKSpriteNode(texture: pipeTextureUp)
    pipeUp.setScale(2.0)
    pipeUp.position = CGPoint(x: 0.0, y: y)
    pipePair.addChild(pipeUp)
    
    // 管道移动动作
    let distanceToMove = CGFloat(self.frame.size.width + 2.0*pipeTextureUp.size().width)
    let movePipes = SKAction.moveBy(x: -distanceToMove, y: 0.0, duration: TimeInterval(0.01 * distanceToMove))
    let removePipes = SKAction.removeFromParent()
    let movePipesAndRemove = SKAction.sequence([movePipes,removePipes])
    pipePair.run(movePipesAndRemove)
    
    pipes.addChild(pipePair)
}
```
```
/// 随机 创建
func startCreateRandomPipes() {
    let spawn = SKAction.run {
        self.creatSpawnPipes()
    }
    let delay = SKAction.wait(forDuration: TimeInterval(2.0))
    let spawnThenDelay = SKAction.sequence([spawn,delay])
    let spawnThenDelayForever = SKAction.repeatForever(spawnThenDelay)
    self.run(spawnThenDelayForever, withKey: "createPipe")
}
```
```
///停止创建管道
func stopCreateRandomPipes() {
    self.removeAction(forKey: "createPipe")
}
/// 移除所有已经存在的上下管
func removeAllPipesNode() {
    pipes.removeAllChildren()
}
```
![birdFly2.gif](http://upload-images.jianshu.io/upload_images/1518951-b17ff3795efd85ab.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

写到这里，我们已经完成了这个游戏效果的一半了，但是细心的你会发现，点击屏幕无效果，小鸟不受重力影响下落，小鸟与水管相撞没反应等。。。
那么牵扯出了下一部分，物理世界~

#### 5.物理世界

然后再在didMove()方法里，背景色代码下面，添加下面的代码
```
//给场景添加一个物理体，限制了游戏范围，确保精灵不会跑出屏幕。
 self.physicsBody = SKPhysicsBody(edgeLoopFrom: self.frame)
 //设置重力
 self.physicsWorld.gravity = CGVector(dx: 0.0, dy: -3.0)
 //物理世界的碰撞检测代理为场景自己
 self.physicsWorld.contactDelegate = self;
```
然后，让GameScene这个类遵守下面的代理协议
`SKPhysicsContactDelegate`
协议方法待会再讲，先声明我们要用到的几个物理体
```
/// 设置物理体的标示符  
let birdCategory: UInt32 = 1 << 0  //1
let worldCategory: UInt32 = 1 << 1  //2
let pipeCategory: UInt32 = 1 << 2  //4
```
在添加地面的代码后面加上:
```
// 配置陆地物理体
    let ground = SKNode()
    ground.position = CGPoint(x: 0, y: groundTexture.size().height)
    ground.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: self.frame.size.width, height: groundTexture.size().height * 2.0))
    ground.physicsBody?.isDynamic = false
//当前物理体
    ground.physicsBody?.categoryBitMask = worldCategory
    self.addChild(ground)
```
在添加小鸟的代码后面加上:
```
    // 配置小鸟物理体
    bird.physicsBody = SKPhysicsBody(circleOfRadius: bird.size.height / 2.0)
    bird.physicsBody?.allowsRotation = false
    bird.physicsBody?.categoryBitMask = birdCategory
    bird.physicsBody?.contactTestBitMask = worldCategory 
```

找到creatSpawnPipes方法中，添加上下水管代码之前加入下面的代码内容：
```
pipeDown.physicsBody = SKPhysicsBody(rectangleOf: pipeDown.size)
pipeDown.physicsBody?.isDynamic = false
pipeDown.physicsBody?.categoryBitMask = pipeCategory
pipeDown.physicsBody?.contactTestBitMask = birdCategory
```
```
 pipeUp.physicsBody = SKPhysicsBody(rectangleOf: pipeUp.size)
 pipeUp.physicsBody?.isDynamic = false
 pipeUp.physicsBody?.categoryBitMask = pipeCategory
 pipeUp.physicsBody?.contactTestBitMask = birdCategory
```
有了物理体之后，我们会发现，运行游戏后，小鸟会直接受重力影响，掉下来~
接下来，我们就不得不考虑，游戏的运行状态了
#### 6.游戏状态

1.初始化状态：小鸟在移动的背景中飞翔而未掉落，无水管出现。
2.运行中状态：小鸟会受重力作用往下坠落，水管开始出现，点击一次屏幕，小鸟就有会受一次上升的力。
3.已结束状态：小鸟碰到水管或地面，游戏结束，小鸟停止飞的动作，场景里的水管和地面都停住不动。

在GameScene类中，定义一个枚举来表示不同的状态，同时增加一个游戏状态的变量
```
/// 游戏状态
enum GameStatus {
    case idle /// 初始化
    case running /// 游戏运行中
    case over /// 游戏结束
}
```
```
/// 游戏状态为初始状态
var gameStatus:GameStatus = .idle
```
实现不同状态对应的不同方法
```
func idleStatus() {
        gameStatus = .idle
}
```
```
func runningStatus() {
        gameStatus = .running
}
```
```
func overStatus() {
        gameStatus = .over
}
```
实现点击屏幕对应不同的状态时，所作出的应对
```
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    switch gameStatus {
    case .idle:
        runningStatus()
        break
    case .running:
        for _ in touches {
            bird.physicsBody?.velocity = CGVector(dx: 0, dy: 0)
            // 施加一个均匀作用于物理体的推力
            bird.physicsBody?.applyImpulse(CGVector(dx: 0, dy: 10))
        }
        break
    case .over:
        idleStatus()
        break
    }
}
```
**小鸟**：这时候，我们可以把小鸟的位置、受外力影响只为否属性和开始飞的方法移到初始化方法里面
```
bird.position = CGPoint(x: self.frame.size.width * 0.35, y: self.frame.size.height * 0.6)
 // isDynamic的作用是设置这个物理体当前是否会受到物理环境的影响，默认是true
bird.physicsBody?.isDynamic = false
self.birdStartFly()
```
接着，在运行中状态，将小鸟受外力影响属性置为是
```
bird.physicsBody?.isDynamic = true
bird.physicsBody?.collisionBitMask = worldCategory | pipeCategory
```
然后，在结束状态中，让小鸟停止飞
```
birdStopFly()
```
**水管**：初始化方法，移除屏幕中上次可能存在的水管
```
removeAllPipesNode()
```
在运行中状态，开始随机生产水管
```
startCreateRandomPipes()
```
已结束状态，停止随机创建水管
```
stopCreateRandomPipes()
```
现在运行代码，来看看效果怎么样~
![MJBird.gif](http://upload-images.jianshu.io/upload_images/1518951-b0e5fb6afbf81da4.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们会看到，只有初始化状态和运行中状态，而碰撞之后，并没有结束，那么下面我们就来实现一下碰撞协议~
```
/// SKPhysicsContact对象是包含着碰撞的两个物理体的,分别是bodyA和bodyB
func didBegin(_ contact: SKPhysicsContact) {
    if gameStatus != .running {
        return
    }
        bird.physicsBody?.collisionBitMask = worldCategory
        overStatus()
}
```
这时运行代码发现，碰撞水管之后，背景及水管还在运动，我们应该让其停止，我们首先创建变量
```
/// 储存陆地、天空和水管
var moving:SKNode!
```
然后再在didMove()方法里初始化变量，然后把之前的储存水管的变量pipes也加到moving里面
```
moving = SKNode()
self.addChild(moving)
pipes = SKNode()
moving.addChild(pipes)
```
把 `self.addChild(sprite)`及`self.addChild(sprite)`替换为`moving.addChild(sprite)`和`moving.addChild(sprite)`

在碰撞协议方法里面，加上让其速度为零的属性
```
moving.speed = 0
```
然后在初始化方法里面，让其速度恢复
```
moving.speed = 1
```
运行效果如下：
![MJBird2.gif](http://upload-images.jianshu.io/upload_images/1518951-fdadbe8ded89b19d.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7.分数显示

首先声明一个分数变量和懒加载一个分时显示Label
```
 /// 分数
 var score: NSInteger = 0
 ///分数Label
 lazy var scoreLabelNode:SKLabelNode = {
     let label = SKLabelNode(fontNamed: "MarkerFelt-Wide")
     label.zPosition = 100
     label.text = "0"
     return label
 }()
```
考虑到加分问题，肯定是在小鸟通过管道的时候，才计分+1，所以我们在管道右端加一个物理体，来实现与小鸟碰撞之后的加分。
在添加管道代码下面添加如下：
```
    let contactNode = SKNode()
    contactNode.position = CGPoint(x: pipeDown.size.width, y: self.frame.midY)
    contactNode.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: pipeUp.size.width, height: self.frame.size.height))
    contactNode.physicsBody?.isDynamic = false
    contactNode.physicsBody?.categoryBitMask = scoreCategory
    contactNode.physicsBody?.contactTestBitMask = birdCategory
    pipePair.addChild(contactNode)

```

然后在碰撞方法中，判断是否相撞，通过管道,分数+1
```
if (contact.bodyA.categoryBitMask & scoreCategory) == scoreCategory || (contact.bodyB.categoryBitMask & scoreCategory) == scoreCategory {
        score += 1
        print(score)
        scoreLabelNode.text = String(score)
        scoreLabelNode.run(SKAction.sequence([SKAction.scale(to: 1.5, duration: TimeInterval(0.1)),SKAction.scale(to: 1.0, duration: TimeInterval(0.1))]))
    }else{
        moving.speed = 0
        bird.physicsBody?.collisionBitMask = worldCategory
        overStatus()
    }
```
在运行中状态，显示分数
```
// 重设分数
 score = 0
 scoreLabelNode.text = String(score)
 self.addChild(scoreLabelNode)
 scoreLabelNode.position = CGPoint(x: self.frame.midX, y: 3 * self.frame.size.height / 4)
```
初始化状态，移除分数labelNode
```
 // 移除分数提示
 scoreLabelNode.removeFromParent()

```
这是运行代码，这款小游戏就基本完成了
![birdFly3.gif](http://upload-images.jianshu.io/upload_images/1518951-ba202e5c9eb79362.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 8. 优化

上面提到，我们留下两个方法，已经用到了didMove() ，那么update()是做什么的呐：update()方法为SKScene自带的系统方法，在画面每一帧刷新的时候就会调用一次。

在这里，我们可以处理让小鸟掉落的时候脸先着地（233~）
```
override func update(_ currentTime: TimeInterval) {
    //调整头先着地
    let value = bird.physicsBody!.velocity.dy * (bird.physicsBody!.velocity.dy < 0 ? 0.003 : 0.001)
    bird.zRotation = min(max(-1, value),0.5)
}
```
碰撞之后，添加一个背景闪光效果
```
func bgFlash() {
    let bgFlash = SKAction.run({
        self.backgroundColor = SKColor(red: 1, green: 0, blue: 0, alpha: 1.0)}
    )
    let bgNormal = SKAction.run({
        self.backgroundColor = self.skyColor;
    })
    let bgFlashAndNormal = SKAction.sequence([bgFlash,SKAction.wait(forDuration: (0.05)),bgNormal,SKAction.wait(forDuration: (0.05))])
    self.run(SKAction.sequence([SKAction.repeat(bgFlashAndNormal, count: 4)]), withKey: "falsh")
    self.removeAction(forKey: "flash")
}
```
添加结束提示语：同样懒加载一个SKLabelNode
```
lazy var gameOverLabel:SKLabelNode = {
     let label = SKLabelNode(fontNamed: "Chalkduster")
     label.text = "Game Over"
     return label
 }()
```
在结束状态中，添加上这个提示语，同时为了让游戏有个缓冲过程，我们让屏幕2秒内不能点击
```
 isUserInteractionEnabled = false;
 addChild(gameOverLabel)
 gameOverLabel.position = CGPoint(x: self.size.width * 0.5, y: self.size.height)

 //让gameOverLabel通过一个动画action移动到屏幕中间
 let delay = SKAction.wait(forDuration: TimeInterval(1))
 let move = SKAction.move(by: CGVector(dx: 0, dy: -self.size.height * 0.5), duration: 1)
 gameOverLabel.run(SKAction.sequence([delay,move]), completion:{
       //动画结束 允许用户点击屏幕
       self.isUserInteractionEnabled = true
})
```
在初始化状态，移除提示语
```
 // 移除 游戏结束提示
 gameOverLabel.removeFromParent()
```
OK,现在一款”飞翔的小鸟”简易款就成型了,可以愉快的玩耍了~

[代码连接](https://github.com/JingJing-Lin/MJFlappyBirdSwift)

升级款：
1.全新的UI视图;
2.动听的音乐组合；
3.计分板和引导提示图的展示。

####资料参考：
[SpriteKit初探](http://www.jianshu.com/p/d18930488df2)
[SKScene类](http://www.jianshu.com/p/c50f4db0b1b9)
[AnchorPoint(锚点)](http://www.jianshu.com/p/aff13f3355f4)
[SKSpriteNode](http://www.jianshu.com/p/72cdb54979a4)
[SKLableNode](http://www.jianshu.com/p/ab75f6b992e5)
[SKTextureAtlas（纹理集）](http://www.jianshu.com/p/fa4b06d7ee35)
[SKAction动作](http://www.jianshu.com/p/2efc153200c9)
[SKSpriteNode拖动](http://www.jianshu.com/p/78bbcc15eac3)
[SKAction常用属性](http://www.jianshu.com/p/5d875bcc19fa)
[SKPhysicsBody物理引擎](http://www.jianshu.com/p/0cc657becbe9)
[节点碰撞](http://www.jianshu.com/p/493686eaf0d7)
[SKPhysicsBody的移动和连接](http://www.jianshu.com/p/ee0d9b1077cc)
[iOS SpriteKit 小游戏开发实例](http://www.jianshu.com/p/304e84a12b91)
[FlappySwift](https://github.com/fullstackio/FlappySwift)