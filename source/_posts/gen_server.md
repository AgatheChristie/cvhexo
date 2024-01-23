---
title: gen_server start 源码阅读
date: 2024-01-23 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

erlang gen_server 源码阅读

<!--more-->


## gen_server
### 入口 
A进程调用
```shell
start_link() ->  
    gen_server:start_link({local, ?SERVER}, ?MODULE, [], []).
```

下面是所有相关代码

```
=======================gen_server==========
start_link(Name, Mod, Args, Options) ->
    gen:start(?MODULE, link, Name, Mod, Args, Options).
    
init_it(Starter, Parent, Name0, Mod, Args, Options) ->
    Name = gen:name(Name0),
    Debug = gen:debug_options(Name, Options),
    HibernateAfterTimeout = gen:hibernate_after(Options),

    case init_it(Mod, Args) of
	{ok, {ok, State}} ->
	    proc_lib:init_ack(Starter, {ok, self()}), 	    
	    loop(Parent, Name, State, Mod, infinity, HibernateAfterTimeout, Debug);
	{ok, {ok, State, TimeoutHibernateOrContinue}} ->
	    proc_lib:init_ack(Starter, {ok, self()}), 	    
	    loop(Parent, Name, State, Mod, TimeoutHibernateOrContinue,
	         HibernateAfterTimeout, Debug);
	{ok, {stop, Reason}} ->
	    %% For consistency, we must make sure that the
	    %% registered name (if any) is unregistered before
	    %% the parent process is notified about the failure.
	    %% (Otherwise, the parent process could get
	    %% an 'already_started' error if it immediately
	    %% tried starting the process again.)
	    gen:unregister_name(Name0),
	    proc_lib:init_ack(Starter, {error, Reason}),
	    exit(Reason);
	{ok, ignore} ->
	    gen:unregister_name(Name0),
	    proc_lib:init_ack(Starter, ignore),
	    exit(normal);
	{ok, Else} ->
	    Error = {bad_return_value, Else},
	    proc_lib:init_ack(Starter, {error, Error}),
	    exit(Error);
	{'EXIT', Class, Reason, Stacktrace} ->
	    gen:unregister_name(Name0),
	    proc_lib:init_ack(Starter, {error, terminate_reason(Class, Reason, Stacktrace)}),
	    erlang:raise(Class, Reason, Stacktrace)
    end.

=========================gen==============
start(GenMod, LinkP, Name, Mod, Args, Options) ->
    case where(Name) of
	undefined ->
	    do_spawn(GenMod, LinkP, Name, Mod, Args, Options);
	Pid ->
	    {error, {already_started, Pid}}
    end.


do_spawn(GenMod, link, Name, Mod, Args, Options) ->
    Time = timeout(Options),
    proc_lib:start_link(?MODULE, init_it,
			[GenMod, self(), self(), Name, Mod, Args, Options],
			Time,
			spawn_opts(Options));

init_it(GenMod, Starter, Parent, Mod, Args, Options) ->
    init_it2(GenMod, Starter, Parent, self(), Mod, Args, Options).

init_it(GenMod, Starter, Parent, Name, Mod, Args, Options) ->
    case register_name(Name) of
	true ->
	    init_it2(GenMod, Starter, Parent, Name, Mod, Args, Options);
	{false, Pid} ->
	    proc_lib:init_ack(Starter, {error, {already_started, Pid}})
    end.

init_it2(GenMod, Starter, Parent, Name, Mod, Args, Options) ->
    GenMod:init_it(Starter, Parent, Name, Mod, Args, Options).

=========================proc_lib==================
start_link(M,F,A,Timeout,SpawnOpts) when is_atom(M), is_atom(F), is_list(A) ->
    ?VERIFY_NO_MONITOR_OPT(M, F, A, Timeout, SpawnOpts),
    sync_start_link(?MODULE:spawn_opt(M, F, A, [link|SpawnOpts]), Timeout).
	
-define(VERIFY_NO_MONITOR_OPT(M, F, A, T, Opts),
        case lists:member(monitor, Opts) of
            true -> erlang:error(badarg, [M,F,A,T,Opts]);
            false -> ok
        end).

sync_start_link(Pid, Timeout) ->
    receive
	{ack, Pid, Return} ->
            Return;
	{'EXIT', Pid, Reason} ->
            {error, Reason}
    after Timeout ->
            kill_flush(Pid),
            {error, timeout}
    end.

spawn_opt(M, F, A, Opts) when is_atom(M), is_atom(F), is_list(A) ->
    Parent = get_my_name(),
    Ancestors = get_ancestors(),
    erlang:spawn_opt(?MODULE, init_p, [Parent,Ancestors,M,F,A], Opts).

init_p(Parent, Ancestors, M, F, A) when is_atom(M), is_atom(F), is_list(A) ->
    put('$ancestors', [Parent|Ancestors]),
    put('$initial_call', trans_init(M, F, A)),
    init_p_do_apply(M, F, A).

init_p_do_apply(M, F, A) ->
    try
	apply(M, F, A) 
    catch
	Class:Reason:Stacktrace ->
	    exit_p(Class, Reason, Stacktrace)
    end.

```
### 图示

