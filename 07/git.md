# git 基础命令

配置本地用户名和邮箱:

```bash
$ git config --global user.name 'zzzzzzz'
$ git config --global user.email 'a@b.c'
$ git commit -m "add apt*gpg tips"
[master (root-commit) c596383] add apt*gpg tips
 1 file changed, 35 insertions(+)
 create mode 100644 07/aptaddkey.md
$ git push
Username for 'https://github.com': 1qaz2wsx007
Password for 'https://1qaz2wsx007@github.com': 
Counting objects: 4, done.
Delta compression using up to 16 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 910 bytes | 910.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To https://github.com/1qaz2wsx007/moxiong.git
 * [new branch]      master -> master

```