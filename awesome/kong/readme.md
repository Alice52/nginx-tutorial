[toc]

## intros/overview

![avatar](/static/image/kong/kong-flow.png)
![avatar](/static/image/kong/kong-layer.jpg)

1. intros

   - Mashape 公司开源, 可以执行 Lua 脚本(OpenResty 已经包含了 lua-nginx-module)的应用
   - 核心: **API 网关和微服务管理平台**: **`数据库抽象|路由和插件管理|插件机制可以注入到请求生命周期的任何位置`**
   - 目的: 帮助开发者构建/管理/扩展现代化的微服务架构和分布式系统

2. feature

   - API 管理: 管理和发布多个微服务的 API(包括路由、插件、身份认证、限流等)
   - 模块化(插件系统): 插件系统可扩展其功能(认证、授权、日志、限流、转发) + 可以自定义开发插件(lua + go)
   - 可扩展性: 高可扩展平台(支持水平扩展和集群部署[gossip-通知其他节点变更]-应对流量和高可用)
   - 监控和分析: 实时监控 API 的性能、健康状况和使用情况
   - 灵活的部署选项: 本地、云端和容器化环境
   -
   - 动态路由: 继承 OpenResty 的动态路由
   - 负载均衡
   - 断路器/健康检测: 能追踪不健康(主动/被动监控)的 upstream services
   - 认证(jwt/hmac/oauth2)
   - 限流
   - 安全(ACL, 机器人检测, 黑白名单 IP)
   - 监控
   - 缓存: 在代理层进行缓存和响应处理
   - 日志记录: 通过 http/tcp/udp 等方式进行日志相关操作
   -
   - 通讯协议: webSockets/grpc/http/xx
   - 转换: 对 TTP 请求和相应进行添加/删除/操纵等操作
   - plugin：可以对 kong 和 API 进行扩展
   - ~~服务发现~~: 可结合 consul 提供服务注册

3. ecology: `kong gateway Server + ~~Apache Cassandra~~/PostgreSQL + Kong Manager/~~Konga~~`

   - kong admin api: kong 内容配置(restful api)
   - kong manager/~~konga~~
   - kong dev portal: 二次开发/管理 api 版本
   - kong vitals: gateway 节点的健康性能指标, 是 kong manager 用户界面的一部分
   - kong gateway plugins

4. config

   - route: 与服务关联, 多对一关系(location)
   - service: 上游服务的抽象, 通过 Kong 匹配到相应的请求要转发的地方(server)
   - upstream: 上游服务, 实现负载
   - target: upstream 负载下的每个节点(物理服务 | ip + port 的抽象)
   - consumer: 代表用户或应用(**核心原则是可以为其添加插件**)
   - **plugins**

5. 请求解析过程

   - `Route >> Service >> Upstream >> Target`

   ![avatar](/static/image/kong/kong-req-flow.jpg)

