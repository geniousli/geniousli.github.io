---
title: java shell docker
date: 2020-09-15
tags: week
---

## A Little Book on Java 的总结
#### Basic

1. 编译 与 运行
编译: javac First.java 产生一个 First.class 文件
运行：java First 将运行 编译之后 First.class
* java  编译器 将 源代码 中的每个 class 转变为 对应的 class file 并存储 其 字节码
* 有 main 函数的 class才能够运行，一个项目中存在多个class 有 main 函数 是为了 将 项目分为不同的 可以运行的单元 方便测试

2. 基本类型
* Number, float, double, int
* Character: char a = ’a’; 
* Boolean: boolean true and false
* Strings: String title = "A Little Book on Java";
* Array: datatype[] ArrayName = new datatype[ArraySize]; 当 使用 index 超过 数组边界 时 会发生 ArrayIndexOutOfBoundsException 错误
3. 流程控制语句
* while loop

  ```java
  while <boolean-expression>
    statement
  ```

* for loop

  ```java
  for (initial-expression, boolean-expression, increment-expression) st  atement
  ```
* if

  ```java
  if (boolean-expression)
    statement
  else if (boolean-expression) 
    statement
  else
    statement
  ```

* break;

4.  抽象机制
5. Procedures
  * its name,
  * what kinds of parameters it expects (if any),
  * what kind of result it might return. 
6. class
  * Syntax of Class Declarations
  
```java
  class Hello{
  }
  ```
  * main method; java foo a b c, params as args array
  
```java
  class Foo {
    public static void main(String args[]){
        /* Body of main */
    }
  }
  ```
  * static variable & function and usage
  
```java
  class Foo {
    static int name;
    public static void showFoo(){
    }
  }

  classname.methodname(parameters); // static method usage
  classname.variablename;           // static variable usage
  ```
  * Visibility Issues （可见控制）: Public data and code is visible to all classes, while private data and code is visible only inside the class that contains it.

7. The Object Concept

  * The State of an Object (Instance variables)
  
```java
  class PointIn3D{
    //Instance Variables
    private double x;
    private double y;
    private double z;
  }
  ```
  * Constructors

  ```java
  class PointIn3D{
    private double x;
    private double y;
    private double z;

     //Constructors
    //This constructor does not take parameters
    public PointIn3D(){
      /* Initializing the fields of this object to the origin,
         a default point */
      x = 0;
      y = 0;
      z = 0;
    }

    //This constructor takes parameters
    public PointIn3D(double X, double Y, double Z){
      /* Initializing fields of this object to values specified by
         the parameters */
      x = X;
      y = Y;
      z = Z;
    }
  }
  ```
  * Creating an Object
  
```java
  //Creates a PointIn3D object with coordinates (0, 0, 0)
  new PointIn3D();
  //Creates a PointIn3D object with coordinates (10.2, 78, 1) new PointIn3D(10.2, 78, 1);
  ```
  * Object References
  
 ```java
   ReferenceType ReferenceName;

   PointIn3D p = new PointIn3D(1, 1, 1);
   ```
  * Accessing the Fields of an Object
  
```java
   ReferenceName.FieldName;
  ```
  *  The Behavior of an Object
  ```java
    ObjectReference.InstanceMethodName(Parameter-List)
  ```

  * The this reference: Inside an instance method, this is a reference to the object on which the instance method is invoked. Inside a constructor, this refers to the object that the constructor just created.

  ```java

  public PointIn3D(){
    this.x = 0;
    this.y = 0;
    this.z = 0;
  }
  public double getX(){
    return this.x;
  }

  ```

  * Inheritance: extends, super can use in subtype to call supertype methods

