## core-point

1. 负载均衡
2. 健康检查

   - 尝试与其连接(telnet 等): 端口探测可能会存在服务僵死
   - 发送 http 请求: 精准
   - 对过往历史数据进行分析: 入一定时间内失败率到达阈值

3. 相关概念

   - 模块化 | lua 脚本化

4. 高可用 & 工作原理(work)
5. 核心配置

   - route: location
   - service: server
   - upstream: server
   - target: server
   - consumer
   - plugins

---

## nginx & openresty & kong

1. Nginx 的核心维度包括

   - 高性能: 异步, 事件驱动(epoll), 高性能/高并发, 低延迟
   - 扩展性: 模块化架构(lua), 可以通过扩展模块来增加功能
   - 反向代理: 反向代理服务器(负载均衡|缓存|访问控制)
   - 高可靠性: 高可靠性/容错性(多个实例实现高可用性)

2. OpenResty

   - 性能: 基于 Nginx 构建
   - 扩展性: 自定义 Lua 代码扩展和定制 API 网关
   - 可编程: 提供可编程的 HTTP 流水线, 可以在请求到达服务器之前或之后执行 Lua 代码
   - 插件: 提供丰富的插件功能(身份验证|访问控制|限流|监控|缓存)

3. Kong

   - 性能: 基于 OpenResty 构建
   - API 管理: 是 openresty 的一个 api 网关应用, 提供了中央 API 管理平台，用于注册、维护和监控 API
   - 插件: openresty
   - 扩展性: 可以通过自定义插件、脚本和模块来扩展和定制 API 网关

   ![avatar](/static/image/nginx/kong-framework.jpg)

4. [kong & nginx 性能对比](https://blog.csdn.net/zz18435842675/article/details/122669030)

   - 两种框架压测时, qps 跟 cpu 核数的提升成正比, 与内存的影响较小(注意带宽): kong 和 nginx 性能差不多(稍差一丝)
     1. `nginx raw qps: 2c4g + 3.1w`
     2. `kong raw qps: 2c4g + 3w`
     3. `nginx proxy qps: 2c4g + 1.3w`
     4. `kong proxy qps: 2c4g + 1.2w`
   - xx
   - kong 架构代理: qps 最高的配置规格为 4 核 2G, qps 大约在 1.2w 左右波动
   - **kong 架构+lua 业务+file-log 插件**: qps 最高的配置规格为 4 核 2G, qps 大约在 9000 左右波动
   - **nginx 架构+lua 业务+nginx 反向代理**: qps 最高的配置规格为 4 核 2G, qps 大约在 1.3w 左右波动
   - nginx 架构+lua 业务: qps 最高的配置规格为 4 核 2G, qps 大约在 3w 左右波动
