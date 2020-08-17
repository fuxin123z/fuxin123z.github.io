---
title: Manjaro安装调教
tags:
  - Manjaro安装和配置
categories:
  - 技术
  - Manjaro
keywords:
  - Manjaro
abbrlink: a55f0545
date: 2020-01-31 21:19:52
---

# 在Windows系统上进行分区

进入windows磁盘管理器，分别新建两个简单卷，只设置大小，**不需要格式化**。一个设置为600M大小用于挂载efi,另一个用于挂载根目录/,大小随意。(我是在一块500g的固态上分出一半来装的，需要注意的是如果重新装系统的时候,挂在efi那个分区不能直接格式化删除，不然后面再分成一样大小的还是挂载efi会报错，没搞懂什么问题，一开始分的1g,后面直接格式化了，一直分1g一直报错，就分了600M装，才莫名成功了，本来都想装虚拟机算了。)

**制作Manjaro镜像**

直接官网下的kde镜像，此次使用的是KDE Plasma 18.1.5版本， 然后用刻录工具刻录到u盘上，这次使用的是[usbwrite](http://7dx.pc6.com/wwb5/USBWriter13.zip) ,也可以使用rufus。
<!-- more -->

# Manjaro系统安装
## BIOS的设置关闭掉Secure Boot
关闭掉Secure Boot  
硬盘模式设置为ahci（根据机型貌似有些不用设置）  
引导模式使用uefi（据说双系统win是mbr则manjaro也使用mbr比较好，我的是GPT）

## 进入安装界面进行安装进入U盘之后首先回车选择最后一个是简体中文
1. 进入U盘之后首先回车选择最后一个是简体中文
2. driver  因为是n卡就选择no free，其他情况选free或网上查bios选项等配置
3. 然后回车 Boot:Manjaro.x86_64 kde
4. 进入安装安装界面之后点击launch installr
5. 地区选择Asia 区域选择Shanghai
6. 分区,双系统有两种情况

### mbr启动
这里我们直接划分出来一块分区文件系统选择ext4，挂载点 `/`  

### uefi启动模式
这里需要一个esp分区  
划出600MB分区，文件系统fat16或fat32，挂载点`boot/efi`，标记boot，esp
还需要一个根分区也就是系统分区  
划出分区，文件系统ext4，挂载点 `/` ，无需标记  
可以进行安装了。(/home,/swap都不用管，要是只有8g内存可以考虑配置/swap，挂载点选linuxswap就行)  
安装完后点击重启计算机，在完全关闭后拔出u盘。开机后进入bios选择Manjaro引导到第一位，保存进入系统。

# 系统设置和软件安装
## 系统设置
**进行软件源设置以及系统更新首先，我们先把它打开文件的方式改为双击**
首先，我们先把它打开文件的方式改为双击
`系统设置->桌面行为->工作空间->双击打开文件和文件夹`

更换中国源
```bash
sudo pacman-mirrors -i -c China -m rank
```
在第命令结束的时候会弹出一个窗口让你选择想要使用的源，选最快的那个就行了。(例如:[https://mirrors.ustc.edu.cn/manjaro/](https://mirrors.ustc.edu.cn/manjaro/) 中科大的)
或者直接编辑文件添加源地址:

```bash
sudo nano /etc/pacman.d/mirrorlist
```
地址从这[获取](https://github.com/archlinuxcn/mirrorlist-repo),添加到文件末尾,例如:
```bash
## SJTUG 软件源镜像服务
Server = https://mirrors.sjtug.sjtu.edu.cn/manjaro/stable/$repo/$arch

## 清华大学镜像源
Server = https://mirrors.tuna.tsinghua.edu.cn/manjaro/stable/$repo/$arch
```
**保存设置方法：用nanoc编辑好文本后按crl + x 然后按y再按回车即可。另外manjaro中的pacman相当于centos（或redhat）中的yum命令** 
然后更新一下数据源
```bash
## 更新源列表
sudo pacman-mirrors -g
```
接着更换增加archlinuxcn软件仓库源:
```bash
sudo nano /etc/pacman.conf
```
在最下方添加：
```bash
#添加archlinuxcn软件源：
[archlinuxcn]
SigLevel = Optional TrustedOnly
#SJTUG 软件源
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch
#清华源
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

[antergos]
SigLevel = TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/antergos/$repo/$arch

[arch4edu]
SigLevel = TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch
```
然后运行两条命令:
```bash
sduo pacman -Syy # -Syy表示将本地的软件与软件仓库进行同步
sudo pacman -S archlinuxcn-keyring #-S表示安装某一软件 安装archlinuxcn签名钥匙
sudo pacman -S antergos-keyring
```
之后更新一下系统:
```bash
sudo pacman -Syu #可以更新系统的一切软件包
```
更新完后安装下系统缺少的字体解决中文乱码问题:
```bash
sudo pacman -S wqy-microhei
```

## 软件安装

### 安装AUR管理工具:
想要使用AUR中的软件，一种方式是在图形的软件安装界面的设置中把AUR打开，然后搜索进行安装，另外是使用命令行工具进行安装。由于Yaourt已经不再维护，这里选择了Yay来管理AUR仓库中的软件。
```bash
sudo pacman -S yay
```
Yay默认使用法国的aur.archlinux.org作为AUR源，可以更改为国内清华大学提供的镜像。  
yay 用户执行以下命令修改aururl:  
```bash
$ yay --aururl "https://aur.tuna.tsinghua.edu.cn" --save
```
修改的配置文件位于 ~/.config/yay/config.json  去掉 # AURURL 的注释，修改为
```json
AURURL="https://aur.tuna.tsinghua.edu.cn"
```
**yay 安装命令不需要加 sudo**,yay的命令参数跟pacman参数基本一致,还可通过以下命令查看修改过的配置:
```bash
$ yay -P -g
```
yay 的常用命令：
```bash
$ yay -S package # 从 AUR 安装软件包
$ yay -Rns package # 删除包
$ yay -Syu # 升级所有已安装的包
$ yay -Ps # 打印系统统计信息
$ yay -Qi package # 检查安装的版本
```

### 安装中文字体:
```bash
sudo pacman -S wqy-zenhei
sudo pacman -S wqy-bitmapfont
sudo pacman -S wqy-microhei
#Manjaro自带了思源系列字体（Noto家族）,补个Emoji：
yay -S noto-fonts-emoji
sudo pacman -S adobe-source-han-sans-cn-fonts
sudo pacman -S adobe-source-han-serif-cn-fonts
```

#### linux字体推荐(可选)
中文字体推荐使用：文泉驿、思源字体。安装如下：
```bash
$ sudo pacman -S wqy-microhei wqy-bitmapfont wqy-zenhei wqy-microhei-lite
$ sudo pacman -S adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
```

西文字体推荐使用dejavu、noto字体。
```bash
$ sudo pacman -S ttf-dejavu
$ sudo pacman -S noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk
```

#### 优化-->字体-->缩放字体-->1.25
字体我用的是思源黑体（adobe家开源的），文泉驿和google家的noto系列也不错。
noto系列cjk指的是中国韩国和日本，中国大陆好像是S开头的字体，因为太大了就没有安装。
chrome等其他浏览器可以在浏览器设置里改一下字体风格，然后在/etc/fonts/新建一个local.conf文件写入：
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <match target="font">
        <edit name="autohint" mode="assign">
            <bool>false</bool>
            </edit>
        <edit name="hinting" mode="assign">
            <bool>false</bool>
        </edit>
        <edit name="hintstyle" mode="assign">
            <const>none</const>
        </edit>
    </match>
</fontconfig>
```
这样软件默认就不会用自己的文字处理方式了，和win下字体基本无太大差别，字体模糊问题解决。



### 中文输入法安装:
fcitx和ibus都可以配置中文输入法
fcitx 或 ibus 两个选其一 推荐fcitx(一开始装了ibus，后面转了fcitx)

#### fcitx5对比fcitx4

- fcitx5的输入体验的确比上个版本好一些，输入相对来说比较流畅，**输入中文分号不需要空格，而是按两下分号键**。
- 自动DPI功能还需要优化。

#### fcitx4安装
~~1. 安装安装输入法模块:~~
~~```bash~~
~~sudo pacman -S fcitx-im #安装全部输入法模块~~

~~#终端出现以下提示:~~
~~$ yay -S fcitx-im~~
~~:: There are 4 members in group fcitx-im:~~
~~:: Repository community~~
   ~~1) fcitx  2) fcitx-gtk2  3) fcitx-gtk3  4) fcitx-qt5~~

