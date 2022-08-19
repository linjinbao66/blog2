---

title: spring 使用websocket主动推送日志到页面展示
date: 2020-09-05

---

## 简介

Java程序运行的时候会产生大量的日志，有的是自己主动打印的，有些的第三方包中打印的日志，一些场景下需要将程序执行过程中的日志主动推送出去，例如使用docker-java制作镜像的时候，我需要将制作过程大日志输出到页面，用来判断制作过程。

## 相关流程

1. logback日志切割
2. 阻塞队列/环形队列处理并发读写
3. websocket推送

## 方案一实现

很明显，日志需要通过log4j等日志框架收集。我们需要自定义一个过滤器，处理需要的日志信息，在过滤器中将日志存到阻塞队列或者环形队列。配置如下：

logback.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration scan="true" scanPeriod="3 seconds">
    <!--设置日志输出为控制台-->
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] [%logger{32}] %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ama.filter.LogFilter"></filter>
    </appender>
    <!--设置日志输出为文件-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ama.filter.LogFilter"></filter>
        <File>logFile.log</File>
        <rollingPolicy  class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>logFile.%d{yyyy-MM-dd}.log.zip</FileNamePattern>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%d{HH:mm:ss,SSS} [%thread] %-5level %logger{32} - %msg%n</Pattern>
        </layout>
    </appender>
    <root>
        <level value="DEBUG"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

其中`ama.filter.LogFilter`是自定义的日志处理过滤器。代码如下：

LogFilter.java

```java
@Slf4j
@Service
public class LogFilter extends Filter<ILoggingEvent> {
    @Override
    public FilterReply decide(ILoggingEvent event) {
        String exception = "";
        IThrowableProxy iThrowableProxy1 = event.getThrowableProxy();
        if(iThrowableProxy1!=null){
            exception = "<span class='excehtext'>"+iThrowableProxy1.getClassName()+" "+iThrowableProxy1.getMessage()+"</span></br>";
            for(int i=0; i<iThrowableProxy1.getStackTraceElementProxyArray().length;i++){
                exception += "<span class='excetext'>"+iThrowableProxy1.getStackTraceElementProxyArray()[i].toString()+"</span></br>";
            }
        }
        LoggerMessage loggerMessage = new LoggerMessage(
                event.getMessage(),
                DateFormat.getDateTimeInstance().format(new Date(event.getTimeStamp())),
                event.getThreadName(),
                event.getLoggerName(),
                event.getLevel().levelStr,
                exception,
                ""
        );
        LoggerQueue.getInstance().push(loggerMessage);
        return FilterReply.ACCEPT;
    }
}
```

重写decide方法，格式化日志，然后放入阻塞队列，` LoggerQueue.getInstance().push(loggerMessage);`

LoggerMessage 是一个简单的实体类，用于存放消息实体类，代码如下：

LoggerMessage.java

```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class LoggerMessage {
    private String body;
    private String timestamp;
    private String threadName;
    private String className;
    private String level;
    private String exception;
    private String cause;
}
```

阻塞队列LoggerQueue的实现：

LoggerQueue.java

```java
public class LoggerQueue {
    //队列大小
    public static final int QUEUE_MAX_SIZE = 1000;
    private static LoggerQueue alarmMessageQueue = new LoggerQueue();
    //阻塞队列
    private BlockingQueue blockingQueue = new LinkedBlockingQueue<>(QUEUE_MAX_SIZE);

    private LoggerQueue() {
    }

    public static LoggerQueue getInstance() {
        return alarmMessageQueue;
    }

    /**
     * 消息入队
     *
     * @param log
     * @return
     */
    public boolean push(LoggerMessage log) {
        return this.blockingQueue.offer(log);//队列满了就抛出异常，不阻塞
    }

    /**
     * 消息出队
     *
     * @return
     */
    public LoggerMessage poll() {
        LoggerMessage result = null;
        try {
            result = (LoggerMessage) this.blockingQueue.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return result;
    }
}

```

利用阻塞队列BlockingQueue 的特点，实现消息的并发读写。至此，日志消息的切割和发送队列完成了，接下来是websocket发送的逻辑，首先需要开启websocket：

WebSocketConfig.java

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry.addHandler(logHandler(), "/buildImage/log").setAllowedOrigins("*");
    }
    private LogHandler logHandler() {
        return new LogHandler();
    }
}
```

消息处理器LogHandler代码如下：

LogHandler.java

```java
public class LogHandler extends TextWebSocketHandler {
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String playload = message.getPayload();
        session.sendMessage(new TextMessage("收到了消息" + playload));
        new Runnable(){
            @SneakyThrows
            @Override
            public void run() {
                while (true){
                    LoggerMessage poll = LoggerQueue.getInstance().poll();
                    session.sendMessage(new TextMessage(poll.getBody()));
                }
            }
        }.run();
    }

}
```

至此可以实现消息的主动推送了，浏览器访问`ws://ip:port/buildImage/log`建立ws连接即可。



