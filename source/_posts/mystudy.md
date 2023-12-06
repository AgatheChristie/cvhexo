---
title: erlang mix
date: 2023-11-21 20:18:51
tags:
- erlang
categories:
- Erlang
---

# 概述
erlang杂项

## erlang杂项

<!--more-->


### UTF编码问题
```shell
%% 这样不会报错
Msg = utils:format(<<("<font color='#36d626'>看了看开服时间你好你好")/utf8>>).

%% 但是这样就会报错

Msg = "<font color='#36d626'>看了看开服时间你好你好".

<<Msg/utf8>>.  %%这里直接报错了
```

很奇怪的原因

### --运算符
```shell
> [1,2,3,2] --[2] --[2].
[1,2,3,2]
> [1,2,3,2]--[2] .
[1,3,2]
```
--符号的运算顺序是从右到左可以分两步执行或者用括号括起来([1,2,3,2] --[2]) --[2].

### ++运算符
```shell
lists:append()
M++N会遍历列表M所以如果必须要使用++也要让数据量小的List在前面
```





