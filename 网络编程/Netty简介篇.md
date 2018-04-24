### Netty简介篇

#### Netty中的Channel与Pipeline

##### Netty中的Channel实现概览：

在Netty中，Channel是通讯的载体，而ChannelHandler负责Channel的逻辑处理。

**channelPipeline**：可以理解为ChannelHandler的容器：一个Channel包含一个channelPipeline,所有的ChannelHandler都会注册到ChannelPipeline中，并按顺序组织起来。

**channelEvent**：是数据或者状态的载体，例如：传输的数据对应MessageEvent,状态改变对应ChannelStateEvent。当对Channel进行操作时，会产生一个ChannelEvent，并发送到ChannelPipeline。ChannelPipeline会选择一个ChannelHandler进行处理。这个ChannelHandler处理之后，可能会产生新的ChannelEvent，并流转到下一个ChannelHandler。

#### channelPipeline的主流程：

Netty的ChannelPipeline包含两条路线，UpStream和DownStream：

 UpStream：对应上行，接收到的消息，被迫的状态改变，都属于Upstream。

 DownStream:对应下行，发送的消息，主动的状态改变，都属于DownStream。

对应的ChannelPipline也包含两类，ChannelUpStreamHandler和ChannelDownStreamHandler。

这里有个值得注意的地方：在一条“流”里，一个ChannelEvent并不会主动的”流”经所有的Handler，而是由上一个Handler显式的调用ChannelPipeline.sendUp(Down)stream产生，并交给下一个Handler处理。也就是说，每个Handler接收到一个ChannelEvent，并处理结束后，如果需要继续处理，那么它需要调用sendUp(Down)stream新发起一个事件。如果它不再发起事件，那么处理就到此结束，即使它后面仍然有Handler没有执行。这个机制可以保证最大的灵活性，当然对Handler的先后顺序也有了更严格的要求。


### Pipleline解决的问题

Pipleline机制解决的问题：

* 1、 提供了ChannelHandler的编程模型，基于ChannelHandler开发业务逻辑，基本不需要关心网络通讯方面的事情，专注于编码/解码/逻辑处理就可以了。

* 2、 实现了所谓的”Universal Asynchronous API”。这也是Netty官方标榜的一个功能。
