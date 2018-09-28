###[深入简出--iOS编程思想](http://www.cocoachina.com/ios/20180928/25053.html)
####1.面向过程编程思想
处理事情以过程为核心，一步一步的实现
####2.面向对象编程思想
万物皆对象
####3.连式编程思想
是将多个操作（多行代码）通过点号（.）链接在一起成为一句代码，是代码可读性好。
####4.响应式编程思想
不需要考虑调用顺序，只需要知道考虑结果，类似于蝴蝶效应，产生一个事件，会影响很多东西，这些东西像流一样的传播出去，然后影响结果。

KVO的实现原理：
当观察某对象A时，KVO机制动态创建一个对象A当前对象类的子类，并为这个新的子类充血了被观察属性keyPath的setter方法，setter方法随后负责通知观察对象属性的改变状况
####5.函数式编程思想
是把操作尽量写成一系列嵌套的函数或者方法调用。



###[打造开源第一iOS图片浏览器](https://www.jianshu.com/p/bffdb9f0036c)
这篇文章主要讲述YBImageBrowser的一些功能技术细节，代码架构思路，设计模式选择等。
[源码](https://github.com/indulgeIn/YBImageBrowser)

###[面向对象设计的六大设计原则](https://knightsj.github.io/2018/09/09/面向对象设计的六大设计原则（附%20Demo%20及%20UML%20类图）/#more)

开闭原则：一个软件实体如类、模块和函数应该对扩展开放，对修改关闭

为了更好的实践开闭原则，在设计之初就要想清楚在该场景里那些数据（或行为）是一定不变（或很难再改变）的，那些是很容易变动的。将后者抽象成接口或抽象方法，以便在将来通过创造具体的实现应对不同的需求

单一职责原则：一个类只允许有一个职责，即只有一个导致该类变更的原因。

在实际开发中，我们需要将不同的职责分开

依赖倒置原则：即，依赖抽象，而不是依赖实现；抽象不应该依赖细节，细节应该依赖抽象，高层模块不能依赖底层模块，二者都应该依赖抽象

今后在处理高低层模块（类）交互的情景时，尽量将二者的依赖通过抽象的方式解除掉，实现方式可以是通过接口也可以是抽象类的方式

接口分离原则：多个特定的客户端接口要好于一个通用性的总接口

在设计接口时，尤其是在向现有的接口添加方法时，我们需要仔细斟酌这些方法是否是处理同一类任务的，如果是则可以放在一起，如果不是则需要做拆分。

迪米特法则：一个对象应该对尽可能少的对象有接触，也就是只接触那些真正需要接触的对象。

今后在做对象与对象之间交互的设计时，应该极力避免引出中间对象的情况（需要倒入其他对象类）：需要什么对象直接返回即可，降低类之间的耦合度

里氏替代原则：所有引用基类的地方必须能透明地使用其子类的对象，也就是说子类对象可以替换其父类对象，而程序执行效果不变。

里氏替换原则是对继承关系的一种检验：检验是否真正符合继承关系，以避免继承的滥用。因此，在使用继承之前，需要反复思考和确认该继承关系是否正确，或者当前的继承体系是否还可以支持后续的需求变更，如果无法支持，则需要及时重构，采用更好的方式来设计程序。


###[iOS APP后台Crash调查](http://mrpeak.cn/blog/ios-background-running/)

已知的iOS App后台运行场景有：
background task 通常情况下，app一旦进入后台，只有数秒的时间继续执行代码，之后就会被系统suspend。除非app显示的调用beginBackgroundTaskWithExpirationHandler API,能延迟App在后台运行代码的时间，iOS7 之前10分钟，iOS7之后缩减至3分钟，一旦时间期限到，系统依旧会suspend进程

background mode 开启这种模式的app可以一直在后台运行

background fetch在系统指定的时间段唤醒app并执行少量逻辑的机制，限制较多

silent push 这是通过apn里设置特定字段来唤醒app的机制，限制较多

pushkit 是用来替代VOIP后台运行模式的新机制，在收到语音或者视频电话时，可以通过pushkit的通知来唤醒app，从而避免app在后台一直运行。

已知的至少有这几类crash是无法被app捕获的：

*前台主线程卡死，app被watchdog强杀

*app在前台或者后台使用过多的内存，被系统强杀，分别为foom和boom

*app在后台被suspend之后，由于违反apple的某个policy，而被系统强杀。


###[保护App不闪退](http://www.cocoachina.com/ios/20180919/24905.html)
由于OC是Message机制，而且对象在转换的时候，回有拿到的对象和预期不一致，所以会有方法找不到的情况，在找不到方法时，查找方法将会静如方法Forward流程，系统给了三次补救的机会。

![流程](images/forward.png)

resolveInstanceMethod:(SEL)sel 这是实例化方法没有找到方法，最先执行的函数，首先回流转到这里，返回值是BOOL，没有找到就是NO，找到就返回YES，如果要解决就需要在当前的事例中加入不存在的Selector，并绑定IMP。

如果resolveInstanceMethod没有处理，将进行到forwardingTargetForSelector这步来，这时候你可以返回nil, 你也可以用一个Stub对象来接住，把消息流程转到Stub，然后在你的Stub里添加不存在的Selector，这样就不会crash。

当methodSignatureForSelector返回nil时，会Crash
如果methodSignatureForSelector返回一个定义好的NSMethodSignature，但是没有实现forwardInvocation，也会闪退，如果实现了forwardInvocation，会先返回到resolveInstanceMethod然后再才会到forwardInvocation
当流转到forwardInvocation,通过以下方法:
[anInvocation invokeWithTarget:xxxtarget1];
[anInvocation invokeWithTarget:xxxtarget2];



###[Hacking Hit Tests](http://khanlou.com/2018/09/hacking-hit-tests/)

-pointInside:withEvent: 给定的点是否在当前视图中

-hitTest:withEvent:返回响应的视图

-convert:from: 转换坐标系

三个方法搭配使用，可以控制视图响应




###[Early returning functions in Swift](https://www.swiftbysundell.com/posts/early-returning-functions-in-swift)

用guard控制函数提前返回。



###[TCP没那么难吧？](https://mp.weixin.qq.com/s/zRelB6uSz07YaCoJoggZZA)

TCP是双向的，可靠的通讯，所以连接就必须确认双方到对方的通讯都是可靠的
SYN表示synchronize，在建立连接时使用；ACK表示acknowledge，表示确认收到了消息，FIN表示finish，在断开连接时使用

![syn](images/syn.png)

![fin](images/fin.png)

###[iOS启动时间优化](http://www.zoomfeng.com/blog/launch-time.html)

####pre-main阶段
影响因素：
动态库加载越多，启动越慢。
ObjC类越多，函数越多，启动越慢。
可执行文件越大启动越慢。
C的constructor函数越多，启动越慢。
C++静态对象越多，启动越慢。
ObjC的+load越多，启动越慢。
优化：
①减少依赖不必要的库，不管是动态库还是静态库；如果可以的话，把动态库改造成静态库；
如果必须依赖动态库，则把多个非系统的动态库合并成一个动态库；
②检查下 framework应当设为optional和required，
如果该framework在当前App支持的所有iOS系统版本都存在，那么就设为required，否则就设为optional，
因为optional会有些额外的检查； 
③合并或者删减一些OC类和函数；
关于清理项目中没用到的类，使用工具AppCode代码检查功能，查到当前项目中没有用到的类（也可以用根据linkmap文件来分析，但是准确度不算很高）；
有一个叫做[FUI](https://github.com/dblock/fui)的开源项目能很好的分析出不再使用的类，准确率非常高，唯一的问题是它处理不了动态库和静态库里提供的类，也处理不了C++的类模板。
④删减一些无用的静态变量，
⑤删减没有被调用到或者已经废弃的方法，
方法见http://stackoverflow.com/questions/35233564/how-to-find-unused-code-in-xcode-7
和https://developer.Apple.com/library/ios/documentation/ToolsLanguages/Conceptual/Xcode_Overview/CheckingCodeCoverage.html。
⑥将不必须在+load方法中做的事情延迟到+initialize中，尽量不要用C++虚函数(创建虚函数表有开销)
⑦类和方法名不要太长：iOS每个类和方法名都在__cstring段里都存了相应的字符串值，所以类和方法名的长短也是对可执行文件大小是有影响的；
因还是object-c的动态特性，因为需要通过类/方法名反射找到这个类/方法进行调用，object-c对象模型会把类/方法名字符串都保存下来；
⑧用dispatch_once()代替所有的 attribute((constructor)) 函数、C++静态对象初始化、ObjC的+load函数；
⑨在设计师可接受的范围内压缩图片的大小，会有意外收获。
压缩图片为什么能加快启动速度呢？因为启动的时候大大小小的图片加载个十来二十个是很正常的，
图片小了，IO操作量就小了，启动当然就会快了，比较靠谱的压缩算法是TinyPNG。


####main阶段
①减少启动初始化的流程，能懒加载的就懒加载，能放后台初始化的就放后台，
能够延时初始化的就延时，不要卡主线程的启动时间，已经下线的业务直接删掉； 
②优化代码逻辑，去除一些非必要的逻辑和代码，减少每个流程所消耗的时间； 
③启动阶段使用多线程来进行初始化，把CPU的性能尽量发挥出来； 
④使用纯代码而不是xib或者storyboard来进行UI框架的搭建，尤其是主UI框架比如TabBarController这种，
尽量避免使用xib和storyboard，因为xib和storyboard也还是要解析成代码来渲染页面，多了一些步骤；


###[动态界面：DSL&布局引擎](https://awhisper.github.io/2017/05/01/DSLandLayoutEngine/)
DSL是Domain Specific Language的缩写，意思就是特定领域下的语言。

###[谈谈DSL以及DSL的应用（以CocoaPods为例）](https://draveness.me/dsl)
GPL是General Purpose Language，即通用编程语言


###[从数据流角度管窥Moya的实现（一）：构建请求](https://mp.weixin.qq.com/s?__biz=MzA5NzMwODI0MA==&mid=2647759707&idx=1&sn=216ce50d0df1a87ae522fb608c679363&chksm=8887e064bff06972a67a055e89fd5918d1029368ef52de37e7565608d57e93368f3cfb7945d8&scene=21#wechat_redirect)
target -> endpoint -> request -> 回调

###[从数据流角度管窥Moya的实现（二）：处理相应](https://mp.weixin.qq.com/s?__biz=MzA5NzMwODI0MA==&mid=2647759751&idx=1&sn=685471017b5655cce360e761534ff78a&chksm=8887e0b8bff069ae01ee885962fe03018c4205fc94e3cb4bfcfe1dbbd28c6cf06917e0e4b2d1&scene=21#wechat_redirect)
response -> result -> 回调


###[移动应用架构演变及泛前段趋势下移动团队破局](https://mp.weixin.qq.com/s/iHehLneIYibWq6IRjuefxg)
第二类石那些异构开发方案， 在大趋势下未来3-5年入锅还继续做移动端技术，rn/weex/flutter三个里面从底层到应用至少要精通一个，PWA可关注。第三类是垂直领域的技术，比如像图像处理，AR/VR，音视频编解码流媒体，浏览器内核，阅读排版引擎，复杂动效，更高级的UI绘制。第四类是周边运维，包括对项目的定制化构建，持续集成，性能监控，代码规范审查自动化，自动化测试。


如何规划未来要做什么：
1。跟随行业热点趋势，就是那些短期内聚集开发者关注的技术，简单的筛选方式是通过GitHub trending挖掘
2.从公司的需求，技术痛点

###[WWDC 2018: iOS内存深入研究](https://juejin.im/post/5b23dafee51d4558e03cbf4f)
+为什么要减少内存使用
可以有更好的用户体验，更快的启动速度，不会因为内存过大而导致crash，可以让APP存活更久

+内存占用
内存是由系统管理的，一般以页为单位划分，在iOS上，每一页包含16kb的空间。一段数据可能回占用多页内存，所占用也总数乘以每页空间得到的就是这段数据使用的总内存

+分析内存占用工具
+图像
+在后台时，对内存优化


###[iOS如何优雅的处理“回调地狱callbackhell”(-)——使用PromiseKit](https://www.jianshu.com/p/f060cfd52f17)
一.PromiseKit简介
PromiseKit是iOS/OS X中一个用来处理异步编程框架。
在PromiseKit中，最重要的一个概念就是Promise的概念的，Promise是异步操作后的future的一个值。
promise也是一个包装着异步操作的一个对象。使用PromiseKit，能够编写出整洁，有序，逻辑简单的的代码，将Promise作为参数，模块化的从一个异步任务到下一个异步任务中去。

PromiseKit就是用来干净整洁的代码，来解决异步操作，和奇怪的错误处理回调的。他将异步操作变成了链式的调用，简单的错误处理方式

PromiseKit里面目前有2个类，一个是Promise<T>(Swift),一个是AnyPromise(Objective-C)，2者的区别就在凉后再难过语言的特征上，Promise<T>是定义精确严格的，AnyPromise是定义宽松，灵活，动态的

二。Promise的使用

三。PromiseKit的源码解析
在异步操作我们可以通过不断的返回promise，传递给后面的then来形成链式调用，所以重点就在then的实现

一个promise可能有三种状态：等待（pending），已完成（fulfilled），已拒绝（rejected）
一个promise的状态只可能从“等待”转到“完成”或者“拒绝”态，不能逆向转换，同时“完成”和“拒绝”不能相互转换
promise必须实现then方法（可以说，then就是promise的核心），而且then必须返回一个promise，同时一个promise的then可以调用多次，并且回调的执行顺序跟他们被定义时的顺序一致
then方法接受两个参数，第一个参数是成功时的回调，在promise由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在promise由“等待”态转换为“”拒绝
态时调用。同时，then可以接受另一个promise传入，也接受一个“类then”的对象或方法，即thenable对象


###[使用asdk性能调优-提升iOS界面的渲染性能](https://draveness.me/asdk-rendering)
iOS中的性能问题大多是阻塞主线程导致用户的交互反馈出现可以感知的延迟
三大原因：
1.UI渲染需要时间较长，无法按时提交结果
2.一些需要密集计算的处理放在了主线程中执行，导致主线程被阻塞，无法渲染UI界面
3.网络请求由于网络状态的问题相应较慢，UI层由于没有模型返回无法渲染

屏幕是如何渲染的？
crt(阴极射线显示管)显示器是比较古老的技术，他使用阴极电子枪发射电子，在阴极高压的作用下，电子由电子枪射向荧光屏，使荧光粉发光，将图像显示在屏幕上，这也就是用磁铁靠近一些老式电视机的屏幕会让他们变色的原因。
而fps就是crt显示器的刷新频率，电子枪每秒会对显示器上内容进行60-100次的刷新，哪怕在我们看来没有任何改变。

但是lcd的原理与crt非常不同，lcd的成像原理跟光学有光：
1.在不加电压下，光线会沿着液晶分子的间隙前进旋转90度，所以光可以通过
2.在加入电压之后，光沿着液晶分子的间隙直线前进，被滤光板挡住

lcd的成像原理虽然与crt截然不同，每一个像素的颜色可以在需要改变时才去改变电压，也就是不需要刷新频率，但是由于一些历史原因，lcd仍然需要按照一定的刷新频率向gpu获取新的图片用于显示

宽泛的说，大多数的CALayer的属性都是由GPU来绘制的，比如图片的圆角，变换，应用纹理；但是过多的几何结构，重绘，离屏绘制以及过大的图片都会导致gpu的性能明显降低

AsyncDisplayKit（ASDK）是由Facebook开源的一个iOS框架，能够帮助最复杂的UI界面保持流畅和快速响应，他更像是对uikit的重新实现，把整个UIkit以及calayer层封装成一个一个node，将昂贵的渲染，图片解码，布局以及其他UI操作移除主线程，这样主线程就可以对用户的操作及时作出反应
在ASDK中最基本的单位就是ASDisplayNode，每一个node都是对UIView以及CALayer的抽象，但是与UIView不同的是，ASDisplayNode是线程安全的，他可以在后台线程中完成初始化以及配置工作。

ASDK的渲染过程
在ASDK中的渲染围绕ASDisplayNode进行，其过程总共有四条主线：
1.初始化ASDisplayNode对应的UIView或者CALayer
2.在当前视图进入视图层级时执行setNeedsDisplay
3.display方法执行时，向后台线程派发绘制事务
4.注册成为runloop观察者，在每个runloop结束时回调

如果ASDisplayNode是layerBacked的，他不会渲染对应的UIView以此来提升性能，因为UIView和CALayer虽然都可以用于展示内容，不过由于UIView可以用于处理用户的交互，所以如果不需要使用UIview的特性，直接使用CALayer进行渲染，能够节省大量的渲染时间

ASDK对于绘制过程的优化有三部分：分别是栅格化视图、绘制图像以及绘制文字
他拦截了视图加入层级翻出的通知-willMOveToWindow:方法，然后手动调用-setNeedsDisplay,强制所有的CALayer执行-display更行内容
然后将上面的操作全部抛入了后台的并发线程中，并在runloop中注册回调，在每次runloop结束时，对已经完成的事务进行-commit，以图片的形式直接穿回对应的layer.content中，完成对内容的更新。
从他的实现来看，确实解决了很多昂贵的cpu以及gpu操作，有效的加快了视图的绘制和渲染，保证了主线程的流畅执行

###[iOS中的JS](https://zhuanlan.zhihu.com/p/34646281)

JSCore的原理和通讯机制
浏览器内核的模块主要是有渲染引擎和JS引擎组成，其中JSCore就是一种JS引擎

JSVirtualMachine js的虚拟机，js运行以来的所有资源提供者
JSContext js的运行环境上下文，用于在native和js间进行数据的传递
JSvalue js的值对象，标记js原始数据类型，双方数据类型转换的桥梁
JSManagedValue 对象管理器，用于处理对象的生命周期
JSExport 接口导出协议，让Native对象直接暴露给JS调用
JSBase.h/JSContextRef.h/JSObjectRef.h。。。。 类，用于C语言的结构


在native应用中我们可以开启多个线程来一步之行我们的不同需求，也就意味着我们可创建多个JSVirtualMachine虚拟机（运行资源提供者），同时相互隔离不影响，这样我们就可以并行执行不同的js任务

在一个jsvirtualMachine中还可以关联多个JSContext（js执行环境上下文），并通过jsvalue（值对象）来和native进行数据传递通讯，同时可以通过jsExport（协议），将native中遵守此解析的类的方法和属性转化为js的接口供其调用


JSPatch
JSPatch是一个Ios动态更新框架，通过引入JSCore，就可以使用js调用任何原生接口，可以为项目动态更新模块，替换原声带吗动态修复bug，也即js传递字符串给oc，oc通过runtime接口调用和替换oc方法



###其他（知识点）
xib的原理就是将xml文件解析出来，找到相应的view，转换成代码，然后创建对象并显示。


nib的生命周期：
1.将nib文件加载入内存
2.“解固化”并实例化出nib文件里对应的对象
这个过程会调用initWithCoder:方法，注意不会再屌用initWithFrame:方法，步骤1虽然将nib加载到了内存，但是他还是“数据”的形式，而这一步吧步骤一中的“数据”变成了“对象”
3.建立connections(outlets,actions)
4.调用awakeFromNib方法
该函数只会在绑定xib的类中调用，不会在他的File's Owner及其内部的object类中调用
5.将xib中可见的控件显示出来

xib,storyboard都属于资源文件，而不是源文件。

内存泄漏：堆空间没有释放
野指针：栈指向的堆空间已经释放了

.Type表示的是某个类型的元类型，而在Swift中，除了class，struct和enum这三个类型外，我们还可以定义protocol。对于protocol来说，有时候我们呢也会想取得接口的元类型。这时我们可以在某个protocol的明智后面使用.Protocol来获取，使用的方法和.Type是类似的

async 表示该函数内部包含异步操作，需要将它交给内置执行器
await表示等待异步操作的实际结果

macos开发中applicationDidFinishLaunching中可以做一些应用启动前的初始化处理。
应用退出前可以在applicationWillTerminate中做一些全局性数据区/内存/资源的清理释放。

####并行与并发
并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是正真的同时
并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是正真的同时。并发往往在场景中有公用的资源。
线程安全：经常用来描述一段代码，指在并发的情况下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。反过来，线程不安全就意味着线程的调度顺序会影响最终的结果
同步：Java中的同步指的是通过认为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。
多线程：指的是这个程序（一个进程）运行时产生了不止一个线程
