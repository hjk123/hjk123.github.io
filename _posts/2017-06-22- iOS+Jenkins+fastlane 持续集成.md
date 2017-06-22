
![](/assets/images/图1.jpeg)
## 安装方式
### 下载安装包
我第一次安装使用下载安装包的方式安装的，如下图所示:

![](/assets/images/图2.png)

但是这种方式安装之后,配置完成 **Jenkins** 之后发现一切进行的都不太顺利，遇到好多问题，但是解决问题的过程算是有点收获，现在列举如下：（先声明一下，按照第二种方式安装以后这一切问题都没了，具体啥原因还没搞清楚）<br>
1. git dowm 代码的速度非常慢，超过设置的10分钟，一直报错，说超时。无奈之下把时间设置为60分钟，但是一个小时过去了 还是down 不下来代码，估计肯定是有问题了，但是Google 了半天也不知道是什么问题,这时候内心是崩溃的。<br>
2. 我用的是系统账户登录的，但是操作某些目录的时候出现 **permission delay** 错误，一种解决办法就是执行 **sudo chown -R username app_path** 命令,**username** 为登录的账户名 **app_path**为文件路径<br>
3. 执行fastlane 相关命令的时候出现 **fastlane command not found** 错误，网上找到一解决办法就是**Jenkins**安装 **environment injector plugin** <br>
4. 下载Jenkins 执行命令： **/Library/Application\ Support/Jenkins/Uninstall.command**
### war包 + 命令行方式
图3
上图是官方文档安装步骤的截图，下载war包 ，cd 到war包路径下面，执行 **java -jar jenkins.war** ，就能将 **Jenkins** 服务起起来，然后访问 localhost:8080 ,先经过一些初始化设置，然后Jenkins会推荐安装一些插件, 我安装了jenkins 推荐安装的一些 **plugins**，安装基本就完成了。
### brew 
执行brew install jenkins 安装，这种方式没试过
## Jenkins 基本使用

1. 新建项目<br>
如下图，输入名称即可 
![](/assets/images/图3.png)

2. 项目基本配置


* 源码管理 
 ![](/assets/images/图4.jpeg)
 如上图 git 方式管理源码，添加git 地址，认证用户名和密码
* 构建触发器
![](/assets/images/图5.png)
如上图我们选择的是日程表的方式触发构建，如图 每6个小时构建一次

* 构建环境 
* 构建 
如下图，我们选择使用fastlane 工具来编译打包工程，也可以使用其他工具来进行编译操作
![](/assets/images/图6.png)

* 构建后操作

我们有两个构建后操作，1. 上传ipa文件到fir.im来下发到测试人员 。 2. 构建成功邮件通知相关开发人员。注意：配置邮件通知的时候，jenskin 管理员的邮箱一定要配置。如下图
![](/assets/images/图7.png)

![](/assets/images/图8.png)

## fastlane 简介
**fastlane** 是一个工具集，常用的工具包括 **deliver：**用来发布应用到 App Store,**match:**通过git来管理项目 certificate，Profile 证书，**pem:** 自动创建通知证书, **sign** 自动创建provisioning 证书，**cert:** 自动创建certificate证书. 等等还有很多其他的工具，可以自己到fastlane 的 GitHub主页查看  我只是列举出了我常用的几种，我这次搭建CI用到了fastlane,也遇到了一些问题，下面这两个问题花费的见识是最多的，记录一下。
### 使用fastlane 遇到的问题
1. **connection reset by ssl_connect**<br> 
当执行 fastlane match development 的时候会报图下图所示的错误 
![](/assets/images/图9.png)

解决办法就是升级ruby 升级OpenSSL,但是升级完成OpenSSL后，但是执行openssl version 看见的还是老版本

2 . **解决了第一个问题以后出现如下图所示的问题** 

 ![](/assets/images/图10.png) 
 
说无法得到访问 https://git.oschina.net 的 username，因为 match 证书文件和项目文件都在 oschina 上面，而Jenkins 已经配置了https 方式访问 oschina 的账户密码，如果 match 还以 HTTPS 方式访问，就没办法拿到账户名和密码，因为 Jenkins 占用了，match 的[方文档](https://github.com/fastlane/fastlane/tree/master/match)也说了这个问题。我目前的解决方式就是 在 oschina 配置RSA公钥，让match 以ssh 的方式访问 git ,问题得到解决。 