6. [部署方式](https://tech.aufomm.com/introduction-of-different-kong-deployment-methods-with-docker/#DBless-Deployment)

   - data place
   - db place
   - hybrid

## [gateway pickup](./01.pickup.md)

1. 社区活跃 + 成熟度高
2. [性能好](https://juejin.cn/post/6844903999582240775)
3. 扩展性好: 插件(lua/go) + 部署
4. 功能全(开箱即用)
5. 耦合低: 没有与 java 生态耦合
6. 二次开发简单
7. 将一些共有逻辑抽象在网关实现(限流/认证/安全/xx): 减小业务的重复代码(**简化应用开发**)

   ![avatar](/static/image/kong/kong-optimizer.jpg)

## plugins(usr/local/share/lua/5.1/kong/plugins)

1. 实现方式

   - Kong 通过在 openresty 各个阶段加入了 lua 代码来实现插件的注入和执行
   - [openresty 执行流程](../openresty/readme.md)

2. code

   - **`kong.access() //1.完成路由匹配, 2.确认加载的插件(并加入缓存) 3.进入balancer阶段`**

   ```lua
   init_by_lua_block {
      kong = require 'kong'
      kong.init() // 完成 Kong 的初始化,路由创建, 插件预加载等
   }
   init_worker_by_lua_block { // 工作进程启动时执行
      // 初始化 Kong 事件, worker 之间的事件, 由 worker_events 来处理
      // cluster 节点之间的事件, 由 cluster_events 来处理, 缓存机制
      kong.init_worker()
   }

   upstream kong_upstream {
      server 0.0.0.1;
      balancer_by_lua_block {
         kong.balancer() //负载均衡
      }
      keepalive 60;
   }
   server {
      server_name kong;
      listen 0.0.0.0:8000 reuseport backlog=16384;
      listen 0.0.0.0:8443 ssl http2 reuseport backlog=16384;

      rewrite_by_lua_block {
         // 此时路由匹配未开始, 只能处理全局插件
         // kong插件级别, 全局(作用于所有请求),route(作用于当前路由), service(作用于匹配到当前service的所有请求)
         kong.rewrite()
      }
      access_by_lua_block {
         kong.access() //1.完成路由匹配, 2.确认加载的插件(并加入缓存) 3.进入balancer阶段
      }

      header_filter_by_lua_block {
         kong.header_filter() //遍历在缓存中的插件列表, 并执行
      }
      body_filter_by_lua_block {
         kong.body_filter() //遍历在缓存中的插件列表, 并执行
      }
      log_by_lua_block {
         kong.log() //遍历在缓存中的插件列表, 并执行
      }

      location / {
         proxy_pass            $upstream_scheme://kong_upstream$upstream_uri;
      }
   }
   ```

3. 插件类型

   - 身份认证插件: Basic Authentication | Key authentication | OAuth2.0 authentication | HMAC authentication | JWT | LDAP authentication
   - 安全控制插件: ACL(访问控制) | CORS | 动态 SSL | IP 限制 | 爬虫检测实现
   - 流量控制插件: 请求限流 | 上游响应限流| 请求大小限制; 限流支持本地、Redis 和集群限流模式
   - 分析监控插件: Galileo(记录请求和响应数据实现 API 分析) | Datadog(记录 API Metric 如请求次数、请求大小、响应状态和延迟, 可视化 API Metric) | Runscope(记录请求和响应数据, 实现 API 性能测试和监控)
   - 协议转换插件: 请求转换(在转发到 upstream 之前修改请求) | 响应转换(在 upstream 响应返回给客户端之前修改响应)
   - 日志应用插件: TCP、UDP、HTTP、File、Syslog、StatsD、Loggly 等

4. [kong oneid plugin with aacs](https://github.com/micro-services-roadmap/kong/tree/master/v3.5.0/plugins/oneid)

---

## reference

1. https://blog.csdn.net/zz18435842675/article/details/118733464
2. [网关选型](https://mp.weixin.qq.com/s?__biz=MzAwMjI0ODk0NA==&mid=2451964472&idx=1&sn=eb05f7d5af78b6635a83a55377ed09d0&chksm=8d1ffba7ba6872b1207a93cf23a9370b49eefb1533b7115d906828e1162bee578a26dfbff0f6&scene=178&cur_album_id=1510122164576911361#rd)
3. https://mp.weixin.qq.com/s/78-CJIqyptnzliPdyQbfNg
4. https://developer.aliyun.com/article/1036809#slide-8
5. https://mp.weixin.qq.com/s?__biz=MzkxNTE3NjQ3MA==&mid=2247496839&idx=1&sn=5d3abf14d91fd9eea8268bd6278ca29c
6. https://mp.weixin.qq.com/s/O2N2ucFLn3vF67RK_aP0UA
7. https://zhuanlan.zhihu.com/p/586308764
8. https://blog.csdn.net/lgxzzz/article/details/121683302
9. https://blog.csdn.net/zz18435842675/article/details/122449579
10. https://cloud.tencent.com/developer/article/2301049
11. https://blog.csdn.net/tao_627/article/details/79298904
12. ***
13. https://zhuanlan.zhihu.com/p/577842078
14. https://www.jianshu.com/p/b44400618c69
15. https://github.com/micro-services-roadmap/roadmap/issues/5
16. [kong logstash](https://blog.csdn.net/why_still_confused/article/details/89244200)
17. https://blog.csdn.net/qq_28410283/article/details/122615304
18. https://blog.csdn.net/mz135135/article/details/122273839
19. https://www.jianshu.com/p/2a63c6d0c09d
20. https://zhuanlan.zhihu.com/p/663243536
