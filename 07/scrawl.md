# Scrawl 

这次调整将架构分两部分：

```bash
1. docker_1->mysql
2. docker_2->scrawl
```

针对mysql，配置相对简单，第一次运行时，重置数据库，然后运行即可:

```bash
$ docker run --restart=always --privileged=true  \
-v /home/kumar/Documents/mysqltmp/mysql/data/:/var/lib/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/logs/:/var/log/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/conf/:/etc/mysql \
-v /home/kumar/Documents/mysqltmp/mysql/my.cnf:/etc/mysql/my.cnf  \
-v /home/kumar/Documents/mysqltmp/mysql/mysql-files:/var/lib/mysql-files \
-p 3306:3306 --name szdb \
-e MYSQL_ROOT_PASSWORD=123456 -d diymysql:1.0
```

针对scrawl，需要对key和程序目录进行挂载:


sudo docker run --privileged -d -p 8848:22 -v /sys/fs/cgroup:/sys/fs/cgroup:ro bluecatr/centos7-systemd-ssh

```bash
$ docker run --restart=always --privileged=true -d  \
-v /home/kumar/Documents/scrawltmp:/scrawl \
-v /sys/fs/cgroup:/sys/fs/crgroup \
-p 8848:22 \
--name szsw \
diyscrawl:1.3

```




因为我需要安装ss，而ss是要求systemctl是能够被使用的，所以需要安装systemctl然后，将其作为init进程：

```bash
$ apt-get install systemd
```

检查ss是否正常运行:

```bash
➜  /scrawl ps -ef|grep senseshield
root        92     1  0 00:58 ?        00:00:00 /usr/lib/senseshield/senseshield
root       594   155  0 01:26 pts/0    00:00:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox senseshield
```

检查elite key是否被识别：

```bash
$ lsusb               
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 008: ID 0951:1666 Kingston Technology DataTraveler 100 G3/G4/SE9 G2/50
Bus 001 Device 002: ID 17ef:608d Lenovo Optical Mouse
Bus 001 Device 009: ID 1bc0:0055 Beijing Senseshield Technology Co.,Ltd. Elite5 v3.x
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```