---
layout: post
title: VS常用正则替换
date: 发布于2019-09-26 10:05:36 +0800
categories: 测试结果杂记
tag: 4
---

* content
{:toc}

匹配整体替换

<!-- more -->

> 查找：.*  
>  替换：$1  
>  栗子：将"str"替换为tr(“str”)  
>  搜索：".*"  
>  替换：tr("$1")

匹配通用部分替换

> 查找：([^"]*)  
>  替换：$1  
>  栗子：将u8"str"替换为tr(“str”)  
>  查找：u8"([^"]*)"  
>  替换：tr("$1")

