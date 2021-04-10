# 容器与镜像

+ 镜像

  镜像是只读文件，提供了运行程序完整的软硬件资源，是应用程序的“集装箱”

+ 容器

  容器是镜像的实例，由docker负责创建，容器之间彼此隔离

# docker 执行流程

![image.png](https://upload-images.jianshu.io/upload_images/26015807-311b192a4a6489f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# docker常用命令

>+ docker version 
>  获取docker服务端、客户端版本号
>
>+ docker pull 镜像名:<tags>
>  获取远程仓库镜像,默认获取最新的镜像
>
>+ docker images
>  查看本地已下载的镜像
>
>+ docker run -p 映射端口:应用端口 -d 镜像名称:<tags> 
>  运行指定镜像，创建容器，启动应用
>  + -d 后台运行
>  + --name 指定容器名称
>  + --link 链接到容器名称
>
>+ docker ps 
>
>  查看正在运行的容器
>
>+ docker rm -f 容器ID
>
>  删除容器
>
>+ docker rmi <-f> 镜像名:<tags>
>
>  删除指定版本的镜像
>  
>+ docker inspect 容器id
>
>  查看docker 元数据信息，可用于查看虚拟ip
>
>  



![image.png](https://upload-images.jianshu.io/upload_images/26015807-aa6cdb2d3bb0ee08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 容器内部执行命令

> docker exec [-it] 容器id 命令
>
> + 在对应容器中执行命令
> + -it 采用交互的方式执行命令



# docker容器的生命周期

![image.png](https://upload-images.jianshu.io/upload_images/26015807-c7e476b3d501265f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





# docker file镜像描述文件

docker通过读取Dockerfile 中的指令自动生成文件

>docker build -t 机构/镜像名<:tags> Dockerfile目录(绝对路径/相对路径)

不加版本号，则默认当前镜像为latest（最新版）

```dockerfile
# 设置基准镜像
FROM tomcat:latest
# 设置镜像维护人/组织
MAINTAINER weizz5
# 切换工作目录
WORKDIR /usr/local/tomcat/webapps
# 添加应用到镜像工作目录
ADD docker-demo ./dockerDemo
```

## dockerfile 基础命令

>+ FROM
>
>+ MAINTAINER
>
>+ label
>
>  + label version = "1.0"
>  + label description = "测试描述命令"
>
>  描述docker镜像，说明镜像文件
>
>+ WORKDIR
>
>  + 如果目录不存在，会自动创建文件夹
>
>+ ADD & copy 复制文件
>
>  + add可以解压压缩文件，如： add test.tar.gz / #添加test文件至根目录，并进行解压
>  + add支持添加远程文件功能
>
>+ ENV 设置环境常量
>
>  + ENV JAVA_HOME /usr/local/openjdk8
>  + RUN ${JAVA_HOME}/bin/java ...



## dockerfile 执行指令

三个命令执行时机不同,由于cmd可以通过外界启动命令进行传参，entrypoint可以和cmd组合使用。

>+ run
>  + 在build时执行命令，构建镜像时执行命令
>  + shell命令书写格式，例：run yum install -y vim
>    + 使用shell执行时，当前shell是父进程，生成一个子进程
>    + 在子shell中执行脚本，脚本执行完成后，退出子进程shell，回到当前shell，子进程不会对父进程shell产生影响
>  + exec命令书写格式
>    + 使用exec方式，会用exec进程替换当前进程，并且进程id会保持不变
>    + 执行完成，直接退出，不会退回之前的进程环境
>+ entrypoint
>  + entrypoint 在容器启动的时候执行，
>  + dockerfile中只有最后一个entrypoint会被执行
>  + entrypoint ["ps"] #推荐使用exec格式书写
>
>+ cmd
>  + 容器启动后执行默认的参数或命令
>  + 如果dockerfile中出现呢多个cmd，则只有最后一个会被执行
>  + 如果容器启动时附加指令，则cmd被忽略
>  + CMD ["ps","-ef"] #推荐使用exec格式



## 构建redis镜像

```dockerfile
FROM centos
# gcc、gcc-c++ ：c语言源代码编译包，用于编译redis代码
# net-tools 网络工具包
# make 安装程序必备组件
RUN ["yum", "install", "-y", "gcc", "gcc-c++", "net-tools", "make"]
workdir /usr/local
ADD redis-4.0.14.tag.gz .
workdir /usr/local/redis-4.0.14
# 编译redis源码
run make && make install
workdir /usr/local/redis-4.0.14
# 添加redis-7000.conf 文件到redis目录下
add redis-7000.conf .
expose 7000
# 执行redis启动命令
cmd ["redis-server","redis-7000.conf"]
```

+ 执行编译命令

  编译镜像到当前目录下，镜像名:docker-redis。

  docker build -t weizz.com/docker-redis .

# 容器间link单向通信

![image.png](https://upload-images.jianshu.io/upload_images/26015807-7c4edeb61d46b663.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 基于应用名进行访问通信

# Bridge网桥双向通信

![image.png](https://upload-images.jianshu.io/upload_images/26015807-2b5017d07d85f6c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 网桥

  + 虚拟网络桥接，虚拟出来的组件，用于docker环境与外部环境进行通信。通过物理网卡与外界进行通信

  + 对容器从网络层面进行分组，将容器绑定到网桥上，则绑定相同网桥的两个容器即可相互通信
  + 相当于一个网关

```shell
# 列出当前环境默认网桥
[mocha@localhost:] ~ $ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
9b4172f0c9d5        bridge              bridge              local
df99909fd963        host                host                local
f731a06bbf92        none                null                local
# 创建网桥
[mocha@localhost:] ~ $ docker network create -d bridge bridge-demo01
ad2875fb754a7c05030be6d6724603673116f4ac78abe40aaf44d229ebdb2ec6
[mocha@localhost:] ~ $ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
9b4172f0c9d5        bridge              bridge              local
ad2875fb754a        bridge-demo01       bridge              local
df99909fd963        host                host                local
f731a06bbf92        none                null                local
# 将容器连接网桥
[mocha@localhost:] ~ $ docker network connect bridge-demo01 tomcat-demo
[mocha@localhost:] ~ $ docker network connect bridge-demo01 tomcate-demo02
# 测试网络连通情况
[mocha@localhost:] ~ $ docker exec -it a82cef766673 bash
root@a82cef766673:/usr/local/tomcat# ping tomcat-demo
PING tomcat-demo (172.18.0.2) 56(84) bytes of data.
64 bytes from tomcat-demo.bridge-demo01 (172.18.0.2): icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from tomcat-demo.bridge-demo01 (172.18.0.2): icmp_seq=2 ttl=64 time=0.089 ms
64 bytes from tomcat-demo.bridge-demo01 (172.18.0.2): icmp_seq=3 ttl=64 time=0.236 ms
64 bytes from tomcat-demo.bridge-demo01 (172.18.0.2): icmp_seq=4 ttl=64 time=0.155 ms

```

## 网桥原理



# volume 容器间共享数据

通过设置 -v 挂载宿主机目录

> + docker run --name 容器名 -v 宿主机路径:**容器内挂载路径** 镜像名
>   + 实例：docker run --name tomcat-demo -v /usr/webapps:/usr/local/tomcat/webapps tomcat
> + 创建共享容器 docker create --name **webpage** -v /webapps:/tomcat/webapps tomcat /bin/true
> + 共享容器挂载点 docker run ***--volumes-from*** **webpage** --name container-demo tomcat 





# 阿里云镜像加速器

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors