---
title: linux下使xx.log输出颜色
date: 2024-08-04 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

linux下使log输出颜色

<!--more-->


## 需求
    在linux下让log中的DEBUG,INFO,ERROR输出不同颜色


### 相关代码 

```
文件1
logger.hrl
-define(log_color_none   , "\e[m").
-define(log_color_red    , "\e[1m\e[31m").
-define(log_color_yellow , "\e[1m\e[33m").
-define(log_color_green  , "\e[0m\e[32m").
-define(log_color_black  , "\e[0;30m").
-define(log_color_blue   , "\e[0;34m").
-define(log_color_purple , "\e[0;35m").
-define(log_color_cyan   , "\e[0;36m").
-define(log_color_white  , "\e[0;37m").

-ifdef(DEBUG_LOG).
-define(DEBUG(Format, Args),
    lager:dispatch_log(lager_event, debug, [{pid, self()}, {module, ?MODULE}, {line, ?LINE}],
        Format, Args, 4096, safe)).
-else.
-define(DEBUG(Format, Args), ok).
-endif.


-ifdef(DEBUG_LOG).
-define(INFO(Format, Args),
    lager:dispatch_log(lager_event, info, [{pid, self()}, {module, ?MODULE}, {line, ?LINE}],
        lists:concat([?log_color_blue, Format,?log_color_none]), Args, 4096, safe)).
-else.
-define(INFO(Format, Args),
    lager:dispatch_log(lager_event, info, [{pid, self()}, {module, ?MODULE}, {line, ?LINE}],
        Format, Args, 4096, safe)).
-endif.


-ifdef(DEBUG_LOG).
-define(ERROR(Format, Args),
    lager:dispatch_log(lager_event, error, [{pid, self()}, {module, ?MODULE}, {line, ?LINE}],
        lists:concat([?log_color_red, Format,?log_color_none]), Args, 409600, safe)).
-else.
-define(ERROR(Format, Args),
    lager:dispatch_log(lager_event, error, [{pid, self()}, {module, ?MODULE}, {line, ?LINE}],
        Format, Args, 409600, safe)).
-endif.

文件2
-module(test).

test_color() ->
    %% 蓝色
    ?INFO("blueblueblue~n"),
    %% 红色
    ?ERROR("redredred~n"),
    %% 无颜色
    ?DEBUG("nonono~n"),
    ok.


```
## 注意事项
### 1.使用-ifdef(DEBUG_LOG).判断是否在log中写入颜色
这个宏定义一般写在 rebar.config中
{erl_opts, [
    debug_info,
    {d, 'DEBUG_LOG'}
     ]}.
生产环境不建议开启颜色不然会导致log文件增大
将生产环境中rebar.config去掉{d, 'DEBUG_LOG'}就好了

### 2.在linux下tail可以正常显示颜色,vim无法显示颜色
解决方案  
vim需要升级到8.1以上开启terminal功能
vim log/test.log后也是无法正常显示颜色的还需要打开新的窗口
在vim模式下输入
:tab term cat log/test.log
即可正常显示颜色

### 3.VIM模式下查看"\e[1m\e[31m"颜色
新终端窗口
:tab term cat log/console.log
新终端窗口左分割
:leftabove vert term cat log/console.log
横向分割
:vert term cat log/console.log
新终端窗口右分割
:rightbelow vert term cat log/console.log
竖向分割
:term cat log/console.log






