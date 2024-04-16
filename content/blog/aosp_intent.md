+++
title = "Intent In AOSP"
date = 2024-04-17

[taxonomies]
tags = ["AOSP", "Android"]
+++
## Intent用途
Intent的主要是用于组件之间的通信. 像常见的APP内的分享到其他应用, 跳转其他APP都是通过Intent实现. Intent大致有以下几个用途:
- 启动activity
- 启动service
  - AOSP蓝牙代码方面大量使用bindService()的方式获取service
- 发送broadcast

{% tip(header="Tip") %}
也就是Android四大组件其中的三个, 所以Intent在源码中随处可见...
{% end %}

## Intent类型
Intent有两种类型:
- Explicit intent 显示Intent需要指定一个完整的组件名称来获取组件. 所以通常用于App内部的一些服务.
- Implicit intent 隐式Intent指定一个通用的Action, 其他组件通过监听这个Action来做出响应, 例如分享操作系统通常会显示许多APP供用户选择(这些APP都是指定了相应的Intent filter).

{% important(header="Important") %}
在Android5.0(API level 21)之后, 只能通过显示Intent调用bindService()
{% end %}

## Intent属性
TODO

## Intent filter
TODO

## Intent在蓝牙中的应用
TODO
