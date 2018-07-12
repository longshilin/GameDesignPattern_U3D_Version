具体Readme请在Asset内分目录查看:

[1. 命令模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/001CommandPattern)

[2. 享元模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/002FlyweightPattern)

[3. 观察者模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/003ObserverPattern)

[4. 原型模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/004PrototypePattern)

[5. 单例模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/005SingletonPattern)

[6. 状态模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/006StatePattern)

[7. 序列模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/007SequencingPatterns)

[8. 行为模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/008BehavioralPatterns)

[9. 解耦模式](https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/009DecouplingPatterns)

---

[TOC]

---

# 命令模式

## 使用说明

- 运行后，WASD控制方块运动，场景中Manager勾选掉IsRun，可以实现时光回流的效果。

## 类说明

### Command

- 抽象基类，包含了时间戳和运行、回退的虚方法

### CommandMove

- Command的子类，可以调用指定Avatar的Move函数

### Avatar

- 执行行为的目标物体，拥有Move函数

### CommandManager

- 当IsRun为true时，由WASD按键生成命令对象，执行Execute(Avatar)
- 当IsRun为false时，调用RunCallBack()，按时间将栈内命令提取出并执行Undo(Avatar)

---

## [笔记](https://gpp.tkchu.me/command.html)

### 是什么（个人理解）

将命令封装，与目标行为解耦，使命令由流程概念变为对象数据

### 为什么

既然命令变成了数据，就是可以被传递、存储、重复利用的：

- 通过命令数据队列或栈可以轻易实现撤销、重做、时光倒流
- 命令数据还可以形成日志，用于复现用户行为，便于重复测试同样序列命令对各种目标的影响
- 这些命令数据可以发送给不同的目标，比如同样的“出发，5分钟后，停止”，发送给飞机就可以变成“起飞，5分钟后，降落”，发送给轮船就成了“离港，5分钟后，抛锚”

### 怎么做（U3D示例）

#### 类图如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/001CommandPattern/UML/001CommandPattern.png)

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/001CommandPattern

### 缺陷

可能会导致大量的实例化，从而浪费内存

### 拓展

可用享元模式代替大量的实例化

---

# 享元模式

## 项目说明

为说明享元模式的意义，此示例展示了开启和关闭Unity内GPU Instancing机制的效率差别

另外要特别指出的是，至少Unity内GPU  Instancing机制已经做得很好了，为了演示差别，我强制造就了一种无法合并(Batch)的状态

## 使用说明

运行后，每帧重新生成1000个立方体，通过勾选/取消Manager中的 IsFlyweight 来控制是否使用享元模式

## 对比

### Stats对比

未开启享元模式：

- FPS：17.6
- Batches（Draw Call 相关）：1003

![1529496951892](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/002FlyweightPattern/ReadmePic/1529496951892.png)

开启享元模式：

- FPS：71.4
- Batches（Draw Call 相关）：5

![1529496533688](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/002FlyweightPattern/ReadmePic/1529496533688.png)

### Profiler对比

![1529497451350](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/002FlyweightPattern/ReadmePic/1529497451350.png)

上图中左半部分为开启享元模式的统计，右图为未开启的统计，可以看出未开启时，CPU的压力大幅上升，而且内存使用也会随实例化而持续增加——因为垃圾回收速度远低于垃圾产生速度

所以享元模式使计算的时间和占用的空间都获得了优化

---

## [笔记](https://gpp.tkchu.me/flyweight.html)

### 是什么（个人理解）

不同的实例共享相同的特性（共性），同时保留自己的特性部分

### 为什么

- 传递的信息享元化，可以节约计算时间
- 存储的信息享元化，可以节约占用的空间
- 所以享元模式可以减少时间与空间上的代价

### 怎么做

#### 类图如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/002FlyweightPattern/UML/002FlyweightPattern.png)

左半部分为享元模式下，只有一个CubeBase，通过ObjInstancing(int num)讲共享的网格、材质及一个Transform信息表传递给GPU，只有一个Draw Call，所以效率极高

右半部分为关闭享元模式后的做法，每生成一个Cube都会重新实例化一个立方体，并向GPU发送一次网格、材质和位置信息，所以1000个立方体就需要1000个Draw Call，效率极低

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/002FlyweightPattern

### 拓展

可与对象池联动，进一步减少内存的开销

---

# 观察者模式

## 项目说明

### Emitter

通过EmitBall()方法释放气球（白色球体），并通知Radio有释放气球的事件（调用 OnEmitEvent (Transform target) ）

### Shooter

告诉Radio要收听（观察）OnEmitEvent事件，事件发生时向气球射击，发出红色子弹

---

## [笔记](https://gpp.tkchu.me/observer.html)

### 是什么（个人理解）

