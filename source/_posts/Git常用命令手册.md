---
title: Git常用命令手册
date: 2018-04-01 16:03:27
tags: [git,工具]
---
### 前言
整理工作中的常用Git命令，以及问题处理方法。

### 代码仓库
```
git init                         # 在本地创建一个空仓库（一般用不到）
git clone   https://xxxx/xx      # 使用账号clone
git clone   git@github.com/xxx   # 使用rsa或者gpk私钥验证，需要在git账户进行配置

git remote -v                    # 查看当前仓库远程地址

git config --list                # 查看当前全局配置
```

### 提交代码
```
git status              # 查看状态，会列出修改的文件.（已经add的为绿色，否则为红色）

git add file1 file2    # 提交指定修改
git add .              # 提交全部修改

git diff file1         # 查看文件的不同

git commit -m '修改注释'     # 将当前add的修改提交
git commit -m 'xxx' file1   # file1不需要add，可直接提交

git push                    # 将本地提交推送到远程仓库，在push之前需要保证本地仓库的代码是最新版本(git pull拉取一下)

git log                     # 查看git提交历史记录
```

### 拉取代码
```
git pull            # 最新远程代码库拉取到本地

git checkout file1  # 将file的内容回滚到远程仓库版本

```
#### 拉取代码出现冲突
在执行pull命令时经常会因为修改同一个文件产生冲突，进而无法拉取代码的情况。需要使用缓存区进行解决。
```
情景再现：
A: 本地修改好的文件已经全部执行了commit操作，需要推送到远程仓库，但是pull命令失败；
B: 本地开发了一半，同事更新了代码需要先更新在继续后续的开发。

解决问题：
1. git stash                       # 将本地所有修改放到暂存区
   git stash save 'name abc'       # 放到暂存区同时起个名字
2. git pull         #拉取最新代码
3. git stash pop                   # 将缓存区的修改同步到本地仓库，同时修改的文件可能会有冲突。
4. 解决冲突， 然后执行 git add file 即可。


其他：
暂存区可以暂存多次；
git stash list 查看暂存列表； 
stash pop xxx 可以弹出指定的暂存版本； （在同时开发多个功能时可能会用到）

```

### 使用开源git仓库
当协同开发时可能会提公共类库，然后大家都可以维护；或者用了第三方开源git库，需要自定义修改，同时还要保证能及时更新； 此时需要用到 submodule 子模块的概念。    
submodule在仓库中增加子模块，子模块有自己的仓库地址，可以自行维护。

拿hexo中的next主题做例子： 
1. blog/ 是一个git仓库，存储hexo的原始md文档、配置文件、themes等内容；
2. next是第三方主题，会不定期更新；
3. 本地使用时也需要修改，但是仓库为他人的，自己的修改无法提交；
4. hexo搭建的博客需要在多台电脑间同时使用。

操作流程：
```
1. 在github中fork主题仓库https://github.com/iissnan/hexo-theme-next 到自己的仓库里

电脑A:
> git submodule add git@github.com:sharehappy11/hexo-theme-next themes/next     #将个人仓库加入子模块中
> cd themes/next;  git add -> commit -> push             # 进入到themes/next目录则可以直接使用git命令进行维护

电脑B:
> git clone blog_repo       # 拉取blog/博客仓库
> git submodule update      # 拉取子模块代码
> cd themes/next; 
>> git pull origin master   # 更新子模块代码


子模块维护：
本地修改了next主题，可以直接提交，因为是自己的仓库所以可以随便修改；
自己修改的功能想提交给官方时，可以给官方发送申请，提交自己的修改，为开源做共享；
官方主题更新时，可以通过github的pull reqeust功能同步最新功能；

```

