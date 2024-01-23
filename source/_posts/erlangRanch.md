---
title: cowboy Ranch1.8.0
date: 2023-12-01 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

erlang cowboy ranch study

<!--more-->


```


werl -name awdaf@127.0.0.1 -setcookie myproj_cookie

    ok = start_application(crypto),
    ok = start_application(asn1),
    ok = start_application(public_key),

    %% ssl 依赖上面那三个
    ok = start_application(ssl),
    %% ranch 依赖ssl
    ok = start_application(ranch),
	
1. ranch app 开始 	
	ok = application:start(ranch).
	
	
	第一个监控树  ranch_sup  supervisor:start_link({local, ?MODULE}, ?MODULE, []). 注册名字

	
	
	创建一个 ETS ranch_server
ranch_server 归属权不重要 因为是public的
	ranch_server = ets:new(ranch_server, [
		ordered_set, public, named_table]),

起 ranch_server 进程 注册名字 gen_server:start_link({local, ?MODULE}, ?MODULE, [], []).
结束




2. 开启echo app 



 application:start(tcp_echo).
start(_Type, _Args) ->
	{ok, _} = ranch:start_listener(tcp_echo,
		ranch_tcp, #{socket_opts => [{port, 5555}]},
        echo_protocol, []),
	tcp_echo_sup:start_link().

		
在  ranch_sup 树下挂起 ranch_listener_sup 监控树  不注册
	
在ranch_server下写入一些数据到ETS




在  ranch_listener_sup 树下挂起 ranch_conns_sup 监控树 





handle_call({set_new_listener_opts, Ref, MaxConns, TransOpts, ProtoOpts, StartArgs}, _, State) ->
	ets:insert_new(?TAB, {{max_conns, Ref}, MaxConns}),
	ets:insert_new(?TAB, {{trans_opts, Ref}, TransOpts}),
	ets:insert_new(?TAB, {{proto_opts, Ref}, ProtoOpts}),
	ets:insert_new(?TAB, {{listener_start_args, Ref}, StartArgs}),
	{reply, ok, State};





	
Ref	             tcp_echo,                       
Transport    	 ranch_tcp, 
TransOpts0	     #{socket_opts => [{port, 5555}]},    
Protocol         echo_protocol, 
ProtoOpts        []                    


```

```
第二个监控树 ranch_acceptors_sup

大部分方法都是在 ranch_tcp 里面调


```

```
客户端连接以后
Transport:accept(LSocket, infinity)


握手


Socket转交给  ranch_conns_sup

echo_protocol:start_link

大部分方法都是在 ranch_tcp 里面调


```





