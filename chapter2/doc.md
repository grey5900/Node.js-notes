# 模块机制 #

#### JavaScript曾经落后后端主要是因为:  ####
1. 没有模块系统
2. 标准库较少
3. 没有标准接口
4. 缺乏包管理系统  

Node借鉴CommonJS的Modules规范实现了一套非常易用的模块系统

#### CommonJS的模块规范 ####
* 模块引用
* 模块定义
* 模块识别

#### Node模块机制 ####
 - 以模块(文件)来划分功能、组织代码和代码复用
 - 用包规范来管理应用和可复用组件
 - 用npm工具和网站进行开源社区代码共享

#### CommonJS包目录包涵文件：####
- package.json: 包描述文件
- bin：用于存放可执行二进制文件目录
- lib：用于存放JavaScript代码目录
- doc：用于存放文档目录
- test： 用于存放单元测试实例的代码

#### 常用的Node核心模块 ####
- console：控制台输入输出
- fs：与文件系统交互（在不同操作系统下是不一样的，node底层有个机制封装，可以使我们用的一样，实现跨平台）
- fttp：提供HTTP服务器功能
- net：有很多C++的库 提供TCP/IP网络功能
- os：提供了一些操作系统相关的实用方法，比如取cpu、内存的信息
- path：一般和os合起来用 提供了一些操作系统相关的实用方法
- url：解析URL
- querystring：解析URL的查询字符串
- crypto：提供加密和解密功能，基本上是对OpenSSL的包装

#### 模块对象 ####
- 模块对象(module)实际上也是一个文件，表示当前模块文件，是一个js对象。模块对象不是全局的，而是每个模块唯一的
- 模块对象的属性
    + module.id，模块的标识符，通常是完全解析后的文件名
    + module.loaded 模块是否已经加载完成，或正在加载中 js运行时要先运行模块中的代码，加载完成后再执行其他的
    + filename 模块完全解析后的文件名
    + parent 调用该模块的模块
    + children 被该模块引用的模块对象
    + exports 模块的导出对象，默认为空对象
#### 模块的导出
- module.exports 模块的导出对象默认为空对象。例如：exptest
    + exports变量是moudule.exports的快捷方式，moudule.exports.f = ...可以被更简洁的写成exports.f = ...
    + exports变量直接赋值（exports = ...）会使之不再是module.exports的快捷方式，导致不能被导出
    + 为避免出错，模块的导出都用module.exports，不用exports
#### 模块引用
- 使用require方法来指定加载模块，方法的参数是模块的标识符
- 模块标识符包括：
    + 核心模块
    + 文件模块
    + 安装包模块
- require.resolve() 可用来解析标识符的绝对路径
#### 模块缓存
- 模块第一次require后会被缓存，多次require不会导致模块的代码被执行多次
- Node 缓存不同于浏览器JavaScript缓存，浏览器只缓存与文件，Node模块缓存的是编辑与执行后的对象
- require.cache对象代表Node模块缓存区，缓存的模块以属性的方式加入该对象，可以用require.cache['模块标识符']来访问具体的缓存模块，也可以delete该缓存。
#### node包规范
- Node 采用包来对一组相互依赖关系的模块进行统一管理，封装为独立的单个组件或单个应用
- Node 目录包括有：
    + package.json 包的描述文件
    + bin目录 包的二进制文件和可执行文件
    + lib 包的js文件
    + doc 包的文档文件
    + test 包的单元测试文件
    + node_modules 本地安装的第三方包
#### 基于包规范开发应用流程
- 创建目录
- 进入目录
- 初始化 （npm init生成packpage.json）
- 配置依赖项
- 安装依赖包
- 配置运行脚本命令 （script的start）
- 编辑代码
- 运行代码 （npm start）

---
CMD规范：  
AMD规范：

require()方法通过解析标识符、路径分析、文件定位，然后加载执行即可。

libuv函数的调用充分表现Node利用libuv实现跨凭他的方式

结束文件模块、核心模块、内建模块、C/C++扩展模块的关系：
C/C++内建模块属于最底层模块，属于核心模块，主要提供JavaScript核心模块和第三方JavaScript文件模块调用。  
核心模块主要是职责两类：一类是作为C/C++内建模块的封装和桥接层，提供文件模块调用；一类是纯粹的功能模块，它不需要跟底层打交道。