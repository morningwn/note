> 最近写代码的时候发现InterruptedException在catch之后不管是抛出新异常还是打日志，发现代码检查仍然提示这样做不对，去查了一下，发现了个小细节，在此记录一下。


### 抛出InterruptedException的场景

> Thrown when a thread is waiting, sleeping, or otherwise occupied, and the thread is interrupted, either before or during the activity.


大概的意思就是线程在未执行结束的时候收到了中断请求就会抛出InterruptedException。

### 处理方法

平时我们处理异常的时候大概就是这样：

```java
try {
    
} catch (ExecutionException e) {
    log.error("调用失败", e);
}
```

或者是这样：

这里的RuntimeException一般会是自己定义的一个异常类型，继承RuntimeException，然后在返回之前统一做捕获，并返回一个友好的提示。

```java
try {
    
} catch (ExecutionException e) {
    log.error("调用失败", e);
    throw new RuntimeException("");
}
```

但是以上这些方法对于InterruptedException就不适合了

为啥？

> InterruptedException在抛出的时候只是给一个提示：有人想我停止运行了。但是如果不做处理的话当前线程仍然会继续执行，并不会终止。


所以`InterruptedException`的处理方法是仍然抛出`InterruptedException`或者如下：

```java
try {

} catch (InterruptedException e) {
    log.error("调用失败", e);
    Thread.currentThread().interrupt();
} catch (ExecutionException e) {
    log.error("调用失败", e);
}
```