8. Rules for Method Lookup and Type Checking.
  * First the rules. Remember that there are two phases: compile time, which is when type checking is done and run time, which is when method lookup happens. Compile time is before run time.
  * The type checker has to say that a method call is OK at compile time.
  * All type checking is done based on what the declared type of a reference to an object is.
  * Subtyping is an integral part of type checking. This means if B is a subtype of A and there is a context that gets a B where A was expected there will not be a type error.
  * Method lookup is based on actual type of the object and not the declared type of the reference.
  * When there is overloading (as opposed to overriding) this is resolved by type-checking.

  ```java
  class myInt {
      private int n;
      public myInt(int n){
          this.n = n;
      }
      public int getval(){
          return n;
      }
      public void increment(int n){
          this.n += n;
      }
      public myInt add(myInt N){
          return new myInt(this.n + N.getval());
      }
      public void show(){
          System.out.println(n);
      }
  }

  class gaussInt extends myInt {
      private int m;  //represents the imaginary part
      public gaussInt(int x, int y){
          super(x);
          this.m = y;
      }
      public void show(){
          System.out.println("realpart is: " + this.getval() +" imagpart is: " + m);
      }
      public int realpart() {
          return getval()
              ;}
      public int imagpart() {
          return m;
      }
      public gaussInt add(gaussInt z){
          return new gaussInt(z.realpart() + realpart(),
                              z.imagpart() + imagpart());
      }
      public static void main(String[] args){
          gaussInt kreimhilde = new gaussInt(3,4);
          kreimhilde.show();
          kreimhilde.increment(2);
          kreimhilde.show();
          System.out.println("Now we watch the subtleties of overloading.");
          myInt a = new myInt(3);
          gaussInt z = new gaussInt(3,4);
          gaussInt w;
          myInt b = z;
          myInt d = b.add(b); //this does type System.out.print("the value of d is:

          // 这里面并没有错误， add 方法为 重载，而非重写，因为 方法签名不同。 同样会通过type check
          // w = z.add(b);
          // w = b.add(z);
          w = ( (gaussInt) b).add(z);//this does type check System.out.print("the value of w is: ");
          w.show();
          myInt c = z.add(a); //will this typecheck? System.out.print("the value of c is: ");
          c.show();
      }
  }

  ```
  9. The Exception Object
  *  分为两类： unchecked exceptions and checked exceptions.
  *  所有的exception 都发生在 runtime， 因为不是的话，要啥编译检查？
  *  Unchecked exceptions 与  checked exception 的区别主要在于： Unchecked exceptions happen because of the programmer’s carelessness，也就是说  unchecked exception 是可以预防的，可以避免的。两个主要的 unchecked exception 主要有： rrayIndexOutofBoundsException and NullPointerException
  *  所有其他的非 unchecked exception 则是：checked exceptions， 两个主要的exception 有 FileNotFoundException and IOException.
  10. 创建 新的 exception
  * 新创建的 exception 应该继承 exception 或者 任何 除 RunTimeException 之外的 子类。 因为 新创建的 exception  为 checked exception
  * An exception is thrown to indicate the occurrence of a runtime error. Only checked exceptions should be thrown, as all unchecked exceptions should be eliminated. 意思是： 只有 checked exceptions 需要throw 声明， unchecked exception 因为无法预测，只能 尽量消除掉。（If a method’s header does not contain a throws clause, then the method throws no checked exceptions.）
  11. Throwing an Exception
  ```java
  public static void main(String[] args) throws IOException,
                                                FileNotFoundException
  ```
  * A method’s header advertises the checked exceptions that may occur when the method executes
  * An exception can occur in two ways: explicitly through the use of a throw statement or implicitly by calling a method that can throw an exception 意思是：异常产生有两种方式：1. 直接抛出异常 2. 调用 能够抛出异常的函数
  12. Catching an Exception: catch 异常的方式同其他 语言一致， 即是 不断的递归的 解开栈，以找到合适的 catch。如果无法找到适合的 catch 则  使用默认的 default exception handler 来捕获异常，所以default exception handler 是在哪一层？main 层面吗？
  ```java
  try{
     code that could cause exceptions
  }
  catch (Exception e1){
     code that does something about exception e1
  }
  catch (Exception e2){
     code that does something about exception e2
  }
  ```

