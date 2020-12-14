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

- ### 运行容器

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

- ### 使用容器进行开发

  1. #### 关于容器运行的问题

     你能进入这个容器进行开发的前提是这个容器必须是运行状态，但是如果你在创建容器也就是docker create的时候没有使用`-it`这个参数的话，你就算是docker start了这个容器，它也会在启动之后马上关闭，而如果你加上了`-it`这个参数，那么容器内就始终运行着一个等待输入的命令行终端进程，这个进程就是`/bin/bash`。如果你是用docker run 来启动一个容器的话，就加入`-itd`参数

  2. #### 与容器进行交互（进入容器）的方法

     - 使用docker attach container命令进入容器的命令行终端，这个方法不推荐，你会发现你退出之后，这个命令行终端会被关闭同时你的容器也关闭了。因为容器内如果没有进程的话，那么这个容器就会被关闭，所以不推荐这个方法
     - 使用`docker exec container /bin/bash`表示启动容器内的bash命令行，这种方法启动容器后再退出的时候不会使容器关闭，因为基础镜像不一样，有时候你可能要把`bash`改成`sh`

  3. #### 案例一：从头开始创建一个容器并在容器内部手动安装软件

     比如说我在ubuntu的容器内里安装python

     1. 首先拉取ubuntu镜像，默认选最新版

        `docker pull ubuntu`

     2. 使用`docker images`确认你已经有这个镜像

     3. 创建并运行一个容器，取名叫pyub

        当然你可以先create这个容器再start运行这个容器，这里我选择更快的方式

        `docker run -idt --name pyub`

     4. 进入容器pyub的命令行终端

        `docker exec -it pyub /bin/bash`（别的Linux系统发行版如alpine是`bin/sh`）

     5. 更新一下apt-get再安装python

        `apt-get update`

        `apt-get install python3.7`

  4. #### 案例二： 利用本地文件传入容器内

     1. 比如把本地的**文件**传到容器内的test文件夹（注意这个文件夹必须要存在）

        `docker cp hah.txt pyub:/test`

     2. 比如把容器内的**文件夹**传到本地

        `docker cp pyub:/test .\somedir`

  5. #### 案例三： 利用Dockerfile构建预装了软件的镜像

     1. Dockerfile是什么？

        是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明，dockerfile的使用方法，官方的推荐是新建一个本地的空的文件夹，在这里面写好Dockerfile文本文件，把需要的文件放到这个文件夹内，然后在这个文件夹的路径下使用命令`docker build  -t new_image_name .`

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
        
        `-t`表示你要为你新建的这个镜像取个名字，可以`-t new_image_name:1`加个版本号`1`或者不加
        
        `.`表示你要使用当前目录下的所有文件作为构建所用的上下文文件，也是Dockerfile的所在目录
     
  6. #### 案例四：构建一个只运行某个特定程序的容器
  
     实现这个需求还是要用到Dockerfiler
  
     `CMD`命令，这个dockerfile命令引擎只会取最后一条执行，也就是写多条也只会执行最后一条，而且他有个特点，可以在`docker run`的时候传的参数覆盖。
  
     `ENDTRYPOINT`命令，跟CMD类似，docker命令引擎也会只取最后一条来执行，有意思的是，`docker run` 不能直接通过传参来修改这个命令里面执行的内容。
  
     - 下面我想要在一个镜像被`docker run`启动为一个容器的时候，这个容器只运行一个app.py文件，而且这个文件有两个参数。我不希望别人用这个镜像运行容器的时候运行别的文件，但是我又需要这个文件的传入参数可以在`docker run`的时候做修改
  
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
       
          指定了其中的一个参数，那么这时候dockerfile里面的`CMD`命令就会因为覆盖而失效（注意是整行失效而不是只第一个-a参数失效），注意这时候的entrypoint命令还是生效的，也就是我这个参数还是传给app.py文件，那么打印出来的结果是`CB`。
       
          原因：这里的第二个参数`-b`我没有去指定，但是因为app.py文件里面`-b`的默认值就是`B`所以会出现我指定的参数拼接上app.py内默认参数的值的现象
  
  7. #### 案例五：构建一个可以实时同步本地主机文件的容器
  
     - 这个案例就要用到**挂载**的知识了，因为我们有时候想要的是在win上面用docker可以边构建边调试一个容器，如果每次都要像**案例二**那样先使用`docker cp`把本地的所需文件复制到容器内，再在容器内部使用这些文件，那真的是太繁琐了
  
       docker提供一个方法可以在使用`docker run`启动一个容器的时候加上`-v`参数，始得运行中的容器能够实时地使用本地某个文件夹的东西
  
       你可以设想成这样，你的win主机有一个文件夹`C:\User\ys\wintest`，你想在`docker run`启动一个镜像为容器的时候用到这个文件夹内的文件
  
     1. 下面我利用一个名为`pyub`的镜像启动一个新的容器，并把wintest文件夹装载到容器的test文件夹
  
        > :roll_eyes: 注意格式是 **本地文件夹绝对路径**:**容器内文件夹的绝对路径**
        >
        > 注意win这边绝对路径要用反斜杠，容器那边要用斜杠
        >
        > :anger:注意不要用相对路径，两边都要用绝对路径
  
        `docker run -itd -v C:\User\ys\wintest:/test  --name new_container image_name` 
  
     2. 然后进入这个容器
  
        `docker exec -it new_container /bin/bash`
  
     3. 进入test文件夹
  
        `cd test` 
  
        `ls test`
  
        你就会发现你在本机的\wintest文件夹下的文件会与容器内的test文件夹同步，你可以通过在容器内操作test文件夹内的文件来直接操作本地\wintest文件夹内的文件，包括对文件的增删改查，都会进行同步。你在本地的wintest文件夹内的文件进行增删改查同样也会同步到容器内的test文件夹
  
     4. 注意一点，你的容器如果销毁了，是不会影响这个wintest里面的文件的
  
  8. #### 案例六：构建一个web服务容器
  
     - ##### 基本方法：直接使用别人的服务器镜像（来自Docker Hub）
  
       1. 为了方便演示我们直接拉一个别人已经做好的flask镜像
  
          `docker pull jcdemo/flaskapp`
  
       2. 然后使用这个镜像运行为容器并随机分配端口
  
          `docker run --name flaskserver -P jcdemo/flaskapp`
  
       3. 查看启动情况`docker ps `
  
          > CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
          > 3c6582469c64        jcdemo/flaskapp     "python /src/app.py …"   11 seconds ago      Up 13 seconds       0.0.0.0:32770->5000/tcp   great_poincare
  
          可以看到docker已经把本地的32770端口绑定到了容器的5000端口上，也就是flask服务的端口上
  
       4. 在浏览器中输入`localhost:32770`可以检查服务是否成功，不出意外你会得到下列信息
  
          | Start Time     | 2020-Nov-18 08:51:21 |
          | -------------- | -------------------- |
          | Hostname       | 998be9824821         |
          | Local Address  | 172.17.0.3           |
          | Remote Address | 172.17.0.1           |
          | Server Hit     | 1                    |
  
     - ##### 进阶方法：改造别人的镜像，定制化我们的flask服务
  
       1. 还是要先把别人的flask镜像拉下来，当然你不嫌麻烦可以自己从ubuntu开始构建
  
          `docker pull jcdemo/flaskapp`
  
       2. 然后我们去DockerHub看看这个镜像的Dockerfile
  
          ~~~dockerfile
          # python:alpine is 3.{latest}
          FROM python:alpine 
          LABEL maintainer="Jeeva S. Chelladhurai"
          RUN pip install flask
          COPY src /src/
          EXPOSE 5000
          ENTRYPOINT ["python", "/src/app.py"]
          ~~~
  
       3. 如果直接用docker run的话就是启动app.py了，但是我想把这个容器当成一个我的app文件定制
  
          `docker run --itd --name myflask --entrypoint /bin/sh -p 5000:5000 jcdemo/flask`
  
          这里的-itd和name参数不再赘述。
  
          说一下这个entrypoint参数，这个参数的出现会使得原来的Dockerfile里面的ENTRYPOINT命令整行失效，不再是运行`python /src/app.py`而是运行`/bin/sh`，有兴趣的话你可以另起一个终端输入`docker top myflask`就可以看看这个容器此时运行了什么程序，你会发现没有app.py运行，而是/bin/sh程序在运行
  
       4. 因为有`/bin/sh`的阻塞，这个容器有这个进程的存在，所以不会被关闭，我们可以进入这个容器
  
          `docker exec -it myflask /bin/sh`
  
          然后就可以在在这里查看这个容器的程序，以及对里面的文件进行改造，配合案例五中的挂载可以轻易地对这个app.py进行改造升级
  
  
  
  
