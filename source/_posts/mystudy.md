---
title: 个人学习笔记
date: 2023-11-21 20:18:51
tags:
- erlang
categories:
- Erlang
---

# 概述
个人学习笔记

## windows下杀死进程

<!--more-->


使用命令 netstat -ano |findstr “8082”

查询当前端口PID为15904的进程 tasklist | findstr 15904

taskkill /f /t /im “15904”
or
taskkill /F /IM node.exe



