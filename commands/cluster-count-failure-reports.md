---
layout: commands
title: cluster-count-failure-reports 命令
permalink: commands/cluster-count-failure-reports.html
disqusIdentifier: command_cluster-count-failure-reports
disqusUrl: http://redis.cn/commands/cluster-count-failure-reports.html
commandsType: cluster
discuzTid: 918
tranAuthor: menwengit
---

这个命令返回指定节点的*故障报告*个数，故障报告是Redis Cluster用来使节点的`PFAIL`状态（这意味着节点不可达）晋升到`FAIL`状态而的方式，这意味着集群中大多数的主节点在一个事件窗口内同意节点不可达。

A few more details:

更多细节：

* 一个节点会用`PFAIL`标记一个不可达时间超过配置中的超时时间的节点，这个超时时间是 Redis Cluster 配置中的基本选项。
* 处于`PFAIL`状态的节点会将状态信息提供在心跳包的流言（gossip）部分。
**failure reports**, remembering that a given node said another given node is in `PFAIL` condition.
* 每当一个节点处理来自其他节点的流言（gossip）包时，该节点会建立**故障报告**（如果需要会刷新TTL），并且会记住发送消息包的节点所认为处于`PFAIL`状态下的其他节点。
* 每个故障报告的生存时间是节点超时时间的两倍。
* 如果在一段给定的事件内，一个节点被另一个节点标记为`PFAIL`状态，并且在相同的时间内收到了其他大多数主节点关于该节点的故障报告（如果该节点是主节点包括它自己），那么该节点的故障状态会从`PFAIL`晋升为`FAIL`，并且会广播一个消息，强制所有可达的节点将该节点标记为`FAIL`。

该命令返回当前节点没有过期的故障报告个数（在两倍的节点超时时间收到的）。该计数值不包含当前节点，该节点是我们要求这个计数值是以我们作为参数所传递的ID的节点，这个计数值只包含该节点从其他节点接收到的故障报告。

当Redis Cluster的故障检测器不能正常工作时，这个命令主要用来调试。


## 返回值

[Integer reply](http://www.redis.cn/topics/protocol.html#integer-reply)：这个节点有效的故障报告个数。