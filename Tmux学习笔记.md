---
title: Tmux学习笔记
date: 2017-07-12 23:40:38
tags: Tmux
categroies: 工具
---
#### tmux介绍    
网上关于tmux教程很多，也比较杂，在这里笔者做个简单的总结，让大家知道感受tmux的牛逼之处。tmux是一个通过一个终端登录远程主机并运行后，在其中可以开启多个控制台的终端复用软件。可以举个例子，比如说我们在本地连上服务器，突然有事要和服务器断开，<!-- more -->这时候我们在服务器的任务很可能与因为与本地失去连接而停止执行，而用tmux不会存在这样的问题，它可一在服务器端保存一个session,即使我们与服务器断开连接，去喝杯茶，回来再和服务器连，服务器上的任务依然在跑着。好了，tmux既然这么厉害，下面让我们来看看如何驾驭它。  
==注意下面的操作均在服务器端进行==
#### 显示所有的session会话
```bash
tmux ls
//或者
ctrl+b s //注意先按ctrl+b，再按s
```
#### 新建会话
```bash
tmux new -s [session-name] // 如tmux new -s nanxuan
```
#### 接入之前的会话
```bash
tmux a -t [session-name] // 如tmux a -t nanxuan
```
#### 从会话断开
```bash
tmux detach
//或者
ctrl+b d
```
#### 关闭会话

