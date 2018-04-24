### NIO编程 ###
---

### NIO简介

NIO是JDK1.4中引入的，之前老的I/O类库是阻塞的，NIO的目标就是让Java支持非阻塞。

### NIO类库简介


* 缓存区Buffer


&emsp;&emsp; Buffer是一个对象，包含了一些写入或读出的数据，在NIO类库中加入Buffer对象，体现了新库与原I/O的重要区别，在面向流的I/O中，可以将数据直接写入或将数据直接读到Stream对象中。

* 通道Channel

&emsp;&emsp;Channel是一个通道，网络数据通过Channel读取和写入，实际上Channel可以分为两大类:用于网络读写的SelectableChannel和用于文件操作的FileChannel。

* 多路复用器Selector

&emsp;&emsp;多路复用器提供选择已经就绪的任务能力，Selector会不断的轮询注册在其上的Channel，如果某个Channel上发生了读或者写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey就可以获取就绪Channel的集合，进行后续的I/O操作。


### NIO服务器端主要的创建步骤