~~Enter a selection (default=all):~~
~~#直接按回车，默认4个都安装，不然后面在有些应用或者终端调不出输入法~~
~~```~~

2020-05-06更新：  
1. 现在网络上大部分老教程还是安装fcitx,fcitx-gtk2,fcitx-gtk3,fcitx-qt4,fcitx-qt5,fcitx-configtool的传统方法，目前在Archlinux的源中，fcitx-im包组已经取消了fcitx-qt4包，但是搜狗输入法Linux版是基于QT4的，少了这个包，搜狗输入法就无法正常启动。这里安装也是担心后面有些应用或者终端不支持QT5调不出输入法。  
首先，如果以前安装了fcitx，全部删除：
```bash
$ sudo pacman -Rsn fcitx-im fcitx-configtool
```

2. 然后安装fcitx-lilydjwg-git这个包，fcitx-lilydjwg-git这个包用来替换fcitx-im包组中除fcitx-qt5之外的所有包，其中包含了fcitx-qt4包。
```bash
sudo pacman -S fcitx-lilydjwg-git
```

3. 安装中文输入法选其一（我选的sunpinyin，rime和goolepinyin据说不支持模糊音）
```bash
sudo pacman -S fcitx-sunpinyin
sudo pacman -S fcitx-rime
sudo pacman -S fcitx-libpinyin
sudo pacman -S fcitx-googlepinyin
```
4. 安装云拼音（可选）
```bash
$ sudo pacman -S fcitx-cloudyinpin
```
安装fcitx-cloudpinyin后，googlepinyin，fcitx自带的pinyin，sunpinyin的候选次列表都会具有云辅助，更加智能。（rime不支持）

> 安装后重启 Fcitx 即可，所选的云拼音输入结果会自动添加到当前输入法的词库中。提醒：建议在fcitx设置里面将“云拼音来源”由Google改为“百度”，Google国内访问不是很顺畅。  
启用云拼音后，从云拼音获得的候选词会默认添加到候选词列表中的第二个，显示位置可以通过云拼音的设置配置。如果云拼音的结果和本地输入法给出的结果一致，云拼音后选项会和本地产生的候选项自动合并，不会产生重复的候选项。  
若安装fcitx-cloudpinyin后，在配置程序里却没有看见云拼音，记得勾上“高级”复选框。这时云拼音会显示出来，再勾上云拼音。  
注意: 不推荐将云拼音候选词设为第一个候选词，因为当网络情况不好，没有及时返回云拼音结果，那么云拼音结果将默认降到第二候选词的位置，于是这个过程可能会涉及到默认候选词的改变。

5. 到这里，Manjaro下的中文输入法基本就安装好了，但是还没有安装fcitx图形设置界面，另外少了个fcitx-qt5包，不能在qt5程序下输入，需要继续安装：  
GNOME（GTK）用户：
```bash
$ sudo pacman -S fcitx-configtool fcitx-qt5
```
注意：fcitx-qt5可选依赖于fcitx-configtool，可以直接安装fcitx-configtool，然后选择fcitx-qt5。  
KDE（QT）用户：
```bash
$ sudo pacman -S kcm-fcitx
```

6. 在/etc/profile或者~/.xprofile文件中，右键粘贴如下代码并保存:
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

7. **注销重新登录或者重启系统。**

8. 扩展点-检查设置
如果出现问题，可使用下面打命令，看下哪里配置异常，处理一下就可以了。
```bash
$ fcitx-diagnose
```

#### fcitx5安装
在一些大佬的博客上看到Fcitx5，试了下发现自带的pinyin准确度确实很高，而且输入设置也整合到了设置中心，对KDE用户相当友好。fcitx的官方Repo在[GitHub](https://github.com/fcitx/fcitx5)  
1. 安装
```bash
#卸载fcitx4
pacman -Rs $(pacman -Qsq fcitx) 

# 可以直接全部安装不用考虑安装顺序(archlinuxcn源)
$ sudo pacman -S fcitx5-git # 安装fcitx5框架
$ sudo pacman -S fcitx5-gtk-git # 增加对gtk程序的支持
$ sudo pacman -S fcitx5-qt5-git # 增加对QT5程序的支持
$ sudo pacman -S fcitx5-qt4-git  # 有QT4程序需要增加对QT4程序的支持
$ sudo pacman -S fcitx5-chinese-addons-git # 增加对中文输入的组件
$ sudo pacman -S kcm-fcitx5-git # 增加内嵌的输入设置，只限KDE

# 也可以直接安装community源下的包，这里我配的是清华的镜像源，貌似没有同步好，找不到文件，所以用了上面的archlinuxcn源安装的。
$ sudo pacman -S fcitx5 fcitx5-gtk fcitx5-qt fcitx5-chinese-addons kcm-fcitx5
```

2. 配置开机自启
系统设置->开机和关机->自动启动->添加程序->搜索Fcitx 选中后确定使Fcitx5自启动（推荐）。  
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_fcitx_autostart.webp)

3. 修改配置文件
直接启动fcitx5是只有西文键盘的，如果是KDE，可以到系统的输入法配置启用拼音；如果是其他发行版，就需要使用配置文件。
首先要确认当前fcitx5是完全退出的状态，如果fcitx5在运行，修改后软件会自动覆写profile配置文件。用户配置在`~/.config/fcitx5/profile`
```
[Groups/0]
# Group Name
Name=默认
# Layout
Default Layout=us
# Default Input Method
DefaultIM=pinyin

[Groups/0/Items/0]
# Name
Name=keyboard-us
# Layout
Layout=

[Groups/0/Items/1]
# Name
Name=pinyin
# Layout
Layout=

[GroupOrder]
0=默认
```
使用默认的pinyin就可以了。

4. 设置环境变量
使用`echo ${XDG_SESSION_TYPE}`命令查看，如果是`x11`即`xorg`用户，则还应当在`~/.xprofile`添加如下内容：
```
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS="@im=fcitx5"
```
如果是`wayland`用户的话，则修改`~/.pam_environment`文件
```
GTK_IM_MODULE=fcitx5
QT_IM_MODULE=fcitx5
XMODIFIERS=@im=fcitx5
```

5. 注销重新登录或者重启系统就可以看到fcitx5的启动了。

6. 个性化设置  
**注意修改配置文件要先彻底关闭fcitx5**  
1. 关闭云拼音
如果注重个人隐私的话，建议关闭该选项。修改`~/.config/fcitx5/conf/pinyin.conf`：
```diff
# Enable Cloud Pinyin
-CloudPinyinEnabled=True
+CloudPinyinEnabled=False
```

2. 关闭自动API  
fcitx5会自动根据多显示器不同的DPI来调整界面大小，但发现经常达不到想要的效果，只好将这一功能关闭，并调整字体大小为14。修改`~/.config/fcitx5/conf/classicui.conf`
```
# 按屏幕 DPI 使用
PerScreenDPI=False

# Font (设置成你喜欢的字体)
Font="Noto Sans Mono Regular 14"
```

3. 使用皮肤
使用了[hosxy的Material-color皮肤](https://github.com/hosxy/Fcitx5-Material-Color)，安装非常方便，依照Readme操作即可。效果惊艳，堪比Windows10自带输入法。  
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_material_color_teal.webp)

直接运行命令安装：
```bash
$ pacman -S fcitx5-material-color
```

4. 使用单行模式(inline_preedit)
对于fcitx5自带pinyin 请修改 `~/.config/fcitx5/conf/pinyin.conf`
```diff
# 可用时在应用程序中显示预编辑文本
-PreeditInApplicaation=False
+PreeditInApplicaation=True
```




#### ibus安装
```bash
sudo pacman -S ibus #安装ibus软件包
sudo pacman -S ibus-qt
sudo pacman -Ss ^ibus-* #查看所有可用的输入法
```
选择一个可用的输入法引擎并安装：
```bash
sudo pacman -S ibus-rime
sudo pacman -S ibus--pinyin
sudo pacman -S ibus-googlepinyin
sudo pacman -S libpinyin
```
```bash
ibus-setup #运行ibus
```
出现提示:![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_ibus.webp)
在`$HOME/.bashrc`中加入下面这段就好了
```bash
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
```
这样就可以使用该输入法了，但是每次开机都要在命令行中输入ibus-setup才能启动ibus，太麻烦了点。所以把原来的$HOME/.bashrc的内容转移到`$HOME/.xprofile`中，并且在最后一行添加一条新的内容:
```bash
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -x -d
```
**再重启发现输入法能开机自动启动了**

### 更改项目文件英文名
修改目录映射文件名:
```bash
sudo nano .config/user-dirs.dirs
```
修改为以下内容:
```bash
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Download"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```
将Home目录下的中文目录名改为对应的英文名:
```bash
mv 桌面 to Desktop
mv 下载 to Downloads
mv 模板 to Templates
mv 公共 to Public
mv 文档 to Documents
mv 音乐 to Music
mv 图片 to Pictures
mv 视频 to Videos
```
**重启系统**

### 常用软件安装

#### 支持复杂运算的计算器

```bash
$ sudo pacman -S speedcrunch
```
有些运算我们需注意单位问题，如角度单位（弧度、度数），默认它是弧度，所以你运算sin(30)不会等于0.5，在设置中或使用快捷键切换即可。
建议勾选 设置 - 行为 - Enable Complex Numbers，不然你可能连pi（圆周率表示）都无法使用。

#### 解压工具安装
```bash
sudo pacman -S unrar unzip p7zip
sudo pacman -S file-roller #图形化的解压软件
```

#### 为知笔记
```bash
sudo pacman -S wiznote
```

