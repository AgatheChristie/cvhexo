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















