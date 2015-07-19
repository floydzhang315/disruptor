#LMAX Disruptor——一个高性能、低延迟且简单的框架

原文地址：[LMAX Disruptor – High Performance, Low Latency and Simple Too](http://www.symphonious.net/2011/07/11/lmax-disruptor-high-performance-low-latency-and-simple-too/)

翻译：杨帆 校对：丁一

Disruptor 是一个用于在线程间通信的高效低延时的消息组件，它像个增强的队列，并且它是让 LMAX Exchange 跑的如此之快的一个关键创新。关于什么是 Disruptor、为何它很重要以及它的工作原理方面的信息都呈爆炸性增长 —— [这些文章](http://code.google.com/p/disruptor/wiki/BlogsAndArticles)很适合开始学习 Disruptor，还可跟着 [LMAX BLOG](http://blogs.lmax.com/) 深入学习。这里还有一份更详细的[白皮书](http://disruptor.googlecode.com/files/Disruptor-1.0.pdf)。


虽然 disruptor 模式使用起来很简单，但是建立多个消费者以及它们之间的依赖关系需要的样板代码太多了。为了能快速又简单适用于 99 %的场景，我为 Disruptor 模式准备了一个[简单的领域特定语言](https://github.com/ajsutton/disruptorWizard)。例如，为建立一个消费者的“四边形模式”：

![](images\10-1.png)

（从 [Trisha Gee’s excellent series explaining the disruptor pattern](http://mechanitis.blogspot.com/2011/07/dissecting-disruptor-wiring-up.html) 偷来的图片）

在这种情况下，只要生产者（P1）将元素放到ring buffer上，消费者 C1 和 C2 就可以并行处理这些元素。但是消费者 C3 必须一直等到 C1 和 C2 处理完之后，才可以处理。在现实世界中的对应的案例就像：在处理实际的业务逻辑（C3）之前，需要校验数据（C1），以及将数据写入磁盘（C2）。

用原生的 Disruptor 语法来创建这些消费者的话代码如下：

```
Executor executor = Executors.newCachedThreadPool();
BatchHandler handler1 = new MyBatchHandler1();
BatchHandler handler2 = new MyBatchHandler2();
BatchHandler handler3 = new MyBatchHandler3()
RingBuffer ringBuffer = new RingBuffer(ENTRY_FACTORY, RING_BUFFER_SIZE);
ConsumerBarrier consumerBarrier1 = ringBuffer.createConsumerBarrier();
BatchConsumer consumer1 = new BatchConsumer(consumerBarrier1, handler1);
BatchConsumer consumer2 = new BatchConsumer(consumerBarrier1, handler2);
ConsumerBarrier consumerBarrier2 =
ringBuffer.createConsumerBarrier(consumer1, consumer2);
BatchConsumer consumer3 = new BatchConsumer(consumerBarrier2, handler3);
executor.execute(consumer1);
executor.execute(consumer2);
executor.execute(consumer3);
ProducerBarrier producerBarrier =
ringBuffer.createProducerBarrier(consumer3);
```

在以上这段代码中，我们不得不创建那些个 handler（就是那些个 MyBatchHandler 实例），外加消费者屏障，BatchConsumer 实例，然后在他们各自的线程中处理这些消费者。DSL 能帮我们完成很多创建工作，最终的结果如下：

```
Executor executor = Executors.newCachedThreadPool();
BatchHandler handler1 = new MyBatchHandler1();
BatchHandler handler2 = new MyBatchHandler2();
BatchHandler handler3 = new MyBatchHandler3();
DisruptorWizard dw = new DisruptorWizard(ENTRY_FACTORY,
	RING_BUFFER_SIZE, executor);
dw.consumeWith(handler1, handler2).then(handler3);
ProducerBarrier producerBarrier = dw.createProducerBarrier();
```

我们甚至可以在一个更复杂的六边形模式中构建一个并行消费者链：

![](images\10-2.png)


```
dw.consumeWith(handler1a, handler2a);
dw.after(handler1a).consumeWith(handler1b);
dw.after(handler2a).consumeWith(handler2b);
dw.after(handler1b, handler2b).consumeWith(handler3);
ProducerBarrier producerBarrier = dw.createProducerBarrier();
```

这个领域特定语言刚刚诞生不久，欢迎任何反馈，也欢迎大家从 [github  上 fork](http://github.com/ajsutton/disruptorWizard) 并改进它。

原创文章，转载请注明： 转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [LMAX Disruptor——一个高性能、低延迟且简单的框架](http://ifeve.com/disruptor-dsl/)