事件与其他对象行为的解耦——例如一个代码描述了日本核电站爆炸的事件，世界人民买盐这种行为显然不应该由核电站爆炸直接调用，而是通过卫星电视告诉广大群众，群众想买盐还是想买仙人掌就由他们自己决定了~

### 为什么

- 解耦，物价局改了粮价不需要挨家挨户通知公民，只需要让电视台播个新闻就好
  - 如果要挨家挨户通知，物价局必须有每个公民的地址，这显然不合理，也会浪费很多资源
  - 扩展困难——如果公民改了地址或者有新公民出生了，那还需要告诉物价局，这也很荒唐

### 怎么做

#### 类图如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/003ObserverPattern/UML/003ObserverPattern.png)

射手（Shooter，观察者，这里是听众）告诉广播电台（Radio）自己要听发射气球的广播

吹气球的人（Emitter） 向上发出气球，并告诉广播电台自己发射了气球

广播电台广播发射了气球的消息，所有射手向气球射击

这个例子中吹气球的人不会关心谁是射手，射手也不用在意谁是吹气球的人

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/003ObserverPattern

---

# 原型模式

## 项目说明

### Dragon

配置在Prefab里，当克隆体被生成后，通过读取Dragons.txt内配置生成不同类型的火龙

### Spawner

生成器，每隔一段时间生成一个指定的Prefab

### Dragons.txt

用以记录不同类型的火龙配置——如胖火龙、高火龙、巨火龙、小火龙和他们各自的尺寸

> 这里为了快速实现使用txt记录配置表（我在偷懒），但实际项目里，往往使用SQL、Json、csv等方式进行配置

---

## [笔记](https://gpp.tkchu.me/prototype.html)

### 是什么（个人理解）

将一个或多个对象当做原型，通过统一的生成器克隆出很多类似原型的对象，同时可以通过配置表更改克隆体属性，制造出很多具有自身个性的对象。

### 为什么

- 复用生成器，而非针对每一个不同的对象做一个生成器
- 与享元模式结合，通过配置表来实现对象的个性，将不同配置与代码解耦

### 怎么做

#### 类图如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/004PrototypePattern/UML/004PrototypePattern.png)

- Unity中Prefab本质就是此模式里的原型，而Spawner要做的只是调用Instantiate方法
- 新的Prefab被生成以后，通过读取Dragons.txt里配置的信息来设置克隆体的名称和尺寸

> 注：这里为了快速实现使用txt记录配置表（我在偷懒），但实际项目里，往往使用SQL、Json、csv等方式进行配置

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/004PrototypePattern

---

# 单例模式

## 项目说明

### TimeLogger

用于生成日志，使用了单例模式

### Speaker

在场景中有多个实例，讲话时会调用TimeLogger单例

### Log.txt

TimeLogger生成的日志

---

## [笔记](https://gpp.tkchu.me/singleton.html)

### 是什么（个人理解）

使用单例意味着这个对象只有一个实例，这个实例是此对象自行构造的，并且可向全局提供

### 为什么

- 减少代码复用，让专门的类处理专门的事情——例如让TimeLog类来记录日志，而不是把StreamWriter的代码写到每一个类里
- 快速访问，任何其他类都可以通过ClassName.Instance来访问单例，使用它的公开变量和方法

### 缺陷

- 因为实现简单，而且使用方便，所以有被滥用的趋势
- 滥用单例会促进耦合的发生，因为单例是全局可访问的，如果不该访问者访问了单例，就会造成过耦合——例如如果播放器允许单例，那石头碰撞地面后就可以直接调用播放器来播放声音，这在程序世界并不合理，而且会破坏架构
- 如果很多很多类和对象调用了某个单例并做了一系列修改，那想理解具体发生了什么就困难了
- 对多线程不太友好——每个线程都可以访问这个单例，会产生初始化、死锁等一系列问题

### 怎么做

U3D中利用MonoBehaviour初始化单例非常简单，只要在Awake中加入Instance = this，不过要注意的是，别的类不能在Awake里使用这个单例

单例在普通C#中还有其他做法，甚至有些泛型、线程安全的扩展，也都不复杂，可以自行查询

#### 类图如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/005SingletonPattern/UML/005SingletonPattern.png)

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/005SingletonPattern

---

# 状态模式

## 项目说明

这个场景内有两个状态机：

- 一个是Switch Case实现的冷暖光，运行后点击“Switch”按键即可观察
- 一个是State Class实现的交通灯，运行后会自动变换

---

## [笔记](https://gpp.tkchu.me/state.html)

### 是什么（个人理解）

现在状态和条件决定对象的新状态，状态决定行为（Unity内AnimationController就是状态机）

### 为什么

- 使流程清晰化、结构化
- 简化判断逻辑，比如嘴的状态是洗牙，那就不应该做出咀嚼的行为；必须是在憋气，那就不应该做出呼吸的行为

### 注解

