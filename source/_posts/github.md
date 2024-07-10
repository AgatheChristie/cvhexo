---
title: github代理学习
date: 2023-11-21 20:18:51
tags:
- github
categories:
- Github
password: 
- ccc
---

# 概述

设置Git全局代理，解决无法pull和push问题

<!--more-->

## 设置Git代理 http or socks5
```shell
git config --global http.proxy 'http://127.0.0.1:15235'
git config --global https.proxy 'http://127.0.0.1:15235'
```
### 全局
```shell
git config --global http.proxy 'socks5://127.0.0.1:15235'
git config --global https.proxy 'socks5://127.0.0.1:15235'
```
### 本地
```shell
git config --local http.proxy '127.0.0.1:15235'
git config --local https.proxy '127.0.0.1:15235'
```

## 取消Git代理
### 全局
```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```
### 本地
```shell
git config --local --unset http.proxy
git config --local --unset https.proxy
```
## 查看Git代理
### 全局
```shell
git config --global http.proxy
git config --global https.proxy
```
### 全局
```shell
git config --local http.proxy
git config --local https.proxy
```





