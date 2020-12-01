# 多容器启动以及管理

## Docker Compose

compose是用来应以和运行多个容器containers的工具文件，可以使用一个YML文件来配置应用程序需要的所需，然后一个命令就从YML文件配置中启动所有的服务

- #### 什么是YAML

  yaml就是Yet Another Makeup Language(仍是一种标记语言)。使用空白符号缩进喝大量依赖外观的数据接口来表达数据结构，配置文件等，YAML文件可以是`.yaml`或者`yml`结尾

  ##### 基本语法

  1. 大小写敏感
  2. 使用缩进表示层级关系
  3. 缩进不允许使用TAB，只允许空格
  4. 所进的空格数不重要，只要相同层级的元素左边对齐即可
  5. `#`开头的行表示注释行

  ##### 数据类型

  1. 对象：键值对组合，类似py的dict
  2. 数组：类似py的list
  3. 纯量：也就是标量，单个的不可再分的值

  ###### 数据类型之对象

  对象键值对使用冒号结构表示，冒号后面要有一个空格。value也可以是一个对象，就像dict

  ~~~yaml
  key: 
  	child-key: value
  	child-key2: calue2
  ~~~

  与py的dict不同的是，对象的key可以是一个数组，value也可以是一个数组，也就是较为复杂的兑现格式，可以使用一个`?`加空格代表一个复杂的key，配合一个`:`加空格代表一个value

  ~~~yaml
  ? 
  	- complexkey1
  	- complexkey2
  : 
  	- complexvalue1
  	- complexvalue2
  ~~~

  代表的就是对象的属性是一个数组 [complexkey1,complexkey2]，对应的值也是一个数组 [complexvalue1,complexvalue2]

  ###### 数据类型之数组

  以`-`开头的的行，同级的表示构成一个数组，如下的ABC一起构成一个数组

  ~~~yaml
  - A
  - B
  - C
  ~~~

  YAML 支持多维数组，可以使用行内表示：

  ```yaml
  key: [value1, value2, ...]
  ```

  数组内的元素可以是对象的集合

  ```yaml
  companies:
      -
          id: 1
          name: company1
          price: 200W
      -
          id: 2
          name: company2
          price: 500W
  ```

  意思是companies的属性是一个数组，每一个数组又有三个属性构成，也可以写成如下形式

  ```yaml
  companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
  ```

  ###### 复合结构

  数组和对象可以构成复合结构

  ```yaml
  languages:
    - Ruby
    - Perl
    - Python 
  websites:
    YAML: yaml.org 
    Ruby: ruby-lang.org 
    Python: python.org 
    Perl: use.perl.org
  ```

  等价于下面的json结构

  ```json
  { 
    languages: [ 'Ruby', 'Perl', 'Python'],
    websites: {
      YAML: 'yaml.org',
      Ruby: 'ruby-lang.org',
      Python: 'python.org',
      Perl: 'use.perl.org' 
    } 
  }
  ```

  注意到了json必须要有双引号把值给括起来

  ###### 数据结构之标量

  使用一个简单的例子

  ```yaml
  boolean: 
      - TRUE  #true,True都可以
      - FALSE  #false，False都可以
  float:
      - 3.14
      - 6.8523015e+5  #可以使用科学计数法
  int:
      - 123
      - 0b1010_0111_0100_1010_1110    #二进制表示
  null:
      nodeName: 'node'
      parent: ~  #使用~表示null
  string:
      - 哈哈
      - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
      - newline
        newline2    #字符串可以拆成多行，每一行会被转化成一个空格
  date:
      - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
  datetime: 
      -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
  ```

## Docker-Compose入门

