# Configuration
## 诉求

1. 构建不同的宏来方便切换相应的配置；
2. 配置不同环境的icon，网络环境，bundle identifier
3. 至少两个类型的包能同时安装在手机上；
4. 最好能使用脚本实现自动化打包放入bugly或者蒲公英等平台供内部测试人员下载；

## 创建多个Configuration
有两种方法可以用来创建我们需要新增的`Build Configuration`，这里新建一个名为`Sit`的配置项，是为了满足App的网络环境的切换。

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/menu_create_configuration.png)

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/create_configuration.png)

## 环境的配置

1. 选择`Duplicate "Debug" Configuration`,得到`Debug copy`重命名为`Sit`
2. 然后`PROJECT` -> `Build Settings` -> `All` -> `Combined` -> search `Preprocessor Macros`，在`Sit`添加一个值为`SIT=1`
3. 重复上述操作分别添加`Dev`,`Uat`

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/multiple_configuration.png)

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/preprocessor_macros.png)

**提示：**如果在`TARGETS`中搜索的`Preprocessor Macros`,你会发现`Multiple values`中有一个值是`$(inherited)`,`TARGETS`中的值会继承自`PROJECT`

在项目中新建一个`Header File`,命名为`ApplicationConfig`,添加一下代码

```
#ifdef DEBUG
// Debug模式

#else
// Release模式

#endif


#ifdef DEV
// DEV环境
#definede kBaseURL @"https://127.0.0.1/"

#endif

#ifdef SIT
// SIT环境
#definede kBaseURL @"https://127.0.0.2/"

#endif

#ifdef UAT
// UAT环境
#definede kBaseURL @"https://127.0.0.3/"

#endif

#ifdef PROD
// PROD环境
#definede kBaseURL @"https://127.0.0.4/"

#endif
```

在`Edit Scheme`中可以选择不同`Configuration`

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/choose_configuration.png)


## Tips

### 配置不同的AppIcon
配置AppIcon有两种必比较方便的方法。

第一种:

首先我们需要找UI设计师要三套不一样的图标，如下图这样取好对应的名称放入`*.xcassets`中：

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/multiple_icon.png)

然后再当前`Target`的`Build Setting`下搜索icon找到`Asset Catalog App Icon Set Name`，然后进行如下配置：

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/set_configuration_icon.png)

然后Edit Scheme选择相应的Configuration进行编译或者打包就能打出不同的图标了。

第二种

使用`User-Defined`配置三种`Configuration`下的变量，在`Info.plist`中进行配置，配置方法与下面的应用名称配置类似，这里不做过多描述。

### 配置不同的AppName

配置不同的应用名称，这里需要使用到User-Defined加上info.plist来进行配置；
首先，我们需要新增一个User-Defined，如下图：

![方法一](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/build_setting_set_app_name.png)
![方法二](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/menu_set_app_name.png)

然后在`info.plist`中加入`Bundle display name`，设置成我们刚刚新建的`User-Defined`值：`$(APP_DISPLAY_NAME)`

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/bundle_display_name.png)


### 配置不同Configuration下Target的`Bundle ldentifier`

`TARGETS` -> `Build Settings` -> `All` -> `Combined` -> search `Product Bundle Identifier`

我们可以根据自己的需要设置各种scheme下的配置不同的Bundle ID，如下图：

