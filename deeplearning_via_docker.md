## Docker深度学习环境

### 环境配置

据说编程初学者有一半会因为环境配置而选择放弃学习，为什么学编程的时候，我们一提到配置环境就头疼？因为这几乎是一门玄学，因为没有一个确定的安装流程可以保证你的安装环境不出错，因为每个人的电脑都不一样，软硬件环境都不一样，可能同样的步骤在我这可以顺利装好，但是到了另一台电脑就会bug满天飞

### 容器技术

- 容器与虚拟机的区别

  容器Container与管理程序虚拟化hypervisor virtualization，HV不一样

  1. 程序虚拟化通过**中间层**将一台或者多台**独立的机器**运行在宿主机的**物理硬件**上

  2. 容器直接运行在宿主机的**操作系统的内核**之上，因此也被称为**操作系统虚拟化**

- 局限性

  由于“客居”于操作系统上，容器只能运行与底层宿主主机相同或者相似的操作系统，如Windows无法在Ubuntu服务器上运行，这看起来确实是失去了一定的灵活性

- 优势

  1. 占用资源比虚拟机小得多，因为它并不需要虚拟层和管理层这些中间层，而是调用宿主机的操作系统的接口。这使得宿主机中可以运行更多的容器
  2. 上手快，容器的启动也快（大多数容器只需要不到一秒）
  3. 方便转移，如docker的容器就像一个盒子，里面可以装很多物件，如果需要这些物件可以直接将大盒子拿走，而不需要从盒子里一件一件取，如果把github理解成代码的托管中心，那docker hub就是系统及环境+代码程序的托管中心

### Docker 基本概念

