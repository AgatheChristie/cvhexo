---
title: erlang反编译
date: 2023-12-05 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

erlang反编译 把beam文件反编译成erl文件

<!--more-->

## GO
### 1
进ebin目录在ebin目录下执行 gogogo:get_all_beam() 把ebin目录下所有文件(这里可能有非beam文件),写入look.txt
### 2
在decompile()函数里把beam文件的名字复制进去,不要加后缀
### 3
执行gogogo:decompile()可以在ebin目录在生成所有的erl文件,这个函数的返回值是ok或者是反编译失败的文件列表
### 4
ps:1和2可以写成一个函数自动读取ebin文件夹下面的beam文件,3也可以把生成的erl文件放到指定目录，反编译失败的文件列表写进一个文件，可以后续优化
### 注意:
此方法只有在erl编译时添加了DEBUG_INFO选项才有效,否则是无法反编译的
### 测试
通过这种方式反编译生成的erl文件会把include里面的record全部写到erl文件中还是挺不方便的，在官网和google没找到去掉自动生成record的办法，有大佬找到了可以互相交流一下
### 代码如下：
```erlang
-module(decompile).
-compile(export_all).

get_all_beam() ->
    {ok, Q} = file:list_dir("."),
    file:write_file("./look.txt", io_lib:fwrite("~p.\n", [Q])).


decompile() ->
    %% 这里是beam文件列表
    BeamX = [test1,test2],
    sources(BeamX, []).

sources([], Result) ->
    Result;
sources([Module | T], Result) ->
    NResult =
        case sourceq(Module) of
            ok ->
                Result;
            _ ->
                [Module | Result]
        end,
    sources(T, NResult).


sourceq(Module) ->
    case catch source(Module) of
        ok ->
            ok;
        _E ->
            null
    end.


source(Module) ->
    Path = code:which(Module),
    {ok, {_, [{abstract_code, {_, AC}}]}} = beam_lib:chunks(Path, [abstract_code]),
    {ok, S} = file:open(atom_to_list(Module) ++ ".erl", write),
    io:format(S, "~s~n", [erl_prettypr:format(erl_syntax:form_list(AC))]),
    file:close(S),
    ok.
```