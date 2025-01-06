---
title: docker 入门
date: 2017-04-25
tags: docker
---

#### docker

#### 概念
  image(景象), container(容器),volume（数据卷）
  ```
    镜像(Image)和容器(Container)的关系,就像是面向对象程序设计中 的 类 和 实例 一样,镜像是静态的定义,容器是镜像运行时的实体。容器可以被 创建、启动、停止、删除、暂停等。
    容器的实质是进程,但与直接在宿主执行的进程不同,容器进程运行于属于自己的 独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、 自己的进程空间,甚至自己的用户 ID 空间
    每一个容器运行时,是以镜像为 基础层,在其上创建一个当前容器的存储层,我们可以称这个为容器运行时读写而 准备的存储层为容器存储层。
    容器不应该向其存储层内写入任何数据,容器存储 层要保持无状态化。所有的文件写入操作,都应该使用 数据卷(Volume)、或者 绑定宿主目录,在这些位置的读写会跳过容器存储层,直接对宿主(或网络存储)发 生读写,其性能和稳定性更高。
  ```
  > 容器是景象的基础， 容器的启动运行按照 存储曾 的概念运行， 但是在容器消亡时候，数据也随之消失， 数据卷随之而出，

#### 一些常用命令
  1. docker pull nginx:latest
  2. docker run -d -p 80:80 --name webserver nginx, docker stop webserver, docker rm webserver
  3. docker run -it(一个是 -i :交互式操作,一个是 -t 终端) --rm(这个参数是说容器退出后随之将其删除)
  4. docker images
  5. docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]] # 提交产生新的景象
#### 使用 Dockerfile 定制镜像
  1. 使用 Dockerfile 定制镜像
    ```
      mkdir mynginx
      cd mynginx
      touch Dockerfile


      FROM nginx
      RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
    ```
    2. 编写Dockerfile的常用命令
    > Dockerfile 中每一个指令都会建立一层, RUN 也不例外。每一个 RUN 的行为,就和刚才我们手工建立镜像的过程一样:新建立一层,在其上执行 这些命令,执行结束后, commit 这一层的修改,构成新的镜像。

     1. run
        > RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html

     2. copy, copy package.json /usr/src/app/, 注意执行的context， copy 的源文件只能是相对路径，并且是在context下的才行， 超出context docker并不知道是否存在
     3. cmd命令， 容器启动时候的默认命令， 可以在run 之后加参数取代掉， 文件中存在多个cmd， 使用最后的cmd
     4. ENTRYPOINT 可以配合cmd，让cmd接受参数，
     5. ENV 设置环境变量
        ```
        ENV VERSION=1.0 DEBUG=on \
            NAME="Happy Feet"
        ```
     6.  ARG 构建参数,  ARG 所设置的 构建环境的环境变量,在将来容器运行时是不会存在这些环境变量的
     7. VOLUME 定义匿名卷, VOLUME /data
     8. WORKDIR 指定工作目录,
        > 使用 WORKDIR 指令可以来指定工作目录(或者称为当前目录),以后各层的当前 目录就被改为指定的目录,如该目录不存在, WORKDIR 会帮你建立目录。
        错误示例：
        RUN cd /app
        RUN echo "hello" > world.txt
        Dockerfile 中,这两行 RUN 命令的执行环境根本不同,是两个完全不同的容器。 这就是对 Dokerfile 构建分层存储的概念不了解所导致的错误。
        之前说过每一个 都是启动一个容器、执行命令、然后提交存储层文件变更。 第一层 的执行仅仅是当前进程的工作目录变更,一个内存上的变 化而已,其结果不会造成任何文件变更。而到第二层的时候,启动的是一个全新的 容器,跟第一层的容器更完全没关系,自然不可能继承前一层构建过程中的内存变 化。
        因此如果需要改变以后各层的工作目录的位置,那么应该使用 WORKDIR 指令。

      9. USER 指定当前用户
      10. ONBUILD 为他人做嫁衣裳
      > 只有当以当前镜像为基础镜像,去 构建下一级镜像的时候才会被执行。
        Dockerfile 中的其它指令都是为了定制当前镜像而准备的,唯有 ONBUILD 是 为了帮助别人定制自己而准备的。

    3. docker build -t nginx:v3 .
      > 镜像构建上下文(Context)， 中的概念比较重要，一般情况下就是指的执行目录. docker会降目录夏的所有文件上传，可以使用类似.gitignore中的.dockerignore来忽略context中不需要的文件

    4. 容器的重要概念理解
      > 对于容器而言,其启动程序就是容器应用进程,容器就是为了主进程而存在的,主 进程退出,容器就失去了存在的意义,从而退出,其它辅助进程不是它需要关心的东西



#### 操作 Docker 容器
  1. 概念
    > 容器是独立运行的一个或一组应用,以及它们的运行态环境。, 容器可以是一组应用吗?

  2. 启动
    docker start
  ```
      当利用 docker run 来创建容器时,Docker 在后台运行的标准操作包括: 检查本地是否存在指定的镜像,不存在就从公有仓库下载
    ￼￼101
    启动
    ￼利用镜像创建并启动一个容器 分配一个文件系统,并在只读的镜像层外面挂载一层可读写层 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去 从地址池配置一个 ip 地址给容器
    执行用户指定的应用程序
    执行完毕后容器被终止
  ```  
  3. docker stop
  4. 后台运行(background)
    > 更多的时候,需要让 Docker在后台运行而不是直接把执行命令的结果输出在当前
宿主机下。此时,可以通过添加 -d 参数来实现。

    > 容器是否会长久运行,是和docker run指定的命令有关,和 -d 参数无关。

    ```
    将当前的日志输出到 屏幕上
    sudo docker run ubuntu:14.04 /bin/sh -c "while true; do echo h
      ello world; sleep 1; done"
      hello world
      hello world
      hello world
      hello world

      sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do ech
      o hello world; sleep 1; done"

      sudo docker logs [container ID or NAMES]
      hello world
      hello world
      hello world
    ```

    5. 进入容器
      ```
      attach 命令
        docker attach 是Docker自带的命令。下面示例如何使用该命令。
        $ sudo docker run -idt ubuntu
        243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
        $ sudo docker ps
        CONTAINER ID
        TED
        243c32535da7
        econds ago
        c_hypatia
        $sudo docker attach nostalgic_hypatia
        root@243c32535da7:/#
      ```

#### docker 数据管理(数据卷， 数据卷容器, Data volumes, Data volume containers)
  1. 数据卷特征：
    1. 数据卷可以在容器之间共享和重用
    2. 对数据卷的修改会即时生效
    3. 对数据卷的更新不会影响镜像
    4. 数据卷一直存在即便容器被删除

    > *注意:数据卷的使用,类似于 Linux 下对目录或文件进行 mount,镜像中的被指定 为挂载点的目录中的文件会隐藏掉,能显示看的是挂载的数据卷。*

  2. 创建数据卷，  docker run -v 标记创建一个数据卷并挂在到容器中， 可以挂在多个数据卷
    > sudo docker run -d -P --name web -v /webapp training/webapp py thon app.py
    也可以在Dockerfile中使用VOLUME添加数据卷到容器
   sudo docker run -d -P --name web -v /src/webapp:/opt/webapp tr aining/webapp python app.py
   使用命令挂载主机的 /src/webapp/ 到容器的/opt/webapp 中， 本地目录必须是绝对路径， Dockerfile中不能使用绝对路径， 因为分线出去不同操作系统不一样， 不支持

  3. 查看数据卷的具体信息
    docker inspect web 从中找到数据卷相关的部分
