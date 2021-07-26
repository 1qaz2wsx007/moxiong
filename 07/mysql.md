# mysql

获取mysql数据目录:

```bash
mysql> show global variables like "%datadir%";
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.01 sec)

```


备份数据库文件，并修改权限：

```bash
$ chown -R xx:xx mysql
```