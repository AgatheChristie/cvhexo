---
title: 一些草稿
date: 2024-03-24 22:21:13
tags:
- erlang
categories:
- Erlang
password: 
- ccc
---

# 概述

一些草稿

<!--more-->


## 需求
    一些草稿




### 相关代码 

```
-module(a_sweep_trace).
-export([generate_route/8, generate_route/7, generate_route/5]).


-define(EXPAND_POINTS(X, Y), [{X - 1, Y}, {X - 1, Y - 1}, {X, Y - 1}, {X + 1, Y - 1}, {X + 1, Y}, {X + 1, Y + 1}, {X, Y + 1}, {X - 1, Y + 1}]).


%% 扩展查找的点
-record(point, {
    x = 0 :: integer(),
    y = 0 :: integer(),
    g = 0 :: integer(),   %% G = 走过的步数
    h = 0 :: integer(),   %% H = 曼哈顿距离
    f = 0 :: integer(),   %% F = G + H
    p_x = 0 :: integer(), %% 前一个位置
    p_y = 0 :: integer()  %% 前一个位置
}).



-define(DEBUG(F), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE])).
-define(DEBUG(F, A), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE | A])).

%% a_sweep_trace2:generate_route(1,3,5,2,1).
generate_route(StartX, StartY, TargetX, TargetY, SceneId) ->
    generate_route(StartX, StartY, TargetX, TargetY, SceneId, 50, 50).

generate_route(StartX, StartY, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY) ->
    generate_route(StartX, StartY, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY, 0).

generate_route(StartX, StartY, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) ->
%% 初始化开始点坐标
    CostH = calculate_cost(StartX, StartY, TargetX, TargetY),
    CostG = 0,
    OpenPoints = [#point{x = StartX, y = StartY, g = CostG, h = CostH, f = CostG + CostH}],
%% 目标点坐标
    case search(OpenPoints, [], #point{x = TargetX, y = TargetY}, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) of
        false ->
            false;
        {Point, ResultOpenPoints, ResultClosePoints} ->
            AA = generate_solution(Point, [], ResultClosePoints ++ ResultOpenPoints),
            PosList = [{P#point.x, P#point.y} || P <- AA],
            PosList
    end.

%% 查找目标点
%% param: OpenPoints 扩充列表,closePoints 已遍历列表,TargetPoint 目标点
search([], _ClosePoints, _TargetPoint, _SceneId, _MaxTraceRangeX, _MaxTraceRangeY, _Flying) ->
    false;
search(OpenPoints, ClosePoints, TargetPoint, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) ->
    ?DEBUG("OpenPoints:~p end", [OpenPoints]),
    [Point | NewOpenPoints] = lists:keysort(#point.f, OpenPoints),
    NewClosePoints = [Point | ClosePoints],
    case is_target(Point, TargetPoint) of
        true ->
            {Point, NewOpenPoints, NewClosePoints};
        false ->
            %% 未找到，
            AddOpenPoints = expand(NewOpenPoints, NewClosePoints, Point, TargetPoint, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying),
            %% 坐标点扩充
            search(AddOpenPoints ++ NewOpenPoints, NewClosePoints, TargetPoint, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying)
    end.


%% 生成结果列表
generate_solution(#point{p_x = X, p_y = Y} = Point, SolutionPoints, Points) ->
    case [Info || #point{x = X2, y = Y2} = Info <- Points, X2 == X, Y2 == Y] of
        [] ->
            SolutionPoints;
        [NewPoint | _] ->
            generate_solution(NewPoint, [Point | SolutionPoints], lists:delete(NewPoint, Points))
    end.

%% 权重计算
calculate_cost(X1, Y1, X2, Y2) ->
    ((X1 - X2) * (X1 - X2) + (Y1 - Y2) * (Y1 - Y2)).

%% 是否到达目标点了,到达目标周围点就可以了
is_target(#point{x = StartX, y = StartY}, #point{x = TargetX, y = TargetY}) ->
    StartX == TargetX andalso StartY == TargetY.

%%	abs(StartX - TargetX) =< 1 andalso abs(StartY - TargetY) =<1.

%% check is block
is_blocked(X, Y, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY, _Flying) ->
    case abs(TargetX - X) > MaxTraceRangeX orelse abs(TargetY - Y) > MaxTraceRangeY of
        true ->
            %% 在查找范围外，作为阻档处理
            true;
        false ->
            %% 在查找范围内
            not is_block(SceneId, X, Y)
    end.

is_block(_SceneId, _X, _Y) ->
    false.


%% 坐标点扩充
expand(OpenPoints, ClosePoints, #point{x = X, y = Y, g = G}, #point{x = TargetX, y = TargetY}, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) ->
    CostG = G + 1,
    lists:foldl(fun({X1, Y1}, AccIn) ->
        case is_blocked(X1, Y1, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) of
            true ->
                AccIn;
            false ->
                case [true || #point{x = OpentX, y = OpentY} <- OpenPoints, OpentX == X1, OpentY == Y1] of
                    [] ->
                        case [true || #point{x = CloseX, y = CloseY} <- ClosePoints, CloseX == X1, CloseY == Y1] of
                            [] ->
                                CostH = calculate_cost(X1, Y1, TargetX, TargetY),
                                [#point{x = X1, y = Y1, g = CostG, h = CostH, f = CostH + CostG, p_x = X, p_y = Y} | AccIn];
                            _ ->
                                %% 已在 close 列表 中不加
                                AccIn
                        end;
                    _ ->
                        %% 已在 opent 列表 中不加
                        AccIn
                end
        end
                end, [], ?EXPAND_POINTS(X, Y)).


```



