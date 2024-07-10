---
title: Erlang实现LOL赛区匹配函数
date: 2024-03-23 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

Erlang实现LOL赛区匹配函数

<!--more-->


## 需求
    现有a,b,c,d四个赛区 每个赛区取前8名玩家,共计32名玩家,进行1V1 PK
    要保证1-4名和7-8名PK,不能一开始就让各赛区的第1名和第1名对决,影响后续收视率

### 图示
![img_1.png](/pic/division/img_1.png)

![img_2.png](/pic/division/img_2.png)

### 相关代码 

```
-module(division).
-define(DEFAULT_SEED_LIST, [1, 2, 3, 4]).
-define(DEFAULT_DIVISION_LIST, [1, 2, 3, 4]).
-define(GROUP_DIVISION_LIST, [
    [1, 2, 3, 4],
    [2, 3, 4, 1],
    [3, 4, 1, 2],
    [4, 1, 2, 3]
]).
-define(GROUP_SEED_LIST, [
    [5, 6, 7, 8],
    [6, 7, 8, 5],
    [7, 8, 5, 6],
    [8, 5, 6, 7]
]).

-record(group, {
    group_id = 1                     % 分组id
    , player_id1 = 0                 % 选手1
    , player_id2 = 0                 % 选手2
}).

-define(DEBUG(F), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE])).
-define(DEBUG(F, A), io:format("##[~w~w:~w] " ++ F ++ "~n", [self(), ?MODULE, ?LINE | A])).

-compile(export_all).

test_data() ->
    [
%% 亚洲赛区前8名 第三位是选手id
        {4, 8, 4810022},
        {4, 7, 4710009},
        {4, 6, 4610025},
        {4, 5, 4510033},
        {4, 4, 4410004},
        {4, 3, 4310018},
        {4, 2, 4210032},
        {4, 1, 4110026},

%% 北美赛区前8名 第三位是选手id
        {3, 8, 3810022},
        {3, 7, 3710009},
        {3, 6, 3610025},
        {3, 5, 3510033},
        {3, 4, 3410004},
        {3, 3, 3310018},
        {3, 2, 3210032},
        {3, 1, 3110026},

%% 韩国赛区前8名 第三位是选手id
        {2, 8, 2810022},
        {2, 7, 2710009},
        {2, 6, 2610025},
        {2, 5, 2510033},
        {2, 4, 2410004},
        {2, 3, 2310018},
        {2, 2, 2210032},
        {2, 1, 2110026},

%% 越南赛区前8名 第三位是选手id
        {1, 8, 1820017},
        {1, 7, 1720008},
        {1, 6, 1620001},
        {1, 5, 1520007},
        {1, 4, 1420018},
        {1, 3, 1320019},
        {1, 2, 1220011},
        {1, 1, 1120009}
    ].


%% lists:zip([a,b,c],[1,2,3]). ->  [{a,1},{b,2},{c,3}]
init_player_list() ->
    List = test_data(),
    %% key为赛区的数据格式转成key为名次的数据格式
    Fun =
        fun({Div, Seed, Id}, Acc) ->
            SeedMap = maps:get(Seed, Acc, #{}),
            Acc#{Seed => SeedMap#{Div => Id}}
        end,
    Map = lists:foldl(Fun, #{}, List),
    ?DEBUG("Map:~p end",[Map]),
    L1 = lists:zip(shuffle(?GROUP_DIVISION_LIST), shuffle(?GROUP_SEED_LIST)),
    ?DEBUG("L1:~p end",[L1]),
    make_group_list(#group{}, init_player_list(Map, L1, []), []).

init_player_list(_Map, [], List) ->
    List;
init_player_list(Map, [{DivList, SeedList} | Rest], List) ->

    SeedDivList1 = lists:sort(lists:zip(DivList, ?DEFAULT_SEED_LIST)),
    ?DEBUG("SeedDivList1:~p end",[SeedDivList1]),
    SeedDivList2 = lists:sort(lists:zip(?DEFAULT_DIVISION_LIST, SeedList)),
    ?DEBUG("SeedDivList2:~p end",[SeedDivList2]),
    %% 取1-4名 Id1是赛区1 Id2是赛区2 Id3是赛区3 Id4是赛区4
    [Id1, Id2, Id3, Id4] = [maps:get(Div, maps:get(Seed, Map)) || {Div, Seed} <- SeedDivList1],
    %% 取5-8名 Id5是赛区1 Id6是赛区2 Id6是赛区3 Id8是赛区4
    [Id5, Id6, Id7, Id8] = [maps:get(Div, maps:get(Seed, Map)) || {Div, Seed} <- SeedDivList2],
    %% 赛区规避原则
    [Id11, Id33] = shuffle([Id1, Id3]),
    [Id66, Id88] = shuffle([Id6, Id8]),
    [Id22, Id44] = shuffle([Id2, Id4]),
    [Id55, Id77] = shuffle([Id5, Id7]),
    %%前4名和后4名打     赛区1 3 和赛区2 4打    赛区2 4 和赛区1 3打
    init_player_list(Map, Rest, [Id11, Id66, Id33, Id88, Id22, Id77, Id44, Id55 | List]).


make_group_list(_, [], GroupList) ->
    lists:reverse(GroupList);
make_group_list(_, [_], _GroupList) ->
    [];
make_group_list(#group{group_id = GId} = Base, [Id1, Id2 | Rest], List) ->
    Group = Base#group{player_id1 = Id1, player_id2 = Id2},
    make_group_list(Base#group{group_id = GId + 1}, Rest, [Group | List]).

%%=======================================================================
%% @doc 打乱list的顺序
shuffle(L) when is_list(L) ->
    List1 = [{rand:uniform(), X} || X <- L],
    List2 = lists:keysort(1, List1),
    [E || {_, E} <- List2];
shuffle(L) ->
    ?DEBUG("err Args:~w", [L]),
    [].

```






















