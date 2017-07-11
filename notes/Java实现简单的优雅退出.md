### 1、netty的优雅退出
> 1、此处不展开讨论netty的优雅退出机制

### 2、在使用自己的工程项目，使用如下方式实现优雅退出
> 1、在启动的时候在注册一个“钩子”，其实也就是注册一个handler

```
public class Keys{
    public final static String SHUTDOWN = "USR2";
}

public void start() throws Exception {
        // netty启动，也可以是其他方式启动，启动之后注册handler
        serverChannelFuture = serverBootstrap.bind(socketAddress).sync();
        
        // 注册接受退出命令的handler
        ShutdownHandler shutdownHandler = new ShutdownHandler();
        shutdownHandler.registerShutdownHook(Keys.SHUTDOWN);
    }
```
> 2、实现Java自带的SignalHandler接口

```
public class ShutdownHandler implements SignalHandler {

    private static final Logger log = LoggerFactory.getLogger(ShutdownHandler.class);

    private volatile boolean isRunning = true;

    // 优雅退出的超时时间（ms）
    private int maxCloseTimeOut = 15000;

    public void registerShutdownHook(String signalName) {
        log.info("初始化ShutdownHandler--------");
        Signal signal = new Signal(signalName);
        Signal.handle(signal, this);
    }

    @Override
    public void handle(Signal signal) {
        String osType = signal.getName();
        log.info("接受到的osType={}",osType);
        if (!osType.equalsIgnoreCase(Keys.SHUTDOWN)){
            // TODO: 2017/6/28 做一些处理
            log.error("操作系统是={}！！！",osType);
            return;
        }

        // 此处根据实际的业务逻辑实现优雅退出
       
    }

    // 获取操作系统的类型
    private String getOSSignalType(){
        return System.getProperties().getProperty("os.name").toLowerCase().startsWith("win") ? "INT" : "USR2";
    }
}
```
> 3、修改停止脚本，以Linux为例：

```
 # 一般的stop都是使用 kill -9 
 #kill -9 $PROGID
 
 # 使用如下特殊符号
 kill -s USR2 $PROGID
```

