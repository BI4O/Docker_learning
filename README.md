# Docker入门

## 安装

因为现在的新版Docker for win 已经支持在win10的家庭版中进行安装了，但是需要用到WSL2(Windows Subsystem for Linux)作为启动时候的内核

- ##### 第一步：安装/更新WSL2

  1. 安装前需要允许wsl2运行这个选项
     在Powershell中键入下列并回车运行

     `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

  2. 因为只要你的系统是跟着微软的官方更新的话，其实wsl早已经存在了的，所以你要做的只是更新这个wsl2，在浏览器中输入并回车

     `ms-settings:windowsupdate`

     下载完后一直安装就可以

- ##### 第二步：安装Docker for win

  1. 下载Docker安装包，浏览器前往

     `https://hub.docker.com/editions/community/docker-ce-desktop-windows/`

     选择**Get Docker Desktop for Windows (stable)**

  2. 一直下一步直至安装完成
  
- ##### （第三步：可能出现的bug）

  - 如果启动docker for win时候发生了错误，可以尝试使用管理员身份打开cmd

    `netsh winsock reset`

    这个命令可以重新初始化网络环境，以解决由于病毒或软件冲突原因造成的参数错误问题

## 具体概念

- ### 镜像，容器和仓库

  1. #### 镜像Images

     Docker的镜像可以简单地比喻为电脑装系统用的系统盘，这里面包括了操作系统以及必要的软件，例如一个镜像可以包含一个完整的ubuntu系统并安装了mysql和python等。**注意镜像是只读的，意味我们应该避免将我们的开发中的项目放进镜像里面**，否则你利用这个镜像每次启动为容器的时候就会包含你的未开发完的的项目代码在里面

     > 查看镜像命令 docker images

  2. #### 容器Containers

     Docker的容器可以理解为提供了系统硬件环境，但是是正常跑项目程序，消耗硬件资源的东西，我们可以把容器理解为一个Linux的电脑，它是基于镜像启动的，并且每个容器（程序）之间是项目隔离的，包括内部的子程序和占用的内存空间。容器在启动的时候基于镜像创建一层**可写层**作为最上层

     > 查看容器命令 docker ps -a
     >
     > 查看正在运行的容器命令 docker ps

  3. #### 仓库Repo

     跟Git很类似，我们可以从中心仓库下载镜像，也可以从自建仓库下载。同时我们可以把经过定制化开发的容器commit到本地成为一个定制化的镜像，然后push到远程仓库。仓库分为公开和私有，最大的公开仓库就是官方的DockerHub，国内也有很多公开的仓库，例如阿里云

## 基本用法

- #### 运行容器

  - ##### 比如运行一个Ubuntu容器

    `docker run -idt --name ub ubuntu /bin/bash`

  - ##### 这里的docker run命令实际上经过了3步

    1. 查看本地有没有ubuntu这个镜像，有的话就下一步，没有的话首先docker pull这个镜像下来，注意还可以通过指定版本号，比如ubuntu:16.2如果不指定的话那就是默认找最新的版本
    2. 现在已经有了ubuntu的镜像（ image）,但是他现在只是可读的状态，要把它变为可写的状态需要使用docker create为他分配资源，但是这个时候创建的容器依然是没有运行的
    3. 使用docker start可以把这个容器运行起来，只有运行的容器才可以在docker ps中展示

  - ##### 参数解释

    - -it表示告诉docker为容器分配一个可以用于交互的虚拟终端（命令行工具），
    - -d表示detach，告诉docker在后台运行容器的守护进程，如果没有这个参数的话，ubuntu容器内部如果没有进程在运行的话，这个容器就会关闭而不是在运行状态
    - --name ub 表示我把容器命名为ub，如果指定容器的名，docker会自动为这个容器名名，以后可以通过完整容器id/容器id前三位/容器名选择某个容器

  - ##### 注意事项

    - 使用的容器暂时不用了不想占用现有的硬件资源，可以用docker stop停掉活着使用docker pause暂停掉，但是这个停掉并不会删掉这个容器，以后你想再用这个以前用过的容器的话可以使用docker start 容器名来进行启动，你之前在容器里面进行过的修改都会在start之后保留

    - 鉴于上面的这一点，我们想要运行某个容器的时候应该首先查看看有没有存在这个相关的镜像或者有没有存在已经create好的容器，使用docker images和docker ps -a进行查看

      case1: 如果已经存在你需要的镜像image_name：

      `docker run -itd --name new_container_name image_name`

      case2: 如果已经存在你需要的容器container_name（处于未启动状态的容器）

      `docker start -i container_name`

      如果你只是想启动并不想对这个容器进行交互，那就不要这个`-i`

