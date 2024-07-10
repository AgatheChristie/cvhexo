---
title: Erlang list笔记
date: 2023-11-21 20:18:51
tags:
- erlang
categories:
- Erlang
---

# 概述

整理一些erlang list的api

<!--more-->

## 一，带函数Pred
### 1, all(Pred, List) -> boolean()
如果List中的每个元素作为Pred函数的参数执行，结果都返回true，那么all函数返回true，
否则返回false

例子：

lists:all(fun(E) -> true end,[1,2,3,4]).

结果

true


### 2, any(Pred, List) -> boolean()
如果List中至少有一个元素作为Pred函数的参数执行，结果返回true，那么any函数返回true，
否则返回false

例子

lists:any(fun(E) -> is_integer(E) end,[q,2,a,4]).

结果

true



### 3，dropwhile(Pred, List1) -> List2
将List1列表中的元素作为参数执行Pred函数，如果返回true，将其丢弃，最后返回剩余元素
组成的列表

例子

lists:dropwhile(fun(E) -> is_atom(E) end,[a,1,2,a,b]).

结果

[1,2,a,b]

### 4，filter(Pred, List1) -> List2
返回一个列表，这个列表是由List1中执行Pred函数返回true的元素组成。

lists:filter(fun(E) -> is_integer(E) end,[q,2,a,4]).

结果：

[2,4]



### 5，map(Fun, List1) -> List2
将List1中的每个元素去在Fun中执行，然后返回一个元素，最后返回的这些元素组成一个列表，
返回给List2
例子：
lists:map(fun(X)->[X,X] end, [a,b,c]).
结果：[[a,a],[b,b],[c,c]]



### 6，flatmap(Fun, List1) -> List2
这个函数和map比较类似，相当于执行了
lists:append(lists:map(List1)).
也就是把map的结果进行append处理
例子：
lists:flatmap(fun(X)->[X,X] end, [a,b,c]).
结果：[a,a,b,b,c,c]



### 7，foldl(Fun, Acc0, List) -> Acc1
Fun这个函数有两个参数
第一个参数是List中的元素，第二个参数是Fun函数执行完后的返回值，这个参数第一次执行时

就是Acc0

感觉原文这里表达有点问题，比较适合的说法应该是当List为空后，返回值是AccN（第二个参数），

lists：foldl源码：

foldl(F, Accu, [Hd|Tail]) ->
foldl(F, F(Hd, Accu), Tail);
foldl(F, Accu, []) when is_function(F, 2) -> Accu.


例子：对[1,2,3,4,5]求和
lists:foldl(fun(X, Sum) -> X + Sum end, 0, [1,2,3,4,5]).
结果：15
执行过程：首先，Fun第一次执行时，X的值取列表List的第一个元素1，Sum取0,
Fun第二次执行时，X的值取列表List的第二个元素2，Sum取Fun第一次的返回值
依次轮推，直到List中每个元素执行完，最后foldl返回最后一次的结果。



### 8，foldr(Fun, Acc0, List) -> Acc1
foldr这个函数和foldl比较相似
不过是Fun执行时，X的值先取List的最后一个，然后取倒数第二个。



### 9，foreach(Fun, List) -> ok
以List中的每个元素为参数执行Fun函数，执行顺序按照List中元素的顺序，这个函数最后返回ok。是单边的
例子 lists:foreach(fun(X)->
%%using X to do somethings
%%
end,List)



### 10，keymap(Fun, N, TupleList1) -> TupleList2
对TupleList1中的每个元素的第N项作为参数在Fun中处理，然后这个第N项最后就被替换为Fun执行完返回的值
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}].
lists:keymap(fun(X)->
list_to_atom(X)
end,2,List1).
结果：
[{name,zhangjing},{name,zhangsan}]


### 11，mapfoldl(Fun, Acc0, List1) -> {List2, Acc1}
这个函数等于是把map和foldl函数结合起来。将List1中的每一个元素执行Fun函数，执行后花括号的第一个值作为返回值返回，
第二个值作为参数传给Fun，作为下一次用。
例子：
lists:mapfoldl(fun(X, Sum) -> {2*X, X+Sum} end,
0, [1,2,3,4,5]).
{[2,4,6,8,10],15}



### 12，mapfoldr(Fun, Acc0, List1) -> {List2, Acc1}
这个函数相当于将map和foldr结合起来


