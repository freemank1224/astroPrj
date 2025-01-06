---
title: 给你的Macbook硬盘省点空间：将微信等APP彻底安装到移动硬盘
description: 总结了一下将微信、QQ等应用安装在移动硬盘的方法，当然还有它们产生的数据！
pubDatetime: 2025-01-06
tags:
  - Mac
  - 小技巧
share: "true"
featured: false
draft: false
---
> Macbook的存储堪比黄金，因此我入了1Tb的丐版机型。主要用来存模型写代码，所以QQ微信这种臃肿的东西，就想转移到外置硬盘上面去。这里记录一下过程，以供参考。
## 几种安装方式的取舍
首先根据你的需求，确定自己采用那种安装风格，第一种无需多讲，主要是后面两类：

✅ **重度微信（QQ）办公者**：如果大部分工作都是在微信上收发文件，建议直接安装在本机，避免找不到文件这种问题；

✅ **微信必须在手边的办公者**：就是说你希望有没有移动硬盘，你都能访问微信，而不是很在乎聊天记录中的文件是否一定随时待命，可以只把主程序安装在本机，数据迁移到移动硬盘；

✅ **惜硬盘如金者**：比如我这种，看到这种不断变大的APP，尤其大部分还是垃圾信息，就头疼，这样就把APP本体和数据都迁移到外置存储上就好了。
## 惜硬盘如金者
显然，这个操作要分两步走：
### 1. 将APP本体安装在移动硬盘
这里仅针对下载的`.dmg`文件，如果是AppStore里面的APP不适用。双击之后，不要将应用拖到旁边的`Applications`文件夹中，而是拖入你的移动硬盘中。当然最好你也在移动硬盘中建立一个类似的文件夹，方便查找和管理。这样，你就可以在移动硬盘上直接点击图标来启动应用。
当然这样操作完，在Mac任务栏中的“启动台”里是看不到这些APP的，可以进一步设置，我个人不需要，此处就略过了。
![](https://i-blog.csdnimg.cn/blog_migrate/e83397af565736afee974aa6d20a5a16.png)

### 2. 将APP的相关数据转移到移动硬盘
#### 2.1 微信
1️⃣ **找到微信文件：** 微信的文件存储在如下位置：
```bash
Users/softmaple/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat
```
右键点击“访达”，菜单里面有一个选项：“前往文件夹”，把上述地址粘贴进去，打开上述目录，里面有个文件夹：`2.0b4.0.9`。

2️⃣ **移动文件：** 将`2.0b4.0.9`这个文件夹复制到移动硬盘中。（同样，建议建立单独文件夹方便管理）

3️⃣ **建立连接：** 运行下面的命令来建立一个连接，相当于把原定的文件存储位置映射到你的移动硬盘中：
```bash
ln -s [A] [B]
```
具体来说，上面的`[A]`是新的文件位置（移动硬盘中的文件位置），在我这里，它是`/Volume/Peanut/AppData/2.0b4.0.9`
`[B]`是原来文件的存储位置，在我这里是`/Users/softmaple/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat`
综上，完整命令为：
```bash
ln -s /Volume/Peanut/AppData/2.0b4.0.9 /Users/softmaple/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat
```

4️⃣ **后处理：**
再运行下属命令，将APP和文件建立起连接，注意路径，要指向你的微信的安装位置，比如我装在了名为`Peanut`的外置硬盘上，所以需要把路径指向它。如果是装在本机的`Applications`文件夹，那么相应的路径就要换成这个文件夹了。
```bash
sudo codesign --sign - --force --deep /Volumes/Peanut/Applications/WeChat.app
```
这个会弹出管理员密码，填入后回车即可。

5️⃣ **打开微信验证：** 此时打开微信验证一下能否正常运行，如果可以，应该正常运行了。
#### 2.2 QQ 
QQ的操作与之类似，关键就是找到文件的存储路径，我找到的是QQ下面的Data文件夹，写在这里，也可能实际的存储位置隐藏得更深，但是目前我是选的这个位置：
`/Users/softmaple/Library/Containers/com.tencent.qq/`
里面有一个叫做`Data`的文件夹。然后后续操作参考上面即可。