#### 思维导图
```bash
yay -S xmind-zen
```

#### WPS及WPS需要的中文字体
```bash
sudo pacman -S wps-office
yay -S wps-office-fonts wps-office-mime ttf-wps-fonts
```

#### 福昕PDF阅读器
```bash
sudo pacman -S foxitreader
```

#### 媒体播放器

```bash
$ sudo pacman -S vlc #一个免费且开源的跨平台媒体播放器以及框架，可以播放大多数格式的多媒体文件以及 DVS，音频 CD，VCD，以及各种流媒体协议
$ sudo pacman -S ffmpeg

$ sudo pacman -S kodi # 一款获得殊荣的免费开源（GPL）软件，支持视频，音乐，图片，游戏以及更多内容的媒体中心。
```

#### 跨平台音乐播放器listen1
```bash
$ yay -S listen1-desktop
```

#### TeamViwer安装
直接运行命令安装的包打开并不能用，需要到[官网](https://www.teamviewer.com/en/download/linux/)下载包解压运行。
选择 Other systems (not officially supported)的[↓ x86 64bit](https://download.teamviewer.com/download/linux/teamviewer_amd64.tar.xz)
下载到的是：`teamviewer_15.3.2682_amd64.tar.xz`

##### 解压
两种解压方式：
- 右键鼠标选择解压
- 命令行解压`tar xvf teamviewer_15.3.2682_amd64.tar.xz`

##### 运行Teamviwer
把解压出来的目录放到你想的任意目录，可以放在`/opt/`下，`cd teamviewer`进入目录里，运行`./teamviewer`

##### 添加到Dash to Dock
修改安装目录下的teamviewer.desktop,修改图标文件路径
```
Icon=/opt/teamviewer/tv_bin/desktop/teamviewer_256.png
```
然后执行
```bash
$ sudo cp /opt/teamviewer/teamviewer.desktop /usr/share/applications/
```

#### ClamAV安装
```bash
sudo pacman -S clamav #Clam 防病毒软件（命令行）
sudo pacman -S clamtk #Clam 防病毒软件（客户端）
```

#### BleachBit安装
```bash
$ sudo pacman -S bleachbit #快速释放磁盘空间并不知疲倦地守卫你的隐私。释放缓存，删除 cookie，清除互联网浏览历史，清理临时文件，删除日志，以及更多功能...
```

#### 截图软件

##### 深度截图
```bash
$ sudo pacman -S deepin-screenshot 
```

##### flameshot
###### flameshot安装
```bash
$ sudo pacman -S flameshot-git
```

###### 配置flameshot截图快捷键
用KDE的桌面环境，可以配置下快捷键：系统设置—->快捷键—->自定义快捷键：
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_flameshot_keybord_00.webp)
编辑—->新建—>全局快捷键—>命令/URL:
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_flameshot_keybord_01.webp)

![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_flameshot_keybord_02.webp)
动作填入为 flameshot gui

###### 配置flameshot守护进程
如果想要配置一些截图选项。执行以下命令：
```bash
$ flameshot config
```
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_flameshot_config_00.webp)

![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_flameshot_config_01.webp)
要配置下开机启动，因为flameshot是有前后端的，守护进程要有运行才能正常截图！

#### 壁纸自动定时切换软件—Variety
```bash
$ sudo pacman -S variety
```

#### 系统状态监控

```bash
sudo pacman -S deepin-system-monitor
```

#### 硬件温度监测

```bash
sudo pacman -S psensor
```

#### 录屏软件
```bash
sudo pacman -S deepin-screen-recorder #录屏软件，可以录制 Gif 或者 MP4 格式
sudo pacman -Ss SimpleScreenRecorder #另外一个更强大的录屏软件
```

#### deepin系列软件必备条件
首先对于非 GNOME 桌面(KDE, XFCE等)需要安装
```bash
sudo pacman -S gnome-settings-daemon
```
运行/usr/lib/gsd-xsettings
系统设置->开机或关机->自动启动->添加脚本->输入/usr/lib/gsd-xsettings

#### 深度QQ
```bash
sudo pacman -S deepin.com.qq.im
```

#### 深度微信
微信是自己从github上下的旧版的包2.7.188版本,并根据文档修改对应的安装配置    
[https://github.com/countstarlight/deepin-wine-wechat-arch](https://github.com/countstarlight/deepin-wine-wechat-arch)
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_deepin_wine_wechat.webp)
安装完后手动切换deepin-wine环境

1. 安装 deepin-wine
```bash
yay -S deepin-wine
```
2. 修改 deepin-wine-wechat 的启动文件
修改如下两个文件中的 WINE_CMD 的值：  
/opt/deepinwine/apps/Deepin-WeChat/run.sh  
/opt/deepinwine/tools/run.sh
```diff
-WINE_CMD="wine"
+WINE_CMD="deepin-wine"
```
3. 对于非 GNOME 桌面(KDE, XFCE等)需要安装 
```bash
sudo pacman Sy gnome-settings-daemon
```
并在 /opt/deepinwine/apps/Deepin-WeChat/run.sh 中加入如下几行：
```diff
 RunApp()
    {
+    if [[ -z "$(ps -e | grep -o gsd-xsettings)" ]]
+    then
+        /usr/lib/gsd-xsettings &
+    fi
        if [ -d "$WINEPREFIX" ]; then
                UpdateApp
        else
```
注意：对 /opt/deepinwine/apps/Deepin-WeChat/run.sh 的修改会在 deepin-wine-wechat 更新或重装时被覆盖，可以单独拷贝一份作为启动脚本
4. 删除原先的微信目录
```bash
rm -rf ~/.deepinwine/Deepin-WeChat
```
5. 修复 deepin-wine 字体渲染发虚
```bash
yay -S lib32-freetype2-infinality-ultimate
```
注意：切换到 deepin-wine 后，对 wine 的修改，如更改dpi，都改为对 deepin-wine 的修改

#### 百度网盘
```bash
yay -S deepin-wine-baidupan
```

#### 迅雷
```bash
yay -S deepin.com.thunderspeed
```

#### Aria2
下载器，支持 HTTP(S)，FTP，SFTP，BitTorrent和Metalink 协议

##### 安装
```bash
sudo pacman -S aria2
```

