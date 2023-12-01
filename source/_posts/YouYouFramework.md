---
title: YouYouFramework
date: 2023-11-21 20:18:51
tags:
- unity
categories:
- Unity
---

# 概述

YouYouFramework学习


<!--more-->




## YouYou

[官网](http://www.u3dol.com/index.html)

YOUYOU unity版本说明


001   03:01     5.2.1f1
039_3 00:19     5.2.3f1
046   00:02:18  5.3.3f1 听说话
049_1 00:28     5.3.3f1


my          5.6.7f1


Common: ResourcesMgr GlobalInit SceneMgr Stat
Core:放了一些Excel转表需要的工具



Core:SocketDispatcher  UIDispatcher 单例基类   UISceneViewBase UIWindowViewBase UIViewBase


Data: 协议和Excel转表以后的数据


SceneCtrl:场上的一些空物体 SelectRoleSceneCtrl SceneLoadingCtrl


UI: UISceneCtrl UIViewMgr UIViewUtil
SceneUICtrl:老的文件夹里面还剩UISceneLoadingCtrl
UIView:放了一些Excel转表需要的工具
UIScene:挂在大预制体(比如UI_Root_LogOn UI_Root_SelectRole)身上的脚本 放在这里
UIWindow:挂在小预制体(比如pan_GameServerEnter pan_LogOn)身上的脚本 放在这里


System: AccountCtrl GameServerCtrl

Utils: 一些工具类  比如获取时间戳 随机名字之类的


AccountCtrl  m_LogOnView 可以控制 pan_LogOn 窗口上面的 m_LogOnView.txtUserName.text 拿到 pan_LogOn 身上的 UILogOnView 组件(脚本)
AccountCtrl   拿到 pan_Reg  身上的 UIRegView  组件(脚本)
监听点击事件




GameServerCtrl   拿到 pan_GameServerSelect 身上的 UIGameServerSelectView 组件(脚本)
GameServerCtrl   拿到 pan_GameServerEnter  身上的 UIGameServerEnterView  组件(脚本)

对两个预制体进行操作比如设置UI或者监听点击事件等  
注意监听的传递，从最底层的GamaServerItem按钮一直传递到最上层的GameServerCtrl



场上的SelectRoleSceneCtrl物体   拿到  UI_Root_SelectRole 身上的 UISceneSelectRoleView 组件(脚本)

场上的SelectRoleSceneCtrl物体   监听TCP协议 监听4个按钮的点击事件 监听拖拽  监听职业选择


m_UISceneSelectRoleView




You框架源码：https://gitee.com/chen_junji/YouYouFramework
https://gitee.com/chen_junji/YouYouFramework
YouYouMini框架源码(去掉了热更新模块, 更加轻量化, 适合独立游戏):
https://gitee.com/chen_junji/YouYouMini
框架教程官网: www.u3dol.com
ouYou初中高级课程-在线播放：
https://b23.tv/z4nAzN

悠游课堂初级课程2023版：
https://www.bilibili.com/video/BV1Fu411s7MV/

http://u3dol.com/YouYouFramework.html


## 破坏神unity客户端


技能模块 1
商场模块 2
副本模块 3
系统模块 4
战斗模块  组队 同步位置 同步动画  5

一 PlayerInfo 脚本初始化做的事情


D 执行穿戴装备函数 目的是为了计算  HP Damage Power
D 分发事件 OnPlayerInfoChanged
D 监听 OnInventoryChange 事件
把背包里面 IsDressed == true 的装备穿上


二 InventoryManager 脚本初始化做的事情
D 把txt里面的数据导入到 inventoryDict 里面
D 随机20个item到玩家背包(inventoryItemList)

D 分发事件 OnInventoryChange



Knapsack


KnapsackRole 挂载在装备面板
监听 OnPlayerInfoChanged 事件
把下面的格子 KnapsackRoleEquip 根据玩家穿戴的装备数据(InventoryItem) 都初始化好

KnapsackRoleEquip 挂载在装备面板里面的装备
监听点击事件 把每个格子的数据发送给  Knapsack 让 Knapsack 调用 equipPopup 并把数据展示在 equipPopup 上

InventoryUI      挂载在背包面板
监听 OnInventoryChange 事件
把下面的格子 InventoryItemUI 根据玩家背包的数据(inventoryItemList) IsDressed == false 都展示出来  把其他格子设置为null

InventoryItemUI  挂载在背包面板里面的格子
监听点击事件 把每个格子的数据发送给  Knapsack 让 Knapsack 调用 InventoryPopup 并把数据展示在 InventoryPopup 上

EquipPopup


InventoryPopup



值类型和引用类型 要分清楚


背包的道具使用、出售   没有同步到服务端 没有同步到 PhotonEngine.Instance.role


背包道具升级的时候同步了  PhotonEngine.Instance.role  同步了 inventoryItemDB  同步了服务端的 inventoryItemDB  和 role
装备穿上的时候                 同步了 inventoryItemDB        同步了服务端
那为什么道具count改变的时候    同步了 inventoryItemDB        没有同步服务端



第一个场景应该有一个GameObject才对
所有脚本都挂在了UIRoot上
ServerController
RoleController
LoginController
RegisterController
StartMenuController


## 破坏神Photon server服务端
一个给用户赋值一个给玩家选择的角色赋值
一个用户下可以有多个角色

LoginHandler
玩家登录的时候
peer.LoginUser = userDB;


RoleHandler
玩家选择角色进入游戏的时候
peer.LoginRole = ParameterTool.GetParameter<Role>(request.Parameters, ParameterCode.Role);



case SubCode.UpdateRole:
Role role = ParameterTool.GetParameter<Role>(request.Parameters, ParameterCode.Role);

那这里为什么没有把 	客户端传过来的role赋值	peer.LoginRole = role;
role.User = peer.LoginUser;         

roleManager.UpdateRole(role2);
role.User = null;
response.ReturnCode = (short)ReturnCode.Success;
break;


InventoryItemDBHandler 这里把role也传过来了  
case SubCode.UpgradeEquip:
InventoryItemDB itemDB4 = ParameterTool.GetParameter<InventoryItemDB>(request.Parameters,
ParameterCode.InventoryItemDB);


Role role = ParameterTool.GetParameter<Role>(request.Parameters, ParameterCode.Role);
                    
peer.LoginRole = role;  //这里需要赋值吗 需要 因为前端的 role 改变了 比如增加了金币  上面就没有
                    
role.User = peer.LoginUser;
                    
itemDB4.Role = role;
                    

inventoryItemDbManager.UpgradeEquip(itemDB4, role);
 break;



``` c#
public void UpdateRole(Role role)
{
    using (var session = NHibernateHelper.OpenSession())
    {
        using ( var transaction= session.BeginTransaction())
        {
        session.Update(role);
        transaction.Commit();
        }
    }
}
```


``` c#
public void UpgradeEquip(InventoryItemDB itemDb4, Role role) {
    using (var session = NHibernateHelper.OpenSession()) {
        using (var transaction = session.BeginTransaction())
        {
            session.Update(itemDb4);
            session.Update(role);
            transaction.Commit();
        }
    }
}
```



## UI框架

破坏神    20150602           unity 4.6
UI框架    20160419


## 冒险世界

service层负责处理网络相关的内容 分发事件


CharacterManager 
PlayerInputController SendEntityEvent 给 EntityController
EntityController 只受数据驱动 控制动画

GameObjectManager 初始化 CharacterManager里面的实体
character.IsCurrentPlayer 如果是自己才给
MainPlayerCamera.Instance.player
Character
entityController
这三个地方赋值





UI层负责展示UI

冒险世界    20181016    unity 2018.2.3f1

Entity Framework 服务端的DB框架



NetworkClient & TcpSocketListener
封包处理器 PackageHandler
消息分发器 MessageDistributer
消息分配处理 MessageDispatch

## 其他


英雄传说

背包 任务 聊天 工会

战斗

冒险岛