![](https://github.com/lightank/iOS_notes/blob/master/Resource/config%26target/target_congfig_bundle_identifier.png)

正常情况下，以上步骤完成之后，如上图选择`Edit Scheme`切换`Build Configuration`就能编译出相应环境下的App，但是如果你的App使用pods来管理第三方库，使用新建的配置项就会报错找不到第三方的库文件，错误信息类似如下：

```
library not found for -lAFNetworking

linker command failed with exit code 1 (use -V to see invocation)
```

原因是`pods`工程并未自动帮我们创建相应的pod配置项，发现这一点之后我手动创建了一个同样名为`Preform`的`pod`配置项，于是编译通过了，但是打ipa包的时候始终通不过，继续查找原因，原来`xcconfig`文件需要终端执行`pod install`进行全面配置，所以大家在新建完了之后记得要`pod install`一下，才能放心使用。

# Target

## 创建target

* 新建target
    * File -> New -> Target -> search `app` -> Single View App,这样新建的target有自己的AppDelegate 和 main，可以通过打开右侧边栏点击文件修改该文件所属Target Membership来复用原来的AppDelegate和 main
* duplicate target
    * project -> TARGETS -> 右键所需复制的Target -> duplicate -> duplicate only.其好外：如果两个target的相同点很多,用duplicate,就可以把相关的设置全部拷贝过来,而不需要做 过多的修改，生成一个新的target。

### duplicate target
1. 修改target名称
2. 修改schemes名称
3. 修改info.plist名称：项目会多一个`xxx copy-Info.plist`,修改名称
4. 配置target对应的info.plist
    * project -> 相应target -> General -> Identity 点击 Choose Info.plist File...，找到相应的文件导入）
    * project -> 相应target -> Build Settings -> 搜索`Info.plist File` -> 点击Info.plist左边value(路径)）,然后输入相对路径`$(SRCROOT)/projectname/infoName.plist`
5. 修改BudleId及版本号
6. 宏定义（preprocessor macros）设置(Swift工程中设置Other Swift Flags,由于swift取消宏定义所以在macros那边设置无效 -- 有坑放置结束后提及)：
由于多个target使用同个文件时，但又存在一定的差异，在代码中可以实现根据不同宏执行不一样的代码，使其区别target
    * project -> TARGET -> build settings -> search: `Preprocessor Macros`,在 Debug/Release 中添加 `targetName(你的target名字)=1`,在写代码的时候，使用
        
        ```
        #if targetName
        #elif
        #else
        #endif
        ```
        来区分不同target
7. Assets.xcassets，一般设置app icon 以及 lanuch image的设置，要区别它俩可以新.xcassets 一个对应一个的.xcassets 或共用 同一个.xcassets
    * 不共用,快捷键`command + n`或菜单键中选择新建file,搜索 Asset 点击“Next”，勾选需绑定的target
    * 共用，新建iOS App Icon / Launch Image,然后Targets -> general -> App lcons and L aunch Images -> 选择相应的icon 跟 Launch Image
    
    
### CocoaPods

多个target共用同一个CocoaPods

方法1：
```
def testing_pods
    pod 'Quick', '0.5.0'
    pod 'Nimble', '2.0.0-rc.1'
end

target 'MyTests' do
    testing_pods
end

target 'MyUITests' do
    testing_pods
end

```

方法2：

```
pod 'ShowsKit'
pod 'Fabric'

# Has its own copy of ShowsKit + ShowWebAuth
target 'ShowsiOS' do
  pod 'ShowWebAuth'
end

# Has its own copy of ShowsKit + ShowTVAuth
target 'ShowsTV' do
  pod 'ShowTVAuth'
end
```
    

# 脚本打包

* [CLI for Building & Distributing iOS Apps (.ipa Files)](https://github.com/nomad/shenzhen)

# 参考链接

* [Xcode Concepts](https://developer.apple.com/library/archive/featuredarticles/XcodeConcepts/Concept-Targets.html) by [Apple][Apple]
* [iOS - 开发一套代码多个app展示不同图标和名称](https://www.cnblogs.com/gongyuhonglou/p/7766291.html)
* [iOS多个Target配置详情操作](https://www.jianshu.com/p/18db54655246)
* [CocoaPods: The Elegant Solution To Installing The Same Pod In Multiple Targets](https://www.natashatherobot.com/cocoapods-installing-same-pod-multiple-targets/#)
* [The Podfile](http://guides.cocoapods.org/using/the-podfile.html)


[Apple]:https://developer.apple.com/library/archive/navigation/