- ### 容器踩坑记录

   1. #### 大坑一：奇怪的WORKDIR

      本来准备写一个flask的web应用，在写dockerfile构建镜像过程时候，发现这个镜像启动的容器可以看到程序运行，你可以`docker top ur_container`来看容器内部的进程，发现`python app.py`确实在运行啊，可是一直都无法请求成功，试了好多的办法都不行，于是我看看别人是怎么写dockerfile的

      以下是`jcdemo/flaskapp`的Dockerfile

      ~~~dockerfile
      FROM python:alpine 
      
      LABEL maintainer="Jeeva S. Chelladhurai"
      
      RUN pip install flask
      
      COPY src /src/
      
      EXPOSE 5000
      
      ENTRYPOINT ["python", "/src/app.py"]
      ~~~

      以下是我写的Dockerfile，**注意这是错误示范**

      ~~~dockerfile
      FROM jcdemo/flaskapp
      
      RUN mkdir project\
      && pip install requests\
      && pip install pymongo
      
      WORKDIR /project  # 这里提前确定了工作目录而且又cd进了/project
      
      COPY . .  # 这里又把文件从本地拉进当前目录，那到底是拉到了/还是拉到/project下呢？
      
      EXPOSE 5001 
      EXPOSE 5002
      
      ENTRYPOINT ["python", "app.py"]  # 一开始从这里看的，我也以为已经相当于进入/project下了
      ~~~

      所以注意到奇怪的地方了么，`WORKDIR`和`COPY`发生了歧义，改成下面之后就莫名其妙好了，服务也可以顺利请求了。这并不是一个寻常的错误（Google不到），是我找了好久自己发现的。
      **注意这才是正确示范**

      ~~~dockerfile
      FROM python:3.7
      
      RUN mkdir project\
      && pip install --upgrade pip\
      && pip install flask\
      && pip install requests\
      && pip install pymongo
      
      COPY . /project/
      
      EXPOSE 5001 
      EXPOSE 5002
      
      WORKDIR /project
      
      ENTRYPOINT ["python", "app.py"]
      ~~~

      ##### 结论就是，`WORKDIR`最好是在`ENTRYPOINT`或者`CMD`的前面一条使用，`COPY`等文件操作最好还是使用容器的绝对路径，而且记得放在`WORKDIR`的前面。（也不知道为啥错误情况下app.py也能跑起来，反正我这样改确实好起来了）

   2. #### 大坑二：胖胖的基础镜像

      这里提个小建议，我们最好在使用Dockerfile构建容器的时候，先把你需要的基础镜像pull到本地。

      可以通过`docker images`命令来查看本地有哪些容器

      **REPOSITORY          TAG                 IMAGE ID               CREATED             SIZE**

      ###### python              3.7                     1297140c6dd2        2 weeks ago       874MB

      ###### python              3.7-slim            af9cfe515812         5 days ago           112MB

      ###### python              3.7-alpine        6b73b71fd64e        2 weeks ago       41.1MB

      如果使用python基础镜像的话，有三种选择，但是完整的python:3.7竟然快有一个g那么大了，再安装点别的包就轻松过1g了。DockerHub的python官方对这两个缩减版本有一些基本的说明

      1. python: \<version\>-slim

         This image does not contain the common packages contained in the default tag and only contains the minimal packages needed to run `python`.

      2. python:\<vresion\>-alpine

         To minimize image size, it's uncommon for additional related tools (such as `git` or `bash`) to be included in Alpine-based images. Using this image as a base, add the things you need in your own Dockerfile

      结论就是推荐使用`python:3.7-alpine`作为dockerfile的基础镜像

   

- ### 容器重利用

   1. 使用`docker commit`然后再`docker save`（非常不推荐）

      也就是先把这个容器运行起来，你再用`docker exec -it /bin/sh`进到容器的命令行终端，自己通过命令行去手动安装requirements，最后把服务跑起来。我推荐你这样去测试容器和宿主机的连接或者一些服务的测试，但是不推荐你在构建好之后直接把这个容器通过`docker commit`来变成你想要移植到别的地方的容器

   2. 使用Dockfile进行构建（推荐使用）
   
      结合上面提到的，使用简单的基础镜像，在dockerfile中用`RUN`等命令把整个requirements的安装过程写下来，如果需要用到外部的文件就用`COPY`命令。
   
      使用Dockerfile构建的镜像不但把整个依赖的安装做到了一目了然，而且还可以使用`CMD`和`ENTRYPOINT`命令指定容器启动的时候需要跑什么程序（注意Docker官方推荐我们每一个容器只运行一个程序），最关键的是，当这个容器需要移植的时候，你只需把包含Dockerfile文件和requirement等文件的文件夹移植过去，再通过`docker build -t new_image_name .`就可以
