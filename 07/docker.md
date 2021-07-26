# docker

docker 出现无限重启的情况:

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                          PORTS     NAMES
64728d075e7e   mysql     "docker-entrypoint.s…"   14 minutes ago   Restarting (1) 59 seconds ago             my-mysql
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                         PORTS     NAMES
64728d075e7e   mysql     "docker-entrypoint.s…"   15 minutes ago   Restarting (1) 7 seconds ago             my-mysql
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                         PORTS     NAMES
64728d075e7e   mysql     "docker-entrypoint.s…"   15 minutes ago   Restarting (1) 8 seconds ago             my-mysql
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                         PORTS     NAMES
64728d075e7e   mysql     "docker-entrypoint.s…"   15 minutes ago   Restarting (1) 8 seconds ago             my-mysql

```

通过inspect命令检查容器状态:

```bash
➜  mysql sudo docker inspect  my-mysql
[
    {
        "Id": "64728d075e7e8305d631a143e663d0565be59832c50aabfd60a1a8f32ca6784d",
        "Created": "2021-07-19T00:22:42.270322433Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "mysqld"
        ],
        "State": {
            "Status": "restarting",
            "Running": true,
            "Paused": false,
            "Restarting": true,
            "OOMKilled": false,  # 并没有看出任何错误
            "Dead": false,
            "Pid": 0,
            "ExitCode": 1,
            "Error": "",
            "StartedAt": "2021-07-19T00:40:43.175222516Z",
            "FinishedAt": "2021-07-19T00:40:43.476235759Z"
        },
...SNIP...
```

只能通过logs检查日志:

```bash
$ docker logs test

2021-07-19T00:45:46.150867Z 0 [ERROR] [MY-010119] [Server] Aborting
2021-07-19 00:46:46+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.25-1debian10 started.
2021-07-19 00:46:46+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2021-07-19 00:46:46+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.25-1debian10 started.
mysqld: Error on realpath() on '/var/lib/mysql-files' (Error 2 - No such file or directory)
2021-07-19T00:46:46.713752Z 0 [ERROR] [MY-010095] [Server] Failed to access directory for --secure-file-priv. Please make sure that directory exists and is accessible by MySQL Server. Supplied value : /var/lib/mysql-files
2021-07-19T00:46:46.713764Z 0 [ERROR] [MY-010119] [Server] Aborting
2021-07-19 00:47:47+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.25-1debian10 started
```

发现果然，是缺少了/var/lib/mysql-files，重新进行文件的挂载：

```
sudo docker run --restart=always --privileged=true  \
-v /home/kumar/Documents/mysqltmp/mysql/data/:/var/lib/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/logs/:/var/log/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/conf/:/etc/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/my.cnf:/etc/mysql/my.cnf  \
-v /home/kumar/Documents/mysqltmp/mysql/mysql-files:/var/lib/mysql-files \
-p 3306:3306 --name test \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql
```


尝试将程序迁移至docker环境中，将配置好的docker文件生成容器：

```bash
$ docker commit -a 'diymysql' -m 'add oh-my-zsh net-tools' 80c919c42526 diymysql:1.0
sha256:21148f1d84f13943b0db26e9be785de37c67fef2030dfc3a130079528c7a8245
$ docker images
REPOSITORY           TAG            IMAGE ID       CREATED         SIZE
diymysql             1.0            21148f1d84f1   7 seconds ago   783MB
ubuntu               latest         c29284518f49   5 days ago      72.8MB
mysql                latest         5c62e459e087   3 weeks ago     556MB
skysider/pwndocker   latest         65d4c8b24db6   2 months ago    3.34GB
phusion/baseimage    master-amd64   0f3e713d1865   2 months ago    224MB
```

分析容器运行时运行的第一条命令:

```bash
$ docker history mysql --no-trunc
...SNIP...

$ docker history mysql           
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
5c62e459e087   3 weeks ago   /bin/sh -c #(nop)  CMD ["mysqld"]               0B        
<missing>      3 weeks ago   /bin/sh -c #(nop)  EXPOSE 3306 33060            0B        
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      3 weeks ago   /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B       
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY file:345a22fe55d3e678…   14.5kB    
<missing>      3 weeks ago   /bin/sh -c #(nop) COPY dir:2e040acc386ebd23b…   1.12kB    
<missing>      3 weeks ago   /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B        
<missing>      3 weeks ago   /bin/sh -c {   echo mysql-community-server m…   420MB     
<missing>      3 weeks ago   /bin/sh -c echo 'deb http://repo.mysql.com/a…   55B       
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENV MYSQL_VERSION=8.0.25-…   0B        
<missing>      3 weeks ago   /bin/sh -c #(nop)  ENV MYSQL_MAJOR=8.0          0B    
```

容器导出：

```bash
$ docker export e87b271a6d35 >~/Desktop/diyscrawl.tar
$ docker export abe991e0881d >~/Desktop/diymysql.tar
```

容器导入：

```bash
cat yuqing-dev.tar | docker import – yuqing
```