- 状态机（自动机）是我最喜欢的一种设计模式，因为这样设计的程序逻辑清晰，稳定性也很强
- 作者对switch case下的状态机理解并不深刻，一般情况下，状态机需要两个switch case，一个用于处理状态变化，另一个用来处理状态行为
- 相比状态类，个人更喜欢switch case的方法，虽然状态类有其有点，但缺点也非常明显——当状态量较大时，代码量激增，可读性也很差，状态变化和状态行为都需要大量的信息传递，十分不便

### 怎么做

这次我实现了两个版本：

1. SwitchCase版本，用按键控制一盏冷暖灯，关灯状态下，按一次打开暖光，再按切换为白光，再按变为暖白光，再按关闭
2. 状态类版本，交通灯 停止、通行、闪烁、等待的切换

另外自动机用类图描述不是好方法，应该用自动机专门的图来说明才对

#### SwitchCase版本类图及自动机如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/006StatePattern/UML/006StatePattern-SwitchCase.png)

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/006StatePattern/UML/LightSwitchCase.png)

#### StateClass版本类图及自动机如下：

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/006StatePattern/UML/006StatePattern-StateClass.png)

![](https://github.com/TYJia/GameDesignPattern_U3D_Version/blob/master/Assets/006StatePattern/UML/LightStateClass.png)

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/006StatePattern

---

# 序列模式

## [笔记](https://gpp.tkchu.me/sequencing-patterns.html)

### 是什么、为什么（个人理解）

包含了

- 双缓冲模式
  - 当一个缓冲准备好后才会被使用——就像一个集装箱装满才会发货一样；当一个缓冲被使用时另一个处于准备状态，就形成了双缓冲
  - 在渲染中广泛使用，一帧准备好后才会被渲染到屏幕上——所以准备时间太长就会导致帧率下降
- 游戏循环
  - 可参考[脚本生命周期](https://docs.unity3d.com/Manual/ExecutionOrder.html)
- 更新方法
  - 同上，实际上是Unity通过反射在生命周期不同时刻调用MonoBehaviour中的相关方法

这三者一定程度上是相辅相成的，在Unity中都已在底层实现，双缓冲可以通过FrameDebugger体会，而游戏循环、更新方法则与脚本生命周期和MonoBehaviour相关

---

# 行为模式

## [笔记](https://gpp.tkchu.me/behavioral-patterns.html)

### 是什么、为什么（个人理解）

包含了

- 字节码
  - 享元模式、原型模式中，不同的属性被存储在数据库中，而字节码是将行为存在数据库中
  - 可用于实现可视化脚本编辑工具
- 子类沙箱
  - 子类使用基类方法，或在基类方法上扩展
- 类型对象
  - 其实是享元模式、原型模式的一种应用，以不同数据（而不是不同类）区分对象类型

---

# 解耦模式

## 项目说明

Game模式下，点击鼠标可以设置多个目标点，Player（方块）会依次移向目标点

> 解耦模式中包含了三种不同的模式，这里只实现了事件序列作为参考，其他两种模式在Unity中均有较好实现，可参考笔记内容

### EventQueue

监听鼠标点击事件，在鼠标点击时向队列注入新的目标点；Update中，若当前目标点为空，则取出队列第一位作为当前目标点，若到达当前目标点，则删除

## [笔记](https://gpp.tkchu.me/decoupling-patterns.html)

### 是什么、为什么（个人理解）

包含了

- 组件模式
  - 本质上是功能的模块化，延伸了面向对象的解耦思想
  - U3D的编程思想就是面向组件的，MonoBehaviour的子类都可作为组件挂在GameObject上
- 事件序列
  - 就像银行办事需要排号一样——每个顾客要处理的事都是一个事件，编号后就形成了天然的事件序列，银行会按一定规则来依次处理队列中的事件
  - 一般在底层实现，但宏观上依然存在，例如RTS游戏中通过Shift对一些单位下达前往不同位置的命令
  - Unity中协程可以用来做消息队列，防止同帧产生大量的计算
- 服务定位器
  - 类似单例模式，在运行时寻找组件（而不是运行前赋值）
  - Unity中GetComponent，FindObjectOfType，Find等方法都可帮助实现相关服务的查找，但此类反射方法要避免在运行时高频循环调用
  - 拓展——还可以建立一个运行前赋值的服务注册中心（当然也可运行中赋值），其他需要服务的对象在运行时去注册中心查找相关服务，这样做一方面可以避免全局反射的恶果，一方面可以保留服务定位器带来的解耦优势——单例模式也可使用这样的方法来替换（对象注册中心）

### 怎么做（事件队列）

点击鼠标时在Queue中添加一个红点，当目标点为空时从Queue中取出第一位位作为目标点，让Player移向目标点，到达目标点时删除目标点

#### 具体实现：

https://github.com/TYJia/GameDesignPattern_U3D_Version/tree/master/Assets/009DecouplingPatterns

---