## Docker 书籍
### Docker 的结构：  客户端 + 服务器。 Docker 服务器 为一个守护进程，下层抽象 Docker 容器，与客户端配合 提供了 一个RESTful API 给 客户端。
### 概念： 镜像 与 容器。镜像是Docker世界的基石，类似于 面向对象中的 类， 所有容器 都是基于 镜像 运行的。也就是说 容器类似于 实例对象。镜像 是 Docker生命周期 中的 构建或 打包阶段，容器则是启动和 执行阶段。
  * Docker容器： 一个镜像格式，一系列标准的操作，一个执行环境
### docker 能够帮助我们做什么：
  * 加速本地开发和构建流程，因为其高效轻量，可以方便本地开发人员进行构建
  * 能够让独立服务在不同的应用环境中，得到相同的运行结果。
  * 创建隔离的环境进行测试
  * 都建一个多用户的平台及服务（PaaS）基础设施
  
### Docker的特性？ linux namespace 的作用：
  * 文件系统隔离：每个容器都有自己的root文件系统
  * 进程隔离： 每个容器都运行在自己的进程环境中
  * 网络隔离：荣期间的虚拟网络接口和IP地址都是分开的
  * 资源隔离和分组： 使用cgroups 将CPU 和内存之类的资源独立分配给每个Docker容器
  * 写时复制： 文件系统都是通过写时复制创建的，这就意味着文件系统是分层的、快速的、占用小的磁盘空间
  * 日志： 容器产生的STDOUT stderr, stdin 这些IO都会被收集并记录入日志，用来进行日志分析和故常排查
  * 交互式shell： 用户可以创建一个伪造tty中断连接的，STDIN，为容器提供一个交互式的Shell 
### Docker 守护进程（服务器）： 
  * 需要以root权限运行，以便处理 诸如 挂载文件系统 等特殊操作。
  * 守护进程 监听 /var/run/docker.sock Unix套接字 来获得 Docker客户端的请求。
  * 启动： ubuntu 中 start docker, stop docker， centos 中 service docker stop service docker start
### Docker 操作：Docker容器 则为 Docker的运行态，运行着 用户的process， 容器内部同linux namespace 一样，进行了完全的隔离
  * 创建容器： docker run -i -t ubuntu /bin/bash  -i 标志打开容器的STDIN， -t 为容器 分配一个伪 tty 终端
  * 停止容器： docker stop daemon_dave; docker stop 2q3412c
  * 删除容器： docker rm daemon_dave // 容器停止运行并不会自动清理，而需要 手动 rm，因为可能存在 重新 start 的需求。类似于进程
  * 命名容器： docker run --name test_container -i -t ubuntu /bin/bash 容器的命名必须是唯一的。
  * 创建守护式容器： (daemonized container): docker run --name daemon_dave -d ubuntu /bin/bash -c "while true; do echo hello world; sleep 1; done"//  -d 为 daemon运行的标志， 容器中的进程不能够退出
* 重启已经停止的容器： docker start test_container
  * 附着到容器： docker attach test_container
  * 查看容器日志： docker logs -f daemon_dave //
  * 查看容器内进程： docker top daemon_dave
  * 在容器内运行进程： docker exec -d daemon_dave touch /etc/new_config_file 可以在现有的容器内 启动新进程，无论是后台任务还是交互式任务, docker exec -it <container_id_or_name> /bin/bash
  * 自动重启容器： docker run --restart=always --name daemon_dave // docker可以通过设定  --restart 标志来检测 容器的退出代码 来决定是否重启容器
  * 深入容器： docker inspect daemon_dave
  * 查看运行中的容器： docker ps

### Docker 镜像: 
  * 列出镜像：docker images
  * 拉取镜像： docker pull ubuntu
  * 查找镜像: docker search puppet
  * 使用Dockerfile构建镜像： 