### 13，merge(Fun, List1, List2) -> List3
这个函数的功能也是把List1和List2合并到一起，只不过是List1和List2的元素要作为参数在Fun中执行，如果
Fun返回true，那么返回值就是List1在前，List2在后。否则，反之。
例子
lists:merge(fun(A,B)-> false end, [3,4],[2,1]).
结果
[2,1,3,4]



### 14，partition(Pred, List) -> {Satisfying, NotSatisfying}
这个函数的功能是将List分成两个List1和List2，List1是将List元素作为参数去Pred函数中执行返回true的元素组成，
List2由Pred返回false的元素组成。
注意，返回的是一个元组
例子
lists:partition(fun(A) -> A rem 2 == 1 end, [1,2,3,4,5,6,7]).
结果
{[1,3,5,7],[2,4,6]}


### 15，sort(Fun, List1) -> List2
如果Fun函数返回true，则排序是从小到大的顺序，否则，从大到小。
其中Fun有两个参数。
例子
lists:sort(fun(A,B)-> false end,[1,2,3]).
结果
[3,2,1]


### 16，splitwith(Pred, List) -> {List1, List2}
将List分成List1和List2，
List1由List中元素在Pred函数返回true的组成，但是有一点，如果遇到为false的，则将剩下的元素
全部放到List2中，List1中就只有前面为true的。
例子
lists:splitwith(fun(A) -> is_atom(A) end, [a,b,1,c,d,2,3,4,e]).
结果
{[a,b],[1,c,d,2,3,4,e]}


### 17，takewhile(Pred, List1) -> List2
List1中的元素element依次执行Pred(element),如果返回true，则获取这个元素，直到有元素执行Pred(element)返回false
例子
lists:takewhile(fun(E)-> is_atom(E) end,[a,b,1,e,{c},[d]]).
结果
[a,b]


### 18,umerge(Fun, List1, List2) -> List3
这个函数和merge不同的是 当Fun返回true时，返回的List3中不能出现相同的元素
疑问：但是当Fun返回false时，List3中可以有相同的元素。
例子(Fun返回true的情况)
lists:umerge(fun(A,B)-> true end,[1,2],[2,3]).
结果
[1,2,3]
(Fun为false的情况)
lists:umerge(fun(A,B)-> false end,[1,2],[2,3]).
[2,3,1,2]
好神奇，竟然2有重复



### 19，usort(Fun, List1) -> List2
按照Fun函数进行排序，如果Fun返回true，那么只返回List1的第一个元素
如果Fun返回false，那么List1从大到小排序
例子1
lists:usort(fun(A,B) -> true end, [1,2,2,3,4]).
结果
[1]

例子2
lists:usort(fun(A,B) -> false end, [1,2,2,3,4]).
结果
[4,3,2,2,1]


### 20，zipwith(Combine, List1, List2) -> List3
将List1和list2中的每个元素执行Combine函数，然后返回一个元素，List3就是由Combine函数返回的一个个元素组成的。
功能和map有点像，但是这里是对两个列表的操作。
例子
lists:zipwith(fun(X, Y) -> X+Y end, [1,2,3], [4,5,6]).
结果
[5,7,9]



### 21，zipwith3(Combine, List1, List2, List3) -> List4
将List1和list2，list3中的每个元素执行Combine函数，然后返回一个元素，List4就是由Combine函数返回的一个个元素组成的。
功能和map有点像，但是这里是对三个列表的操作。
例子
lists:zipwith3(fun(X, Y, Z) -> X+Y+Z end, [1,2,3], [4,5,6],[7,8,9]).
结果
[12,15,18]


## 二，不带函数Pred

### 1，append(ListOfLists) -> List1
ListOfLists都是由List组成的，而List一个列表，里面可以是任何类型的元素
这个函数就是将ListOfLists里面的所有列表的元素按顺序编成一个列表
提示：ListOfLists里面的元素必须都是列表才能用这个函数

例子

lists:append([[1, 2, 3], [a, b], [4, 5, 6]]).

结果：

[1,2,3,a,b,4,5,6]


### 2，append(List1, List2) -> List3
将List1和List2两个列表连接起来，组成一个列表，然后返回新的这个列表
这个函数的功能等同于List1 ++ List2

例子

lists:append("abc", "def").

结果

"abcdef"



### 3，concat(Things) -> string()
这里的Things是一个列表，里面由atom() | integer() | float() | string()
将这个列表里面的元素拼成一个字符串，然后返回

例子

lists:concat([doc, '/', file, '.', 3]).

结果

doc/file.3"



