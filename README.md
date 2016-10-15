# pitfalls-ios


#### 2016-10-15

1、MJ Refresh 使用过程的崩溃解决方案    

有时候在释放一个包含MJRefresh的控件或者View的时候崩溃，然后报了如下的崩溃信息    

*** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'An instance 0x102031200 of class UITableView was deallocated while key value observers were still registered with it. Current observation info: <NSKeyValueObservationInfo 0x17422f4e0> (
<NSKeyValueObservance 0x174447380: Observer: 0x100e07860, Key path: contentSize, Options: <New: YES, Old: NO, Prior: NO> Context: 0x0, Property: 0x1744473b0>
<NSKeyValueObservance 0x174447530: Observer: 0x100e07860, Key path: contentOffset, Options: <New: YES, Old: NO, Prior: NO> Context: 0x0, Property: 0x174447560>
<NSKeyValueObservance 0x1744476b0: Observer: 0x100e3e810, Key path: contentOffset, Options: <New: YES, Old: NO, Prior: NO> Context: 0x0, Property: 0x174447560>
)'
*** First throw call stack:
(0x18c7241c0 0x18b15c55c 0x18c724108 0x18d171df4 0x18b175fe0 0x19256b134 0x1926f18f8 0x1000c03f0 0x100225870 0x18c713e30 0x18c605c70 0x1926f1d08 0x1926fcbb4 0x10009ba0c 0x18b15af10 0x18b1676e0 0x18b167744 0x19290afe0 0x1926f1d30 0x18c601

解决方法：在dealloc方法内部移除将MJRefresh对ScrollerViewcontentOffset的KVO移除   
example:   
    -(void) dealloc   
    {   
            [self.tableView removeObserver:self.mjheader forKeyPath:@"contentOffset"];   
    }


#### 2016-07-13

1、外部链接打不开的问题：    
情景：当在运用中使用［［UIApplication sharedApplication］openURL：［NSURL urlWithString：urlString］］去打开一个外部链接时，往往都能正常的打开，凡事都有例外，也有怎么都打不开的情形。    
i、当urlString存在空格的时候，比如第一个字符为空格，那么，打开外部链接的命令将不生效。    
ii、当urlString 存在中文字符时也打不开。    

成因分析：    
构建URL的时候（不管是＋号方法还是减号方法构建的，只要第一个字符为空格，可能苹果为了提高效率考虑，直接返回URL空对象，所以打开外部链接不生效）。    

解决方案  
1、去空格    
2、统一UTF－8 编码    
代码：    

    NSString *urlString = [NSString stringWithString:string];
    
    NSString *URL = [urlString stringByReplacingOccurrencesOfString:@" " withString:@""];
    
    NSURL *url = [NSURL URLWithString:[URL stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding]];
    
    return url;

#### 2016-05-12


1、iOS工程图片导出的问题

情景：当有新项目需要在原本原有的项目进行二次开发的时候，设计师往往会找我们要我们之前工程的图片，而这时我们往往会想到对工程目录当中的图片或者Imagee.xcassets 里面的图片进行复制过来再传给他（她）。可最终的结果是，设计师后来给的图片和之前使用的图片出现了不同程度的突差别，导致在开发过程中不尽人意。

解析：如果全部图片放在了工程目录下，复制的时候，直接复制过去，问题不是很大；可如果图片是放在了Imagee.xcassets文件夹下，而你直接把这个Imagee.xcassets里面的文件全部复制给了设计师，那么设计师的工作量估计不小。Xcode 下图片只要放入了Imagee.xcassets，Xcode自动回为图片包上了一个.imageset的文件夹，并生成了对应的content.json文件，设计师拿到的时候，要把图片一张张抽出来，工作量可想而知。这种方发绝不是一种好的做法。

最佳做法是：先给我们之前的工程打一个包（测试包或者，App Store 包）都行，用归档使用工具进行打开，这时候你会发现，我们的包变成了Payload的文件夹，打开这个Payload文件夹会看到一个.app文件，接着显示包内容你会看到工程所有工程当中使用的图片的在这里了纯粹的图片，没有使用的图片不在，（不行你自己实验一下），只需要把这里边的图片全部拷出来，发给设计师，大功告成。

原理：Xcode其实在打包的时候，会对应用当中的资源进行检测，没有用到的资源不会放到包里面，所以，在这里找到的图片一定涵盖了工程中使用的所有图片。在这一点上，感觉Xcode还是挺强大的。

#### 2016-04-07（四）


1.iOS 数据库升级导致程序崩溃 

情景：应用程序更新到最新版本，运行出现程序崩溃

原因：为了使应用程序文件命名规范化，工程里的.h 和.m等文件类前缀改为LX，当应用重新启动后，程序读取残留下来的，未更改类前缀名字的数据，
与更改名字后的类有冲突，导致程序崩溃。

解决方法：由于该数据只是用作缓存，不包含有用户的信息，因此在新版本程序中，在读取数据之前，可以使用NSFileManager把旧的数据库文件删掉，重新创建新的数据库文件，即可解决崩溃问题
#### 2016-03-31(四)

1.iOS 闪屏图片对应用分辨率的影响  
情景1：  
很多时候我们会发现（尤其是使用6/6S或是6/6sPlus的时候），同尺寸安卓设备和iOS设备，iOS设备的分辨率明显却差了很多，有些模糊的感觉，给人很low的感觉，甚至有时候会疑惑  
情景2：
有时候当我们使用设备（6/6S，6/6sPlus）进行开发新项目或是调试的时候，会发现这样的一种现象：我们所要写的控件，图片或是字体，明显的比设计师给的效果图或是切图有很大的突兀，有时候甚至觉得设计师弄错了，很多时候，我们放大或是缩小我们的字体控件等等以使得UI效果和设计师提供的原型一致。  
情景3：
有时候版本发App Store或是打测试包在不同的设备上运行的时候，我们会发现有些尺寸的屏幕闪屏没有，譬如4S，6/6s等等，表现为在设备运行的时候一片漆黑一闪而过。  
以上的问题，其实都是由于各种iOS设备屏幕的闪屏没有添加完毕（可能缺少某一种），app在运行的时候会根据闪屏去唯一确定分辨率，如果6/6s plus 的闪屏缺少，系统会去选择同iphone 5 屏幕尺寸一致的图片作为更大尺寸设备屏幕的闪屏，系统默认的分辨率就是iphone 5手机分辨率，所以我们在6/6s上看到的效果就会差很多，当我们重新添加上这些缺少的图片以后，6/6s plus的分辨率就恢复正常了。
