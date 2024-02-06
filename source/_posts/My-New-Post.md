---
title: Golang context
date: 2024-02-06 17:07:28
tags:
---

包context定义了`Context`类型，它带有截止时间，取消信号，以及其他（跨API和进程）请求范围的值

传入服务器的请求应该创建一个`Context`对象，并且处理请求的服务器应该接受一个`Context`对象，（服务器的请求处理的）回调函数链必须传播`Context`对象本身，或者传播通过`WithCancel`, `WithDeadline`, `WithTimeout`, `WithValue`得到的`Context`对象，当某个`Context`对象取消时，所有的由该对象得到的对象都会被取消。

`WithCancel`, `WithDeadline`以及`WithTimeout`接收一个`Context`对象(作为父对象)，返回得到的子`Context`对象和一个`CancelFunc`回调函数，通过调用这个取消回调函数会取消掉该子`Context`对象以及所有的由该子对象产生的子子对象，移除父对象对该子对象的引用


