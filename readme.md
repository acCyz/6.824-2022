## 思路和笔记待更
  
  最近在忙论文，mit的授课老师（大名鼎鼎的Robert Morris ）貌似不让公开代码实现，据说写了个爬虫自动发邮件警告，不知道是不是本人。后面有时间了把伪代码和详细思路（这总不能不让吧）更新在这个仓库，当作对整个lab的最终总结吧

## intro
- 断断续续利用两个月的周末，终于把这门课的lab写完了，所有lab均稳定通过3k次
- lab1要求实现MapReduce，我实现了共享内存+锁、channel+无锁通信两个版本，个人感觉后一个版本比较优雅，代码逻辑比较简洁
- lab2raft层，遵守了22lab的建议，不实现CondInstallSnapshot()，另外废了心思来保证应用层和共识层之间安装快照的原子性
- lab3应用层，单机kvraft，主要的难点在于如何在多客户端并发的情况下，保证线性一致性，重复命令去重、快照和恢复以及在不可靠网络情况下，leader变更导致的前后反馈不一致等
- lab4分片kvraft，难点在于并发的重新配置以及如何处理shard迁移，并且每个shard相互独立，不能阻塞其他正常分片的正常服务。个人觉得push模式优于pull模式
- 两个Challenge 已完成

## 优化
所有的优化都是在824的代码框架中进行的，且均再次通过了3k次测试，但由于引入了某些优化机制，部分参数需要调整，例如prevote需要调整心跳间隔等等：
- 原来课程lab给的框架使用的是大数rand方案给客户端生成分布式ID，实现了雪花算法进行替换
- 实现了PreVote 和 no-op（注意824的在raft层的测试框架不支持no-op，因此要么在raft层实现no-op跳到lab4测试，要么在server层实现no-op）
- 实现了ReadIndex 和 Follow read加速线性一致读，读请求再也不用走raft log啦，效率++。同时客户端的读请求会随机打到follower服务器，借鉴了一致性hash进行了简单的负载均衡。
### todo list：
- pipeling
- lease read
- lab的rpc模块貌似是基于反射实现的模拟，后面自己手撸个rpc替换掉试试（但是这样可能就用不了lab提供的测试代码了，因为整个集群网络模拟（包括网络故障、拥塞、乱序、节点shotdown和recover）都是基于labrpc的config模拟的）
- 参考etcd的事件状态机（不过都这样了，为什么不去做pingcap的tinykv呢）



