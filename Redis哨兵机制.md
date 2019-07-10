Redis主从复制有一个很大的缺点就是没有办法对 master 进行动态选举（当 master 挂掉后，会通过一定的机制，从 slave 中选举出一个新的 master），需要使用 Sentinel 机制完成动态选举。

Sentinel哨兵主要解决以下问题：

- **监控**，监控每个节点以及哨兵运行状态
- **报警**，当发现某个节点或哨兵出现问题，通知其他哨兵
- **自动故障转化**，当主节点宕机时，哨兵从原主节点下的所有可用从节点中选举出一个作为主节点，原主节点降为从节点，并将其他从节点的主节点配置改为指定新主节点
- **配置中心**，客户端初始化连接的是哨兵节点集合

哨兵配置如下：

![img](https://github.com/CoinShine/coinshine.github.io/blob/master/media/picture/8.png)

哨兵工作原理：

1. 每个 Sentinel（哨兵）进程以**每秒钟一次**的频率向整个集群中的 **Master 主服务器，Slave 从服务器以及其他 Sentinel（哨兵）进程**发送一个 PING 命令。（此处我们还没有讲到集群，下一章节就会讲到，这一点并不影响我们模拟哨兵机制）
2. 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为**主观下线**（**SDOWN**）。
3. 如果一个 Master 主服务器被标记为主观下线（SDOWN），则正在监视这个 Master 主服务器的**所有 Sentinel（哨兵）** 进程要以每秒一次的频率 **确认 Master 主服务器**的确**进入了主观下线状态**。
4. 当 **有足够数量的 Sentinel（哨兵）** 进程（大于等于配置文件指定的值）在指定的时间范围内确认 Master 主服务器进入了主观下线状态（SDOWN）， 则 Master 主服务器会被标记为 **客观下线（ODOWN）**。
5. 在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有Master 主服务器、Slave 从服务器发送 INFO 命令。
6. 当 Master 主服务器被 Sentinel（哨兵）进程标记为 **客观下线(ODOWN)** 时，Sentinel（哨兵）进程向下线的 Master 主服务器的所有 Slave 从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
7. 若没有足够数量的 Sentinel（哨兵）进程同意 Master 主服务器下线， Master 主服务器的客观下线状态就会被移除。若 Master 主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master 主服务器的主观下线状态就会被移除。