### 4，delete(Elem, List1) -> List2
List1是由很多Element组成的，这个函数的功能是在List1中寻找第一个和Elem元素一样的，
然后删除之，返回删除后新的列表。

例子

lists:delete({name,"zhangsan"},[{name,"lisi"},{name,"zhangsan"},{name,"wangmazi"})).

结果

[{name,"lisi"},{name,"wangmazi"}]



### 5，duplicate(N, Elem) -> List
返回一个由N个Elem组成的列表。

例子

lists:duplicate(5,"test").

结果

["test","test","test","test","test"]



### 6，flatlength(DeepList) -> integer() >= 0
我的理解是DeepList就是列表里面套列表
计算列表的长度，即用flatten函数将DeepList转化成List后元素的个数
这个函数和length()的区别就是：
length函数是得到列表元素的个数，
而flatlength函数是先将DeepList转化成List后的个数
譬如说List = [1,2,[3,4]]这个列表用
length(List)求的值是：3
lists:flatlength(List)求的值是：4
其实lists:flatlength(List) = length(flatten(List))

### 7，flatten(DeepList) -> List
将DeepList变成只有term()的list
例子：
lists:flatten([[a,a],[b,b],[c,c]]).
结果：
[a,a,b,b,c,c]



### 8，flatten(DeepList, Tail) -> List
就是将DeepList变成只有term的List后，在后面再加一个Tail。
例子：
lists:flatten([[a,a],[b,b],[c,c]],[dd]).
结果：
[a,a,b,b,c,c,dd]



### 9,keydelete(Key, N, TupleList1) -> TupleList2
这个函数适合处理列表里面的元素是元组的情况
删除TupleList1中元素第N个元素和Key一致的元素，只删除第一个一样的，后面一样的不删除
例子：
List = [{name,"zhangjing"},{sex,"male"},{name,"zhangsan"},{sex,"male"}],
lists:keydelete("male",2,List)
结果：
[{name,"zhangjing"},{name,"zhangsan"},{sex,"male"}]



### 10,keyfind(Key, N, TupleList) -> Tuple | false
查找TupleList中的一个Tuple，如果查找到，返回，如果没有查找到，则返回false
这个Tuple必须满足第N个元素和key是一样。
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}].
lists:keyfind("zhangjing",2,List1)
结果：{name,"zhangjing"}



### 11，keymember(Key, N, TupleList) -> boolean()
如果TupleList中的元素中存在第N个元素和key一致，则返回true，否则返回false
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}].
lists:keymember("zhangjing",2,List1).
结果：true



### 12，keymerge(N, TupleList1, TupleList2) -> TupleList3
将TupleList1和TupleList2进行混合，组成一个TupleList，
新组成的TupleList是按照Tuple的第N个元素进行排序的
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}].
List2 = [{nick,"zj"},{nick,"zs"}].
lists:keymerge(2,List1,List2).
结果：
[{name,"zhangjing"},
{name,"zhangsan"},
{nick,"zj"},
{nick,"zs"}]



### 13，keyreplace(Key, N, TupleList1, NewTuple) -> TupleList2
在TupleList1的Tuple中找出第N个元素和Key一致，然后用NewTuple将这个Tuple替换掉，如果没有找到
，则返回原来的TupleList1
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}]
lists:keyreplace("zhangjing",2,List1,{nickname,"netzj"}).
结果：
[{nickname,"netzj"},{name,"zhangsan"}]



### 14，keysearch(Key, N, TupleList) -> {value, Tuple} | false
这个函数和keyfind差不多，就是返回值的结构不一样
也是在TupleList中找一个Tuple，这个Tuple的第N个元素和Key一样。
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"}]
lists:keysearch("zhangjing",2,List1).
结果：
{value,{name,"zhangjing"}}



### 15，keysort(N, TupleList1) -> TupleList2
对TupleList1中的Tuple按照第N个元素进行排序，然后返回一个新的顺序的TupleList。
不过这种排序是固定的。
例子：
List1 = [{name,"zhangsan"},{name,"zhangjing"}].
lists:keysort(2,List1).
结果：
[{name,"zhangjing"},{name,"zhangsan"}]