#### 使用Dockerfile构建镜像： Dockerfile 有一系列 指令和参数构成，每条命令 都必须为大写字母 比如 RUN FROM 后面跟随参数， Dockerfile 从上到下执行，每条指令都会创建一个新的镜像层并对镜像进行提交。流程如下： 

```shell
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
EXPOSE 80

docker build -t="static_web" ./
docker run -d -p 80 --name static_web static_web nginx -g "daemon off;"
```

##### 1. 流程:
  * Docker 从基础镜像运行一个容器
  * 执行一条指令，对容器做出修改
  * 执行类似docker commit 操作，提交一个新的镜像层
  * Docker在基于刚才提交的镜像层运行一个新容器
  * 执行Dockerfile 中的下一条命令，知道所有指令运行完毕
##### 2. 详细： 
  * 每个Dockerfile 的第一条指令 都是从FROM 开始的，该镜像 被称为基础镜像
  * expose 指令 指明 容器 使用的端口号，但 docker 在运行容器时候，并不会打开端口，需要使用指令来明确docker 打开特定的端口号
  * Dockerfile 的构建方式，导致如果 在因为一些命令失败，则可以得到一个 最近成功命令 的镜像，可以基于该命令 运行一个交换式的容器 来方便调试, 比如 docker run -t -i 最后commit  /bin/bash
##### 构建: docker build -t="xxxx" ./
  * docker 构建 过程会添加缓存： 可以如此 docker build --no-cache -t="xxxx" 来跳过缓存
  * docker 在修改命令之后的 的命令 重建缓存
  * 可以添加 ENV REFRESHED_AT 2020-10-09 在头部，当希望 重建镜像时候，可以更改 其时间来 进行  之后命令的重建
  * 查看镜像： docker images; 
  * 查看镜像的构造过程: docker history
  * 容器端口： -p 用来指定端口， 方式有： -p 80:80 -p 127.0.0.1:80:80, -p 127.0.0.1::80 -P 前两个指定端口绑定到 容器中的端口， 后面两个则 将随机端口绑定到 容器中的端口， -P 表示 将随机的本地端口 绑定到 Dockerfile中的expose 的端口