## 方案二实现

此种方案使用了环形队列，以及stomp来实现。

### 日志消息处理

和上面一直，定义过滤器统一处理，发送到环形队列。

logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true">
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <appender name="IMGFILE" class="ch.qos.logback.core.FileAppender">
        <file>imageBuild.log</file>
        <encoder>
            <pattern>
                %date %level [%thread] %logger{10} [%file:%line] %msg%n
            </pattern>
        </encoder>
    </appender>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
        <filter class="ama.filter.LogFilter"></filter>
    </appender>

    <logger name="com.github.dockerjava">
        <appender-ref ref="IMGFILE" />
    </logger>
    <logger name="ama">
        <appender-ref ref="IMGFILE" />
    </logger>

    <logger name="org.springframework.messaging.simp.broker" level="ERROR">
    </logger>

    <logger name="org.springframework.web.SimpLogging" level="ERROR">
    </logger>

    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

LogFilter.java

```java
@Service
public class LogFilter extends Filter<ILoggingEvent>{
    @Override
    public FilterReply decide(ILoggingEvent event) {
        LoggerMessage loggerMessage = new LoggerMessage(
                event.getMessage()
                , DateFormat.getDateTimeInstance().format(new Date(event.getTimeStamp())),
                event.getThreadName(),
                event.getLoggerName(),
                event.getLevel().levelStr
        );
        LoggerDisruptorQueue.publishEvent(loggerMessage);
        return FilterReply.ACCEPT;
    }
}
```

### 环形队列实现

环形队列使用Disruptor实现，代码如下：

LoggerDisruptorQueue.java

```java
@Component
public class LoggerDisruptorQueue {

    private Executor executor = Executors.newCachedThreadPool();

    // The factory for the event
    private LoggerEventFactory factory = new LoggerEventFactory();

    private FileLoggerEventFactory fileLoggerEventFactory = new FileLoggerEventFactory();

    // Specify the size of the ring buffer, must be power of 2.
    private int bufferSize = 2 * 1024;

    // Construct the Disruptor
    private Disruptor<LoggerEvent> disruptor =
            new Disruptor<>(factory, bufferSize, executor);;

    private Disruptor<FileLoggerEvent> fileLoggerEventDisruptor =
            new Disruptor<>(fileLoggerEventFactory, bufferSize, executor);;

    private static  RingBuffer<LoggerEvent> ringBuffer;

    private static  RingBuffer<FileLoggerEvent> fileLoggerEventRingBuffer;

    @Autowired
    LoggerDisruptorQueue(LoggerEventHandler eventHandler,FileLoggerEventHandler fileLoggerEventHandler) {
        disruptor.handleEventsWith(eventHandler);
        fileLoggerEventDisruptor.handleEventsWith(fileLoggerEventHandler);
        this.ringBuffer = disruptor.getRingBuffer();
        this.fileLoggerEventRingBuffer = fileLoggerEventDisruptor.getRingBuffer();
        disruptor.start();
        fileLoggerEventDisruptor.start();
    }

    public static void publishEvent(LoggerMessage log) {
        long sequence = ringBuffer.next();  // Grab the next sequence
        try {
            LoggerEvent event = ringBuffer.get(sequence); // Get the entry in the Disruptor
            // for the sequence
            event.setLog(log);  // Fill with data
        } finally {
            ringBuffer.publish(sequence);
        }
    }

    public static long getSize(){
        return ringBuffer.getBufferSize();
    }

    public static void publishEvent(String log) {
        if(fileLoggerEventRingBuffer == null) return;
        long sequence = fileLoggerEventRingBuffer.next();  // Grab the next sequence
        try {
            FileLoggerEvent event = fileLoggerEventRingBuffer.get(sequence); // Get the entry in the Disruptor
            // for the sequence
            event.setLog(log);  // Fill with data
        } finally {
            fileLoggerEventRingBuffer.publish(sequence);
        }
    }

}
```

环形队列和阻塞队列在这里的区别不大，都是用作消息的处理，实现并发读写。此处一方面实现了输出控制台日志，另一方面也实现了从日志文件读写然后推送。

### 控制台日志相关

LoggerEvent.java

```java
public class LoggerEvent {

    private LoggerMessage log;

    public LoggerMessage getLog() {
        return log;
    }

    public void setLog(LoggerMessage log) {
        this.log = log;
    }
}
```

LoggerEventFactory.java

```java
public class LoggerEventFactory implements EventFactory<LoggerEvent> {
    @Override
    public LoggerEvent newInstance() {
        return new LoggerEvent();
    }
}
```

LoggerEventHandler.java