### 16，keystore(Key, N, TupleList1, NewTuple) -> TupleList2
这个函数和keyreplace函数比较像，不同的是，这个keystore在没有找到对应的Tuple时，
会将这个NewTuple追加在这个TupleList1的最后。
例子：
List1 = [{name,"zhangsan"},{name,"zhangjing"}].
找到了的情况
lists:keystore("zhangjing",2,List1,{name,"netzhangjing"}).
[{name,"netzhangjing"},{name,"zhangsan"}]
没有找到的情况
lists:keystore("zhanging",2,List1,{name,"netzhangjing"}).
[{name,"zhangjing"},{name,"zhangsan"},{name,"netzhangjing"}]



### 17，keytake(Key, N, TupleList1) -> {value, Tuple, TupleList2} | false
在TupleList1中找Tuple，这个Tuple的第N个元素和Key一致，如果找到了这么一个Tuple
那么返回，{value, Tuple, TupleList2} 其中TupleList2是去掉Tuple的TupleList1.
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"},{name,"lisi"}].
lists:keytake("zhangjing",2,List1).
结果：
{value,{name,"zhangjing"},[{name,"zhangsan"},{name,"lisi"}]}



### 18，last(List) -> Last
返回：List最后一个元素
例子：
List1 = [{name,"zhangjing"},{name,"zhangsan"},{name,"lisi"}].
lists:last(List1).
结果：
{name,"lisi"}



### 19，max(List) -> Max
取出List中最大的元素，一般List是整型时比较适合。
例子：
lists:max([1,10,15,6]).
结果：
15



### 20，member(Elem, List) -> boolean()
如果Elem和List中的某个元素匹配（相同），那么返回true，否则返回false
例子
lists:member({sex,"1"},[{sex,"1"},{sex,"2"},{sex,"3"}]).
结果：
true


### 21，merge(ListOfLists) -> List1
ListOfLists是一个列表，里面由子列表构成
这个函数的功能就是将这些子列表合并成一个列表。
例子：
lists:merge([[{11}],[{22}],[{33}]]).
结果
[{11},{22},{33}]



### 22，merge(List1, List2) -> List3
List1和List2分别是一个列表，这个函数的功能是将这两个列表合并成一个列表。
例子：
lists:merge([11],[22]).
结果
[11,22]



### 23, merge3(List1, List2, List3) -> List4
将List1，List2，List3合并成一个列表
例子
lists:merge3([11],[22],[33,44]).
结果：
[11,22,33,44]



### 24，min(List) -> Min
返回List中的最小的元素，和max函数对应
例子
lists:min([1,2,3]).
结果
1



### 25，nth(N, List) -> Elem
返回List中的第N个元素。
例子
lists:nth(2,[{name,"zhangsan"},{name,"lisi"},{name,"wangmazi"}]).
结果
{name,"lisi"}



### 26，nthtail(N, List) -> Tail
返回List列表中第N个元素后面的元素
例子
lists:nthtail(3, [a, b, c, d, e]).
结果
[d,e]


### 27，prefix(List1, List2) -> boolean()
如果List1是List2的前缀(也就是说List1和List2前部分相同)，
那么返回true，否则返回false

### 28，reverse(List1) -> List2
将List1反转
例子
lists:reverse([1,2,3,4]).
结果
[4,3,2,1]



### 29,reverse(List1, Tail) -> List2
将List1反转，然后将Tail接在反转List1的后面，然后返回
例子
lists:reverse([1, 2, 3, 4], [a, b, c]).
[4,3,2,1,a,b,c]



### 30，seq(From, To) -> Seq
其中From和To都是整型，这个函数返回一个从From到To的一个整型列表。
例子
lists:seq(1,10).
结果
[1,2,3,4,5,6,7,8,9,10]



### 31，seq(From, To, Incr) -> Seq
返回一个整型列表，这个列表的后一个元素比前一个元素大Incr。
例子
lists:seq(1,10,4).
[1,5,9]



### 32，sort(List1) -> List2
将List1中的元素从小到大排序，然后返回新的一个列表。
例子
lists:sort([3,2,1]).
结果
[1,2,3]


### 33，split(N, List1) -> {List2, List3}
将List1分成List2和List3
其中List2包括List1的前N个元素，List3包含剩余的。
例子
lists:split(3,[1,2,3,4,5]).
结果
{[1,2,3],[4,5]}


这个函数和partition数有区别，partition是遍历全部的List，而splitwith在遍历时遇到false的情况
则马上结束遍历，返回结果。

### 34，sublist(List1, Len) -> List2
返回从第一个元素到第Len个元素的列表，这个Len大于List1的长度时，返回全部。
例子
lists:sublist([1,2,3,4,5,6],3).
结果
[1,2,3]



