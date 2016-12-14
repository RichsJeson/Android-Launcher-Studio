###### 本人邮箱:richsjeson@gmail.com
###### 欢迎转载，请注明出处。
###### 本文出自[自己动手编写插件化](http://www.jianshu.com/users/0a2868f025e0/latest_articles)
---
###### 由于笔者的电脑是Mac OSX EI Captian版本，因此本教程所有的操作都是在Mac OSX环境下进行源码编译和LLDB调试。本教程从Android7.0源码进行分析。
>*主机配置：CPU:2.2 ghz IntelCore i7,内存：16G DDR3 <br/> 
*操作系统：OSX EI Captian 10.11.6 <br/> 
*源码地址：https://android.googlesource.com <br/> 
*源码大小：由于笔者所使用的是repo命令，当执行repo sync后，会将现有的全套git上的源码都下载下来。<br/> 
*目标编译地址：[Target]/WORKSPACE/ANDROIDNOUGAT <br/> 

---
 1. [关于Android 7.0](#1)
    * [1.1 版本变化](#1.1)
    * [1.2 总结](#1.2)
 2. [源码编译](#2)
    * [2.1 环境设置](#2.1)
    * [2.2 下载源码](#2.2)
    * [2.3 源码编译](#2.3)
    * [2.4 Repo的使用](#2.4)
    
---
<h2 id="1">1.关于Android 7.0</h2>
######Android 7.0即 “Nougat”是Google开发的Android操作系统的第7个系统，Android 7.0引入操作系统及其开发平台显著的变化，包括屏幕上的分屏视图同时显示多个应用程序的能力，内联通知回复的支持，以及一个基于OpenJDk的JAVA环境，引入先进的Vulkan 2D/3D图形渲染的支持，并支持设备“无缝” 系统更新。

<h3 id="1.1">1.1 版本变化</h3>

######现在，让我们来回顾下Android版本中比较显著的变化，由于笔者是基于Android 1.5学起，因此文章从1.6开始讲解。

* Android 4.0 之前的变化

######和早期的诺基亚时代一样，在1.5之前，Android的键盘始终是实体键盘。为了方便，google学习了苹果的虚拟键盘，从Android 1.5开始进行大量的改进，谷歌中加入了拍摄/播放视频、WebKit内核、手机屏幕旋转、相机启动加速。这些都是对1.5版本以下做了补充。然而在2009年9月的Android 1.6版本后，针对当时的移动网络做了优化，提供了CDMA网络支持，手势支持、文本转语音系统、虚拟屏幕键盘，VPN、OpenCore2媒体引擎。在Android2.0中更是对原先版本进行优化，支持硬件加速、内置相机闪光灯、支持数码变焦、支持蓝牙2.1，支持动态桌面的设计、同时也支持更多的分辨率、支持Adboe Flash（ps:于是有了AIR技术）、加强软件即时编译的速度。到了Android 2.3，google修复UI、支持近场通讯、强化电源、应用程序管理功能、支持从YAFFSZ转化至ext4文件系统（ps:sd 卡的加载）。到了3.0，我们拥有了平板的时代，在这个时代下，google提供了平板电脑的使用，增加了fragment技术，3D加速处理、加强多任务处理、支持多核心处理器、FAC音视频播放支持、高性能的WIFI锁、对新的厂商硬件支持、增加了应用兼容性功能。

* Android 4.4  之前的变化

######至4.0起，google开始注意到android碎片化的严重性。于是提出了Android的规范。统一了手机和平板电脑的使用习惯，提升了硬件的性能和系统的优化、完善了虚拟按键，界面以标签页的形式展示、内置流量监控、人脸识别技术、提供了随时关闭正在使用的应用程序、增强硬件加速的功能、WIFI直连技术。到android 4.2以后，更是引入360全景照相技术、、改善蓝牙A2DP流问题。在4.3以后，支持多用户登录、蓝牙低功耗、OpenGL ES3.0 、增加纹理支持、多重渲染目标、TRIM指令、多缓冲器对象。

* Android 4.4  之后的变化

######Android 4.4，可以说是一个历史的转折点，按照以往之前，android的虚拟机都是由dalvik。由于dalvik是典型的虚拟机模式，因此每个App创建进程时都要求Dalvik在后台迅速把字节码进行运行时编译。这样运行时编译，导致代码的执行速度比较慢。为了解决这个问题。google在android 4.4引入了新的虚拟机机制，即ART.由于之前多任务处理时，内存消耗过大，因此在4.4以后，google开始对内存进行大量的优化、存储访问框架（ps:SAF,即让用户能够在其所有首选文档存储提供程序中方便地浏览并打开文档、图像及其它文件。 ）。直至2014年6月Google在i/o大会上，提出Android 5.0所有的平台都应遵循新的设计模式-Material Design.随着手机的高速发展，64位处理器，应运而生，google为此提供了64位处理器支持。面向碎片化service的随意使用，尤其是app国内产商用户恶意使用service服务，造成用户的怨气上深，google提出了以系统服务为主的省电优化API（ps:使用JobScheduler来替代service技术，实现后台运作（早期是由asyn adapter执行操作【需要service配合】，国内开发者不会使用，强制要求servie后台进程运作），防止service的恶意使用）。然而在今天，依然有APP产商还在持续的实现这种恶意的操作。（ps:google的意思是可以做，但是要在我google的system server的把控下进行操作。不允许你乱来。相信google为了性能，后面会完全封闭service这条通道。毕竟google已经给你一个技术方案了。）为了防止app产商的恶意操作，google在android 6.0开始做起了防御措施。即运行的权限系统。用户未授权的情况下，不得随意访问操作。删除访问设备的本地硬件标识符。将手机自带的存储空间和SD存储卡空间合并，统一为一个存储空间。通过Doze功能，减少电源的消耗、新增指纹扫描、支持USB Type C接入、强制杀死进程，封闭jni层的进程创建（ps:早期service的守护进程的方式有几种，其中一种方式就是利用c fork进程，来增强进程守护的能力。）

* Android 7.0

######Android 7.0 可以说是对整个Android操作系统应用程序的提升。并且使整个android系统更趋向于简洁，阻止开发商加载非公共的API。继续完善应用授权能力（即系统底层操作加入权限控制）、禁止应用公开file://URI操作。如需操作需要授权。改进action操作、加强JobScheduler能力（ps:7.0以后，JobScheduler可随系统启动时，自动启动。来实现后台任务。）、新增分屏支持、增强通知的能力（可自动回复）、组件的模块化、Vulkan API、密钥认证、APK signature scheme v2的支持、VR技术、强化Doze 的省电功能。

<h3 id="1.2">1.2 总结</h3> 

######从Android的历史演变，我们发现google的android系统在Launcher方面开始将所有的底层进行模块化操作。因此，对android 7.0的源码研究对我们的插件化课程有着更深层次的意义。现在我们来编译android源码。</h6>
<h2 id="2">2.源码编译</h2>
#######本文将介绍如何搭建Android源码开发环境。下面将详细介绍如何下载Android的源码以及如何解决在源码编译过程中所遇到的问题的。

<h3 id="2.1">2.1 环境设置</h3>

* 部署JDK

######Android N的编译依赖于Open JDK 8，所以首先要做的就是下载OpenJDK1.8。在Ubuntu下使用的是open jdk8,而在MAC OSX情况下依然使用的是sun jdk 8 。所以首先要做的就是去Oracle官网下载Sun JDK8版本，下载得到的文件后缀名为.pkg，这时候双击.pkg，会弹出一个应用程序。安装这个应用程序后，系统会默认将jdk安装到/Library/Java/JavaVirtualMachines/目录下。

######1)在根目录下执行"vi ~/.bash_profile"命令，打开.bash_profile文件，在该文件中输入以下内容：

![这里写图片描述](http://img.blog.csdn.net/20161214095952689?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmljaHNqZXNvbnM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#######配置完成后，使用“source ~/.bash_profile”方式，立即生效当前的配置。之后再终端中输入“javac”和"java"命令，来检测当前配置是否生效。

 * 安装Mac Ports
 
######在编译源码之前，首先我们要安装好编译的环境。在Mac OSX情况下，按照google的官方文档。需要安装Mac Ports。

######1)打开 https://distfiles.macports.org/MacPorts/ ，找到对应系统的mac ports（ps：由于笔者用的是OSX EI Captian）版本。点击下载安装mac Ports。
![95849DC8-C1E1-430E-AF16-808120FB4A3B.png](http://upload-images.jianshu.io/upload_images/641423-b951b008cdf46fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

* 安装相关的依赖库

#######执行语句：$POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg

![这里写图片描述](http://img.blog.csdn.net/20161213130137745?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmljaHNqZXNvbnM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 设置环境变量

#######在编译前，请确认你当前的路径在/usr/bin，下，设置环境变量"export PATH=/opt/local/bin:$PATH”。

>richsjesondeMacBook-Pro:~ export  PATH=/opt/local/bin:$PATH <br />
richsjesondeMacBook-Pro:~ source ~/.bashrc_brofile 

* 安装 make 3.81版本

#######由于之前安装的make版本是3.82的，为了让系统支持3.81版本，需要在/opt/local/etc/macports/sources.conf文件下新增一行：file:///Users/Shared/dports。并且创建该文件夹。之后从svn下载并更新mac ports下的make版本。并且通过portindex来创建ports 的索引。

![95849DC8-C1E1-430E-AF16-808120FB4A3B.png](http://upload-images.jianshu.io/upload_images/641423-b951b008cdf46fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

<h3 id="2.2">2.2 下载源码</h3>

######完成以上操作后，开始进行源码的下载。首先我们先从git下载repo脚本。

![这里写图片描述](http://img.blog.csdn.net/20161213140534911?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmljaHNqZXNvbnM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

######由于笔者使用的自己的移动硬盘进行编译，如果大家需要使用移动硬盘编译，请将移动硬盘格式成MAC OSX 日志扩展盘（区分大小写）。
######具体操作如下：打开磁盘工具，点击分区，在分区栏下选择分区的格式为：MAC OSX 日志扩展盘（区分大小写）即可。
######默认的磁盘位置在根目录下的/Volumes文件夹下，在/Volumes文件夹下可以看到当前已mount进来的的磁盘分区，笔者选择了WORKSPACE分区，并且在该分区下创建ANDROID_NOUGAT文件，用于存放android nougat的源码。执行repo init命令下载git文件。

![这里写图片描述](http://img.blog.csdn.net/20161213140457300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmljaHNqZXNvbnM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

######从源码中看到目前仓库中android已经更新到7.0.0_r6版本。我们使用repo init -u 方法来定位到r6版本。这时系统弹出相关提示，询问是否要用颜色的方式来区分内核和普通文件，选择是。

><h6>Your identity is: richsjeson@gmail.com <richsjeson@gmail.com> <br/>
If you want to change this, please re-run 'repo init' with --config-name
Testing colorized output (for 'repo diff', 'repo status'):
  black    red      green    yellow   blue     magenta   cyan     white 
  bold     dim      ul       reverse 
Enable color display in this user account (y/N)? y</h6>

######执行repo init -u https://android.googlesource.com/platform/manifest -b android-7.0.0_r6后，根据提示信息，执行repo sync命令来下载android 7.0源码。

<h3 id="2.3">2.3 源码编译</h3>

#######在经过漫长的时间后，笔者终于把android源码下载下来。在编译前，请确保你的上述环境都搭建完成。执行“ make clobber”检测全局环境是否搭建完成。如果为搭建完成，会给出相应的提示。否则会出现以下的错误:

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

######环境配置好以后，让我们来进行Android的编译操作。首先我们要了解下源码build下的envsetup.sh文件。该文件主要用来设置些Android编译环境。假设如果我在vendor目录下存放一些产商的驱动。那么在执行envsetup.sh时，会初始化该相应的操作。接下来我们来执行一些相关操作：<

><h6>richsjesondeMacBook-Pro:ANDROID_NOUGAT richsjeson$ source build/envsetup.sh <br/></h6>
<h6>系统输出指令:</h6>
><h6>including device/asus/fugu/vendorsetup.sh <br/>
including device/generic/mini-emulator-arm64/vendorsetup.sh <br/>
including device/generic/mini-emulator-armv7-a-neon/vendorsetup.sh <br/>
including device/generic/mini-emulator-mips/vendorsetup.sh <br/>
including device/generic/mini-emulator-mips64/vendorsetup.sh <br/>
including device/generic/mini-emulator-x86/vendorsetup.sh <br/>
including device/generic/mini-emulator-x86_64/vendorsetup.sh <br/>
including device/google/dragon/vendorsetup.sh <br/>
including device/htc/flounder/vendorsetup.sh    <br/>
including device/huawei/angler/vendorsetup.sh <br/>
including device/lge/bullhead/vendorsetup.sh   <br/>
including device/linaro/hikey/vendorsetup.sh   <br/>
including device/moto/shamu/vendorsetup.sh  <br/>
including sdk/bash_completion/adb.bash        <br/>
</h6>

######执行完成后，就可以编译系统了，执行“lunch”命令。

><h6>
You're building on Darwin <br/>
Lunch menu... pick a combo: <br/>
     1. aosp_arm-eng <br/>
     2. aosp_arm64-eng <br/>
     3. aosp_mips-eng <br/>
     4. aosp_mips64-eng <br/>
     5. aosp_x86-eng <br/>
     6. aosp_x86_64-eng <br/>
     7. full_fugu-userdebug <br/>
     8. aosp_fugu-userdebug <br/>
     9. mini_emulator_arm64-userdebug <br/>
     10. m_e_arm-userdebug <br/>
     11. m_e_mips-userdebug <br/>
     12. m_e_mips64-eng <br/>
     13. mini_emulator_x86-userdebug <br/>
     14. mini_emulator_x86_64-userdebug <br/>
     15. aosp_dragon-userdebug <br/>
     16. aosp_dragon-eng <br/>
     17. aosp_flounder-userdebug <br/>
     18. aosp_angler-userdebug <br/>
     19. aosp_bullhead-userdebug <br/>
     20. hikey-userdebug <br/>
     21. aosp_shamu-userdebug <br/>
Which would you like? [aosp_arm-eng] <br/>
</h6>

######不管执行以上哪个操作，默认情况下，编译完成后，都会生成emulator，来针对不同的内核版本做模拟器。nexus6的内核选择的是21。当笔者执行21的时候，发现居然会出现以下错误:

><h6>Which would you like? [aosp_arm-eng] 21 <br/>
-bash: Saving: command not found  <br/>
-bash: ...copying: command not found  <br/>
-bash: ...saving: command not found  <br/>
-bash: ...completed.: command not found  <br/>
-bash: Deleting: command not found  <br/>
** Don't have a product spec for: 'aosp_shamu'  <br/>
** Do you have the right repo manifest?</h6>  

######刚开始的时候，笔者也对这种问题感到困惑，不管是百度还是google都无法解决该问题。后来笔者想到，是不是要通过root权限下去执行envsetup.sh脚本，然后再通过lunch呢。想到这，笔者进行了尝试。首先：使用root权限执行envsetup.sh，执行lunch。这时候系统会报“lunch”错误。于是，笔者再次使用普通用户进行如上操作，终于解决了上述的问题。</h6>

######如果你认为完成上述操作，就可以高枕无忧的进行编译，那你就错了。mac osx 下编译的环境不同ubuntu编译的环境，笔者是通过不断的尝试和研究，总结出一些编译经验。在编译过程中还会遇到以下一个问题：</h6>

><h6>问题1：linux/netfilter/xt_DSCP.h: No such file or directory（没有找到当前文件。）<br/>
原因：mac osx情况下，下载到android源码，缺少了基于linux版本的xt_DSCP.h文件。<br/>
解决：在目录external/iptables/extensions/../include/linux/netfilter中创建文件xt_DSCP.h。</h6>

![这里写图片描述](http://img.blog.csdn.net/20161213125653895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmljaHNqZXNvbnM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

><h6>问题2：No space left on device <br/>
原因：磁盘空间不足，请确保你的磁盘空间满足。如果不满足的话，可以在别的分区下使用。默认情况下源码默认编译的路径是out目录。为了解决该问题，笔者在build/buildspec.mk.default下配置目标输出路径为：/DOC/out(ps:DOC分区下)。buildspec.mk.default是一个envsetup.sh的参数模板，默认情况下该模板的所有指令都是关闭的,这时候就要开启OUT_DIR这个操作，当我们执行envsetup.sh时，系统就会到buildspec.mk.default读取参数。代码如下：<br/>
找到OUT_DIR:语句，输入OUT_DIR:=/Volumes/DOC/NEXUS6</h6>

######经过上述操作后，最终在Mac上编译成功AOSP，并成功运行emulator，接下来就可以在emulator上来进行系统的烧录和源码的改写。在此之前我们要对当前的版本进行创建一个分支，用于防止修改后的操作，在源码更新后代码被覆盖。

<h3 id="2.4">2.4 Repo的使用</h3>

######repo，是google用于管理多个git项目。在repo脚本中包含了repo的配置信息，以及repo所管理的git项目集合。通过repo init命令下载或更新好repo配置和脚本集。一般源码下载完成后，都会生成一个repo目录。该目录下用于提取相应项目的项目集合。通过manifests.git文件，我们可以获取到当前已下载好的所有git版本。执行repo branches可以查看当前所编译的分支。在这里我们主要学习repo一些常用的指令。

><h6>*repo diff：比对提交的代码和当前目录代码的差异 <br/>
>*repo start 分支名称 ：创建并切换分支 <br/>
>*repo checkout -b 分支名称：在现有的分支基础上创建特性分支。 <br/>
>*repo abandon 分支名称：删除指定分支  <br/>
>*repo status ：显示分支及修改的情况</h6>

######为了保证现有的代码质量，我们需要在本地搭建一个代码审核服务器（Gerrit）。将其提交至本地服务器上。
