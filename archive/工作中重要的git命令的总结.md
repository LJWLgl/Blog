---
title: 工作中重要的git命令的总结
date: 2017-08-06 17:59:02
tags: Git
categroies: Git
---
### 从远程master上切一个分支下来
有时候master分支上被更新后，可以master切一个分支下来查看更新
```
gco -b feature/nanxuan/dev origin/master
```
<!-- more -->
如果需要切换到别人的分支可以用这条命令，只需把origin/master,改成别人的分支即可，如
```
gco -b feature/nanxuan/170806/dev origin/feature/mark/170620-ad_preview
```
以上这些命令，都可以通过intellij轻松完成
（右键->Git的精彩世界）
### 解决与远程代码的冲突问题
本周五我push本地分支到Git上之后，pull request的时候，Git提示我不能合并master,原因是因为远程分支有改动，在merge前需要解决冲突，可以通过以下命令把master拉下来
```
git pull origin master
```
<<<<<<<标记冲突开始，后面跟的是当前分支中的内容。
HEAD指向当前分支末梢的提交。
=======之后，>>>>>>>之前是要merge过来的另一条分支上的代码。之后的dev是该分支的名字。对于简单的合并，手工编辑，然后去掉这些标记，最后像往常的提交一样先add再commit就可以了。
```
<<<<<<< HEAD

test in master

=======

test in dev

>>>>>>> dev
```
### 撤销本地commit
在工作的时候，你万一错误的commit不需要的文件，
可以通过```git revert HEAD ```回退到上一个commit版本，不过这样话，你在所做的修改都没有了，最便捷的方法，就是使用Intellij的revert快捷键，把这个文件还原。
### 注意
+ ```git add .```这条命令要慎用，经常会把一些没必要的提交存入暂存区，比如一些单元测试
+ 总之，很多git命令都有快捷键，使用起来可以让我们事半功倍
