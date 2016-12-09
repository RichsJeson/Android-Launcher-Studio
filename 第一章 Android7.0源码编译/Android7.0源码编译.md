###### 本人邮箱:richsjeson@gmail.com
###### 欢迎转载，请注明出处。
###### 本文出自[自己动手编写插件化](http://www.jianshu.com/users/0a2868f025e0/latest_articles)
---
###### 由于笔者的电脑是Mac OSX EI Captian版本，因此本教程所有的操作都是在Mac OSX环境下进行源码编译和LLDB调试。本教程从Android7.0源码进行分析。
---
* [1 关于Android 7.0](#1)
* [1.1 版本变化](#1.1)
* [1.2 总结](#1.2)
* [2   源码编译](#2)
* [2.1 环境设置](#2.1)
* [2.2 下载源码](#2.2)
* [2.3 源码编译](#2.3)

---
<h2 id="1">1.关于Android 7.0</h2>
###### Android 7.0即 “Nougat”是Google开发的Android操作系统的第7个系统，Android 7.0引入操作系统及其开发平台显著的变化，包括屏幕上的分屏视图同时显示多个应用程序的能力，内联通知回复的支持，以及一个基于OpenJDk的JAVA环境，引入先进的Vulkan 2D/3D图形渲染的支持，并支持设备“无缝” 系统更新。
<h3 id="1.1">1.1 版本变化</h3>
###### 现在，让我们来回顾下Android版本中比较显著的变化，由于笔者是基于Android 1.5学起，因此文章从1.6开始讲解。
* Android 4.0 之前的变化
###### 和早期的诺基亚时代一样，在1.5之前，Android的键盘始终是实体键盘。为了方便，google学习了苹果的虚拟键盘，从Android 1.5开始进行大量的改进，谷歌中加入了拍摄/播放视频、WebKit内核、手机屏幕旋转、相机启动加速。这些都是对1.5版本以下做了补充。然而在2009年9月的Android 1.6版本后，针对当时的移动网络做了优化，提供了CDMA网络支持，手势支持、文本转语音系统、虚拟屏幕键盘，VPN、OpenCore2媒体引擎。在Android2.0中更是对原先版本进行优化，支持硬件加速、内置相机闪光灯、支持数码变焦、支持蓝牙2.1，支持动态桌面的设计、同时也支持更多的分辨率、支持Adboe Flash（ps:于是有了AIR技术）、加强软件即时编译的速度。到了Android 2.3，google修复UI、支持近场通讯、强化电源、应用程序管理功能、支持从YAFFSZ转化至ext4文件系统（ps:sd 卡的加载）。到了3.0，我们拥有了平板的时代，在这个时代下，google提供了平板电脑的使用，增加了fragment技术，3D加速处理、加强多任务处理、支持多核心处理器、FAC音视频播放支持、高性能的WIFI锁、对新的厂商硬件支持、增加了应用兼容性功能。
* Android 4.4  之前的变化
###### 至4.0起，google开始注意到android碎片化的严重性。于是提出了Android的规范。统一了手机和平板电脑的使用习惯，提升了硬件的性能和系统的优化、完善了虚拟按键，界面以标签页的形式展示、内置流量监控、人脸识别技术、提供了随时关闭正在使用的应用程序、增强硬件加速的功能、WIFI直连技术。到android 4.2以后，更是引入360全景照相技术、、改善蓝牙A2DP流问题。在4.3以后，支持多用户登录、蓝牙低功耗、OpenGL ES3.0 、增加纹理支持、多重渲染目标、TRIM指令、多缓冲器对象。
* Android 4.4  之后的变化
###### Android 4.4，可以说是一个历史的转折点，按照以往之前，android的虚拟机都是由dalvik。由于dalvik是典型的虚拟机模式，因此每个App创建进程时都要求Dalvik在后台迅速把字节码进行运行时编译。这样运行时编译，导致代码的执行速度比较慢。为了解决这个问题。google在android 4.4引入了新的虚拟机机制，即ART.由于之前多任务处理时，内存消耗过大，因此在4.4以后，googlek开始对内存进行大量的优化、存储访问框架（ps:SAF,即让用户能够在其所有首选文档存储提供程序中方便地浏览并打开文档、图像及其它文件。 ）。直至2014年6月Google在i/o大会上，提出Android 5.0所有的平台都应遵循新的设计模式-Material Design.随着手机的高速发展，64位处理器，应运而生，google为此提供了64位处理器支持。面向碎片化service的随意使用，尤其是app国内产商用户恶意使用service服务，造成用户的怨气上深，google提出了以系统服务为主的省电优化API（ps:使用JobScheduler来替代service技术，实现后台运作（早期是由asyn adapter执行操作【需要service配合】，国内开发者不会使用，强制要求servie后台进程运作），防止service的恶意使用）。然而在今天，依然有APP产商还在持续的实现这种恶意的操作。（ps:google的意思是可以做，但是要在我google的system server的把控下进行操作。不允许你乱来。相信google为了性能，后面会完全封闭service这条通道。毕竟google已经给你一个技术方案了。）为了防止app产商的恶意操作，google在android 6.0开始做起了防御措施。即运行的权限系统。用户未授权的情况下，不得随意访问操作。删除访问设备的本地硬件标识符。将手机自带的存储空间和SD存储卡空间合并，统一为一个存储空间。通过Doze功能，减少电源的消耗、新增指纹扫描、支持USB Type C接入、强制杀死进程，封闭jni层的进程创建（ps:早期service的守护进程的方式有几种，其中一种方式就是利用c fork进程，来增强进程守护的能力。）
* Android 7.0
###### Android 7.0 可以说是对整个Android操作系统应用程序的提升。并且使整个android系统更趋向于简洁，阻止开发商加载非公共的API。继续完善应用授权能力（即系统底层操作加入权限控制）、禁止应用公开file://URI操作。如需操作需要授权。改进action操作、加强JobScheduler能力（ps:7.0以后，JobScheduler可随系统启动时，自动启动。来实现后台任务。）、新增分屏支持、增强通知的能力（可自动回复）、组件的模块化、Vulkan API、密钥认证、APK signature scheme v2的支持、VR技术、强化Doze 的省电功能。
<h3 id="1.2">1.2 总结</h3> 
###### 从Android的历史演变，我们发现google的android系统在Launcher方面开始将所有的底层进行模块化操作。因此，对android 7.0的源码研究对我们的插件化课程有着更深层次的意义。现在我们来编译android源码。
<h2 id="2">2.源码编译</h2>
<h3 id="2.1">2.1 环境设置</h3>
###### * 安装openjdk8
###### 由于N支持OpenJDK8，在Ubuntu下使用的是open jdk8,而在MAC OSX情况下依然使用的是sun jdk 8 。<br/>
###### * 安装Mac Ports
###### 在编译源码之前，首先我们要安装好编译的环境。在Mac OSX情况下，按照google的官方文档。需要安装Mac Ports。
###### 1)打开 https://distfiles.macports.org/MacPorts/ ，找到对应系统的mac ports（ps：由于笔者用的是OSX EI Captian）版本。点击下载安装mac Ports。
![95849DC8-C1E1-430E-AF16-808120FB4A3B.png](http://upload-images.jianshu.io/upload_images/641423-b951b008cdf46fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) <br />
###### 2）安装相关的依赖库。<br />
###### * 执行语句：<br />
###### $ POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg <br/>
><h6>richsjesondeMacBook-Pro:~ richsjeson$ POSIXLY_CORRECT=1 sudo port install gmake libsdl gnupg <br/>
Password:
Sorry, try again.
Password:
--->  Computing dependencies for gmake 
--->  Cleaning gmake 
--->  Computing dependencies for libsdl <br/>
--->  Cleaning libsdl<br/>
--->  Computing dependencies for gnupg <br/>
--->  Dependencies to be installed: libusb-compat libusb openldap cyrus-sasl2 db46 icu perl5 tcp_wrappers readline <br/>
</h6>
###### 3)设置环境变量<br/>
###### 在编译前，请确认你当前的路径在/usr/bin，下，设置环境变量"export PATH=/opt/local/bin:$PATH”。
><h6>richsjesondeMacBook-Pro:~ export  PATH=/opt/local/bin:$PATH <br />
 richsjesondeMacBook-Pro:~ source ~/.bashrc_brofile <br/></h6>

4)安装 make 3.81版本
###### 由于之前安装的make版本是3.82的，为了让系统支持3.81版本，需要在/opt/local/etc/macports/sources.conf文件下新增一行：file:///Users/Shared/dports。并且创建该文件夹。之后从svn下载并更新mac ports下的make版本。并且通过portindex来创建ports 的索引。
> <h6>richsjesondeMacBook-Pro:~ vi /opt/local/etc/macports/sources.conf <br/>
richsjesondeMacBook-Pro:~  mkdir /Users/Shared/dports <br/>
richsjesondeMacBook-Pro:/ richsjeson$ cd /Users/Shared/dports <br/>
richsjesondeMacBook-Pro:dports richsjeson$ svn co --revision 50980 http://svn.macports.org/repository/macports/trunk/dports/devel/gmake/ devel/gmake/ <br/>
A    devel/gmake/Portfile <br/>
Checked out revision 50980. <br/>
richsjesondeMacBook-Pro:dports richsjeson$ portindex /Users/Shared/dports <br/>
Creating port index in /Users/Shared/dports <br/>
Adding port devel/gmake  <br/>
Total number of ports parsed: 	1 <br/>
Ports successfully parsed:	1 <br/>
Ports failed:			0   <br/>
Up-to-date ports skipped:	0 <br/>
richsjesondeMacBook-Pro:dports richsjeson$ sudo port install gmake @3.81
--->  Computing dependencies for gmake
--->  Cleaning gmake
--->  Scanning binaries for linking errors
--->  No broken files found.
</h6>

<h3 id="2.2">2.2 下载源码</h3>
###### 完成以上操作后，开始进行源码的下载。首先我们先从git下载repo脚本。
><h6>richsjesondeMacBook-Pro:dports richsjeson$ mkdir ~/bin <br/>
richsjesondeMacBook-Pro:dports richsjeson$ ls <br/>
PortIndex	PortIndex.quick	devel <br/>
richsjesondeMacBook-Pro:dports richsjeson$ PATH=~/bin:$PATH <br/>
richsjesondeMacBook-Pro:dports richsjeson$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo <br/>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current <br/>
                                 Dload  Upload   Total   Spent    Left  Speed   
100 27759  100 27759    0     0  35051      0 --:--:-- --:--:-- --:--:--  107k  <br/>
richsjesondeMacBook-Pro:dports richsjeson$ chmod a+x ~/bin/repo <br/>
</h6>

###### 由于笔者使用的自己的移动硬盘进行编译，如果大家需要使用移动硬盘编译，请将移动硬盘格式成MAC OSX 日志扩展盘（区分大小写）。选择源码目录在/Volumes/DOCUMENT下。在该目录下创建ANDROID_NOUGAT文件夹，用于存储android nougat的源码。执行repo init命令下载git文件。
><h6>richsjesondeMacBook-Pro:dports richsjeson$ cd /Volumes/DOCUMENT/ANDROID_NOUGAT
richsjesondeMacBook-Pro:ANDROID_NOUGAT richsjeson$ ls
richsjesondeMacBook-Pro:ANDROID_NOUGAT richsjeson$ repo init -u https://android.googlesource.com/platform/manifest
gpg: 钥匙环‘/Users/richsjeson/.repoconfig/gnupg/secring.gpg’已建立
gpg: 钥匙环‘/Users/richsjeson/.repoconfig/gnupg/pubring.gpg’已建立
gpg: /Users/richsjeson/.repoconfig/gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 920F5C65：公钥“Repo Maintainer <repo@android.kernel.org>”已导入
gpg: 密钥 692B382C：公钥“Conley Owens <cco3@android.com>”已导入
gpg: 合计被处理的数量：2
gpg:           已导入：2  (RSA: 1)
Get https://gerrit.googlesource.com/git-repo/clone.bundle
Get https://gerrit.googlesource.com/git-repo
remote: Counting objects: 2, done
remote: Finding sources: 100% (35/35)
remote: Total 35 (delta 13), reused 35 (delta 13)
Unpacking objects: 100% (35/35), done.
From https://gerrit.googlesource.com/git-repo
   01b7d75..16889ba  master     -> origin/master
   39252ba..16889ba  stable     -> origin/stable
 *[new tag]         v1.12.35   -> v1.12.35
 *[new tag]         v1.12.36   -> v1.12.36
Get https://android.googlesource.com/platform/manifest
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (22) The requested URL returned error: 404 Not Found
Server does not provide clone.bundle; ignoring.
remote: Counting objects: 356, done        
remote: Finding sources: 100% (356/356)           
remote: Total 4022 (delta 1133), reused 4022 (delta 1133)        
Receiving objects: 100% (4022/4022), 2.66 MiB | 3.10 MiB/s, done.
Resolving deltas: 100% (1133/1133), done.
From https://android.googlesource.com/platform/manifest
*[new tag]         android-6.0.1_r7 -> android-6.0.1_r7
*[new tag]         android-6.0.1_r8 -> android-6.0.1_r8
*[new tag]         android-6.0.1_r9 -> android-6.0.1_r9
*[new tag]         android-7.0.0_r1 -> android-7.0.0_r1
*[new tag]         android-7.0.0_r3 -> android-7.0.0_r3
*[new tag]         android-7.0.0_r4 -> android-7.0.0_r4
*[new tag]         android-7.0.0_r5 -> android-7.0.0_r5
*[new tag]         android-7.0.0_r6 -> android-7.0.0_r6
</h6>
###### 从源码中看到目前仓库中android已经更新到7.0.0_r6版本。我们使用repo init -u 方法来定位到r6版本。这时系统弹出相关提示，询问是否要用颜色的方式来区分内核和普通文件，选择是。
><h6>Your identity is: richsjeson@gmail.com <richsjeson@gmail.com>
If you want to change this, please re-run 'repo init' with --config-name
Testing colorized output (for 'repo diff', 'repo status'):
  black    red      green    yellow   blue     magenta   cyan     white 
  bold     dim      ul       reverse 
Enable color display in this user account (y/N)? y</h6>

###### 执行repo init -u https://android.googlesource.com/platform/manifest -b android-7.0.0_r6后，根据提示信息，执行repo sync命令来下载android 7.0源码。

<h3 id="2.3">2.3 源码编译</h3>
###### 在编译前，请确认你的环境都搭建完成。执行“ make clobber”检测环境是否搭建完成。如果为搭建完成，会给出相应的提示。比如最为常见的错误:""
><h6>异常1：build/core/config.mk:600: *** Error: could not find jdk tools.jar at /System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/../lib/tools.jar, please check if your JDK was installed correctly.  Stop.<br/>
原因：系统没有找到可对应的jdk版本。</br>
异常2：
Checking build tools versions… 
build/core/main.mk:117: ************************************************** 
build/core/main.mk:118: You are building on a case-insensitive filesystem. 
build/core/main.mk:119: Please move your source tree to a case-sensitive filesystem. 
build/core/main.mk:120: ************************************************** 
build/core/main.mk:121: * Case-insensitive filesystems not supported. Stop.<br/>
原因：makefile无法找到Mac osx扩展（区分大小写）的分区。</br>
</h6>