```java
@Component
public class LoggerEventHandler implements EventHandler<LoggerEvent> {

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    @Override
    public void onEvent(LoggerEvent stringEvent, long l, boolean b) {
        messagingTemplate.convertAndSend("/consoleLog",stringEvent.getLog());
    }
}
```

### 日志文件相关

FileLoggerEvent.java

```java
public class FileLoggerEvent {
    private String log;

    public String getLog() {
        return log;
    }

    public void setLog(String log) {
        this.log = log;
    }
}
```

FileLoggerEventFactory.java

```java
public class FileLoggerEventFactory  implements EventFactory<FileLoggerEvent> {
    @Override
    public FileLoggerEvent newInstance() {
        return new FileLoggerEvent();
    }
}
```

FileLogListening.java

```java
@Service
public class FileLogListening implements ApplicationContextAware{

    private long lastTimeFileSize = 0;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        String logPath = "imageBuild.log";

        try {
            File logFile = ResourceUtils.getFile(logPath);
            final RandomAccessFile randomFile = new RandomAccessFile(logFile, "rw");
            //指定文件可读可写
            ScheduledExecutorService exec = Executors.newScheduledThreadPool(2);
            exec.scheduleWithFixedDelay(new Runnable() {
                public void run() {
                    try {
                        randomFile.seek(lastTimeFileSize);
                        String tmp = "";
                        while ((tmp = randomFile.readLine()) != null) {
                            String log=new String(tmp.getBytes("ISO8859-1"));
                            LoggerDisruptorQueue.publishEvent(log);
                        }
                        lastTimeFileSize = randomFile.length();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }, 0, 1, TimeUnit.SECONDS);
        }catch (IOException e){
            e.printStackTrace();
        }

    }

    /**
     * 监听日志文件
     *
     * @throws IOException
     */
    @PostConstruct
    public void start() throws IOException {

    }
}
```

FileLoggerEventHandler.java

```java
@Component
public class FileLoggerEventHandler implements EventHandler<FileLoggerEvent> {
    @Autowired
    private SimpMessagingTemplate messagingTemplate;
    @Override
    public void onEvent(FileLoggerEvent fileLoggerEvent, long l, boolean b) {
        messagingTemplate.convertAndSend("/fileLog",fileLoggerEvent.getLog());
    }
}
```



### websocket配置

WebSocketConfig.java

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/log")
                .setAllowedOrigins("*")
                .addInterceptors()
                .withSockJS();
    }
}
```

### 页面示例

index.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>WebSocket Logger</title>
    <script src="https://cdn.bootcss.com/jquery/2.1.4/jquery.js"></script>
    <script src="https://cdn.bootcss.com/sockjs-client/1.1.4/sockjs.min.js"></script>
    <script src="https://cdn.bootcss.com/stomp.js/2.3.3/stomp.min.js"></script>
</head>
<body>
<h1>jvm进程内的日志</h1>
<button onclick="openSocket()">开启日志</button><button onclick="closeSocket()">关闭日志</button>
<div id="log-container" style="height: 300px; overflow-y: scroll; background: #333; color: #aaa; padding: 10px;">
    <div></div>
</div>
<h1>指定监听日志文件的日志</h1>
<button onclick="openSocket()">开启日志</button><button onclick="closeSocket()">关闭日志</button>
<div id="filelog-container" style="height: 300px; overflow-y: scroll; background: #333; color: #aaa; padding: 10px;">
    <div></div>
</div>
</body>
<script>
    var stompClient = null;
    $(document).ready(function() {openSocket();});
    function openSocket() {
        if(stompClient==null){
            var socket = new SockJS('http://localhost:9005/log?token=kl');
            stompClient = Stomp.over(socket);
            stompClient.connect({token:"kl"}, function(frame) {
                stompClient.subscribe('/consoleLog', function(event) {
                    var content=JSON.parse(event.body);
                    $("#log-container div").append(content.timestamp +" "+ content.level+" --- ["+ content.threadName+"] "+ content.className+"   :"+content.body).append("<br/>");
                    $("#log-container").scrollTop($("#log-container div").height() - $("#log-container").height());
                },{
                    token:"kltoen"
                });
                stompClient.subscribe('/fileLog', function(event) {
                    var content=event.body;
                    $("#filelog-container div").append(content).append("<br/>");
                    $("#filelog-container").scrollTop($("#filelog-container div").height() - $("#filelog-container").height());
                },{
                    token:"kltoen"
                });
            });
        }
    }
    function closeSocket() {
        if (stompClient != null) {
            stompClient.disconnect();
            stompClient=null;
        }
    }
</script>
</body>
</html>
```

## 总结

推送的流程如下：

logback收集日志->自定义filter拦截->发送到消息队列（本地或者远程，需要实现并发读写）->websocket线程东队列读取数据推送。

## 注意

sprint-boot-websocket与spring-boot-devtools依赖冲突，会导致无法读到数据。