## intros

1. 可以理解为一个集成了很多模块的定制版 nginx
2. nginx with lua

   ```conf
   http {
       server {
           listen 8080;
           location / {
               default_type text/html;
               content_by_lua '
                   ngx.say("<p>hello, world</p>")
               ';
           }
       }
   }
   ```

## 执行流程

![avatar](/static/image/openresty/workflow.png)
