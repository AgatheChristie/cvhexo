---
title: erlang data2csv
date: 2024-08-04 22:21:13
tags:
- erlang
categories:
- Erlang
---

# 概述

用excel导出的erlang data文件重新转回csv

<!--more-->


## 需求
    用excel导出的erlang data文件重新转回csv


### 数据文件

```
%%===================================================
%%作    者：泰奇  https://agathechristie.github.io/  QQ：544682223
%%创建时间：2024-01-24 17:51:20
%%备    注：此代码为工具生成 请勿手工修改
%%===================================================
-module(scene_data).
-include("common.hrl").
-include("scene.hrl").
-export([list/0, get/1]).
list() -> [100, 101, 102, 103].
get(100) ->
    #ets_scene{
        sid = 1001,
        type = 0,
        name = ?T("仙灵岛"),
        x = 15,
        y = 17,
        safe = [0, 0, 0, 0],
        npc = [[21034, 29, 28], [10103, 21, 22], [10109, 43, 40]],
        mon = [[40004, 19, 65, 1], [44104, 10, 110, 1], [40004, 22, 61, 1]],
        elem = [],
        requirement = [{item, 0}, {lv, 0}, {time, 0}],
        id = 100
    };
get(101) ->
    #ets_scene{
        sid = 101,
        type = 1,
        name = ?T("雷泽"),
        x = 102,
        y = 214,
        safe = [87, 150, 1, 0],
        npc = [[21034, 29, 28], [10103, 21, 22], [10109, 43, 40]],
        mon = [[40004, 19, 65, 1], [44104, 10, 110, 1], [40004, 22, 61, 1]],
        elem = [[1, 119, ?T("野外竞技场"), 46, 128], [2, 300, ?T("九霄"), 81, 18], [3, 251, ?T("太昊城城郊"), 94, 222], [4, 201, ?T("娲皇城城郊"), 102, 203], [5, 281, ?T("华阳城城郊"), 110, 223]],
        requirement = [{item, 0}, {lv, 0}, {time, 0}],
        id = 101
    };
get(102) ->
    #ets_scene{
        sid = 102,
        type = 1,
        name = ?T("洛水"),
        x = 106,
        y = 57,
        safe = [0, 0, 0, 0],
        npc = [[21034, 29, 28], [10103, 21, 22], [10109, 43, 40]],
        mon = [[40004, 19, 65, 1], [44104, 10, 110, 1], [40004, 22, 61, 1]],
        elem = [[1, 103, ?T("苍莽林"), 5, 13], [2, 300, ?T("九霄"), 110, 12]],
        requirement = [{item, 0}, {lv, 0}, {time, 0}],
        id = 102
    };
get(103) ->
    #ets_scene{
        sid = 103,
        type = 1,
        name = ?T("苍莽林"),
        x = 52,
        y = 183,
        safe = [0, 0, 0, 0],
        npc = [[21034, 29, 28], [10103, 21, 22], [10109, 43, 40]],
        mon = [[40004, 19, 65, 1], [44104, 10, 110, 1], [40004, 22, 61, 1]],
        elem = [[1, 102, ?T("洛水"), 15, 19]],
        requirement = [{item, 0}, {lv, 0}, {time, 0}],
        id = 103
    };
get(_SceneId) ->
    undefined.
```
```
%%===================================================
%%作    者：泰奇  https://agathechristie.github.io/  QQ：544682223
%%创建时间：2024-01-24 17:51:20
%%备    注：此代码为工具生成 请勿手工修改
%%===================================================
-module(base_realm_data).
-include("common.hrl").
-export([all/0, get_id/1]).
all() -> [1, 2, 3, 100].
get_id(1) -> #{
    id => 1,
    name => ?T("女娲")
};
get_id(2) -> #{
    id => 2,
    name => ?T("神农")
};
get_id(3) -> #{
    id => 3,
    name => ?T("伏羲")
};
get_id(100) -> #{
    id => 100,
    name => ?T("新手")
};
get_id(_Id) ->
    undefined.
```


