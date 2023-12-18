---
title: erlang学习
date: 2023-11-21 20:18:51
tags:
- erlang
categories:
- Erlang
---

# 概述

erlang学习

<!--more-->

## Q1

protobuf_pb
encode_public_success_s2c
decode_public_success_s2c
1001001
0
#public_success_s2c{}
%% 协议组合
-record(sys_protobuf_pb, {
pb_mod = "",                                                          %% 协议模块
encode_pb = "",                                                       %% encode协议模块
decode_pb = "",                                                       %% decode协议模块
cor_cmd_id = 0,                                                       %% 对应协议id
is_public = 0,                                                        %% 是否公共协议
pb_record = none                                                      %% 对应record
}).



protobuf_pb.beam 这个文件没有
解析DataBin 用的也还是传统的方式
<<Len:?PROTOBUF_MSG_LEN, MsgSeq:?PROTOBUF_SEQ_LEN, MsgId:?PROTOBUF_MSG_ID, TDataBin/binary>> = BinData,
ByteSize = erlang:byte_size(TDataBin),
先取到2位(16bit)的长度 再取 xxx 再取XXX

<<Rest:Len/binary, TRest/binary>> = TDataBin,
DataBin = Protobuf:DecodePb(Rest),







## E1
日志模块使用
lager(依赖goldrush)


打包模块用到了
msgpack


## C1


### ETS_GOODS_ONLINE 不是配置表

玩家进入游戏

**mod_player**这条进程
init时会拉起mod_goods进程

**mod_goods** 进程init时
init_goods(PlayerId)


### ETS_BASE_SHOP  ETS_BASE_GOODS 是配置表

**mod_kernel**这条进程
init时会把所有配置表从DB导入到ETS
为什么要存在数据库里面？然后还导到ETS？
为什么不直接用配置表呢？
猜测是为了用ETS里面的一些匹配方法，比如ets:match_object(Tab, Pattern),
如果是配置表就要自己写筛选方法了，但是不占用DB和ETS内测呀！！！

### 背包系统 增加道具  扣除道具  

mod_goods进程state

null_cells = [1,4,6,22,31] %% 空位















