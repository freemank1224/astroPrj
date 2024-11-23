---
title: Netlify博客筑巢装修记
description: Test whether the blog file can be correctly display LATEX
pubDatetime: 2024-10-13
tags:
  - TEST
share: true
---
> 一直想搭建一个个人主页，更系统的记录一些碎片思考和感悟，能顺手帮到别人更好，然后又想能够自动同步本地内容，了解了一圈，目前看来`Obsidian + Github + Netlify` 部署的方案是我能力范围内的最佳选择了。这里简单记录一下这个过程。

## Table of content


## 0. 知识基础
在做一件事之前，总要有一个基本的框架性的知识，否则无法知道我们走在哪一步。在做这个事情之前

## 1. `Astro`与`Netlify`部署
`Astro`是一种现代化的前端框架，专注于构建快速、高效的静态网站。对小白极为友好，友好到在[它的网站](https://astro.build/)上，直接就有部署到`Netlify`的选项。这里回顾一下这个简单的流程：
##### 1.1 **选择一个主题模板（Theme）**

##### 1.2 **部署到`Netlify`**

##### 1.3 **去Github 查看**


#### **个性化：贴心的`AstroPaper`作者**
由于他本身也是Astro的使用者，因此很清楚使用者的需求，因此在默认部署的Blog中，包含了许多实用的个性化技巧。其中一个就是关于主题的个性化。


## 2. 搞定`Obsidian`同步

## 3. 个性化装饰
### 3.1 **中文字体实在是太丑了**
`Astropaper`本来就是外国人设计的模板，当然英文的观感是很棒的。虽说显示中文没问题，但默认中文字体，是真的丑到**不能忍**！作为一个审美在线的理工男，第一件事就是换掉中文字体！
于是我就按照[Astro使用自定义字体](https://docs.astro.build/zh-cn/guides/fonts/)中的说明，选择了第二种方法——使用字体资源包（Fontsource项目中的开源字体）来新增字体。具体方法上述页面都有，这里我用自己的第一视角来更详细的展示步骤：
1. **选择字体**。从[Fontsource](https://fontsource.org/)目录中找出你喜欢的字体，我用了比较稳重的[Noto Serif SC](https://fontsource.org/fonts/noto-serif-sc)，这个页面里就提供了各种粗细的预览，而页面`Install`标签里则可以看到如何安装使用这些字体的命令；
2. **安装字体包**。说明中只说了要使用`npm`命令安装：
   ```bash
		npm install @fontsource-variable/noto-serif-sc
	```
	但在运行这个命令之前，最好先在命令行中把当前目录切换到你本地`astro`的哪个目录（`repository`）下，就是已经和Github同步了的那个本地目录，再运行这个命令。
3. **导入字体包**。在要使用该字体的布局或组件中导入字体包。一般我们在基本布局组件中执行这个操作，做到全站通用。在`src/layouts/BaseLayout.astro`文件中：
   ``` bash
	---
	import '@fontsource/noto-serif-sc'
	---
	```
4. **使用字体**。在任何Astro项目中能编写CSS的地方都可以用，我需要在全文和首页中都使用，因此我把相关的位置，即`src/styles/base.css`文件中相关的标签都做了增加，这样标题和正文的中文字体都会得到替换：
   ```css
    h1 {
		font-family: 'Noto Serif SC Variable', serif;
	}

	body {
	    @apply flex min-h-[100svh] flex-col bg-skin-fill font-mono text
	    skin-base selection:bg-skin-accent/70 selection:text-skin-inverted;
	    font-family: 'Noto Serif SC Variable', serif;
	}

	```   

### 3.2 **如何同步`Github`仓库中的图片**
在主页增加图片，可能是我这种对网站和前端技术一窍不通的人最头疼的地方了，填个图片都添不明白。如果你也正受此折磨那一定要看这部分。
总体来说，你要加入的图片无非是两类，一种来自于网络，一种来自与本机。前者自然不必多说，因为图片地址是一个URL，你在Obsidian里面插入或粘贴一个图片，就会自动帮你生成Markdown格式引用，然后你部署到主页的时候，同样可以正常显示。这里要说的是第二种情况：就是你这个**图片来自于本机**，现在要把它贴在你的博文中。这时又有两种思路：
##### 3.2.1 **网络图片**
==找一个网站，能让你上传图片，并提供图片外链==。这就转化成了第一种情况（即图片来自网络）。那这样的网站我们一般叫它“图床”，国内能访问的基本都是收费的。常见的有腾讯、阿里等等，大家感兴趣可以自己选择，当然提醒注意下面两点：
- ! 墙的问题，选择国外图床的要关注一下可访问性，你能打开，观众未必；
- ! 国内图床收费不仅是存储空间费用，还有访问的流量费用，请综合考虑；
##### 3.2.2 **往服务器上传的本地图片**
==将图片放在部署的服务器上==。这是个很直接的办法，只要你的部署服务器空间够用，所以我只有一些装饰图片和最重要的图片才会选择存储在这里。看似简单，没想到让我这个小白搞了好久，下面就以自己的需求示例来说明具体做法。
> 需求：因为`Astro`这个模板足够简洁又太过简洁，甚至有些单调。所以我打算往首页放张图调节一下气氛。
1. 找到 `src/pages/index.astro` 文件，在里面需要先声明如下部分：
```javascript
import { Image } from "astro:assets"
import sample_image from "../assets/images/sample_image.png"
```

[Images | Docs (astro.build)](https://docs.astro.build/en/guides/images/)


![RainbowRGB.png](https://s2.loli.net/2024/10/13/lZd4DLRnJ8cwCAX.png)

![NEW_LOGO4.png](https://s2.loli.net/2024/10/13/gxFdE3c16U4IwAn.png)