### 主文件
```
-module(csv_gen_tests).
-include("scene.hrl").
-export([
    base_realm_data/0
    , scene_data/0
    , gen_all/0
]).
gen_all() ->
    scene_data(),
    base_realm_data().

%% 生成地图场景表
scene_data() ->
    gen_record_data([?FUNCTION_NAME], record_info(fields, ets_scene), get),
    ok.

%% 生成阵营表
base_realm_data() ->
    gen_map_data([?FUNCTION_NAME], get_id),
    ok.

gen_map_data([], _F) ->
    ok;
gen_map_data([Mod | T], F) ->
    {ok, File} = file:open(atom_to_list(Mod) ++ ".csv", [write]),
    A = [FirstId | _] = get_all_ids(Mod),
    FirstMap = Mod:F(FirstId),
    csv_gen:row(File, maps:keys(FirstMap)),
    get_map_data_(A, File, Mod, F),
    file:close(File),
    gen_map_data(T, F).

get_all_ids(Mod) ->
    case catch Mod:list_ids() of
        TmpA when is_list(TmpA) ->
            TmpA;
        _ ->
            case catch Mod:list() of
                TmpB when is_list(TmpB) ->
                    TmpB;
                _ ->
                    Mod:all()
            end

    end.

gen_record_data([], _RecordName, _F) ->
    ok;
gen_record_data([Mod | T], RecordName, F) ->
    {ok, File} = file:open(atom_to_list(Mod) ++ ".csv", [write]),
    csv_gen:row(File, RecordName),
    AllIds = get_all_ids(Mod),
    gen_record_data_(AllIds, File, Mod, F),
    file:close(File),
    gen_record_data(T, RecordName, F).


get_map_data_([], _File, _Mod, _F) ->
    ok;
get_map_data_([H | T], File, Mod, F) ->
    case Mod:F(H) of
        A when is_map(A) ->
            B = maps:values(A),
            csv_gen:row(File, B);
        _E ->
            ok
    end,
    get_map_data_(T, File, Mod, F).

gen_record_data_([], _File, _Mod, _F) ->
    ok;
gen_record_data_([H | T], File, Mod, F) ->
    case Mod:F(H) of
        A when is_tuple(A) ->
            [_ | B] = tuple_to_list(A),
            csv_gen:row(File, B);
        _E ->
            ok
    end,
    gen_record_data_(T, File, Mod, F).
```

```
-module(csv_gen).
-export([newline/1, comma/1, field/2, row/2]).
newline(File) ->
  file:write(File, "\n").

comma(File) ->
  file:write(File, ",").

field(File, Value) when is_tuple(Value) ->
  file:write(File, "\""),
  file:write(File, io_lib:format("~p",[Value])),
  file:write(File, "\"");
field(File, Value) when is_binary(Value) ->
  Match = binary:match(Value, [<<",">>, <<"\n">>, <<"\"">>]),
  case Match of
    nomatch ->
      file:write(File, Value);
    _ ->
      file:write(File, "\""),
      file:write(File, binary:replace(Value, <<"\"">>, <<"\"\"">>, [global])),
      file:write(File, "\"")
  end;
field(File, Value) when is_list(Value) ->
%%  field(File, unicode:characters_to_binary(Value));
%%  field(File, term_to_binary(Value));
  field(File, list_to_tuple(Value));
field(File, Value) when is_integer(Value) ->
  file:write(File, integer_to_list(Value));
field(File, Value) when is_atom(Value) ->
  file:write(File, io_lib:write_atom(Value));
field(File, Value) when is_float(Value) ->
  file:write(File, io_lib:format("~f",[Value]));
field(File, Value) ->
  file:write(File, io_lib:format("\"~p\"",[Value])).

row(File, Elem) when is_tuple(Elem) ->
  ListElem = tuple_to_list(Elem),
  row(File, ListElem);
row(File, []) ->
  newline(File);
row(File, [Value]) ->
  field(File, Value),
  newline(File);
row(File, [Value | Rest]) ->
  field(File, Value),
  comma(File),
  row(File, Rest).
```
### 头文件
```
common.hrl
-define(T(Text), (<<Text/utf8>>)).

scene.hrl
-record(ets_scene, {
    sid,                                    %% 场景id
    type = 0,                               %% 0:城镇 1:野外场景, 2:副本, 5:氏族领地, 6:竞技场, 7:封神台,8:秘境,9:神岛,
    name,                                   %% 场景名称
    x = 0,                                  %% 默认x坐标
    y = 0,                                  %% 默认y坐标
    requirement,                            %% 进入需求
    elem,                                   %% 场景元素
    npc,                                    %% 场景NPC数据结构,
    mon,                                    %% 场景怪物数据结构,
    mask,                                   %% 场景移动坐标信息
    safe,                                   %% 安全区
    id = 0                                  %% 场景唯一ID
}).

```


## 图示
![img.png](/pic/e2csv/img.png)

![img_1.png](/pic/e2csv/img_1.png)

![img_2.png](/pic/e2csv/img_2.png)