##### 配置
配置文件
```bash
cd ~
mkdir aria2
cd aria2
touch aria2.conf
touch aria2.session
touch aria2.log
```
aria2.conf配置文件内容，aria2.session是用来存地址的。
```bash
## 文件保存相关 ##
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/home/johnnychan/Downloads
# 日志文件
log=/home/johnnychan/aria2/aria2.log
# 日志级别
log-level=warn
# 检查完整性，默认true
check-integrity=true
# 启用磁盘缓存
disk-cache=32M
# 断点续传
continue=true
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
file-allocation=trunc

## 进度保存相关 ##
# 从会话文件中读取下载任务
input-file=/home/johnnychan/aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/home/johnnychan/aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60


# 下载设置
# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=10
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=10
# 单个任务最大线程数, 添加时可指定, 默认:5
split=5
# 整体下载速度限制, 运行时可修改, 默认:0
max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=500KB
# 单个任务上传速度限制, 默认:0
max-upload-limit=0
# 禁用IPv6, 默认:false
disable-ipv6=true
# 最大重试次数, 设置为0表示不限制重试次数, 默认:5
max-tries=5
# 设置重试等待的秒数, 默认:0
retry-wait=0
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M


# BitTorrent/Metalink 设置
# BitTorrent/Metalink 设置
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# 本地节点查找, PT需要禁用, 默认:true
bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 种子最大打开文件数
bt-max-open-files=16
# 单个种子最大连接数, 默认:55
bt-max-peers=55
# DHT文件路径
dht-file-path=/home/johnnychan/aria2/dht.dat
dht-file-path6=/home/johnnychan/aria2/dht6.dat
# DHT文件监听端口，默认:6881-6999
dht-listen-port=6881-6999
# 打开IPv6 DHT功能, PT需要禁用，默认false
#enable-dht6=false
# BT监听端口，当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 种子分享率，默认1.0
seed-ratio=1.0
# 种子分享时间，默认60分
seed-time=60
# 额外Tracker服务器
bt-tracker=udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.open-internet.nl:6969/announce,udp://tracker.skyts.net:6969/announce,udp://tracker.piratepublic.com:1337/announce,udp://tracker.opentrackr.org:1337/announce,udp://9.rarbg.to:2710/announce,udp://retracker.coltel.ru:2710/announce,udp://pubt.in:2710/announce,udp://public.popcorn-tracker.org:6969/announce,udp://wambo.club:1337/announce,udp://tracker4.itzmx.com:2710/announce,udp://tracker1.wasabii.com.tw:6969/announce,udp://tracker.zer0day.to:1337/announce,udp://tracker.xku.tv:6969/announce,udp://tracker.vanitycore.co:6969/announce,udp://ipv4.tracker.harry.lu:80/announce,udp://inferno.demonoid.pw:3418/announce,udp://open.facedatabg.net:6969/announce,udp://mgtracker.org:6969/announce,udp://tracker.mg64.net:6969/announce,udp://tracker.tiny-vps.com:6969/announce,udp://tracker.internetwarriors.net:1337/announce,udp://tracker.grepler.com:6969/announce,udp://tracker.files.fm:6969/announce,udp://tracker.dler.org:6969/announce,udp://tracker.desu.sh:6969/announce,udp://tracker.cypherpunks.ru:6969/announce,udp://p4p.arenabg.com:1337/announce,udp://open.stealth.si:80/announce,udp://explodie.org:6969/announce,udp://bt.xxx-tracker.com:2710/announce,http://tracker.city9x.com:2710/announce,udp://tracker.tvunderground.org.ru:3218/announce,udp://tracker.torrent.eu.org:451/announce,udp://t.agx.co:61655/announce,udp://sd-95.allfon.net:2710/announce,udp://santost12.xyz:6969/announce,udp://sandrotracker.biz:1337/announce,udp://retracker.nts.su:2710/announce,udp://retracker.lanta-net.ru:2710/announce,http://retracker.mgts.by:80/announce,udp://tracker.uw0.xyz:6969/announce,http://tracker.skyts.net:6969/announce,udp://torr.ws:2710/announce,udp://thetracker.org:80/announce,http://retracker.telecom.by:80/announce,http://0d.kebhana.mx:443/announce,wss://tracker.openwebtorrent.com:443/announce,wss://tracker.fastcast.nz:443/announce,wss://tracker.btorrent.xyz:443/announce,ws://tracker.btsync.cf:2710/announce,udp://zephir.monocul.us:6969/announce,udp://tracker.martlet.tk:6969/announce,udp://tracker.kamigami.org:2710/announce,udp://tracker.cyberia.is:6969/announce,udp://tracker.bluefrog.pw:2710/announce,udp://tracker.acg.gg:2710/announce,udp://peerfect.org:6969/announce,https://evening-badlands-6215.herokuapp.com:443/announce,udp://z.crazyhd.com:2710/announce,udp://tracker.swateam.org.uk:2710/announce,udp://tracker.justseed.it:1337/announce,udp://packages.crunchbangplusplus.org:6969/announce,udp://104.238.198.186:8000/announce,https://open.kickasstracker.com:443/announce,http://tracker2.itzmx.com:6961/announce,http://tracker.vanitycore.co:6969/announce,http://tracker.torrentyorg.pl:80/announce,http://tracker.tfile.me:80/announce,http://tracker.mg64.net:6881/announce,http://tracker.electro-torrent.pl:80/announce,http://t.nyaatracker.com:80/announce,http://share.camoe.cn:8080/announce,http://servandroidkino.ru:80/announce,http://open.kickasstracker.com:80/announce,http://open.acgtracker.com:1096/announce,http://open.acgnxtracker.com:80/announce,http://mgtracker.org:6969/announce,http://fxtt.ru:80/announce,http://bt.dl1234.com:80/announce,http://agusiq-torrents.pl:6969/announce,http://104.238.198.186:8000/announce
# 客户端伪装
peer-agent=uTorrent/2210(25130)
peer-id-prefix=-UT2210-
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true
#下载完成后删除.ara2的同名文件
on-download-complete=/home/johnnychan/aria2/delete_aria2

# RPC 设置
# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# RPC证书
#rpc-certificate=/opt/var/aria2/aria2.pfx
# 允许非外部访问, 默认:false
rpc-listen-all=true
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret=faketoken
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
#rpc-secure=true

# 高级设置
# 进程守护
daemon=true
#enable-mmap=true
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
force-save=true
# 整体下载速度限制, 运行时可修改, 默认:0
max-overall-download-limit=0
```

##### 配置后台守护运行：
```bash
sudo vim /etc/systemd/user/aria2.service
```
```bash
[Unit]
Description=Aria2c download manager
After=network.target

[Service]
User=<你的用户名>
Type=forking
ExecStart=/usr/bin/aria2c --conf-path=/home/johnnychan/aria2/aria2.conf -D

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable aria2
sudo systemctl start aria2
```

#### uGet

强大的下载软件，比迅雷更强，推荐搭配 aria2 使用
uGet 使用 aria2：编辑 > 设置 > 插件 > 插件匹配顺序：aria2; URI:http://localhost:6800/jsonrpc
参数：–enable-rpc=true -D –disable-ipv6 –check-certificate=false
chrome,firefox启用 uget 下载
```bash
sudo pacman -S uget-intedrator
sudo pacman -S uget-intedrator-chrome
sudo pacman -S uget-intedrator-firefox
```

#### 蓝牙使用
安装软件包
```bash
sudo pacman -S bluez bluez-utils pulseaudio-bluetooth pulseaudio-alsa #安装蓝牙模块需要的软件
```
确保未禁用蓝牙
```bash
rfkill #查看TYPE类型是bluetooth的是否是unblocked
rfkill unblock 0 #取消阻止
```
启动蓝牙服务
```bash
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```
打开`系统设置->蓝牙>适配器`看是否能识别适配器,如果不能识别则需要更换蓝牙适配器。能识别的话再继续下一步，可以直接系统设置中点击连接，也可以终端启动bluetoothctl交互命令。  
使用bluetoothctl连接到蓝牙设备：
```bash
bluetoothctl
[bluetooth]#power on #打开控制器电源，默认是关闭的。
[bluetooth]#devices #获取要配对设备的 MAC 地址
[bluetooth]#scan on #进行扫描以检测你的蓝牙设备
[bluetooth]#pair $MAC #开始配对,MAC地址(支持 tab 键补全)
[bluetooth]#trust $MAC #再次连接可能需要手工认，所以需要这句命令
[bluetooth]#connect $MAC #连接到设备
[JBL Reflect Mini2]# info #以下是输出信息
Device F8:DF:15:40:E3:C3 (public)
        Name: JBL Reflect Mini2
        Alias: JBL Reflect Mini2
        Class: 0x00240404
        Icon: audio-card
        Paired: yes
        Trusted: yes
        Blocked: no
        Connected: yes
        LegacyPairing: no
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        RSSI: -55
```

#### 系统信息查看软件(类似AIDA64)  
```bash
$ yay -S hardinfo
```

#### SSD优化配置

