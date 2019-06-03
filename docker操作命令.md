## 一, ==docker操作==

- ##### 1, 安装docker

  - 进入源码docker文件中

  - 执行命令

    - sudo apt-key add gpg

    - sudo dpkg -i docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb


- ##### 2,拉取云端镜像


  - sudo docker image  pull  hello-world

  
- ##### 3,检查docker是否安装成功

  - sudo docker run hello-world

  - 显示: hello from Docker! 表示已经安装成功

    > 提示1: 如果不想每次都输入sodu, 执行, sudo usermod -a -G docker $USER
    >
    > 注销, 或者重启ubuntu

- ##### 4,启动,停止docker服务

  - 启动
    - sudo service docker start
  - 停止
    - sudo service docker stop
  - 启动docker
    - sudo service docker restart

  

- ##### 5,列出所有docker镜像

  - docker image ls
  - 或者 docker images

  

- ##### 6,加载本地镜像

  - docker load -i  本地镜像包

- ##### 6.1,拉取云端镜像


- sudo docker image  pull  镜像包


- ##### 7,删除镜像

  - docker image rm 镜像名或id

  

- ##### 8,使用镜像,创建交互式容器

  - docker run [option] 镜像 [传入的命的命令]
  - 例如:
    - docker run -it --name=myubuntu ubuntu 
    - 解释: 使用ubuntu镜像创建交互式容器, 名字为myubuntu
    - 退出容器: ctrl +d

  

- ##### 9,进入创建的容器

  - docker exec -it myubuntu /bin/bash
  - 注意点: 容器需要启动
  - 守护进程创建容器:
    - docker exec -==d==it myubuntu /bin/bash
    - 退出之后,不会停止容器

  

- ##### 10,查看容器

  - 查看所有运行的容器
    - docker container ls
  - 查看运行和停止的容器
    - docker container ls -all
    - 或者: docker ps -a

  

- ##### 11,停止运行容器

  - 运行容器
    - docker container start 容器名或id
  - 停止容器
    - docker container stop 容器名或id
    - docker container kill 容器名或id

- ##### 12,删除容器

  - docker container rm 容器名或id

  

  ## 二, ==使用Docker安装FastDFS==

  

  - ##### 1, 获取fastdfs镜像

    - 远程下载镜像
      - docker image pull delron/fastdfs
    - 本地镜像加载
      - docker load -i 文件路径/fastdfs_docker.tar

  - ##### 2,创建tracker容器

    ```html
    docker run -dti --network=host --name tracker -v /var/fdfs/tracker:/var/fdfs delron/fastdfs tracker
    ```

    - 解释: 将delron/fastdfs目录映射到/var/fdfs/目录中,创建名字为tracker的容器

      

  - ##### 3,运行tracker容器

    - docker container start tracker

      

  - ##### 4,创建storage容器

    ```
    docker run -dti --network=host --name storage -e TRACKER_SERVER=172.16.12.134:22122 -v /var/fdfs/storage:/var/fdfs delron/fastdfs storage
    ```

    - 解释: 将delron/fastdfs镜像目录, 映射到/var/fdfs/, 创建容器的名字为storage, 绑定追踪服务器的地址

      

  - ##### 5,运行storage容器

    - docker container start storage

      > 如果不能运行删除`/var/fdfs/storage/data`**目录下的**`fdfs_storaged.pid`**文件

    

    

    

    

  

  

  

  

  

  

  

  