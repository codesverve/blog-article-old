# Ubuntu shadowsocks 客户端代理记录（本地端用ubuntu setting替代chrome浏览器插件）
-------------

基于Ubuntu 18.04测试，这里记录的是客户端使用代理的方式，服务端搭建代理服务器的移步

## 安装
-----------

* sudo apt-get update
* sudo apt-get install python-pip
* sudo apt-get install shadowsocks
* sudo vim /etc/shadowsocks.json

输入以下内容后保存
 
>
> {
>     "server":"xx.xx.xx.xx", // 远程服务器地址
>     "server_port":xxxx, // 远程服务器端口
>     "local_address": "127.0.0.1", // 本机地址
>     "local_port":1080, // 本机用于代理的端口
>     "password":"xxxxx", // 密码
>     "timeout":300, // 超时
>     "method":"aes-256-cfb", // 加密方式
>     "fast_open": true,
>     "workers": 1
> }

sudo sslocal -c /etc/shadowsocks.json -d start      启动服务（以后可以通过这个命令启动socks服务）

## chrome浏览器代理
-----------

打开操作系统的设置 -> 网络 -> 网络代理，选择手动代理，只需填两行，Socks主机：127.0.0.1  1080，忽略主机：localhost, 127.0.0.0/8, ::1，忽略主机项表示不经过代理的网站，还可以逗号补上不想被代理的地址，如：www.baidu.com等。这时浏览器就能通过代理加速访问需要访问的网站。

## 命令行终端代理
-----------

前面的设置下，命令行终端并没有经过代理。需要经过以下步骤进行：

* git clone https://github.com/rofl0r/proxychains-ng.git
* cd proxychains-ng
* ./configure
* make
* sudo make install
* sudo cp ./src/proxychains.conf /etc/proxychains.conf
* cd .. && rm -rf proxychains-ng       (根据需要选择删除原来的安装源码)
* sudo vim /etc/proxychains.conf       修改最后一行为：socks5  127.0.0.1 1080

这时候，命令行终端如果需要经过代理，可以在命令前面添加：sudo proxychains4 指令，如果不想输出proxychains的日志信息可以使用：sudo proxychains4 -q，如：

> 添加docker ppa时，由于网络问题，原来的命令
> sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
> 不能成功执行，这时命令修改成：
> sudo proxychains4 add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
> 使得该命令的网络经过代理，可以成功完成
>
> 还可以：
> sudo proxychains4 ping www.docker.com
> sudo proxychains4 apt-get update
> sudo proxychains4 -q apt-cache madison mysql-server       (-q 安静模式，不输出proxychains4的日志)
> sudo proxychains4 apt-get install mysql-server


参考自：https://blog.csdn.net/lee_j_r/article/details/54019691
主要不同点： chrome浏览器代理（修改理由：在没有代理之前，不能打开插件市场安装管理插件，因此修改了代理方式）
 