##### 开启Trim功能
SSD TRIM是一个高级技术附件(ATA)命令，它使操作系统能够通知NAND闪存固态硬盘(SSD)哪些数据块可以删除，因为它们已经不再使用了。使用TRIM可以提高向SSD写入数据的性能，延长SSD的使用寿命。可以参考[ArchWiki](https://wiki.archlinux.org/index.php/Solid_state_drive#TRIM),
在使用Trim功能之前需要查看固态硬盘是否支持，否则可能造成数据丢失:
```bash
$ lsblk --discard
```
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_manjaro_ssd_io_check.webp)
`DISC-GRAN`和`DISC-MAX`关于使用的Trim方式，Nvme 协议固态是不推荐使用的`ContinuousTRIM`方式的。(详见[ArchWiki](https://wiki.archlinux.org/index.php/Solid_state_drive/NVMe#Discards))
所以使用的定期执行fstrim的方式，即添加一个定时任务或服务让其自动执行，如每周执行一次trim操作。 参考[PeriodicTRIM](https://wiki.archlinux.org/index.php/Solid_state_drive#Periodic_TRIM)
```bash
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer #开启fstrim
```
启用fstrim.timer服务则会自动每周运行一次fstrim.service去进行trim,不用手动调用。

##### IO调度器选择
一般来说，IO调度算法是为低速硬盘准备的，对于固态，最好是不使用任何IO调度器，或使用对硬盘干预程度最低的调度算法。这里可以照搬官方的[配置](https://wiki.archlinux.org/index.php/Improving_performance#Storage_devices)
```bash :/etc/udev/rules.d/60-schedulers.rules
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"
# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
```
然后重启电脑永久生效，再查看当前固态的IO调度器:
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_manajro_ssd_io.webp)
可以看到我当前NVME盘没有使用任何调度器，SATA固态使用的是deadline，而机械硬盘使用的是bfq。

#### 提取图片文字Tesseract
##### Tesseract概述
Tesseract 是一款被广泛使用的开源 OCR 工具,Tesseract(/'tesərækt/) 这个词的意思是"超立方体"，指的是几何学里的四维标准方体，又称"正八胞体"。。不过这里要讲的，是一款以其命名的开源 OCR(Optical Character Recognition, 光学字符识别) 软件。
所谓 OCR 是图像识别领域中的一个子领域，该领域专注于对图片中的文字信息进行识别并转换成能被常规文本编辑器编辑的文本。
Tesseract 已经有 30 年历史，开始它是惠普实验室的一款专利软件，然后在 2005 年开源，自 2006 年后由 Google 赞助进行后续的开发和维护。
在 1995 年 Tesseract 曾是世界前三的 OCR 引擎，而且在现在的免费 OCR 引擎中，其识别精度也仍然是出类拔萃的。因为其免费与较好的效果，许多的个人开发者以及一些较小的团队在使用着 Tesseract ，诸如验证码识别、车牌号识别等应用中，不难见到 Tesseract 的身影。
[Tesseract](https://github.com/tesseract-ocr/tesseract)目前已作为开源项目发布在GitHub上，其最新版本已经支持中文OCR，并提供了一个命令行工具。

##### Tesseract安装
```bash
$ sudo pacman -S tesseract

$ tesseract -v                     
tesseract 4.1.1
 leptonica-1.79.0
  libgif 5.2.1 : libjpeg 8d (libjpeg-turbo 2.0.4) : libpng 1.6.37 : libtiff 4.1.0 : zlib 1.2.11 : libwebp 1.1.0
 Found AVX2
 Found AVX
 Found FMA
 Found SSE
 Found libarchive 3.4.2 zlib/1.2.11 liblzma/5.2.4 bz2lib/1.0.8 liblz4/1.9.2 libzstd/1.4.4
```

##### 安装语言文件
```bash
$ sudo pacman -S tesseract-data-chi_sim #简体中文
$ sudo pacman -S tesseract-data-chi_tra #繁体中文
$ sudo pacman -S tesseract-data-eng #英文

$ tesseract --list-langs #查看有哪些可用的语言   
List of available languages (3):
chi_sim
chi_tra
eng
osd
```
这里的 "eng" 和 "chi_sim"，"chi_tra"是 Tesseract 提供的英文，简体中文和繁体中文的语言文件，而其他的则是训练得到的。
另外要说明的是，这里的 "语言文件" 的本质是包含了某种 "自然语言" 的文字的特征等辅助识别的一些资源，但像 chi_sim 这个中文简体里也包含了英文字母与阿拉伯数字的资源。而我们也可以为了特定的用途而去训练产生对应的资源。

##### 命令行测试
假设我们有一张名为 paper.png 的图片，则通过以下命令对图片进行识别
```bash
$ tesseract paper.png result -l chi_sim
```
- 第一个参数是待识别的图像的文件名
- 第二个参数用于指定输出，如果希望直接输出而不是保存到文件，那么就使用 stdout，否则这个参数将会作为保存结果的文件的前缀
- -l chi_sim 这个应该很好理解，就是用来指定使用哪个 "语言文件"，如果是使用 英文(eng) ，这个参数可以不加，因为默认就是使用英文的 "语言文件" 来进行识别
以上命令如不出错，结果将会保存到 result.txt 这个文本文件中。
此外 Tesseract 还提供非常丰富的可选参数来对识别过程进行调整，可用的参数及其默认值可以通过以下命令进行查看:
```bash
$ tesseract --print-parameters
```
参数的使用有两种:
- 使用 -c 选项来设定单项参数的值，比如:
```bash
$ tesseract paper.png result -l chi_sim -c language_model_ngram_on=1
```
允许使用多个 -c 选项来设置多个参数的值。

- 将多项参数设置写入文件，然后在识别时使用该文件，比如:
```bash
$ tesseract paper.png result -l chi_sim tess.conf
```
需要注意的是，如果使用配置文件，用作参数的配置文件名要放在最后面——这里也支持多个配置文件，但它们必须要在最后面。假如我有两个配置文件 tess_1.conf 和 tess_2.conf，那么这样是正确的:
```bash
$ tesseract paper.png paper -l chi_sim tess_1.conf tess_2.conf
```
而这样则是错误的:
```bash
$ tesseract paper.png paper tess_1.conf -l chi_sim tess_2.conf
```
至于 Tesseract 其他那些参数各有什么含义，官方没有提供任何文档来进行解释，这里有一个[链接](http://www.sk-spell.sk.cx/tesseract-ocr-parameters-in-302-version)提供了部分参数的用处说明，应该是阅读了 Tesseract 源代码后得到的结论。

##### Tesseract可视化界面
根据不同的GUI安装相应支持的Tesseract软件
```bash
$ sudo pacman -S gimagereader-qt #QT
$ sudo pacman -S gimagereader-gtk #GTK
```

### 开发软件
#### 网络工具
```bash
$ sudo pacman -S wireshark-qt  # 抓包工具

$ sudo pacman -S bind-tools #Dig（Domain Information Groper）是一个功能强大的命令行工具，用于查询DNS名称服务器
```

#### jdk
手动安装oracle-jdk，可选择低版本,下载tar包
解压
```bash
tar -zxvf xxx.tar.gz
```
移动到 `/usr/src`目录下
```bash
sudo mv xxx /usr/src/
```
先试一下前面这个，不行再加上后面那个
```bash
vim ~/.bashrc
```
在后面加上， 地址根据jdk修改
```bash
export JAVA_HOME=/home/johnnychan/Public/programs/java/jdk8/jdk1.8.0_241
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
启用配置
```bash
source .bashrc
```
查看是否配置成功
```bash
java -version
```
显示
```bash
java version "1.8.0_241"
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
```
前面那个不行在加上底下这个配置jdk环境变量
修改配置文件`/etc/profile`  
setting for jdk-oracle
```bash
JAVA_HOME=/home/johnnychan/Public/programs/java/jdk8/jdk1.8.0_241
CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
```
启用配置
```bash
source /etc/profile
```

#### Tomcat安裝配置
到[Tomcat官网](https://tomcat.apache.org/)选择相应版本的tar.gz包下载

解压并移动到opt目录下
```bash
$ tar -zxvf apache-tomcat-8.5.53.tar.gz
sudo cp -r apache-tomcat-8.5.53 /opt/
```
发现没有读写查权限，于是开放tomcat及其子目录的权限，-R表示启动递归处理
```bash
$ sudo chmod -R 755 /opt/apache-tomcat-8.5.53
```
启动Tomcat
```bash
sudo /opt/apache-tomcat-8.5.53/bin/startup.sh
```
会看到控制台有以下输出：
```bash
Using CATALINA_BASE:   /opt/apache-tomcat-8.5.53
Using CATALINA_HOME:   /opt/apache-tomcat-8.5.53
Using CATALINA_TMPDIR: /opt/apache-tomcat-8.5.53/temp
Using JRE_HOME:        /home/johnnychan/Public/programs/java/jdk8/jdk1.8.0_241/jre
Using CLASSPATH:       /opt/apache-tomcat-8.5.53/bin/bootstrap.jar:/opt/apache-tomcat-8.5.53/bin/tomcat-juli.jar
Tomcat started.
```
在浏览器地址栏输入http://localhost:8080/ 出现了那幅经典的猫图,成功。

Tomcat 默认运行在8080端口
```bash
$ sudo netstat -anpt |grep :8080
tcp        0      0 :::8080                     :::*                        LISTEN      3318/java  
```
关闭Tomcat
```bash
$ sudo /opt/apache-tomcat-8.5.53/bin/shutdown.sh
或者直接kill进程号,例如:
$ sudo kill -9 3318
```

为了以后方便启动tomcat 可设置软链接
```bash
sudo ln  -s /opt/apache-tomcat-8.5.53/bin/startup.sh  /usr/bin/tomcatup
sudo ln  -s /opt/apache-tomcat-8.5.53/bin/shutdown.sh  /usr/bin/tomcatdown
```
以后直接使用 sudo tomcatup 和sudo tomcatup即可。

#### Maven安裝配置

在/usr/local/lib 目录下新建一个文件夹maven：
```bash
sudo mkdir /usr/local/lib/maven
```
将文件解压到这个目录下：
```bash
$ tar -zxvf apache-maven-3.6.3-bin.tar.gz -C  /usr/local/lib/maven
```
也可以放到别的路径下，可以看一下linux目录一般存放规则：[http://blog.csdn.net/fuzhongyu2/article/details/52437161](http://blog.csdn.net/fuzhongyu2/article/details/52437161)
配置环境变量  
环境变量分为用户变量和系统变量。  
用户变量配置文件：~/.bashrc（在当前用户主目录下的隐藏文件，可以通过`ls -a`查看到)  
系统环境配置文件：/etc/profile  
用户变量和系统变量的配置方法一样，本文以配置用户变量为例。  
编译配置文件.bashrc:  
在终端输入:  

```bash
sudo nano ~/.bashrc
```
在打开的文档末输入：
```bash
export MAVEN_HOME=/usr/local/lib/maven/apache-maven-3.6.3
export PATH=${MAVEN_HOME}/bin:${PATH}
```
点击保存，这样maven环境变量就配置好了。 
执行命令:
```bash
source  ~/.bashrc
source  /etc/profile
```
测试是否安装成功
输入：
```bash
mvn -v
```
如果出现下列字样，则安装成功：
```bash
  Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
  Maven home: /usr/local/lib/maven/apache-maven-3.6.3
  Java version: 1.8.0_241, vendor: Oracle Corporation,runtime:/home/johnnychan/Public/programs/java/jdk8/jdk1.8.0_241/jre
  Default locale: zh_CN, platform encoding: UTF-8
  OS name: "linux", version: "5.4.13-3-manjaro", arch: "amd64", family: "unix"
```

#### Git
```bash
sudo pacman -S git
```

#### Vim
```bash
sudo pacman -S vim
```
##### Vim右键不能粘贴的解决办法
修改/usr/share/vim/vim81/defaults.vim文件，不同发行版位置可能位置不一样，
```bash
find /usr/ -type f -name 'defaults.vim'
```
发现我的是/usr/share/vim/vim81/defaults.vim这个文件
```bash
sudo vim /usr/share/vim/vim81/defaults.vim
```
大概在第70多行的地方:
```bash
if has('mouse')
  set mouse=a
endif
```
把set mouse=a修改为set mouse-=a
```bash
if has('mouse')
  set mouse-=a
endif
```
:wq保存退出即可。

#### Markdown编辑器
```bash
sudo pacman -S typora
```

#### VSCode
```bash
sudo pacman -S visual-studio-code-bin
```

#### REST工具
```bash
sudo pacman -S postman-bin
```

#### GIT管理工具
```bash
sudo pacman -S gitkraken
```

#### idea(JAVA IDE)
```bash
sudo pacman -S intellij-idea-ultimate-edition
```

#### 数据库管理工具
```bash
sudo pacman -S datagrip
sudo pacman -S dbeaver
```

#### 离线API文档管理
```bash
sudo pacman -S zeal
```

#### NodeJS与NPM
```bash
cd /usr/local/
sudo mkdir node
cd node
#下载&解压
sudo wget https://nodejs.org/dist/v12.14.1/node-v12.14.1-linux-x64.tar.xz && sudo tar   zxvf node-v12.14.1-linux-x64.tar.xz
#设置全局
sudo ln -s /usr/local/node/node-v12.14.1-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /usr/local/node/node-v12.14.1-linux-x64/bin/npm /usr/local/bin/npm
#安装cnpm
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org  
#设置全局
sudo ln -s /usr/local/node/node-v12.14.1-linux-x64/bin/cnpm /usr/local/bin/cnpm
```
然后查看版本信息
```bash
$ node -v
v12.14.1
$ npm -v
6.13.4
$ cnpm -v
cnpm@6.1.1 (/usr/local/node/node-v12.14.1-linux-x64/lib/node_modules/cnpm/lib/parse_argv.js)
npm@6.13.6 (/usr/local/node/node-v12.14.1-linux-x64/lib/node_modules/cnpm/node_modules/npm/lib/npm.js)
node@12.14.1 (/usr/local/node/node-v12.14.1-linux-x64/bin/node)
npminstall@3.27.0 (/usr/local/node/node-v12.14.1-linux-x64/lib/node_modules/cnpm/node_modules/npminstall/lib/index.js)
prefix=/usr/local/node/node-v12.14.1-linux-x64/node_global 
linux x64 5.4.13-3-MANJARO 
registry=https://r.npm.taobao.org
```

##### npm全局模块路径的设置
设置全局模块的存放路径和cache路径。例如我希望将以上两个文件夹放在nodejs内（不要问我为什么，因为我希望以后在别的电脑上配置起来简单，不用每次都去获取各个模块），便在nodejs目录下新建"node_global"和"node_cache"两个文件夹。
在终端输入下面两行命令:
```bash
npm config set prefix "/usr/local/node/node-v12.14.1-linux-x64/node_global"
npm config set cache "/usr/local/node/node-v12.14.1-linux-x64/node_cache"
```
接下来，安装一个模块试试，例如vue，这里直接在终端输入npm install vue -g（-g就是全局安装模块的意思，就是将vue模块安装到你修改后的模块存放路径/usr/local/node/node-v12.14.1-linux-x64/node_global），等待下载安装。

#### 阿里云OSS
阿里云[OSS客户端](https://github.com//oss-browser): 点击下载：[oss-browser-linux_64_1.9.5](https://oss.chenjunxin.com/files/oss-browser-linux_64_1.9.5.zip),解压到目录，运行`oss-browser`即可。
若出现：`./oss-browser: error while loading shared libraries: libgconf-2.so.4: cannot open shared object file: No such file or directory`,
解决：
```bash
sudo pacman -S gconf
```

#### Docker与Docker-Compose
```bash
sudo pacman -S docker docker-compose
```
设置普通用户使用 Docker 不需要使用 sudo
```bash
sudo groupadd docker
```
启动docker服务
```bash
sudo systemctl start docker
```

查看docker服务的状态
```bash
sudo systemctl status docker
```

设置docker开机启动服务
```
sudo systemctl enable docker
```

干掉讨厌的 sudo,如果还没有 docker group 就添加一个
```bash
sudo groupadd docker
```
将自己的登录名(${USER} )加入该 group 内，然后退出并重新登录就生效了。
```bash
sudo gpasswd -a ${USER} docker 
```
或者
```bash
sudo usermod -aG docker $USER
```
重启 docker 服务
```bash
sudo systemctl restart docker
```
切换当前会话到新 group 或者重启 X 会话
注意，这一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错。
```bash
newgrp - docker
```
或者
```bash
pkill X
```
使用中国官方镜像加速:
为了永久性保留更改，我们可以修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值。
```bash
 {
  "registry-mirrors": ["https://registry.docker-cn.com"]
 }
```
修改保存后重启 Docker 以使配置生效。

#### Nginx
```bash
sudo pacman -S nginx
systemctl start nginx #启动Nginx服务
systemctl enable nginx #Nginx服务开机时启动
```
http://127.0.0.1 的默认页面是:/usr/share/nginx/html/index.html,你可以修改在 /etc/nginx/ 目录中的文件来更改配置 ./etc/nginx/nginx.conf 是主配置文件。

##### Nginx配置下载目录
在原有nginx配置中增加location模块，对127.0.0.1访问路径设置为下载目录根目录/home/johnnychan/Downloads/ThunderDownloads,并且对该location块开启目录文件列表，详细配置如下：
```json
location / {
root            /home/johnnychan/Downloads/ThunderDownloads;
autoindex on;  # 开启目录文件列表
autoindex_exact_size on;  # 显示出文件的确切大小，单位是bytes
autoindex_localtime on;  # 显示的文件时间为文件的服务器时间
charset utf-8,gbk;  # 避免中文乱码
}
```

##### Nginx的403 Forbidden解决的办法(权限文件和文件不存在)
要修改nginx运行用户为拥有配置的root路径拥有权限的用户，或者修改目录的权限。在/etc/nginx/nginx.conf 前面加上一句：
```json
user xxx;
```
就可以了，其中，xxx就是运行nginx的用户。  
完成了上面的，访问还出现错误，很有可能是你的目录里没有文件，然后又没有列出目录的权限。   
检查你的/home/xxx/website/nginxweb文件夹里面是否有配置的默认文件，默认文件在nginx.conf里面的index，或者使用上面的方法生成文件索引。

### 终端软件

```bash
sudo pacman -S neofetch # 终端打印出你的系统信息
sudo pacman -S htop #命令行显示进程信息
sudo pacman -S bashtop #终端Linux资源监控器
sudo pacman -S yakuake # 堪称 KDE 下的终端神器，KDE 已经自带，F12 可以唤醒
sudo pacman -S net-tools # 这样可以使用 ifconfig 和 netstat
yay -S tree
yay -S qrcp #电脑手机通过wifi互传文件工具
```
上述工具中[bashtop](https://github.com/aristocratos/bashtop)和[qrcp](https://github.com/claudiodangelis/qrcp)是[GitHub](https://github.com/)上的开源项目。

#### ZSH和Oh-my-zsh
```bash
sudo pacman -S zsh
```
接着配置oh-my-zsh：
```bash
sh -c“$（curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh）”
```
查看本地有哪些shell
```bash
cat /etc/shells
```
最后更换默认的shell为zsh：
```bash
chsh -s /bin/zsh
```

#### 更改zsh主题
```bash
vim ~/.zshrc
```
修改配置文件中的 " ZSH_THEME ",例如设置为随机主题
```bash
ZSH_THEME  = "random"
```

##### 设置为powerlevel10k主题
推荐使用[powerlevel10k](https://github.com/romkatv/powerlevel10k)主题,兼容powerlevel9k，10-100倍的性能提升。  
由于使用的是Oh My Zsh，所以只需要将 github 上的 repo 下载到 Oh My Zsh 的目录下然后配置即可。  
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k

# 然后设置 .zshrc 中的变量 ZSH_THEME
ZSH_THEME="powerlevel10k/powerlevel10k"
```

#### 安装字体
安装完 powerlevel10k 后你可能会发现有乱码，这是因为你终端字体不支持那么多的字符，这就需要使用扩展字体了。  
字体的安装，我们可以参考 [powerlevel9k 的官方介绍](https://github.com/bhilburn/powerlevel9k/wiki/Install-Instructions#step-2-install-a-powerline-font)：  
安装 powerline 字体有四种方式：

- Option 1: Install Powerline Fonts 请看官方文档
- Option 2: Use a Programmer Font 请看官方文档
- Option 3: Install Awesome-Powerline Fonts 请看官方文档
- Option 4: Install Nerd-Fonts:Nerd 字体应该是支持字形最多的，所以不用多想，安装它就好了。

GitHub 上有[Nerd-Fonts详细的安装介绍文档](https://github.com/ryanoasis/nerd-fonts#font-installation)
可以直接 [nerdfonts官网](https://www.nerdfonts.com/)，找到Downloads，直接下载 Hack 字体，，直接双击安装下载的字体即可，然后修改 oh my zsh 的变量文件 ~/.zshrc，在设置主题的配置前添加一行设置：
```diff
+ POWERLEVEL9K_MODE='nerdfont-complete'
ZSH_THEME="powerlevel10k/powerlevel10k"
```
ArchLinux下可以运行命令安装:
```bash
$ sudo pacman -S nerd-fonts-hack 
```
**注意：**zsh安装powerlevel10k主题后启动终端会让用户选择各种配置，之后生成相应的配置文件，并在~/.zshrc文件中加入运行该主题自定义配置文件的命令`[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh`，会在其中看到已经配置了主题的字体`typeset -g POWERLEVEL9K_MODE=nerdfont-complete`，因此在~/.zshrc中不用再额外设置`POWERLEVEL9K_MODE`字段。  
最后，如果字体没有生效，那就需要手动在`系统设置`->`字体`->`等宽字体`中设置Hack字体了。如下图：
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_zsh_font_demo.webp)


#### zsh必备插件安装

##### 参数补全插件 zsh-completions
```bash
git clone https://github.com/zsh-users/zsh-completions ~/.oh-my-zsh/custom/plugins/zsh-completions
```

##### 语法高亮插件 zsh-syntax-highlighting
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

##### 命令自动补全插件 zsh-autosuggestions
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

##### thefuck插件
```bash
pip install thefuck
```

##### fzf，模糊搜索神器
```bash
sudo pacman -S fzf
```

##### 自动跳转插件 autojump
在终端输入d，可以显示刚刚走过的路径，然后按数字选择进入哪一个目录。（这个插件需要自己下)
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_autojump.webp)
```bash
# clone 到本地
git clone git://github.com/joelthelion/autojump.git
# 进入clone目录，接着执行安装文件
cd autojump
./install.py
# 接着根据安装完成后的提示，在~/.bashrc最后添加下面语句：
vim ~/.bashrc
[[ -s /home/misfit/.autojump/etc/profile.d/autojump.sh ]] && source /home/misfit/.autojump/etc/profile.d/autojump.sh
```
autojump 工作原理：它会在你每次启动命令时记录你当前位置，并把它添加进它自身的数据库中。这样，某些目录比其它一些目录添加的次数多，这些目录一般就代表你最重要的目录，而它们的“权重”也会增大。

##### 安装完后启用插件
```bash
# 编辑~/.zshrc   
vim ~/.zshrc    
# 在plugins后括号里添加安装的插件名字
plugins=( git 
          zsh-completions
          autojump 
          zsh-completions
          zsh-autosuggestions 
          zsh-syntax-highlighting
          docker
          docker-compose
          fzf
       )
# 最后刷新使配置生效
autoload -U compinit && compinit
source ~/.zshrc
```

#### 配置环境变量使得zsh和bash都生效
为了便于在bash和zsh切换后可以使用同样的配置的alias等配置，采用如下方案：  
自定义配置放在.profile中  
.bashrc配置文件中使用source ~/.profile加载自定义配置  
.zshrc配置文件中使用[[ -e ~/.profile ]] && emulate sh -c ‘source ~/.profile’加载自定义配置  
配置文件示例如下：

- 在.bashrc原有配置最下面新加一行配置
```
source ~/.profile
```
- .在zshrc原有配置最下面新加一行配置
```
[[ -e ~/.profile ]] && emulate sh -c 'source ~/.profile'
```
- .profile
```bash
export EDITOR=/usr/bin/nano
# oh-my-zsh autojump配置
[[ -s /home/misfit/.autojump/etc/profile.d/autojump.sh ]] && source /home/misfit/.autojump/etc/profile.d/autojump.sh
# 在后面加上地址,根据你jdk配置
# 配置Maven环境变量
```
然后执行:
```bash
$ source ~/.profile
```
在bash终端执行:
```bash
$ source ~/.bashrc
```
在zsh终端执行:
```bash
$ source ~/.zshrc
```
这样把算定义配置放在.profile里，即可在bash和zsh中使用同样的自定义环境了。
    
### 科学上网

#### Shadowsocks客户端
先用manjaro自带的octopi搜索shadowsocks-qt5，然后安装
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_shadowsocks_qt5_00.webp)
安装成功并配置好你自己的ss后，如下:
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_shadowsocks_qt5_01.webp)
此时进入系统设置-代理下配置已经连接上的代理端口，如下图:
![](https://chenjunxin.oss-cn-shenzhen.aliyuncs.com/picture/blogPicture/a55f0545_global_proxy.webp)
此时，Chrome浏览器就可以科学上网了，但这是全局的设置，而且没有规则绕过一些国内的网址，因此还要继续设置。
打开Chrome网上应用商店-搜索安装Proxy SwitchyOmega扩展  
然后设置一个proxy
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_switchyomega_proxysetting.webp)
再新建设置一个auto switch,规则列表填写地址[https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt) ，点击立即更新情景模式，按照如下配置:
![](https://oss.chenjunxin.com/picture/blogPicture/a55f0545_switchyomega_proxysetting_autoswitch.webp)
把这个auto switch情景模式设置为插件默认的就可以了，实现按照规则科学上网，这里是符合规则内的才翻墙，规则之外的直连访问。**记得改回系统刚才设置的全局代理**。

#### V2Ray客户端
运行官方一键安装脚本:
```bash
$ bash <(curl -L -s https://install.direct/go.sh)
```
或者使用Manjaro自带的包管理器安装
```bash
sudo pacman -S v2ray
```
编辑/etc/v2ray/config.json文件，可以用[配置生成器](https://intmainreturn0.com/v2ray-config-gen/)
ArchLinux下的v2ray/config.json
使用v2ray自带了一个检查工具v2ray -test检查json文件
```bash
$ v2ray -test -config /etc/v2ray/config.json` #检查json
V2Ray 4.22.1 (V2Fly, a community-driven edition of V2Ray.) Custom (go1.13.5 linux/amd64)
A unified platform for anti-censorship.
Configuration OK.
```
显示OK就表示没问题了，可以开启本机的开机自启服务
```bash
systemctl enable v2ray  #开机自启v2ray
systemctl start v2ray   #启动v2ray
```
浏览器代理实现同上的shaodowsocks方式。

#### 终端代理
为了解决终端下载被墙服务器的安装包失败的问题，所以需要让终端也可以翻墙，顺便提升下载速度。工具有polipo 和 privoxy 两种,polipo 貌似只能全局代理，privoxy 全局/自动两种代理方式都可以实现。**全局代理下，访问 localhost 时也会走代理，可能导致无法正常访问本地服务**。

##### privoxy 实现全局和自动代理
privoxy 可以配置 .action 格式的代理规则文件。通过控制规则文件实现全局和自动代理。
action 文件可以手动编辑，也可以从 gfwlist 生成。
下面将先介绍 privoxy 的安装配置，再介绍 action 文件的生成。

###### 安装配置
安装 privoxy：
```bash
$ yay -S privoxy
```
进入目录 `/etc/privoxy`，可以看到目录结构大致为：

- `config` 配置文件，这个文件很长。。
- `*.action` 代理规则文件
- `*.filter` 过滤规则文件
- `trust` 不造干嘛用
- `templates/` 同上

开始修改配置文件。
privoxy 有 filter （过滤）的功能，可以用来实现广告拦截。不过这里只希望实现自动代理，在配置文件中把 filter 部分注释掉：
```bash
# 大约在435行
# filterfile default.filter
# filterfile user.filter      # User customizations
```

我们将使用自定义的 action 文件，所以把默认的 action 文件注释掉，并添加自定义文件：
```bash
# 386行左右
# 默认的 action 文件
# actionsfile match-all.action # Actions that are applied to all sites and maybe overruled later on.
# actionsfile default.action   # Main actions file
# actionsfile user.action      # User customizations
# 自定义 action 文件
actionsfile my.action
```
可以指定转换后的 HTTP 代理地址，这里直接使用默认端口 8118：
```bash
# 785行左右
listen-address  127.0.0.1:8118
listen-address  [::1]:8118
```
如果代理规则直接写在配置文件 config 中，那么代理规则和本地 SS 代理地址是写在一起的：
```bash
# / 代表匹配全部 URL，即全局代理
forward-socks5 / 127.0.0.1:1081 .
```
或
```bash
# 根据规则自动代理
forward-socks5 .google.com 127.0.0.1:1081 .
```
**注意！每行最后还有一个点。**  
实现全局代理就是第一种写法了。  
但是如果要自动代理，第二种直接写在配置文件里的做法其实不太合适，更合适的做法是写成 action 文件，配置文件中只管引用。  
把上面的注释掉。  
新建 action 文件 my.action，内容如下：
```bash
# 这一行表示本 action 文件中所有条目都使用代理
{+forward-override{forward-socks5 127.0.0.1:1081 .}}
# 添加一条规则
.google.com
```
把 privoxy 转换后的地址 http://127.0.0.1:8118 添加到环境变量，可以参照 polipo 部分。
启动 privoxy，这时应该可以正常访问 Google 了：
```bash
service privoxy start
curl www.google.com
```
下面看一下怎么用 gfwlist 生成 action 文件。

###### 生成 action 文件
配置文件 config 或 action 文件修改后不需要重启 privoxy。
使用的工具是 [gfwlist2privoxy](https://github.com/snachx/gfwlist2privoxy)。这个工具很简单，文档就几行，写得也很清楚。
安装：
```bash
pip install gfwlist2privoxy
```
gfwlist2privoxy 不支持 python3.x，安装时注意使用的是 pip2 还是 pip3。
参数说明：

- `-i/--input` 输入，本地 gfwlist 文件或文件 URL。这里使用上面的 gfwlist
- `-f/ --file` 输出，即生成的 action 文件的目录。这里输出到  `/etc/privoxy/gfwlist.action`
- `-p/ --proxy` SS 代理地址，生成后可以修改。这里是 `127.0.0.1:1081`
- `-t/ --type` 代理类型，生成后也可以修改。这里是 `socks5`
- `--user-rule` 用户自定义规则文件，这个文件中的规则会被追加到 gfwlist 生成的规则后面

示例：
```bash
gfwlist2privoxy -i https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt -f /etc/privoxy/gfwlist.action -p 127.0.0.1:1081 -t socks5
```
得到文件 `/etc/privoxy/gfwlist.action`,[下载地址](https://oss.chenjunxin.com/files/gfwlist.action)。

最后，把` /etc/privoxy/config` 中的`actionsfile my.action` 改为 `actionsfile gfwlist.action`就完成了。

启动 privoxy.service 服务
```bash
systemctl start privoxy.service
systemctl -l status privoxy.service
```

配置环境变量  
在~/.bashrc或者~/.zshrc中输入
```
export https_proxy=127.0.0.1:8118
export http_proxy=127.0.0.1:8118
```
这样就完成代理的设置了。

## KDE
### KDE插件
- Resource Monitor（系统资源监控）：
```bash
yay -S plasma5-applets-resources-monitor
```
- Netspeed Widget（网络监控）：
```bash
yay -S plasma5-applets-netspeed
```
- Latte Dock：
```
sudo pacman -S latte-dock
```
- Global Menu
可以访问[https://store.kde.org/](https://store.kde.org/) 找插件，或者在AUR包的网站上根据插件名找是否有相应的包安装。

### KDE设置
一些桌面设置：  
显示：  
屏幕 120% 放大： 系统设置 > 显示 > 全局缩放> 1.2 

全局菜单：  
因为有了 Latte Dock，不再需要任务栏了，取而代之的是全局菜单。需要添加全局菜单的桌面部件
```
sudo pacman -S appmenu-gtk-module
sudo pacman -S libdbusmenu-glib  # For electron apps menu
```

锁屏：  
系统设置 > 工作空间行为 > 锁屏 > 键盘快捷键 设为 Meta + L 。

KDE 桌面动画：  
系统设置 > 工作空间行为 > 桌面特效 设置你喜欢的桌面效果。

打开文件：  
KDE 默认是单击打开文件，需要修改成跟Window一样的话：  
系统设置 > 工作空间行为 > 常规行为 > 点击行为

# 一些命令与技巧

## 常用pacman命令
### 更新系统
在 Archlinux系 中，使用一条命令即可对整个系统进行更新:
```bash
$ pacman -Syu
```
如果你已经使用pacman -Sy将本地的包数据库与远程的仓库进行了同步，也可以只执行:
```bash
$ pacman -Su
```

### 安装包
`pacman -S` 包名：例如，执行 `pacman -S firefox` 将安装 Firefox
你也可以同时安装多个包，只需以空格分隔包名即可  
`pacman -Sy` 包名：与上面命令不同的是，该命令将在同步包数据库后再执行安装  
`pacman -Sv` 包名：在显示一些操作信息后执行安装  
`pacman -U`：安装本地包，其扩展名为 pkg.tar.gz  
`pacman -U http://www.example.com/repo/example.pkg.tar.xz` 安装一个远程包（不在pacman配置的源里面）

### 删除包
`pacman -R` 包名：该命令将只删除包，保留其全部已经安装的依赖关系  
`pacman -Rs` 包名：在删除包的同时，删除其所有没有被其他已安装软件包使用的依赖  
`pacman -Rsc` 包名：在删除包的同时，删除所有依赖这个软件包的程序  
`pacman -Rd` 包名：在删除包时不检查依赖

### 搜索包
`pacman -Ss` 关键字：在仓库中搜索含关键字的包  
`pacman -Qs` 关键字： 搜索已安装的包  
`pacman -Qi` 包名：查看有关包的详尽信息  
`pacman -Ql` 包名：列出该包的文件

### 其他用法
`pacman -Sw` 包名：只下载包，不安装  
`pacman -Sc`：清理未安装的包文件，包文件位于/var/cache/pacman/pkg/目录   
`pacman -Scc`：清理所有的缓存文件

### pacman替代命令yay
```bash
sudo pacman -S yay
```
yay 的命令参数跟pacman参数基本一致。

## 常用命令
### 查看网卡信息
```bash
$ lspci|grep -i net
```

### 查看已经启用的服务
```bash
systemctl list-unit-files --state=enabled
```

### 查看关联性服务启动耗费时间
```bash
systemd-analyze critical-chain xxx.service
```

### 按时间排序，查看服务启动耗费时间
```bash
systemd-analyze blame
```

### GIT代理设置
推荐放到 .zshrc 中作为常用命令
```bash
git-proxy(){
  git config --global http.proxy socks5://127.0.0.1:1080
  git config --global https.proxy socks5://127.0.0.1:1080
}
git-noproxy(){
  git config --global --unset http.proxy
  git config --global --unset https.proxy
}
```

## 一些小技巧
### 快捷键
F12：拉幕式终端  
Alt+空格：调出全局搜索  
Ctrl+F8：切出多桌面窗口  

### 鼠标操作
在左上角撮几下，平铺所有窗口。  
鼠标滚轮好慢：
```bash
sudo pacman -S imwheel #配置文件自己上网查
```

# 参考链接

- [Manjaro 安装体验小结](https://michael728.github.io/2019/08/03/linux-manjaro-install/)
- [Manjaro的尝试](https://zzycreate.github.io/2018/11/03/Manjaro%E7%9A%84%E5%B0%9D%E8%AF%95/)
- [Manjaro安装，配置，美化指南](https://www.cnblogs.com/zryabc/p/11408297.html)
- [Manjaro Linux 踩坑调教记录](https://printempw.github.io/setting-up-manjaro-linux/)
- [Manjaro常用软件和命令行推荐](https://blog.csdn.net/was172/article/details/82826607)
- [polipo/privoxy 实现 Linux 系统全局/自动代理](juejin.im/post/5c91ff5ee51d4534446edb9a)
- [centos privoxy action 分流，黑白名单，不走代理](https://www.codeleading.com/article/22561057096/)
- [学习利器V2ray了解一下](https://www.teaper.dev/2019/06/02/v2ray/)
- [Linux bash终端设置代理(proxy)访问](https://aiezu.com/article/linux_bash_set_proxy.html)
- [Linux-zsh与bash共用](https://xueyp.github.io/linux/2019/01/15/linux-%E5%88%87%E6%8D%A2%E5%88%B0zsh.html)
- [Manjaro中文输入法安装](https://studytous886237090.wordpress.com/2018/07/04/manjaro%E4%B8%AD%E6%96%87%E8%BE%93%E5%85%A5%E6%B3%95%E5%AE%89%E8%A3%85/)
- [比较几种中文输入法后，我最终选择了sunpinyin + cloudpinyin组合](https://forum.manjaro.org/t/sunpinyin-cloudpinyin/114282)
- [manjaro xfce 18.0 踩坑记录](https://www.codetd.com/article/4515916)
- [将干净的 Manjaro 快速配置为工作环境](https://blog.triplez.cn/manjaro-quick-start/)
- [记录一次linux系统迁移过程](https://rovo98.coding.me/posts/3babee60/)
- [manjaro踩坑记](https://mrswolf.github.io/zh-cn/2019/05/24/manjaro%E8%B8%A9%E5%9D%91%E8%AE%B0/#SSD%E9%85%8D%E7%BD%AE)
- [在Manjaro Linux下安装驱动程序的两种方法](https://ywnz.com/linuxjc/4850.html)
- [Tesseract:安装与命令行使用](http://www.zmonster.me/2015/04/17/tesseract-install-usage.html)
- [linux 环境下安装tesseract](https://www.wj0511.com/site/detail.html?id=257)
- [提取图片文字——linux下tesseract-ocr安装编译](https://blog.csdn.net/pangyunsheng/article/details/79372845)
- [Fcitx (简体中文)](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [在Manjaro上优雅地使用Fcitx5](https://www.wannaexpresso.com/2020/03/26/fcitx5/)
- [尝试Fcitx5](https://zjukuny.github.io/posts/fcitx5/)