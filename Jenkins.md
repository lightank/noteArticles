# 使用 Jenkins 实现持续集成 (iOS)

# 环境
操作环境在 Mac OS 10.14 中进行。Jenkins 的版本是 2.19.1，fastlane 的版本是 2.28.2

# 安装 fastlane 

可能需要更新系统自带的Ruby
* 查看软件源：`gem sources -l`
* 移除原有的软件源：`gem sources --remove xxx`
* 添加新的软件源：`gem sources --add https://gems.ruby-china.com/`
* 也可以在移除同时添加软件源：`gem sources --add https://gems.ruby-china.com/ --remove xxxx`

安装fastlane

```
# Using RubyGems
sudo gem install fastlane -NV

# Alternatively using Homebrew
brew cask install fastlane
```

安装成功会提示

```
Successfully installed fastlane-2.106.2
```
    
相应资源地址：
* [GitHub地址](https://github.com/fastlane/fastlane)
* [docs 地址](https://docs.fastlane.tools)

# 安装 Jenkins 

先安装[JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

下载[Jenkins .war](https://jenkins.io/download/)文件

下载完成后，打开终端，进入到 war 包所在目录，执行以下命令：

```
java -jar jenkins.war --httpPort=8080
```

可能报错

```
严重: Running with Java class version 55.0, but 52.0 is required.Run with the --enable-future-java flag to enable such behavior. See https://jenkins.io/redirect/java-support/
java.lang.UnsupportedClassVersionError: 55.0
	at Main.main(Main.java:139)

Jenkins requires Java 8, but you are running 11+28 from /Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home
java.lang.UnsupportedClassVersionError: 55.0
	at Main.main(Main.java:139)
```

这个时候先执行下列代码去开启支持更新版本的Java：

```
java -jar jenkins.war --enable-future-java
```

安装完会提示

```
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

52b328a3495e4485b5461b9a4dc3e43b

This may also be found at: /Users/huanyu.li/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

这个时候在浏览器中打开

```
http://localhost:8080
```

![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/jenkins_localhost.png)

复制终端的密码，进入`Customize Jenkins`，选择`Install suggested
plugins`，安装完进入`Create First Admin User`

![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/create_first_admin_user.png)

![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/jenkins_is_ready.png)

![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/instance_configuration.png)

## 安装插件
比如`GitLab Plugin`、`Xcode integration`、`Keychains and Provisioning Profiles Management`插件

```
常用插件如下：
Git plugin
Git client plugin
Subversion Plug-in
Subversion Release Manager plugin
Subversion Tagging Plugin
SVN Publisher plugin
SSH Credentials Plugin
Gradle plugin： Android
Xcode integration：iOS
Keychains and Provisioning Profiles Management：iOS/Mac OS
```
 
 ![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/plugins.png)
 
 ![](https://github.com/lightank/iOS_notes/blob/master/Resource/Jenkins/download_plugins.png)

## 配置Keychains和Provisioning
 
 `系统管理` -> `Keychains and Provisioning Profiles Management`
 
## 重置 Jenkins 新建的用户

使用dmg包安装的Jenkins会默认新建一个叫`Jenkins`的用户，可以在偏好设置中的`用户与群组`，可重置密码，记得打开`允许用户管理这台电脑`
 
## 卸载Jenkins
 
 ```
//进入以下目录，双击运行
/Library/Application Support/Jenkins/Uninstall.command
//也可以这样运行
sh "/Library/Application Support/Jenkins/Uninstall.command"
//删除配置，这个可选
sudo rm -rf /var/root/.jenkins ~/.jenkinssudo launchctl 
unload /Library/LaunchDaemons/org.jenkins-ci.plistsudo 
rm /Library/LaunchDaemons/org.jenkins-ci.plistsudo 
rm -rf /Applications/Jenkins "/Library/Application Support/Jenkins" /Library/Documentation/Jenkinssudo 
rm -rf /Users/Shared/Jenkinssudo dscl . -delete /Users/jenkinssudo dscl . -delete /Groups/jenkinssudo 
rm -f /etc/newsyslog.d/jenkins.confpkgutil --pkgs | grep 'org\.jenkins-ci\.' | xargs -n 1 sudo pkgutil --forget

//如果使用brew安装的，可以执行以下命令
brew uninstall jenkins
 ```