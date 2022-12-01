[Nginx基本使用 (yuque.com)](https://www.yuque.com/wexiao/nginx/nr0app#Z86pM)

[Nginx（三）------nginx 反向代理 - YSOcean - 博客园 (cnblogs.com)](https://www.cnblogs.com/ysocean/p/9392908.html)

[Nginx基本使用 - Alley-巷子 - 博客园 (cnblogs.com)](https://www.cnblogs.com/nanianqiming/p/10630899.html)

## 反向代理与反向代理的区别

### 反向代理
反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669892028481.png)

#### 作用
- 保证内网的安全，阻止web攻击，大型网站，通常将反向代理作为公网访问地址，Web服务器是内网
- 负载均衡，通过反向代理服务器来优化网站的负载

### 正向代理
比如我们国内访问谷歌，直接访问访问不到，我们可以通过一个正向代理服务器，请求发到代理服，代理服务器能够访问谷歌，这样由代理去谷歌取到返回数据，再返回给我们，这样我们就能访问谷歌了
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669892209612.png)

### 总结

**正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端.**

**反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端**