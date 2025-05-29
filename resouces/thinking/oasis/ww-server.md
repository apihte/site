# ww-server

## ZoneExtensionServer

### 主要属性

· ServerConfig config 服务器配置

· ScheduledThreadPoolExcutor scheduler 定时任务调度器

· RequestBaseManager requestBaseManager 请求管理器

· SessionManager sessionManager 会话管理器

### 启动流程


· ZoneExtensionServer

    · initLogger 初始化日志

    · loadConfig 加载配置

    · initEventBus 创建事件总线 - 监听 server.properties 更新事件

    · initScheduler 初始化定时任务调度器

    · initRequestBaseManager 初始化请求管理器 - ZoneExtensionRequestManager（初始化各种线程池 login、logout、payment、cmd 等）

    · initExtensions 初始化区域 - 游戏服

    · initRPCServer 初始化 RPC 服务端，使用 Apache Thrift 框架

    · initPRCClient 初始化 RPC 客户端，使用 Apache Thrift 框架

    · initNettyNetWorkEngine 初始化 Netty 网络引擎，绑定消息 Handler（MessageHandler，消息处理调用 ZoneExtensionRequestManager）
    
    · initHttpServer

### 一条消息的执行过程

· 客户端发消息到服务器，服务器由 Netty 接受到消息，交给 MessageHandler 处理

· MessageHandler 调用 ZoneExtensionRequestManager 中的 handleRquest 方法处理消息

· handleRequest 方法中，根据消息类型，调用对应的处理方法，如 login、logout、payment、cmd 等

· 现在我们默认是一条逻辑消息，则构建一条 UserRequestAction 对象，参数为 actionQueue、cmdExecutor、message 等，即 UserRequestAction 中有对 actionQueue 的引用，并调用 UserRequestAction 的 enqueue 方法

· 在构建 UserRequestAction 对象后执行 enqueue 方法时，会尝试将这个对象 offer 到 actionQueue 对象的 Queue 队列中，并执行这个 UserRequestAction 的逻辑，也就是它的 run 方法

· run 方法执行完后，则会调用 actionQueue 的 dequeue 方法，将这个对象从 actionQueue 的 Queue 队列中移除

### 同一个玩家的逻辑都在同一条线程吗

不在，使用线程池

### 如何保证玩家逻辑执行顺序

sychronized 玩家的 actionQueue，只有当 actionQueue 的大小为 1 时才允许执行 action

然后当一个 action 执行完毕后会调用 dequeue 方法，在 dequeue 方法中，会先移除 actionQueue 的队首 action，然后判断 actionQueue 中是否还有 nextAction，若存在着继续执行 nextAction

而 action 中又有 actionQueue 的引用，在 action 的 run 方法的 finnaly 块中，会调用 actionQueue 的 dequeue 方法，将 action 从 actionQueue 的队列中移除，这样就形成了一个循环
