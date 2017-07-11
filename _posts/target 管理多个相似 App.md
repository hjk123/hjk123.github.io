项目有一个需求就是私有化（拥有同样的功能但是图标，启动页，一些小功能有些不一样），以前是用分支来应对这个需求的，但是随着私有化越来越多，每次都要多开一个分支，且做的工作大同小异，看见 Android 组能可配置的打渠道包，iOS应该也可以。网上搜索发现唐巧大神早就这么实现了，[唐巧文章](http://blog.devtang.com/2013/10/17/the-tech-detail-of-ape-client-1/)，[这篇文章](http://blog.startry.com/2015/09/06/Optional-solutions-for-iOS-multi-target/)提出了另一张解决方案，使用脚本通过修改配置文件和替换资源文件来实现。这样省去了维护多个target，维护多套资源文件的麻烦，我们项目中还是按照唐巧说的的那种方式实现的，业务可能没他们那么复杂，接下来具体说。

## 如何创建target

>- 选中target -> 右键 -> 选择Duplicate 就能创建一个target，target 配置同时也会复制过来
>- Xcode -> manage schemes -> 修改schemes的名字 和 target 的名字一样
>- 创建target 的同时Xcode 会自动创建一个plist 文件,这时也应该修改plist文件的名称,且在Build Setting 里面修改Info.plist file 文件

 
## 不同target如何加载不同资源实现不同功能
>- 不同target 实现不同AppIcon，LaunchImage 非常简单，新建一套AppIcon 和LaunchImage, 在Xcode配置页面, general -> App Icons and Launch Image -> 选择新建的appIcon 和 LaunchImage 即可。
>- 如何实现不同target 加载不同图片资源?代码中图片名称保持一致，新增一个target 添加一个Imagexcassets，不同target引用不同Imagexcassets。如下图在我们项目中有Image.xcassets IBona.xcassets shixinOnLine.xcassets三个Imagexcassets,Image.xcassets 中是共有的图片,IBona.xcassets 和 shixinOnLine.xcassets 分别是对应target中不同的图片。

![](/assets/images/target-图1.png)

>- 如何实现不同target 实现不同功能?我们项目中采取的解决方案是 1.每一个target 创建一个json配置文件,配置文件里面主要包括了服务器地址,第三方key,某些功能是否展示key。2. 为每个target 创建不同的预编译宏定义，编译的时候根据不同的宏定义读取不同的配置文件,如下代码,在项目中配置了developer,test,production,shixin 四个预编译宏

```
/**
 读取配置文件
 */
-(void)readConfig
{
    NSString * filePath = nil;
#ifdef developer
    filePath = [[NSBundle mainBundle] pathForResource:@"IbonaDeveloperConfig" ofType:@"json"];
#elif test
    filePath = [[NSBundle mainBundle] pathForResource:@"IbonaTestConfig" ofType:@"json"];
#elif production
    filePath = [[NSBundle mainBundle] pathForResource:@"IbonaConfig" ofType:@"json"];
#elif shixin
    filePath = [[NSBundle mainBundle] pathForResource:@"IbonaShixinOnLine" ofType:@"json"];
#endif
    NSData * data = [NSData dataWithContentsOfFile:filePath];
    NSError * error;
    NSDictionary * json = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:&error];
    if (error) {
        NSLog(@"解析失败");
    }
    else
    {
        _setting = json;
    }
}

```

## swift compiler 999+ 问题及解决方案
在定义预处理宏的时候发现一坑，本来 production 刚开始定义的是 release ，由于我们的项目是Objective-C 和 swift 混编。每次编译都报swift编译器错误,而且错误999+,搞的一脸懵逼.找了好久才发现 然来是添加了release 预编译宏。不知道是系统已经定义了 还是swift 编译器的bug。反正把名字换一下就好了。  