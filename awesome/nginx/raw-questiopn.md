## nginx

1. 是什么
   - 高性能, 高负载, 高扩展, 高可靠, 热部署: 50k 的并发量
   - 代理服务器 + static resource server
2. 有什么
   - 反向代理: proxy_pass[server/location] + 概念好处
   - 负载均衡: upstream[http] + proxy_pass[server/location] + 轮询/ip-hash/权重/响应时间
   - 配置动静分离
3. 怎么用
4. 反向代理默认会将请求头去掉: 需要时要自己写以便传递

## 四层转发

1. 1.9.0 版本中增加了 4 层(TCP/UDP)代理功能: ngx_stream_core_module
2. 要求 nginx 需要开启 stream 模块, 编译的时候加载 --with-stream 选项
3. config sample

   ```conf
   stream {
      # 方式一
      upstream dns {
         server 192.168.0.1:53535;
         server dns.example.com:53;
      }
      server {
         listen 127.0.0.1:53 udp reuseport;
         proxy_timeout 20s;
         proxy_pass dns;
      }

      # 方式二
      server {
         listen 12001;
         proxy_pass 172.36.111.204:12001;
      }
      server {
         listen 12002;
         proxy_pass 172.36.109.137:12001;
      }

      # 方式三
      server {
         listen [::1]:12345;
         proxy_pass unix:/tmp/stream.socket;
      }
   }
   ```

## nginx 调优参数

1. worker_processes number: 工作进程数

   - 每个 worker 进程都是单线程的进程
   - 如果这些模块确认不会出现阻塞式的调用: CPU 核心数
   - 如果有可能出现阻塞式调用: 大于 CPU 核心数

2. worker_connections number: 每个 worker 进程的最大连接数

   - 默认是 1024

3. [仅 linux 有效]worker_cpu_affinity cpumask[cpumask……] 绑定 Nginx worker 进程到指定的 CPU 内核

   - 每一个 worker 进程都独享一个 CPU, 就在内核的调度策略上实现了完全的并发
   - sample: `worker_processes 4; worker_cpu_affinity 1000 0100 0010 0001`

4. worker_priority: 进程优先级设置

   - 优先级由静态优先级和内核根据进程执行情况所做的动态调整(目前只有 ±5 的调整)共同决定
   - 如果用户希望 Nginx 占有更多的系统资源, 那么可以把 nice 值配置得更小一些, 但不建议比内核进程的 nice 值（通常为–5）还要小

5. worker_rlimit_nofile limit: worker 进程可以打开的最大句柄描述符个数

   - 默认为为操作系统的限制
   - 解决 too many open files 问题

6. accept_mutex[on|off]: Nginx 的负载均衡锁(默认开启)

   - accept (非堵塞锁-取不到会立刻返回)锁默认是打开的, 因此不建议关闭它
   - 当某一个 worker 进程建立的连接数量达到 worker_connections 配置的最大连接数的 7/8 时, 会大大地减小该 worker 进程试图建立新 TCP 连接的机会
   - 如果关闭它, 那么建立 TCP 连接的耗时会更短, 但 worker 进程之间的负载会非常不均衡

7. accept_mutex_delay Nms;
   - 默认 500ms
   - 如果只有一个 worker 进程试图取锁而没有取到, 至少要等待 accept_mutex_delay 定义的时间才能再次试图取锁
