---
title: linux实用软件
---

#### 1.搜狗输入法
有个好的输入法，可以增强我们对新系统的舒适度，搜狗输入法也为linux用户提供了相应的颁布，下载地址:[http://pinyin.sogou.com/linux/](http://pinyin.sogou.com/linux/)　，具体配置可以看看这篇文章：[http://jingyan.baidu.com/article/08b6a591cb06f114a8092209.html](http://jingyan.baidu.com/article/08b6a591cb06f114a8092209.html)

#### 2.Linux环境下JDK配置和安装
jdk下载安装之后，将其解压到/opt目录下，然后在命令行输入``` sudo gedit /etc/profile```，亦可用文本编辑器打开，将下面的代码加入文件的尾部．
```
export JAVA_HOME=/opt/jdk1.8.0_91
export JRE_HOME=/opt/jdk1.8.0_91/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
，保存之后，注销系统，打开命令行```java -version```，出现以下信息，就说明jdk已经配置成功
```
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```
当然你也可以解压到别的目录下，需要注意的是，要将下面的jdk目录换成相应的目录，

#### 3.WPS
WPS下载地址：[http://community.wps.cn/download/](http://community.wps.cn/download/)
可以直接下载deb版的，双击安装即可，不过安装之后，会提示你wps缺少字体，解决这个问题需要下载对应的缺失字体，字体我放在我的百度网盘了，需要的可以直接下载，分享链接：[https://pan.baidu.com/s/1nvmaYYP](https://pan.baidu.com/s/1nvmaYYP) ，字体解压之后，将里面的字体文件放到主目录的.fonts目录下（如果没有该目录就建一个），然后注销，再打开wps即可．


#### 4.FileZilla
&#160; &#160; &#160; &#160;FileZilla是一个免费开源的FTP软件，分为客户端版本和服务器版本，具备所有的FTP软件功能。可控性、有条理的界面和管理多站点的简化方式使得Filezilla客户端版成为一个方便高效的FTP客户端工具，而FileZilla Server则是一个小巧并且可靠的支持FTP&SFTP的FTP服务器软件。对于使用Linux系统的学生来说，这是交作业的不二之选，不过交作业之前得先连上学校的内网，注意传输协议改为带SFTP-SSH字样的。
这个软件可以再官网下：[点击这里下载](https://nchc.dl.sourceforge.net/project/filezilla/FileZilla_Client/3.17.0.1/FileZilla_3.17.0.1_src.tar.bz2)），也可以在软件源中点击安装下载。

#### 5.Remmina
&#160; &#160; &#160; &#160;Remmina是一个用远程桌面软件，提供了RDP、VNC、XDMCP、SSH等远程连接协议的支持。这个客户端最大的优点在于界面清爽，方便易用，创建远程连接的界面与Windows自带的远程桌面十分相近。可以到Linux软件管理器中搜索下载安装，十分方便．

#### 6.Kazam
一款不错的截图，截视频的软件，可以到软件源下载．


#### 7.Chrome推荐插件
Chrome应用市场：[点击进入](https://chrome.google.com/webstore/category/extensions?hl=zh-CN)
+ chrome美化插件：Infinity
+ 快速安全通道：一款免费的网站加速插件

#### 8.winqq
QQ2012基于WineTM版，支持双击deb包安装、支持全局热键、不会自动离线、文件传输正常、ibus中文输入法正常。
下载地址：[http://www.ubuntukylin.com/application/show.php?lang=cn&id=279](http://www.ubuntukylin.com/application/show.php?lang=cn&id=279)

#### 9.网易云音乐
下载地址：[http://music.163.com/#/download](http://music.163.com/#/download)

#### 10.小书匠
一款不错的markdown编辑器，可以把写好的文件保存到有道云笔记，支持同步预览，个人感觉很不错，下载地址：[https://github.com/suziwen/markdownxiaoshujiang/releases/tag/v2.1.0](https://github.com/suziwen/markdownxiaoshujiang/releases/tag/v2.1.0)