![img.png](/pic/gen_server/img.png)

### 分析
可以看到我们主要用到的模块有
**gen_server**、**gen**、**proc_lib**、**erlang**

A进程从入口gen_server进来以后经过gen层  
gen:start(  
(这一层是OTP行为的通用层  
This module implements the really generic stuff of the generic standard behaviours (e.g. gen_server, gen_fsm))  

来到proc_lib层  
proc_lib:start_link(  
(这一层是erlang给我们封装好的开启**标准**进程的模块  
This module is used to set some initial information in each created process)

当进入proc_lib层以后sync_start_link会挂起A进程等待ACK响应  
接着来到erlang:spawn_opt(  
这里会新起一个spawn_opt进程执行gen模块里面的init_it函数给这个spawn_opt进程注册名字
```shell
init_it(GenMod, Starter, Parent, Name, Mod, Args, Options) ->
    case register_name(Name) of
	true ->
	    init_it2(GenMod, Starter, Parent, Name, Mod, Args, Options);
	{false, Pid} ->
	    proc_lib:init_ack(Starter, {error, {already_started, Pid}})
    end.

```
然后执行gen_server模块的init
```shell
init_it2(GenMod, Starter, Parent, Name, Mod, Args, Options) ->
    GenMod:init_it(Starter, Parent, Name, Mod, Args, Options).  
```
部分代码
```shell
init_it(Starter, Parent, Name0, Mod, Args, Options) ->
    Name = gen:name(Name0),
    Debug = gen:debug_options(Name, Options),
    HibernateAfterTimeout = gen:hibernate_after(Options),

    case init_it(Mod, Args) of
	{ok, {ok, State}} ->
	    proc_lib:init_ack(Starter, {ok, self()}), 	    
	    loop(Parent, Name, State, Mod, infinity, HibernateAfterTimeout, Debug);
```
这里会执行gen_server回调模块中的init函数  
```shell
init_it(Mod, Args) ->
    try
	{ok, Mod:init(Args)}
    catch
	throw:R -> {ok, R};
	Class:R:S -> {'EXIT', Class, R, S}
    end.
```
然后给A进程发送ACK响应结束A进程的挂起,至此gen_server进程初始化完毕，可以用gen_server:call给spawn_opt进程发消息了  

对于proc_lib的学习还可以参考cowboy里面的ranch模块，有一个简单的应用
## cowboy_ranch
ranch_conns_sup模块
```shell
start_link(Ref, Transport, Protocol) ->
	proc_lib:start_link(?MODULE, init,
		[self(), Ref, Transport, Protocol]).
```
下面是所有相关代码
```shell
=========================proc_lib==================
start_link(M, F, A) when is_atom(M), is_atom(F), is_list(A) ->
    start_link(M, F, A, infinity).

start_link(M, F, A, Timeout) when is_atom(M), is_atom(F), is_list(A) ->
    sync_start_link(?MODULE:spawn_link(M, F, A), Timeout).

sync_start_link(Pid, Timeout) ->
    receive
	{ack, Pid, Return} ->
            Return;
	{'EXIT', Pid, Reason} ->
            {error, Reason}
    after Timeout ->
            kill_flush(Pid),
            {error, timeout}
    end.

spawn_link(M,F,A) when is_atom(M), is_atom(F), is_list(A) ->
    Parent = get_my_name(),
    Ancestors = get_ancestors(),
    erlang:spawn_link(?MODULE, init_p, [Parent,Ancestors,M,F,A]).

init_p(Parent, Ancestors, M, F, A) when is_atom(M), is_atom(F), is_list(A) ->
    put('$ancestors', [Parent|Ancestors]),
    put('$initial_call', trans_init(M, F, A)),
    init_p_do_apply(M, F, A).

init_p_do_apply(M, F, A) ->
    try
	apply(M, F, A) 
    catch
	Class:Reason:Stacktrace ->
	    exit_p(Class, Reason, Stacktrace)
    end.

```
### 图示


![img_1.png](/pic/gen_server/img_1.png)


### 分析
对比gen_server的流程图可以发现有大部分的流程是一模一样的

![img_2.png](/pic/gen_server/img_2.png)

A进程挂起等待proc_lic给我们创建一个**标准**进程，给这个标准进程注册名字，标准进程执行回调模块的init函数，然后给A进程发送ACK响应结束A进程的挂起  

![img_3.png](/pic/gen_server/img_3.png)

