- #### 快速开始

  1. ##### 第一步：安装预备工作

     新建一个文件夹并进入如`dctest`，创建一个app.py文件

     ```python
     import time
     
     import redis
     from flask import Flask
     
     app = Flask(__name__)
     cache = redis.Redis(host='redis', port=6379)
     
     def get_hit_count():
         retries = 5
         while True:
             try:
                 return cache.incr('hits')
             except redis.exceptions.ConnectionError as exc:
                 if retries == 0:
                     raise exc
                 retries -= 1
                 time.sleep(0.5)
     
     @app.route('/')
     def hello():
         count = get_hit_count()
         return 'Hello World! I have been seen {} times.\n'.format(count)
     ```

     创建另一个文件requirements.txt

     ~~~txt
     flask
     redis
     ~~~

  2. ##### 第二步：创建一个Dockerfile文件

     ```dockerfile
     FROM python:3.7-alpine
     WORKDIR /code
     ENV FLASK_APP=app.py
     ENV FLASK_RUN_HOST=0.0.0.0
     RUN apk add --no-cache gcc musl-dev linux-headers
     COPY requirements.txt requirements.txt
     RUN pip install -r requirements.txt
     EXPOSE 5000
     COPY . .
     CMD ["flask", "run"]
     ```

     注意这里的copy .. 意思是把当前文件夹`dctest`的所有文件放到了容器的工作目录`/code`下

  3. ##### 第三步：在Compose文件中定义服务`services`

     在当前目录`dctest`下创建一个`docker-compose.yaml`文件

     ```yaml
     version: "3.8"
     services:
       web:
         build: .
         ports:
           - "5000:5000"
       redis:
         image: "redis:alpine"
     ```

  4. ##### 第四步：一行命令用compose文件build和run你的app

     命令行：`docker-compose up`

     然后你就可以在浏览器中输入网址`localhost:5000`就可以看到

     > ```
     > Hello World! I have been seen 1 times.
     > ```

     再刷新一次就可以看到数字变化

  5. ##### 第五步：修改Compose文件添加共享文件夹

     > 因为有时候我们会用容器作为开发环境，但是没改一次都要重新build一个新的容器确实是太麻烦了，下面我们实现在容器中更改代码并实时让代码更新到flask应用中去

     在compose文件中新增挂载文件说明，并设置变量告诉flask我们是在用开发环境，使得flask允许app实时根据代码的修改更新。需要注意的是，这种方法必须是你用命令行flask run来运行而不是用python代码app.run()方法

     ```yaml
     version: "3.8"
     services:
       web:
         build: .
         ports:
           - "5000:5000"
         volumes:
           - .:/code
         environment:
           FLASK_ENV: development
       redis:
         image: "redis:alpine"
     ```

  6. ##### 第六步：重新新建这个app

     命令行：`docker-compose up`

  7. ##### 第七步：更新这个app的代码

     比如你把app.py文件中`/`视图函数的返回值改一下，变成hello from Docker，然后直接在浏览器中刷新一下，看看这个改动有没有实时更新。

  8. ##### 第八步：实验使用更多的参数

     保持容器们在后台运行：`docker-compose up -d`

