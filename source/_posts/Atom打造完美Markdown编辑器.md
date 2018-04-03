---
title: Atom打造完美Markdown编辑器
date: 2017-12-08 22:03:27
tags: [博客建站,Atom,Markdown]
---
### 前言
用Markdown写笔记有一段时间了，目前主要在有道记录、简书两个平台上使用。


**有道**  
支持实时预览，但不能插入图片，只能从别的地方上传后在笔记中。主要用来记录一些私人的、工作上内容。  
插入图片方法：新建一篇笔记，并分享；然后将图片粘贴在该笔记中，通过访问分享地址进而获取图片地址。

**简书**  
在线编辑器，支持实时预览，图片的ctrl+v粘贴(良心之作)。文章还能投稿引流很不错，唯一的问题就是浏览器经常用来搜索资料，再切换回标签编辑不是很流畅。   
看到有文章说使用Hexo建站后，会选择在简书编辑然后两边发布，其实也是一个不错的选择。

**Atom**  
Atom是github发布的一款文本编辑器，支持大量快捷键可以让程序员做到手指不离键盘。模块化是其主要的特点，有大量开源的插件，相比Sublime模块管理会更加友好。

目前已有很多优秀Markdown插件，所以选择该编辑器来编写文章。

### 安装Atom
在其[官网](https://atom.io/)下载安装即可。

安装成功后，点击右上角File->Setting->Install安装插件。

### 汉化包 simplified-chinese-menu
初次使用时可以先安装汉化包了解软件，目前支持Atom所有版本的汉化。

### 增强预览 markdown-preview-plus
Atom自带的mardow-preview功能比较简单，markdown-preview-plus对齐进行了升级，支持实时预览(ctrl+shift+m)

### 同步滚动 markdown-scroll-rsync
同步滚动编辑和预览页面，方便定位位置进行修改。

### 图片粘贴 markdown-img-paste [重点]   
https://atom.io/packages/markdown-img-paste   
使用ctrl+shift+v，将剪切板中的截图直接插入的文档中。支持三种模式：
1. Save images in assets floder: 将图片保存在./assets/中
2. Use sm.ms for image link: sm.ms是一个免费图床，但是国内好像访问不了
3. Use qiniu for imgage link: 使用七牛云存储图片，需要配置七牛的秘钥、空间名、域名。

注册七牛账号并实名认证后，每个月回给免费的10G存储，用来做个人博客图床非常不错。  
七牛注册地址：[https://portal.qiniu.com/signup?code=3lmwk75lqbviq](https://atom.io/packages/markdown-img-paste), 可以使用这个地址注册，这样就可以多给我5G空间了，谢谢~
![markdown-img-paste插件配图](http://p0mqjeixa.bkt.clouddn.com/markdown-img-paste-%E7%A4%BA%E4%BE%8B.gif)

### 表格编辑 markdown-table-editor
markdown的表格编写需要特定的语法格式，而且新行的数据太多是无法和上面的内容对齐，编写起来很不方便。   
table-editor插件就能很好的解决该问题了：自动补全、自动调节表格宽度。
https://atom.io/packages/markdown-table-editor  目前在1.4.3安装会报错。
![markdown-table-editor](http://p0mqjeixa.bkt.clouddn.com/markdown-table-editor-%E7%A4%BA%E4%BE%8B.gif)


### minimap
同sublime右边栏一样，让你了解当前屏幕所处位置，快速翻页。

### 结束语
按照好上面的插件 + Atom好看的主题，打造的Markdown编辑器就非常完美啦。   

网上都说Atom比Sublime卡，目前来看内存确实占用比较多，倒是还没有出现卡顿的情况。
先用用看，不行就安装个新版本，目前使用的是Pentestbox自带的1.4.3版本。
