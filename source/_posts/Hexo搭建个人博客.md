---
title: Hexo搭建个人博客
date: 2017-12-07 15:03:27
tags: [博客建站,Hexo]
---

使用Hexo和github.io搭建个人免费博客，配合Atom打造的Markdown编辑器(支持图片ctrl+v粘贴)，再把剁手买的域名利用上，这一切看起来是这么的完美。
<!-- more -->

### 起因
脑子一抽买了个小众域名yachao.wang，目测也只能用来做个人博客。第一印象是用WordPress，但是需要自己花钱买vps，并且我本身一直从事PHP开发，所以想换个套路。   
Hexo搭建静态个人网站，支持Markdown编写文章、本身又有大量的精美主题，依托于github.io能够免费部署，这种种优势对我都很有吸引力。


Hexo博客搭建的基础大致流程为：   
> 安装Node.js →安装Git → 安装Hexo → 安装主题 → 本地测试运行 → 注册给github与coding并创建pages仓库 → 部署


### 安装Node.js
访问[Node.js官网](https://nodejs.org/en/)下载自身系统对应的32位(x6)或64位(x64)安装包。软件安装之后，检查PATH是否配置了Node.js的环境变量。
```
Win+R -> 输入cmd
> PATH
    PATH=***.;C:\Users\sks\AppData\Roaming\npm

>node --version
    v8.9.2
```
如果获得以上的输出，说明已经成功安装了Node.js.


### 安装Git
访问[Git官网](https://git-scm.com/download/win)下载对应安装包，一直点击next进行安装即可。
```
Win+R -> 输入cmd -> 输入git --version
    git version 2.15.1.windows.2
```

### 安装Hexo
通过npm分别安装hexo客户端和服务端。   
```
npm install -g hexo-cli
```
[警告]npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3:
wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})   
原因：fsenents是mac osx系统的，在win或linux上使用都会有该警告，忽略即可。

#### 【踩坑1】
要按照npm官方[https://www.npmjs.com/package/hexo](https://www.npmjs.com/package/hexo) 或者 Hexo官网[https://hexo.io/](https://hexo.io/)的说明进行操作，只安装hexo-cli即可。    

网上的教程在此时要安装一起hexo-server，是针对旧版本的的方法。如果你此时安装了hexo-server,在执行hexo init时会在[hexo-server@0.2.2]处卡死！！   
![表情1](http://upload-images.jianshu.io/upload_images/1849253-963d1e0f4792c11d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/220)   
npm默认安装最新版软件，软件作者会更新软件，而博主的文章一般都是针对以前版本的的。所以会出现这种情况，看官网才是王道！


### 体验Hexo
新建一个文件夹D:/blog，进入该目录先安装必要的文件，执行下面的命令：
```
d:> cd d:/blog
blog> hexo init     #在目标文件夹建立网站所需要的内容
blog> npm install   #安装依赖包
blog> hexo server   #即可在本地访问测试
```

#### 配置   
配置比较简单，参考官网即可：https://hexo.io/zh-cn/docs/configuration.html  
**注**：配置中冒号[:]后面需要有空格。

#### 常用指令
```sh
hex new [layout] <filename>
#新建一篇文章，默认模板为配置文件中的default_layout，可自定义。如文件名有空格，则需要引号["]括起来
hexo generate  Or    hexo g
#生成静态文件，生成public目录
hexo server [-p 端口]
#启动服务端，支持自定义端口
hexo deploy
#将静态页部署到线上，在_config.yml的deploy地方进行配置，发布到github coding静态页等。 建议预览后在部署
hexo clean
#清除db.json和静态文件，在更换主题后使用，重新生成静态文件。

```
部署文件时主要操作流程： 清空->预览->部署。  
写文章时的主要操作流程：开启服务->编辑->同步预览。

#### 更换Hexo主题
Hexo支持用户发布自己的主题，其官网有很多[个性化主题](https://hexo.io/themes/)可自行下载安装，很喜欢next、Cactus Dark主题，简介大气，有文档锚点，翻阅时能迅速定位到自己想要的。
```
#更换主题非常简单，在bolg目录下载
blog>git clone https://github.com/iissnan/hexo-theme-next themes/cactus-dark

#修改_config.yml中的theme配置
    theme: next

#修改 themes/next/_config.yml进行主题内的自定义配置
    一定要按照github上的说明来做即可！
```
**说明**:   
配置主题时先读主题文件夹中的ReadMe文档，了解主题功能和配置方法。
cactus-dark
    按照Readme或者github说明即可，配置比较简单,但是感觉侧栏不如Next好。
Next
    主题功能比较强大，有官方网站。支持三种外观、侧栏；支持第三方插件百度统计、阅读次数统计、Algolia搜索、评论系统等。
    按照[Next官网文档](http://theme-next.iissnan.com/getting-started.html)说明即可进行个性化定制。


### 配置Github仓库
1. 注册免费Github账号
2. 点击页面【New repository】新建代码仓库
3. 在创建页面【Repository Name】处填写{$yourname}.github.io,  {$yourname}必须和用户名一致，不能有大小写(即和前面Owner字母一致).
    同时勾选【Initialize this repository with a README】用自述文件初始化这个库,点击创建按钮；
![创建仓库](http://p0mqjeixa.bkt.clouddn.com/hexo%E5%88%9B%E5%BB%BA%E5%8D%9A%E5%AE%A2.gif)

4. 仓库创建成功后，会给出访问地址：【Your site is published at https://{$yourname}.github.io/】
5. 配置Github使用RSA秘钥登录，可以配置账户的SSH-key，也可以对单个仓库配置deploy-key. 为方便使用直接配置账户的ssh-key.
生成RSA公钥私钥，点击鼠标右键选择[Git Bash Here]
```sh
$ ssh-keygen -t rsa -C 'github注册邮箱'
#一路回车，将在用户.ssh/目录下生成两个文件，一个公钥一个私钥
```
6. 在gitbub右上角，进入账户设置页面Setting， 选择SSH and GPG keys,添加公钥id_rsa.pug的内容。地址：https://github.com/settings/ssh/new
```
#在本机执行该命令测试连接是否成功
$ ssh -T git@github.com
Hi xxxx You've successfully authenticated, but GitHub does not provide shell access.  
```
7. 修改hexo网站配置文件_config.yml， deploy块
```yaml
deploy:
  type: git
  repository: https://github.com/{$yourname}/{$yourname}.github.io
  branch: master

```
8. 执行hexo deploy命令部public文件.  
如果有报错ERROR Deployer not found: git，则先安装hexo-deployer-git
```sh
> npm install hexo-deployer-git --save
```
9. 完成，访问 http://{$yourname}.github.io就能看到你自己的博客啦。


### 绑定个人域名
1. ping {$yourname}.github.io 获取服务器IP地址;
2. 域名在万网购买的，直接打开控制台->产品与服务->域名->域名列表，对域名进行解析配置；将www和@ 配置到该IP;
3. 在Hexo目录的soruce的文件下添加一个名为CNAME的文件，文件无后缀，内容为：
> yachao.wang
4. 再次部署，访问该域名即可看到博客内容。



**相关阅读**
1. nodejs、git、hexo.io、github 官网
2. [github.io搭建教程](https://www.cnblogs.com/liulangmao/p/4323064.html)
