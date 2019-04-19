# 异步I/O

#### 为什么要异步I/O
1. 用户体验，消耗时间为max(M,N),同步I/O消耗为max(M+N)
2. 资源配置，让单线程原理阻塞，更好利用CPU

#### 非阻塞IO和事件驱动
- 非租塞I/O
    + 操作系统进入内核I/O操作有两种方式：
        * 阻塞，应用要读取磁盘某个文件时，要等待系统内核完成硬盘寻道、读取数据、复制数据到内存等操作。
        * 非租塞，应用的I/O调用可以立即返回，但返回的是调用状态，需要轮询的方式确定系统内核是否完成所有操作。
        * Windows的IOCP及*nix的liev有不同的实现
    + Node的非阻塞I/O
        * 底层是基于libuv的线程池
        * 非阻塞I/O不再限于文件I/O,还包括了网络资源等的I/O
    + libuv
        * libuv是为node设计开发的跨平台抽象层，用于抽象的windows的IOCP及*nix的libv
        * 特性
            1. 异步文件I/O
            2. 异步TCP、UDP套接字
            3. 异步DNS解析
            4. 线程池调度
    + 非阻塞IO的调用过程和流程

#### 事件驱动
- 事件驱动：只有当事件发生时候才会调用回调函数的函数执行方式
- JavaScript异步的本质是事件驱动，其关键要素包括：
    + 消息队列（message queue），不占用JavaScript主线程运算时间的任务完成后或外部触发一个事件都会在消息队列里增加一条消息（事件）
    + 回调函数（callback），每个消息有对应的处理函数
    + 无限循环（even loop），不停地从消息队列（即观察者的队列）中检索消息，只要消息队列不空并且主线程空闲，就把消息对应的回调函数加载到EC Stack上执行
        * 主线程空闲：EC Stack上没有函数EC执行，Global EC的全局代码也执行完毕
- 每个请求不创建线程，用事件驱动调回调函数的方式 大大提高效率
- node一般配的服务器是nginx主要用于反向代理和负载均衡

#### 事件驱动的高性能服务器
- 传统web服务器（Apache, Tomcat, IIS等）基本上都是为每个请求启动一个线程来处理，直到该请求结束，大并发请求到来时，内存会耗光，导致服务器缓慢
- 基于事件驱动处理请求，无须为每一请求创建额外的服务线程（通过线程池的方式节省开销），节省了线程创建、管理、销毁的开销，能极大地提高并发请求的性能。
- Nginx web服务器是基于事件驱动的，在反向代理和负载均衡上有着大量的应用
- Node可以独立作为事件驱动的高性能Web服务器，也可以和各种Web服务器配合使用。

#### Node event模块
- Node event模块是一个基于发布／订阅模式的实现，比javascript客户端的事件模型简单，主要包括：
    + 事件源：可以发出事件的对象
    + 事件：通过事件源以唯一的字符串（事件名）来进行识别
    + 事件订阅：为某个事件设置回调函数（可以有多个）
    + 事件发布：事件源触发某个事件（可以触发任意次）


```js 
var events = require('events');
var eventEmitter = events.EventEmitter;
var count = 0;
var em = new eventEmitter(); //事件源

em.on('timed',function(data){ //事件订阅
    console.log('timed '+data);
})

em.on('timed', function(data){ //事件订阅
    console.log('事件数据：'+data);
})

setInterval(function(){
    em.emit('timed',count++); //事件发布
},1000); 
```
- EventEmitter
    + 事件源对象基本都继承于EventEmitter
    + Node核心模块有近半数继承于EventEmitter
    + 自定义对象的事件驱动可以通过继承于EventEmitter来实现
```js
var events = require('events');
var util = require('util');
function MyEventSource() { //自定义事件源的构造函数
    events.EventEmitter.call(this); //调用父类的构造函数
}
util.inherits(MyEventSource,events.EventEmitter); // 继承
MyEventSource.prototype.doit = function (data) {
    this.emit('myevent',data); //事件发布
}
// 可以通过下面的代码来使用上面的事件源
var obj = new MyEventSource();
obj.on("myevent",function(data){ //事件订阅
    console.log('Received data: "'+ data + '"');
})
obj.doit('foobar');
// Received data: "foobar"  
```
使用ES6的语法：
```js
var events = require('events');
class MyEmitter extends events.EventEmitter {};
const myEmitter = new MyEmitter();
myEmitter.on('event',() => {
   console.log("触发了一个事件！");
});
myEmitter.emit('event');
// 触发了一个事件！
```

- EventEmitter.prototype方法
    + 事件默认最大的订阅数（监听器数）为10个，可以设置
        * getMaxListeners()，返回当前最大订阅数
        * setMaxListeners(n)，修改当前最大订阅数
    + 为某事件增加订阅
        * addListener(eventName, listener)，一个事件可以多个订阅
        * on(eventName, listener)，一个事件可以多个订阅
        * once(eventName, listener)，一个事件只能一个订阅
    + emit(eventName[, …args])，按监听器的注册顺序，同步地调用每个注册名为eventName事件的监听器，并传入提供的参数。如果事件有监听器，则返回true，否则返回false
    + eventNames()，返回触发器上已注册监听器的事件的数组
    + listenerCount(eventName)，返回正在监听名为eventName的事件的监听器的数量
    + listeners(eventName)，返回名为eventName的事件的监听器数组的副本
    + removeAllListeners([eventNames])，移除全部或指定eventName的监听器
    + removeListener(eventName,listener)，从名为eventName的事件的监听器数组中移除指定的listener