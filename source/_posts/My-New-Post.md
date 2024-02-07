---
title: Golang context
date: 2024-02-06 17:07:28
tags:
---
## 1.概述

包context定义了`Context`类型，它带有截止时间，取消信号，以及其它（跨API和进程）请求范围的值

传入服务器的请求应该创建一个`Context`对象，并且处理请求的服务器应该接受一个`Context`对象，（服务器的请求处理的）回调函数链必须传播`Context`对象本身，或者传播通过`WithCancel`, `WithDeadline`, `WithTimeout`, `WithValue`得到的`Context`对象，当某个`Context`对象取消时，所有的由该对象得到的对象都会被取消。

`WithCancel`, `WithDeadline`以及`WithTimeout`接收一个`Context`对象(作为父对象)，返回得到的子`Context`对象和一个`CancelFunc`回调函数，通过调用这个取消回调函数会取消掉该子`Context`对象以及所有的由该子对象产生的子子对象，移除父对象对该子对象的引用，停止任何关联的定时器（用于timeout和deadline计时）。如果没有调用`CancelFunc`将会导致该子`Context`对象以及其子子`Context`对象发生泄漏直到其父`Context`对象的`CancelFunc`被调用或者定时器被触发。go vet工具会检查整个控制流路径的`CancelFunc`是否被调用。

`WithCancelCause`函数（go1.20版本新增）返回的`cancelFunc`不同于需要传入一个error对象用来记录取消的原因。调用一个已经取消的`Context`对象(或者其子对象)的`Cause`方法会检索出cancel的原因（就是返回cancalFunc传入的error），如果没有指定cause（调用cancelFunc时传递的error为nil），它的返回结果将等于`ctx.Err`。

程序使用`Context`对象时应该遵守如下规则从而保证接口的跨包一致性和启用静态分析工具检查`Context`的传播
* 不要把`Context`对象存储在结构体里，取而代之的是明确地作为第一个参数传递给任何一个需要`Context`对象的函数，并且命名为`ctx`
```
func DoSomething(ctx context.Context, arg Arg) error {
    // ... use ctx somewhere ...
}
```
* 当你不确定应该使用哪个`Context`对象,并且要掉用一个需要`Context`入参的函数时，不要传递`nil`，取而代之的是使用`context.TODO()`
* 仅使用Values存储和传递（跨API和进程）请求范围的值，不要传递可选的参数。
注：请求范围的值如可以进行链路追踪的TraceID。如数据库连接，日志实例这样的全局对象不适用于Values传递。

* 补充：不推荐使用基本数据类型作为Values的Key，避免不同包的key发生冲突。Key的类型应该是未导出的，外部包想要访问Key中的值时需要使用包内导出的`SomethingFromContext`方法

同一个Context对象可能被传递到不同goroutines的函数中，Context对象同时被多个goroutines使用是安全的。

访问https://blog.golang.org/context 可以浏览到关于服务使用context的实例代码
<hr>

## 2.Context interface

`Context`带有截止时间，取消信号和其它跨API边界的值。
`Context`的方法可能被多个goroutines同时调用。

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}
```
`Deadline` 返回工作完成的截止时间，代表这个Context对象应该被取消。如果没有截止时间被设置，`ok`将为`false`，同一个`Context`对象多次调用`Deadline`返回相同的结果。