##### Dockerfile 指令: 
  * CMD: 指定容器启动时候的运行的命令， 区分于RUN 为镜像被构建时运行的命令，CMD 则是容器启动时候运行的命令。同docker run时候指定的命令， docker run中的命令会覆盖CMD命令，即 docker run中指定了命令，则CMD中的命令将不会被执行
  * ENTRYPOINT： 区分于 CMD，需要传递 --entrypoint 来代替,  docker run  custom-cmd ...  会替代 CMD 一起传递给 ENTRYPOINT，示例：(即  ENTRYPOINT 是 docker的 默认入口， cmd 为默认 参数， docker run cmd 时候， cmd 代替 默认cmd 传递给 entrypoint， 但是 entrypoint  并非 不可代替， 可以指定  --entrypoint 来执行 出 /bin/sh 之外的 可执行文件 代替)  参考文档 (stackoverflow)[https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile] (docker)[https://yeasy.gitbook.io/docker_practice/image/dockerfile/entrypoint] (docker hub) [https://docs.docker.com/engine/reference/builder/] (images) [https://stackoverflow.com/questions/44769315/how-to-see-docker-image-contents]

    ```shell
    ENTRYPOINT ["/usr/sbin/nginx"]
    CMD ["-h"]

      docker  run -t -i static_web -g "daemon off;"
    ```
  * WORKDIR: 为后续的指令 执行  设定工作目录
  * ENV: 指定环境变量， 在后续的 RUN 中使用，也可以在其他命令中使用环境变量。 该变量 持久的保存到 从我们的镜像创建的任何容器中。 相反 在docker run -e 中传递的环境变量 则一次性有效
  
    ```shell
    ENV RVM_PATH /home/rvm
    RUN gem install unicorn
    // 等同于  RVM_PATH=/home/rvm gem install unicorn

    ENV TARGET_DIR /opt/app
    WORKDIR $TARGET_DIR
    ```
  * VOLUME: 向 从该镜像创建的容器 添加卷。卷是容器中的特殊目录 ，可以跨越文件系统，进行共享，提供持久化功能，有如下特性
    * 卷 可以再 容器间 共享和重用
    * 一个容器 可以不是必须 和 其他容器共享卷
    * 对卷的修改 立即生效
    * 对卷的修改不会影响镜像
    * 卷会一直存在知道没有任何 容器使用它。(标志着 卷 是由 docker管理的，而非容器，也非操作系统)
    * VOLUME ["/opt/project", "/data"] 可以使用数组形式 创建多个挂载点
  * ADD: 将 构建环境下 的文件或 目录  复制到 镜像中。ADD software /opt/application/software; ADD source target 
    * 其中source可以是 文件或者目录 或者url，不能对构建目录之外的文件进行ADD操作。(因为docker只关心 构建环建， 构建环境之外的任何东西 在命令中都是不可用的)
    * target 如果目录不存在的话，则 docker会创建 全路径，新建文件目录的权限 为0755
    * ADD命令会屎之后的命令不能够使用缓存。
    * ADD 会将 归档文件 进行 解压，例如 ADD latest.tar.gz /var/www/wordpress/
  * COPY： 区分于 ADD， copy只做纯粹的复制操作。不会进行解压缩操作.
  * 产出镜像： docker rmi static_web

### 实践：
-v 允许我们将宿主机的目录作为卷，挂在到容器里。-v source:target 

* 构建 Redis 镜像

```shell
FROM ubuntu:14.04
ENV REFRESHED_AT 2020-10-09
RUN apt-get update
RUN apt-get -y install redis-server redis-tools
EXPOSE 6379
ENTRYPOINT ["/usr/bin/redis-server"]
CMD []
```
* 连接到 redis 容器: 容器之间 互连。 如果使用 映射到宿主机 的ip 来连接到 对应的容器，在 容器重启之后，因为其port会改变（当然可以使用 参数固定 其对应的宿主机 port） 会导致 之前的链接配置失效，从而无法使用 该种 方法 建立长久的固定的链接。
* docker 提供了另一种方法: --link 使用 该标志 创建两个容器的父子链接。链接让 父容器有能力访问子容器， 并将子容器的一些详细信息分享给父容器，应用程序可以利用 这些信息 建立链接。示例：

```shell
docker run  -d --name redis_con  redis

docker run -p 4567 --name webapp --link redis:db -t -i sinatra /bin/bash
// 该命令中 使用--link标志创建了  sinatra 到 redis_conn 的父子链接关系
```
* 链接的特点： 
  * 使子链接 无需公开端口，从而更安全一些。容器端口不需要在宿主机 上 公开，就可以限制被攻击的方面，减少应用暴露的网络
  * 被连接的容器 必须运行在同一个 Docker宿主机上，不同的Docker宿主机 上的容器不能够互相链接
* 链接的实现方法： docker 在父容器里的两个地方写入了链接信息。 
  *  /etc/hosts 文件
  *  包含链接信息的环境变量 ( 自动创建的环境变量包括： 子容器的名字， 子容器服务所运行的 协议 ip 端口 )  

```shell

docker run -d --name redis_con redis
docker run --link redis:db -i -t ubuntu /bin/bash

root@31c4f6ac36a4:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	db 949baab68dd4 redis_con
172.17.0.4	31c4f6ac36a4


root@31c4f6ac36a4:/# env

DB_PORT_6379_TCP_ADDR=172.17.0.3
DB_PORT_6379_TCP=tcp://172.17.0.3:6379
DB_PORT=tcp://172.17.0.3:6379
....
```
* 所以： 在应用程序中  通过使用 环境变量 来链接 子容器 是非常方便的方法。

```shell
require 'uri

uri = URI.parse(ENV['DB_PORT'])
redis = Redis.new(:host => uri.host, :port => uri.port)
```


#### 实践： 通过 Jekyll Apache 来构建 自动构建一个博客网站

* jekyll 镜像

```shell
FROM ubuntu:14.04
ENV refreshed_at 2020-10-10

RUN apt-get update
RUN apt-get install -y ruby ruby-dev make nodejs
RUN gem install --no-rdoc --no-ri jekyll

VOLUME /data
VOLUME /var/www/html

WORKDIR /data

ENTRYPOINT [ "jekyll", "build", "--destination=/var/www/html" ]

docker build -t jekyll ./
```

* apache 镜像

```shell
FROM ubuntu:14.04
ENV REFRESHED_AT 2020-10-10

RUN apt-get update
RUN apt-get install -y apache2

VOLUME [ "/var/www/html" ]
WORKDIR /var/www/html

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/logapache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2

RUN mkdir -p $apache_run_dir $apache_lock_dir $apache_log_dir
EXPOSE 80

ENTRYPOINT [ "/usr/sbin/apache2" ]
CMD ["-D", "foreground"]

docker build -t apache ./
```

> docker run -v /home/james/blog:/data/ --name jekyll_con jekyll
> docker run -d -P --volumns-from jekyll_con --name apache_conn apache
> // 这里使用了 标志 --volumes-from 标志 将指定容器 中的所有卷 添加到 新创建 的容器中。意味着容器 apache_conn 可以访问 容器 jekyll_conn 中的所有卷，即： 可以访问 jekyll_conn 产生的博客文件 目录 /var/www/html 中的内容。
> // 卷 只有在没有容器 使用的时候才会被清理，也就是说 在 删除 docker rm jekyll_conn 之后 /var/www/html 中的内容就不复存在了 （这里面是否需要 同时删除 apache_conn 才可以？ 因为apache_conn 依然在使用，把持 该卷. 可以进行实验验证）


* 备份卷：

> docker run --rm --volumnes-from jekyll_conn -v $(pwd):/backup ubuntu tar cvf /backup/blog.tar /var/www/html
> 创建一个 docker 容器，将 共享的 /var/www/html 卷，进行打包 到 外部目录中。


#### 不使用 ssh 管理 Docker 容器

* 传统上将，通过ssh 登入运行环境或者虚拟机 来管理服务，在Docker世界中， 大部分容器只运行一个进程，所以不能够使用该方法进行访问。可以通过如下方式进行访问： 使用卷 或者 链接 完成大部分管理操作。比如服务通过某个网络接口做管理， 或者使用 Unix套接字 做管理， 就可以通过 卷 来公开这个套接字，或者发信号 可以 docker kill -s <signal> <container>
* 如果是登录 容器，则可以使用 nsenter 工具。使用方法如下：

```shell
  PID=$(docker inspect --format {{.State.Pid}} 949baab68dd4)
  nsenter --target 16870 --mount --uts --ipc --net --pid
  nsenter --target 16870 --mount --uts --ipc --net --pid ls
```
#### 对 Docker 容器的编排，或者是非常重要的一步. K8s

### ? 实际中遇到的一些问题： 
##### ssh：  ssh 经常使用， ssh ip "command" 可以用来在远端 ip 上执行command， 但是 当command存在复杂的 command 比如 for 循环，则总是不能够正确执行， 比如 sh: 2: Syntax error: word unexpected (expecting "do")， 在进行了一番查找之后， https://stackoverflow.com/questions/26325685/execute-for-loop-over-ssh  发现自己忽略到了 在 command 中的 $ 会，会在 shell 传递给 $ip 机器 之前，就会展开， 这将导致  远端$ip 接收到的 command并非 我们传递给 ssh的command。两种解决办法：
* 1. 将command 中所有的 $ 进行转义 \$
* 2. 将 "command" 替换为 'command', 即展开 command中的所有内容 
