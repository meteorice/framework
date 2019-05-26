# Reactor

> "高效"是指什么?
>
> * 消息从A传递到B时，产生很少的内存垃圾，甚至不产生。
> * 解决消费者处理消息的效率低于生产者时带来的溢出问题。
> * 尽可能提供非阻塞异步流

## 解决的问题

* 阻塞等待：如 Future.get\(\)
* 不安全的数据访问：如 ReentrantLock.lock\(\)
* 异常冒泡：如 try…​catch…​finally
* 同步阻塞：如 synchronized{ }
* Wrapper分配\(GC 压力\)：如 new Wrapper\(event\)

## 提供的方案