### 35，sublist(List1, Start, Len) -> List2
返回从List1的第Start个位置开始，后面Len个元素的列表。
例子
lists:sublist([1,2,3,4], 2, 2).
结果
[2,3]



### 36，subtract(List1, List2) -> List3
等同于 List1 -- List2
这个函数功能是返回一个List1的副本，对于List2中的每个元素，第一次在List1副本中出现时被删掉。
例子
lists:subtract("112233","12").
结果
"1233"



### 37，suffix(List1, List2) -> boolean()
如果List1是List2的后缀，那么返回true，否则返回false
例子
lists:suffix("22","1122").
结果
true



### 38，sum(List) -> number()
返回List中每个元素的和。其中List中的元素都应该是number()类型的。
例子
lists:sum([1,2,3,4]).
结果
10


### 39，ukeymerge(N, TupleList1, TupleList2) -> TupleList3
TupleList1和TupleList2里面的元素都是元组
将TupleList1和TupleList2合并，合并的规则是按照元组的第N个元素，如果第N个元素有相同的，那么保留TupleList1中
的，删除TupleList2中的。



### 40，ukeysort(N, TupleList1) -> TupleList2
TupleList1里面的元素都是元组
这个函数也同样返回一个元素是元组的列表，返回的这个列表是按照元组的第N个元素来排序的，如果元组中有出现
第N个元素相同的情况，删除掉后面的一个元组。
例子
lists:ukeysort(1,[{name,"zhangsan"},{sex,"male"},{name,"himan"}]).
结果
[{name,"zhangsan"},{sex,"male"}]



### 41，umerge(ListOfLists) -> List1
这个函数和merge唯一不同的就是，里面不能出现相同的元素，如果出现相同的，那么删除之，只保留一个唯一的
例子
lists:umerge([[1,2],[2,3]]).
结果
[1,2,3]
分析：由于[[1,2],[2,3]]中merge后是[1,2,2,3],这个时候有两个相同的元素2，所以只保存一个2，所以结果是[1,2,3].


### 42，umerge3(List1, List2, List3) -> List4
将List1, List2, List3合并
和merge3不同的是返回的List4中不能出现重复的元素
例子
lists:merge3([1,2],[2,3],[3,4]).
结果
[1,2,3,4]



### 43，unzip(List1) -> {List2, List3}
List1里面的元素是元组，每个元组由两个元素组成，返回值List2包含每个List1中每个元组的第一个元素
返回值List3包含每个List1中每个元组的第二个元素。
例子
lists:unzip([{name,"zhangsan"},{sex,"male"},{city,"hangzhou"}]).
结果
{[name,sex,city],["zhangsan","male","hangzhou"]}



### 44，unzip3(List1) -> {List2, List3, List4}
List1里面的元素是元组，每个元组由三个元素组成，返回值List2包含每个List1中每个元组的第一个元素；
返回值List3包含每个List1中每个元组的第二个元素；返回值List4包含每个List1中每个元组的第三个元素。
例子
lists:unzip3([{name,"zhangsan","apple"},{sex,"male","banana"},{city,"hangzhou","orange"}]).
结果
{[name,sex,city],
["zhangsan","male","hangzhou"],
["apple","banana","orange"]}
注意，最终返回的是一个元组。

### 45，usort(List1) -> List2
将List1按照从小到大的顺序排序，如果排序后有重复的元素，删除重复的，只保存一个唯一的。
例子
lists:usort([4,3,2,1,2,3,4]).
结果
[1,2,3,4]


### 46，zip(List1, List2) -> List3
将两个长度相同的列表合并成一个列表
List3是里面的每一个元组的第一个元素是从List1获取的，而每个元组的第二个元素是从List2中获取的
例子
lists:zip([name,sex,city],["zhangsan","male","hangzhou"]).
结果
[{name,"zhangsan"},{sex,"male"},{city,"hangzhou"}]
注意，如果List1和List2长度不一致，那么这个函数将会报错。



### 47，zip3(List1, List2, List3) -> List4
将三个长度相同的列表合并成一个列表
List3是里面的每一个元组的第一个元素是从List1获取的，而每个元组的第二个元素是从List2中获取的
每个元组的第三个元素是从List3中获取的。
例子
lists:zip3([name,sex,city],["zhangsan","male","hangzhou"],["nick","1","zhejiang"]).
结果
[{name,"zhangsan","nick"},
{sex,"male","1"},
{city,"hangzhou","zhejiang"}]