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
5. tcp 端口转发

   - 要求 nginx 需要开启 stream 模块, 编译的时候加载 --with-stream 选项

     ```conf
     stream {
        server {
           listen 12001;
           proxy_pass 172.36.111.204:12001;
        }

        server {
           listen 12002;
           proxy_pass 172.36.109.137:12001;
        }
     }
     ```
