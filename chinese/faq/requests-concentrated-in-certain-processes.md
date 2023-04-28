# 请求集中在某些进程

### 现象
有时候我们通过命令`php start.php status` 看到，请求被集中在特定的某些进程中处理，其它进程完全空闲。

### 抢占机制
workerman多个进程获取连接的方式**默认**是**抢占式**的，也就是说当客户端有连接发起时，所有空闲的进程都有机会去获取这个连接，快者先得。到底谁快，是由操作系统内核调度决定的。操作系统可能会优先选取最近一次使用的进程获得cpu使用权，因为cpu寄存器里可能还存在上个进程的上下文信息，这可以减少上下文切换开销。所以当业务足够快的时候或者压测过程中，更容易出现连接集中被某些进程处理的情况，因为这个策略可以避免频繁的进程切换，性能往往是最优的，并不是什么问题。

### 轮询机制
workerman可以通过设置 `$worker->reusePort = true;`的方式将获取连接的方式改为**轮询**的方式，轮询的方式内核会将连接近似平均的方式分配给所有进程，这样所有的进程将会一起处理请求。

### 误区
很多开发者认为所有进程都参与请求处理性能越好，实际上不一定。当业务足够简单时，参与处理请求的进程数越趋近于cpu核心数服务器吞吐量越高。例如4核服务器，进程数设置为4时，helloworld压测QPS一般是最高的。如果参与处理的进程数超过cpu核数太多，进程上下文开销越大，性能反而越差。而一般带数据库业务时，进程数设置为cpu核数的3倍-6倍性能可能会更好。


