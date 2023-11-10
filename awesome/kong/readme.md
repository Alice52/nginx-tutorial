[toc]

## intros/overview

![avatar](/static/image/dist/gateway/kong-flow.png)

1. ecology: `kong gateway Server + Apache Cassandra/PostgreSQL + Kong Manager/Konga`

   - Kong Admin API: kong 内容配置(restful api)
   - Kong Manager/Konga
   - Kong Dev Portal: 二次开发/管理 API 版本
   - Kong Vitals: Gateway 节点的健康性能指标, 是 Kong Manager 用户界面的一部分
   - Kong Gateway plugins

## deploy

## config: `Route >> Service >> Upstream >> Target`

1. route: 与服务关联, 多对一关系(location)
2. service: 上游服务的抽象, 通过 Kong 匹配到相应的请求要转发的地方(server)
3. upstream: 上游服务, 实现负载
4. target: upstream 负载下的每个节点(物理服务 | ip + port 的抽象)
5. consumer: 代表用户或应用(核心原则是可以为其添加插件)
6. plugins

## plugins 开发

## practice

1. oneid plugin with aacs

---

## reference

1. https://zhuanlan.zhihu.com/p/577842078
2. https://www.jianshu.com/p/b44400618c69
3. https://github.com/micro-services-roadmap/roadmap/issues/5
4. https://blog.csdn.net/lgxzzz/article/details/121683302
5. https://cloud.tencent.com/developer/article/2301049
6. https://mp.weixin.qq.com/s/O2N2ucFLn3vF67RK_aP0UA
7. https://blog.csdn.net/zz18435842675/article/details/122449579
8. [kong logstash](https://blog.csdn.net/why_still_confused/article/details/89244200)
