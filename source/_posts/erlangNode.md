---
title: erlangNode
date: 2023-12-01 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

erlang node study

<!--more-->

## 本地注册
```
werl -name 'fries@127.0.0.1'

之前的版本需要加''现在加了会直接报错

werl -name fries@127.0.0.1

werl -name ketchup@127.0.0.1

erlang:send(Dest, Msg) -> Msg
This is the same as using the send operator: Dest ! Msg.

```
### 不连接 直接给在本地注册名字的节点发消息
此时两个节点的cookie是一样的
```
erlang:get_cookie().
erlang:register(uzi,self()).
global:register_name(uzi, self()).
fries 下执行
{uzi,'fries@127.0.0.1'} ! {hellocc,fromqqq,self()}.
```
1.可以用一个一模一样的名字在本地和全局同时注册
2.!操作符传注册名的话只能收到本地注册名的消息
3.上面可以收到消息是因为在本地注册了uzi erlang:register(uzi,self())
4.如果 不在本地注册uzi只全局注册global:register_name(uzi, self()).
这样在K节点向F节点发送消息F节点是收不到的,如果只想向一个全局注册的名字进程发送消息需要
取到pid比如global:whereis_name(uzi) ! {hellocc,fromqqq,self()}.

![img_3.png](/pic/erlangNode/img_3.png)

给K节点发
```
{uzi,'ketchup@127.0.0.1'} ! {hellocc,fromqqq,self()}.
```
也可以收到消息

![img_7.png](/pic/erlangNode/img_7.png)




### 连接后直接给在本地注册名字的节点发消息 这个就不试了
此时两个节点的cookie是一样的
因为发送完消息以后就 连接在一起了，个人猜测在发消息之前节点自动执行了
```
net_adm:ping('fries@127.0.0.1').
```



### cookie不一样会怎么样
```
werl -name fries@127.0.0.1 -setcookie 'awdwad223'

werl -name ketchup@127.0.0.1 -setcookie 'awdwaq773'
```
失败了

F节点试图连接K节点被拒绝了


![img_8.png](/pic/erlangNode/img_8.png)

## global注册

![img_10.png](/pic/erlangNode/img_10.png)

所有的操作都需要两个节点连在一起