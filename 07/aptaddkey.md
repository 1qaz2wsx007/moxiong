# Apt list 添加公钥

在ubuntu新添加了软件源后，尝试apt-get update出现下面报错:

```bash
root@b6189b21ba4e:/etc/apt# apt-get update
Get:1 http://mirrors.aliyun.com/ubuntu bionic InRelease [242 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease [88.7 kB]
Err:1 http://mirrors.aliyun.com/ubuntu bionic InRelease    
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 3B4FE6ACC0B21F32
Get:3 http://mirrors.aliyun.com/ubuntu bionic-backports InRelease [74.6 kB]
Err:2 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease            
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 3B4FE6ACC0B21F32

```

核心原因是3B4FE6ACC0B21F32没有被系统认可，需要手动添加:

```bash
$ gpg --keyserver keyserver.ubuntu.com --recv 3B4FE6ACC0B21F32
gpg: directory '/root/.gnupg' created
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 3B4FE6ACC0B21F32: public key "Ubuntu Archive Automatic Signing Key (2012) <ftpmaster@ubuntu.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
$ gpg --export --armor 3B4FE6ACC0B21F32|apt-key add -
OK
$ apt-get update
Get:1 http://mirrors.aliyun.com/ubuntu bionic InRelease [242 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease [88.7 kB]
...SNIP...                       
Reading package lists... Done
```