Docker就是一个这样的工具，人们把自己**配置好的环境**打包成**镜像**（实际是由一系列指令构建而成，可以理解为是**容器**的源代码），我们可以从[Docker官网](<https://hub.docker.com/search?q=&type=image>)把它们pull下来，启动这个镜像后，生成一个容器，相当于有了别人搭建好的环境，我们只需要提供硬件支持就好。

image，镜像，是一个个配置好的环境。 container，容器，是image的具体实例。 image和container的关系，相当于面向对象中类与对象的关系

### 安装Docker

1. ##### 使用源码安装Docker（在终端中进入你的虚拟环境）

   添加公钥
   `sudo apt-key add gpg`

   安装docker 
   `sudo dpkg -i docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb`

   检查是否安装成功
   `sudo docker run hello-world`  只要出现Hello from Docker！表示安装成功

2. ##### 安装 Nvidia-docker

   因为原来的docker不支持GPU加速，所以英伟达单独做了一个docker，让docker镜像可以使用英伟达自家的GPU

   对于 Ubuntu 14.04/16.04/18.04, Debian Jessie/Stretch 系统可以在终端中用以下命令

   ~~~
   # If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
   docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
   sudo apt-get purge -y nvidia-docker
   
   # Add the package repositories
   curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
     sudo apt-key add -
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
     sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   sudo apt-get update
   
   # Install nvidia-docker2 and reload the Docker daemon configuration
   sudo apt-get install -y nvidia-docker2
   sudo pkill -SIGHUP dockerd
   
   # Test nvidia-smi with the latest official CUDA image
   docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
   ~~~

   关于更新的命令

   ~~~
   # On Ubuntu/Debian
   sudo apt upgrade
   ~~~

3. ##### 安装深度学习镜像

   这里使用的镜像是deepo是一款由中国人贡献的深度学习镜像，版本也很新。需要注意的是，你必须要把前面的docker和nvidia-docker安装好，然后通过命令

   `docker pull ufoym/deepo `

   就可以把各种框架都下载下来，但是比较大10G多，也可以只下一个框架，以tensorflow为例

   `docker pull ufoym/deepo:tensorflow `

   另外全框架版还有jupyter notebook版的镜像，注意python和cuda版本（根据显卡来定）

   `docker pull ufoym/deepo:all-jupyter-py36-cu100`
   
   如果你手上有直接的安装包也就是哪个rar结尾文件，那么你可以先cd到安装包所在的文件夹，然后
   
   `docker load -i xxxx.rar`

   然后通过命令来查看你安装好的镜像

   `docker images`

   

## 安装完成，开始Docker操作

1. ##### 容器的创建/查看/删除

   创建某个镜像的容器实例container，同一个镜像可以通过这种方式创建多个容器，加上it表示马上进入交互模式
   `docker run [-it] some-image` 

   查看当前所有的容器
   `docker ps` 

   查看所有的容器，包括运行的和没有运行的
   `docker ps -a `

   删除某个没有在运行的容器
   `docker rm container-id` 

   强制删除某个容器
   `docker rm container-id -f`  

2. ##### 容器的启动/进入/退出

   启动某个容器，加上-i表示直接进入交互模式
   `docker start [-i] container-id`

   如果在启动后再进入的话
   `docker attach container-id`

   退出交互模式

   1. 退出但保持容器运行：按ctrl + Q + P三个键（有时候不灵，多按几次）
   2. 退出并停止容器：ctrl + D

### Docker 的 Jupyter notebook服务：端口映射（力荐）

1. 确保你的**镜像**已经安装过了，也就是前面的

   `docker pull ufoym/deepo:all-jupyter-py36-cu100`

   查看是否有这个镜像

   `docker images`

2. 然后根据镜像创建**容器**

   ~~~shell
   sudo nvidia-docker run -it -p 7777:8888 --ipc=host -v /home/shcd/Documents/gby:/gby --name gby-notebook  90be7604e476
   ~~~

   其中：

   - `-it`为直接进入交互式 
   - `-p 7777:8888`是把主机的7777端口映射到容器的8888端口 
   - `-ipc=host`可以让容器与主机共享内存 - 
   - 还可以加一个`--name xxxxx`给容器定义一个个性化名字 
   -  `-v /home/shcd/Documents/gby:/gby`可以将主机上的/home/shcd/Documents/gby地址挂载到容器里，并命名为/data文件夹，这样这个文件夹的内容可以在容器和主机之间共享了。因为容器一旦关闭，容器中的所有改动都会清除，所以这样挂载一个地址可以吧容器内的数据保存到本地。 - `90be7604e476`则是你安装的jupyter镜像的id，可以在刚刚docker images命令后面查看，当然你也可以直接写全名`ufoym/deepo:all-py36-jupyter`

3. 创建容器后就之后，就可以启动jupyter notebook了

   ~~~shell
   jupyter notebook --no-browser --ip=0.0.0.0 --allow-root --NotebookApp.token= --notebook-dir='/gby'
   ~~~

   其中：

   - `--no-browser`即不通过浏览器启动
   - `--ip`指定容器的ip
   - `--allow-root`允许root模型运行 
   - `--NotebookApp.token`可以指定jupyter 登录密码，可以为空 
   - `--notebook-dir='/gby'`指定jupyter的根目录

4. 开启本地与服务器的端口映射，从而远程登陆jupyter:

   在本地机器上，执行如下命令
   `ssh username@host-ip -L 1234:127.0.0.1:7777`

   这样，可以将本地的1234端口，映射到服务器的localhost的7777端口（即你前面创建jupyter容器时候的指定的服务器端口） 这样，你在本地电脑的浏览器里输入'localhost:1234'，即可登录到服务器上的jupyter notebook了！

### 容器的备份

如果好不容易配置好的环境，突然学校服务器要重装，那么可以把配置好的环境备份一份，然后重新加载进来

在docker中，只需要配置相应的容器container即可，因为之前具体的配置都是在容器中完成的，而镜像是网上下载的，我们不需要准备

1. 先通过`docker ps`或者`docker ps -a`来查看要备份哪个容器的id，然后通过

  ~~~shell
  docker commit -p [your-container-id] [your-backup-name]
  ~~~

  来将id为your-container-id的容器创建成一个镜像快照

2. 接着，通过`docker images`就可以查看刚刚创建好的镜像快照了。然后通过：

   ~~~shell
   docker save -o [path-you-want-to-save/your-backup-name.tar]] [your-backup-name]
   ~~~

   把那个镜像打包成tar文件，保存到服务器上。 后面就可以把服务器上打包好的tar文件，下载到本地了。。

3. 恢复：

   ~~~shell
   docker load -i your-backup-name.tar docker run -d -p 80:80 your-backup-name
   ~~~

   
