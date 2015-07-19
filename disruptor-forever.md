#Disruptor Wizard已死，Disruptor Wizard永存！

原文地址：[The Disruptor Wizard is Dead, Long Live the Disruptor Wizard!](http://www.symphonious.net/2011/08/13/the-disruptor-wizard-is-dead-long-live-the-disruptor-wizard/) 

译者：杨帆 校对：丁一

Disruptor Wizard（上一篇中提到的 DSL 组件）目前已经正式并入 [Disrupto r的代码树](http://code.google.com/p/disruptor/)当中。既然 .net 移植版包含了 Wizard 风格的语法很久了，并且看起来还挺受欢迎，所以为什么还要让人们非得搞两个 jar 而不是一个？

我跟随 Disruptor 在术语命名上的变动做出了相应的更新。以前的 Customer（消费者），现在叫 EventProcessor（事件处理器）和 EventHandler（事件句柄）。这样的命名更好的说明了实际上的情况：消费者事实上可以向事件添加附加值。另外，ProducerBarrier（生产者屏障）被合并到 Ring Buffer 一起，并且 Ring Buffer Entry（条目）被改名为 Event （事件）。新的命名更贴切了，因为实际上围绕 Disruptor的编程模型大部分时候都是基于事件的。


除了以下两点，Wizard API 与以往并没有太大的不同：

* consumeWith 方法改名为 handleEventsWith
* getProducerBarrier 方法被替换成了一个返回值为 ring buffe r的 start 方法。这就不会混淆地认为 getProducerBarrier 方法也被用作触发事件处理器线程的启动。

现在的方法命名清楚地表示了该方法的其它作用。

原创文章，转载请注明： 转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [Disruptor Wizard已死，Disruptor Wizard永存！](http://ifeve.com/disruptor-wizard/)