- #### Compose详细介绍

  - ##### compose使用的三个步骤

    1. 使用Dockerfile定义应用程序的环境
  2. 使用docker-compose.yml定义构成应用程序的服务，这样他们可以在隔离的环境中一起运行
    3. 最后使用`docker-compose up`命令来启动并运行整个应用程序

  - ##### version（top key）
  
    指定yml文件依从的compose的哪个版本来制定的，一般都放在首行
  
  - ##### app（name of a application，can be named whatever you want）
  
    这个名字在你用`docker-compose logs -f `的时候可以把容器进行区分
  
  - ##### image（under app key）
  
    说明这个服务的来源，app下build或者image必须有有一个作为app的来源
  
    ~~~yaml
    web:
      image: python:3.7-alpine
    ~~~
  
  - container_name（under app key）
  
    可以指定这个容器的名字
  
    ~~~yaml
    web:
      container_name: my-web-container
    ~~~
  
  - ##### environment（under app key）
  
    如果你想在compose文件中向容器中添加环境变量，你可以如下格式书写
  
    ```yaml
    web:
      environment:
        - DEBUG=1
    ```

    这有点类似`docker run -e VARIABLE=VALUE`

    > 如果你不给这个`DEBUG`赋值的话，那容器内的`DEBUG`的值就会取当前的这个终端的同名环境变量的值

    
  
  - ##### build（属于在服务名的下级）
  
    类似于运行`docker build`过程中要制定的参数，比如你要指向Dockerfile的位置。你可以指定包含了Dockerfile的具体文件夹（假设这个要新建的镜像名为webapp）
  
    ```yaml
    version: "3.8"
    services:
      webapp:
        build: ./dir
    ```
  ```
  
  也可以指定具体的Dockerfile，此时的build对象的值就是一个以对象为元素的数组
  
    ```yaml
  version: "3.8"
    services:
      webapp:
        build:
        context: ./dir
          dockerfile: Dockerfile-alternate
  ```
  
  1. ##### build下的参数context
  
     作为build数组内的一个对象的键，对应的值为一个包含了Dockerfile的文件夹或者是一个git的仓库。路径支持相对路径，但是以docker-compose.yaml为对照的相对路径
    
  2. ##### build下的参数ARGS
  
   作为docker-copmose.yaml文件向Dockerfile传递参数的方法，首先你需要在Dockerfile里面定义了这个参数
  
   ~~~dockerfile
       ARG buildno
     ARG gitcommithash
       
     RUN echo "Build number: $buildno"
       RUN echo "Based on commit: $gitcommithash"
   ~~~
  
     然后你再在build下面添加args参数（注意在参数中如果含有布尔值的话，就把整个元素的内容包在双括号里面以保证正常工作）
  
     ```yaml
       build:
       context: .
         args:
       - buildno=1
           - gitcommithash=cdc3b19
     ```
  
    3. ##### build下的参数cache_from
    
       你可以使用一些已有的镜像来作为构建的新镜像的基础，这样就可以不用什么都重新下载了
    
       ```yaml
        build:
       context: .
       cache_from:
         - alpine:latest
           - corp/web_app:3.14
       ```
     ```
  
  
  
  ## Docker-Compose案例之更新部署
  
     ```

比如说你对生产环境compose文件中的某个服务，比如说是app，你想修改里面的代码，你要怎么做？

###### 做法一：效仿本文的`Compose快速开始`里面提到的，直接在生产环境挂载本地目录实时更改

  ###### 做法二：先在开发环境里面创建新的compose，在里面进行实时修改，改好了再去生产环境部署

  显然第二种做法是更符合安全要求的，但是有个问题，更新部署的时候，如果其他服务需要很依赖的话，那重新build全部服务不就会浪费很多时间？可以只build这个更新的app服务吗？

  使用以下命令可以只针对`app`这个作了修改的服务/镜像进行更新而不必重建compose其他的镜像

  ~~~cmd
$ docker-compose build app
  $ docker-compose up --no-deps -d app
  ~~~



  ## Docker-Compose案例之network

  由于compose文件会同时起多个容器，容器之间的网络通信该怎么使用呢？最笨的方法就是让每个容器分别跟宿主机绑定端口，然后再利用宿主机的ip:port来进行信息传输，但是这样很麻烦

  缺点一：你在代码里面写好了宿主机的ip万一换了一台机器进行部署的时候就会出现问题

缺点二：会占用宿主机的端口，而且有可能因为绑定的宿主机的端口暴露而被别人进行数据库攻击

所以compose文件建立一个只允许容器之间交流的内部网络

  - #### default网络
  
    Compose 默认给你的 app 服务设置一个网络（没有network命令指定也是有的）。 service 中的每个容器默认都加入这个网络，容器之间彼此是互通的。并且，可以利用容器名字识别到
  
    > 这个默认的网络的名字就是你的compose文件所在的文件夹名（项目名）决定的，比如你的文件夹名`db`，那么这个默认的网就叫`db_default`，当然如果你不想用文件夹名来做你的项目名，那你就在`docker-compose up`添加`-p`参数指定你的项目名称

  ```yaml
    version: "3"
  services:  
      web:    
      build: .    
        ports:      
          - "8000:8000"  # 把宿主机的8000端口映射到容器的8000端口
      db:    
        image: postgres    
        ports:      
          - "8001:5432"  # 把宿主机的8001端口映射到容器的5432端口
  ```

    假设你的项目文件夹（compose文件所在的文件夹）名为`myapp`，那么当`docker-compose up -d`运行后就会有一个`myapp_default`的网络，然后
    
    1. 一个使用`web`配置创建的容器会被创建，他以`web`这个hostname加入到`myapp_default`内部网络中
    2. 一个使用`db`配置创建的容器会被创建，他以`db`这个hostname加入到`myapp_default`内部网络中
      
    现在，**`myapp_default`网络内的任一容器都可以通过hostname（也就是服务名）找到`web`和`db`**，例如，`web` 应用的代码可以使用 URL `postgres://db:5432` 连接数据库并使用它。再例如，`web`应用的py代码可以使用`cache = redis.Redis(host='redis', port=6379)`连接redis数据库
      
    > 注意搞清楚`db`服务，内部容器可以直接使用`db`作为hostname和5432作为内部网络的端口访问，
    >
    > 而外部宿主机想要访问这个数据库却需要使用8001端口号

  - #### 自定义网络
  
    默认的网络是只要在同一个compose文件下的容器都可以使用，但是有时候为服务之间想要保持网络的独立性，比如需求：B服务可以连接A服务或者C服务，但是A只能连接B服务，C也只能连接B服务。这种情况很常见，比如B服务就是后端，A服务就是前端，C服务就是数据库。这种情况下AC服务实际上是不需要有连接的
  
    比如下面这个例子用到的两个相对独立的网络`frontend`和`backend`
  
    ```yaml
    version: '3.8'
    services:  
      proxy:    
        build: ./proxy  # 指向proxy内的Dockerfile   
        networks:      
          - frontend  
          
      app:    
        build: ./app    # 指向app内的Dockerfile
        networks:      
          - frontend      
          - backend  
        
      db:    
        image: postgres    
        networks:      
          - backend
          
    networks:  
      frontend:  
      backend:
    ```

  

  ## Docker-Compose案例之多文件使用

  在现实中，有时候dev是一套配置，prod又是另一套配置了，可以使用多个compose文件同时启动

  ###### 基础compose：docker-compose.yml

  ```yml
  web:
    image: example/my_web_app:latest
    depends_on:
      - db
      - cache
  
  db:
    image: postgres:latest
  
  cache:
    image: redis:latest
  ```

  ###### 测试/开发环境compose：docker-compose.override.yml

  ```yaml
  web:
    build: .
    volumes:
      - '.:/code'
    ports:
      - 8883:80
    environment:
      DEBUG: 'true'
  
  db:
    command: '-d'
    ports:
      - 5432:5432
  
  cache:
    ports:
      - 6379:6379
  ```

  当你使用`docker-compose up`时候，这个override会被自动读取而不是读`docker-compose.yml`

  ###### 生产环境compose：docker-compose.prod.yml

  ```yaml
  web:
    ports:
      - 80:80
    environment:
      PRODUCTION: 'true'
  
  cache:
    environment:
      TTL: '500'
  ```

  当你想用这个prod的compose的时候，你可以使用命令
  `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d`当然如果你觉得太麻烦你可以直接在prod的compose中写完所有的东西，然后只启动一个compose文件

  