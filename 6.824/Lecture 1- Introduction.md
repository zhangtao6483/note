#Lecture 1

##分布式系统

### 1. 什么是分布式系统

multiple networked cooperating computers

### 2. 为什么分布
- 连接物理上分散的节点
- 通过各个节点的资源隔离保证安全
- 通过各个节点的复制容忍故障
- 通过并行提供CPUs/mem/disk/net的性能

缺点：

- 复杂，难以debug
- 新类型的问题 e.g. 局部异常
- 不可知问题

> Leslie Lamport: "A distributed system is one in which the failure of a computer you didn't even know existed can render your own computer unusable."

- 建议：如果系统能很好工作，就不要使用分布式 

