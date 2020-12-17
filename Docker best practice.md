# Docker best practice

### MySql as a container

1. 拉取镜像

   `docker pull mysql`

2. 检查镜像

   `docker images`

3. 创建容器（根据Dockerhub上面的mysql官方推荐启动方式）

   `docker run -d -p 8888:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 mysql`

4. 检查容器是否运行

   `docker ps`

5. 检查`mysql`容器的端口是否成功映射

   `docker port mysql `
   `docker inspect mysql` （可以查看一个容器运行的所有详细信息）

6. 使用命令行操作容器（运行容器内的bash）

   `docker exec -it mysql bash`

7. 进行一些简单的操作

   `create database test
   use test`

   ```sql
   create table test(
   id int unsigned primary key not null auto_increment,
   name varchar(10) not null
   );
   ```

   `describe test`你能看到

   ```mysql
   +-------+--------------+------+-----+---------+----------------+
   | Field | Type         | Null | Key | Default | Extra          |
   +-------+--------------+------+-----+---------+----------------+
   | id    | int unsigned | NO   | PRI | NULL    | auto_increment |
   | name  | varchar(10)  | YES  |     | NULL    |                |
   +-------+--------------+------+-----+---------+----------------+
   2 rows in set (0.00 sec)
   ```

### Things-Board as container

1. 拉取最新的镜像

   使用的事官网推荐的整合数据库的镜像

   > [thingsboard/tb-postgres](https://hub.docker.com/r/thingsboard/tb-postgres/) - single instance of ThingsBoard with PostgreSQL database.
   >
   > Recommended option for small servers with at least 1GB of RAM and minimum load (few messages per second). 2-4GB is recommended.

   `docker pull thingsboard/tb-postgres`

2. 新建两个文件夹

   `mkdir mytb-data`

   `mkdir mytb-log`

3. 把官网的启动配置文件复制下来作为`docker-compose.yml`

   ```yaml
   version: '2.2'
   services:
     mytb:
       restart: always
       image: "thingsboard/tb-postgres"
       ports:
         - "8080:9090"
         - "1883:1883"
         - "5683:5683/udp"
       environment:
         TB_QUEUE_TYPE: in-memory
       volumes:
         - ./mytb-data:/data
         - ./mytb-logs:/var/log/thingsboard
   ```

4. 可以进行访问

   `localhost:8080`

5. 当你不想用了可以直接`Ctrl + C`

6. 如果想把这个容器运行在后台（detach模式）

   `docker-compose up -d`

   不想用了

   `docker-compose stop`

   