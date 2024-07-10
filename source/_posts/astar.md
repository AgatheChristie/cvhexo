---
title: Erlang实现A星寻路
date: 2024-03-24 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

Erlang实现A星寻路

<!--more-->


## 需求
    Erlang实现A星寻路

### 图示
![img.png](/pic/astar/img.png)

![img_1.png](/pic/astar/img_1.png)

### 相关代码 

```
-module(a_sweep_trace2).
-export([
    generate_route/8
    , generate_route/7
    , generate_route/5
    , expand_points/2
]).

%% 八个方向  走九宫格
-define(EXPAND_POINTS_8(X, Y), [
    {X - 1, Y}, {X - 1, Y - 1},
    {X, Y - 1}, {X + 1, Y - 1},
    {X + 1, Y}, {X + 1, Y + 1},
    {X, Y + 1}, {X - 1, Y + 1}
]).

%% 六个方向  走六宫格(地图为蜂窝六边形)
-define(EXPAND_POINTS_6(X, Y), [
    {X, Y + 1}, {X + 1, Y},
    {X + 1, Y - 1}, {X, Y - 1},
    {X - 1, Y}, {X - 1, Y + 1}
]).

%% 四个方向  走十字
-define(EXPAND_POINTS_4(X, Y), [
    {X - 1, Y}, {X + 1, Y},
    {X, Y + 1}, {X, Y - 1}
]).

%% 障碍物
-define(BLOCKS, [{3, 4}, {4, 3}, {3, 2}, {1, 2}]).


%% 扩展查找的点
-record(point, {
    x = 0 :: integer(),   %% 当前点位X
    y = 0 :: integer(),   %% 当前点位Y
    g = 0 :: integer(),   %% G = 走过的步数
    h = 0 :: integer(),   %% H = 曼哈顿距离
    f = 0 :: integer(),   %% F = G + H
    p_x = 0 :: integer(), %% 前一个位置
    p_y = 0 :: integer()  %% 前一个位置
}).

expand_points(X, Y) ->
    ?EXPAND_POINTS_4(X, Y).

-define(DEBUG(F), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE])).
-define(DEBUG(F, A), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE | A])).

%% a_sweep_trace2:generate_route(1,3,5,2,1).

%% [{2,3},{3,3},{4,3},{4,2},{5,2}] 没有墙
%% [{2,3},{2,2},{2,1},{3,1},{4,1},{4,2},{5,2}] 有墙
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
            Solution = generate_solution(Point, [], ResultClosePoints ++ ResultOpenPoints),
            PosList = [{LX, LY} || #point{x = LX, y = LY} <- Solution],
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

%% 坐标点扩充
expand(OpenPoints, ClosePoints, #point{x = X, y = Y, g = G}, #point{x = TargetX, y = TargetY}, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) ->
    CostG = G + 1,
    lists:foldl(fun({X1, Y1}, AccIn) ->
        case is_blocked(X1, Y1, TargetX, TargetY, SceneId, MaxTraceRangeX, MaxTraceRangeY, Flying) of
            true ->
                AccIn;
            false ->
                case lists:any(fun(#point{x = OpentX, y = OpentY}) ->
                    (OpentX == X1 andalso OpentY == Y1) end, OpenPoints) of
                    false ->
                        case lists:any(fun(#point{x = CloseX, y = CloseY}) ->
                            (CloseX == X1 andalso CloseY == Y1) end, ClosePoints) of
                            false ->
                                CostH = calculate_cost(X1, Y1, TargetX, TargetY),
                                [#point{x = X1, y = Y1, g = CostG, h = CostH, f = CostH + CostG, p_x = X, p_y = Y} | AccIn];
                            true ->
                                %% 已在 close 列表 中不加
                                AccIn
                        end;
                    true ->
                        %% 已在 opent 列表 中不加
                        AccIn
                end
        end
                end, [], ?EXPAND_POINTS_4(X, Y)).

%% 生成结果列表
generate_solution(Point = #point{p_x = X, p_y = Y}, SolutionPoints, Points) ->
    %% 找到终点前面的格子A 找到格子A前面的格子B 找到格子B前面的格子C
    PPoints = [Info || Info = #point{x = X2, y = Y2} <- Points, X2 == X, Y2 == Y],
    case PPoints of
        [] ->
            SolutionPoints;
        [One] ->
            generate_solution(One, [Point | SolutionPoints], lists:delete(One, Points));
        [NewPoint | _] = Tmp ->
            %% 一般来说不会走到这里来
            ?DEBUG("Tmp:~p end", [Tmp]),
            generate_solution(NewPoint, [Point | SolutionPoints], lists:delete(NewPoint, Points))
    end.

%% 权重计算
calculate_cost(X1, Y1, X2, Y2) ->
    ((X1 - X2) * (X1 - X2) + (Y1 - Y2) * (Y1 - Y2)).

%% 是否到达目标点了,到达目标周围点就可以了
is_target(#point{x = StartX, y = StartY}, #point{x = TargetX, y = TargetY}) ->
    StartX == TargetX andalso StartY == TargetY.
%%	abs(StartX - TargetX) =< 1 andalso abs(StartY - TargetY) =<1.


%% 检查是否是边界或者障碍物 {3,3},  {5,2}
is_blocked(X, Y, TargetX, TargetY, _SceneId, MaxTraceRangeX, MaxTraceRangeY, _Flying) ->
    case abs(TargetX - X) > MaxTraceRangeX orelse
        abs(TargetY - Y) > MaxTraceRangeY of
        true ->
            %% 在查找范围外，作为阻档处理
            true;
        false ->
            %% 在查找范围内
            lists:member({X, Y}, ?BLOCKS)
    end.

```
