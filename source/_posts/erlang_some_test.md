---
title: Erlang中的一些测试结果
date: 2024-05-23 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

Erlang中的一些测试结果

<!--more-->



## 1.匿名函数重新生成 
测试环境 
Erlang/OTP 17 [erts-6.0] [64-bit] [smp:6:6] [async-threads:10]

```
[lists:foldl(fun(X, Sum) -> X + Sum end, 0, [1,2,3,4,5]) || X <- lists:seq(1,5)].
和下面
Fun = fun(X, Sum) -> X + Sum end.
#Fun<erl_eval.12.82930912>
[lists:foldl(Fun, 0, [1,2,3,4,5]) || X <- lists:seq(1,5)].
```
看到大佬们17年的博客说到用方式1每次循环匿名函数都会重新生成

测试下看看
```
-module(test_anonymous_func).
-compile(export_all).
-define(DEBUG(F), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE])).
-define(DEBUG(F, A), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE | A])).
%%======================测试匿名函数是否会重新生成=======================================
test_func(Type, Num) ->
    case Type of
        1 -> %% 这里的F每次都会重新生成?
            myfoldl(fun(X, Items) ->
                [X | Items]
                        end, [], lists:seq(1, Num));
        2 ->
            F = fun(X, Items) ->
                [X | Items]
                end,
            myfoldl(F, [], lists:seq(1, Num))
    end.

%% 从lists:foldl复制出来的
myfoldl(F, Accu, [Hd|Tail]) ->
    ?DEBUG("make_ref:~p end",[F]),
    myfoldl(F, F(Hd, Accu), Tail);
myfoldl(F, Accu, []) when is_function(F, 2) -> Accu.

%% 从源码可以看出 A ++ B的时候会遍历A 那么如果先对AB length一下再传参会怎么样？
%% timer:tc(lists,append,[lists:seq(1,100000000),[1]]).
%% timer:tc(lists,append,[[1],lists:seq(1,100000000)]).
%% timer:tc(erlang,length,[lists:seq(1,1000000000)]).
%% timer:tc(test_anonymous_func,test_append,[lists:seq(1,100000000),[1]]).
%% timer:tc(test_anonymous_func,test_append,[[1],lists:seq(1,100000000)]).
test_append(A, B) ->
    case length(A) > length(B) of
        true ->
            lists:append(B, A);
        false ->
            lists:append(A, B)
    end.
```
### 图示

![img_4.png](/pic/erlang_some_test/img_4.png)

### 分析
从打印的make_ref看匿名函数没有重新生成，官网已经下载不到更低的版本了，遂放弃，不知道是在OTP 17以后编译器做了优化还是要到底层的汇编码去找区别，因个人能力有限，只能测到这步，如有了解汇编的大佬还请指点一二

![img_3.png](/pic/erlang_some_test/img_3.png)


## 2.lists:append执行时间分析

### 图示

![img_5.png](/pic/erlang_some_test/img_5.png)

### 分析
从图可以看出来速度差了有10倍，length这个nif函数Erlang其实已经优化得很好了，为什么lists:append不先通过成length排序再把长度短的放前面查入呢？








