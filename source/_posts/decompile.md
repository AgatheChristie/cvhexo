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
### (1)
随便找个地方把my_decompile.erl编译成my_decompile.beam,把my_decompile.beam复制进想要反编译的ebin目录，在ebin目录下右键打开终端
### (2)
在终端输入erl进入erl shell环境 执行my_decompile:decompile()后会在ebin目录在生成所有的erl文件,这个函数的返回值是ok或者是反编译失败的文件

### 注意
此方法只有在erl编译时添加了DEBUG_INFO选项才有效,否则是无法反编译的
### 测试
通过这种方式反编译生成的erl文件会把include里面的record全部写到erl文件中,还是挺不方便的，在官网和google没找到去掉自动生成record的办法，有大佬找到了优化record的方法可以互相交流一下
### 代码如下：
```
-module(my_decompile).
-compile(export_all).

decompile() ->
    {ok, Tmp} = file:list_dir("."),
    %% 自己这个文件my_decompile没有加DEBUG_INFO反编译不了，筛选掉
    BeamFiles = [list_to_atom(filename:rootname(X)) || X <- Tmp,
        (filename:extension(X) == ".beam" andalso filename:rootname(X) =/= "my_decompile")],
    %% BeamFiles是beam文件原子列表
    sources(BeamFiles, []).

sources([], Result) ->
    Result;
sources([Module | T], Result) ->
    NResult =
        case check_source(Module) of
            ok ->
                Result;
            _ ->
                [Module | Result]
        end,
    sources(T, NResult).


check_source(Module) ->
    case catch source(Module) of
        ok ->
            ok;
        _E ->
            io:format("ERR:~p end~n",[_E]),
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