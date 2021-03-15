# 背景

## nginx 和 apache httpd 相比优点

### 优点

- nginx更轻量级，相比apache，启同样的web服务，占用更少的内容和资源 ？为什么能占用更少的资源？
- 高并发，nginx采用异步非阻塞模型，apache则是阻塞IO，在高并发的情况下，nginx能够保持低资源低消耗，性能更高
- 高度模块化的设计，编写模块相对简单
- nginx的社区活跃度更高

### apache相对于nginx的优点

- rewrite， （重写）

- 模块组件多，bug少，性能稳定

**nginx配置简洁，apache配置相对复杂，二者最为核心的区别在于，nginx是异步的，多个连接对应一个进程，进程会fork出线程进行请求；apache是同步多进程模型，一个连接对应一个进程；**



## nginx解决的问题

- 高并发 
- 负载均衡
- 高可用
- 虚拟主机
- 伪静态， url rewrite
- 动静分离

# 安装



## 官网

http://tengine.taobao.org/

- 官网下载最新版本tag.gz包
- 解压
  - tar -xzvf  tengine-2.3.2.tar.gz
- yum install gcc openssl-devel pcre-devel zlib-devel -y
-  ./configure --prefix=/home/parallels/software/tengine-2.3.2/bin
- make && make install