- #### 使用容器进行开发

  1. ##### 关于容器运行的问题

     你能进入这个容器进行开发的前提是这个容器必须是运行状态，但是如果你在创建容器也就是docker create的时候没有使用`-it`这个参数的话，你就算是docker start了这个容器，它也会在启动之后马上关闭，而如果你加上了`-it`这个参数，那么容器内就始终运行着一个等待输入的命令行终端进程，这个进程就是`/bin/bash`。如果你是用docker run 来启动一个容器的话，就加入`-itd`参数

  2. ##### 与容器进行交互（进入容器）的方法

     - 使用docker attach container命令进入容器的命令行终端，这个方法不推荐，你会发现你退出之后，这个命令行终端会被关闭同时你的容器也关闭了。因为容器内如果没有进程的话，那么这个容器就会被关闭，所以不推荐这个方法
     - 使用`docker exec container /bin/bash`表示启动容器内的bash命令行，这种方法启动容器后再退出的时候不会使容器关闭

  3. ##### 案例一：从头开始创建一个容器并在容器内部手动安装软件

     比如说我在ubuntu的容器内里安装python

     1. 首先拉取ubuntu镜像，默认选最新版

        `docker pull ubuntu`

     2. 使用docker images确认你已经有这个镜像

     3. 创建并运行一个容器，取名叫pyub

        当然你可以先create这个容器再start运行这个容器，这里我选择更快的方式

        `docker run -idt --name pyub`

     4. 进入容器pyub的命令行终端

        `docker exec -it pyub /bin/bash`

     5. 更新一下apt-get再安装python

        `apt-get update`

        `apt-get install python3.7`

  4. ##### 案例二： 利用本地文件传入容器内

     1. 比如把本地的**文件**传到容器内的test文件夹（注意这个文件夹必须要存在）

        `docker cp hah.txt pyub:/test`

     2. 比如把容器内的**文件夹**传到本地

        `docker cp pyub:/test .\somedir`

  5. ##### 案例三： 利用Dockerfile构建预装了软件的镜像

     1. Dockerfile是什么？

        是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明，dockerfile的使用方法，官方的推荐是新建一个本地的空的文件夹，在这里面写好Dockerfile文本文件，把需要的文件放到这个文件夹内，然后在这个文件夹的路径下使用命令`docker build `

     2. 基本使用

        使用apt-get安装，前提要有网络

        ~~~shell
        FROM ubuntu
        RUN apt-get update\
        	&& apt-get python3.7
        ~~~

        使用源码安装需要本地编译，前提先下载好python源码而且需要你的镜像有gcc编译器，建议先确认安装

        `apt-get install build-essential -y`

        ```
        FROM ubuntu
        COPY . /test
        RUN cd /test\
        && tar -zxvf Python-3.7.9.tgz\
        && cd Python-3.7.9\
        && ./configure --prefix=/usr/local/python3.7\
        && make && make install
        ```

        写好你的Dockerfile之后，你就可以使用命令

        `docker build -t new_image_name .`
        
        `-t`表示你要为你新建的这个镜像取个名字，可以`-t new_image_name:1`加个版本号
        
        `.`表示你要使用当前目录下的所有文件作为构建所用的上下文文件
        
     
  6. ##### 案例四：构建一个只运行某个特定程序的容器
  
     实现这个需求还是要用到Dockerfiler
  
     `CMD`命令，这个dockerfile命令引擎只会取最后一条执行，也就是写多条也只会执行最后一条，而且他有个特点，可以在`docker run`的时候传的参数覆盖。
  
     `ENDTRYPOINT`命令，跟CMD类似，docker命令引擎也会只取最后一条来执行，有意思的是，`docker run` 不能直接通过传参来修改这个命令里面执行的内容。
  
     - 下面我想要在一个镜像被`docker run`启动为一个镜像的时候，运行一个app.py文件，而且这个文件有两个参数。我不希望别人用这个镜像运行容器的时候运行别的文件，但是我又需要这个文件的传入参数可以在`docker run`的时候做修改
  
     - 首先还是本地创建一个新的空白文件夹pp，放入我写好的app.py和Dockerfile两个文件
  
     - pp/app.py
  
       ~~~python
       import argparse
       
       parser = argparse.ArgumentParser(description='字符串拼接')
       parser.add_argument('-a', type=str, default='A')
       parser.add_argument('-b', type=str, default='B')
       args = parser.parse_args()
       
       print(args.a + args.b)
       ~~~
  
       pp/Dockerfile
  
       ~~~dockerfile
       # 为了展示方便，我使用的是一个已经安装了python的ubuntu镜像
       FROM pyub
       RUN mkdir test
       COPY app.py ./test
       ENTRYPOINT ["python", "./test/app.py"]
       CMD ["-a", "cmd默认1", "-b", "cmd默认2"]
       ~~~
  
     - 在pp这个文件夹路径下打开cmd
  
       `docker build -t test .` 根据当前的Dockerfile新建一个镜像文件名叫test
  
       1. case1: `docker run test`
  
          直接跑的结果就是按照Dockfile的优先，也就是会打印`cmd默认1cmd默认2`
       
       2. case2: `docker run test -a C`
       
          指定了其中的一个参数，那么这时候dockerfile里面的`CMD`命令就会因为覆盖而失效，注意这时候的entrypoint命令还是生效的，也就是我这个参数还是传给app.py文件，那么打印出来的结果是`CB`。
       
          这里的第二个参数`-b`我没有去指定，但是因为app.py文件里面`-b`的默认值就是`B`所以会出现我指定的参数拼接上app.py内默认参数的值的现象
  
  7. 总结
  
     在我们需要定制化开发所需要的容器的时候，推荐的做法是先在一个网络环境良好的主机上进行容器设计，然后使用`docker commit container_name`把设计好的容器提交为一个镜像。然后用`docker save image_name`来保存，需要用的时候就用`docker load`
  
     
  
     
