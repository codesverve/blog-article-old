# shadowsocks docker 镜像实现代理 - 客户端（Ubuntu电脑访问服务器使用代理）
===============

基于Ubuntu 18.04测试，这里记录的是客户端使用代理的方式，服务端搭建代理服务器的移步
docker 安装见 https://blog.csdn.net/Vincent_Field/article/details/82086197

## 镜像获取
-------------

docker pull uetty/shadowsocks:client

## 使用方法
-------------
### 初次启动

> 1. 宿主机上运行启动容器命令：
>    docker run --net=host --name=socks -t uetty/shadowsocks:client
>    ctrl + c 退出
> 2. 进入容器内部：
>    docker exec -it socks bash
>    修改 /etc/shadowsocks.json 文件(vim /etc/shadowsocks.json)：
>    按i键编辑文件
>    修改服务器地址、端口、密码、以及加密方式(method)
>    保存(依次：ctrl + c  ->  输入:wq  ->  回车)
> 3. 在容器内部运行命令：
>    sslocal -c /etc/shadowsocks.json -d start
>    退出容器，先后按下：
>    ctrl + p     ctrl + q
> 4. 在宿主机上操作：
>    依次进入：设置 -> 网络 -> 网络代理 -> 手动 -> socks主机那行：
>    主机地址填写： 127.0.0.1 端口填写： 1080 忽略主机行填写（默认值）： localhost, 127.0.0.0/8, ::1
>    关闭设置
在浏览器上尝试是否代理成功

### 关闭代理方式（第二步根据需要选择）

> 1. 将网络代理手动重新改为禁用
> 
> 2. 进入容器：
>    docker exec -it socks bash
>    执行停止命令：
>    sslocal -c /etc/shadowsocks.json -d stop

### 重启电脑后重新启动代理

> 1. docker start socks
> 
> 2. docker exec -it socks bash
> 
> 3. 此时已进入容器中，执行：
>    sslocal -c /etc/shadowsocks.json -d start
> 
> 4. 退出容器，设置中设置手动代理

### 命令行下使用代理

> 1. 在宿主机上执行（从container中将已准备好的工具拷贝出来）：
>    docker cp socks:/data/proxychains-ng.tar ~/data/proxychains-ng.tar
>    拷贝出来的位置随意自己决定
> 
> 2. 解压
>    cd ~/data
>    tar xf proxychains-ng.tar
> 
> 3. 安装
>    ./configure
>    make
>    sudo make install
>    sudo cp ./src/proxychains.conf /etc/proxychains.conf
>    cd .. && rm -rf proxychains-ng
> 
> 4. 修改配置
>    sudo vim /etc/proxychains.conf
>    修改最后一行为：
>    socks5 127.0.0.1 1080
> 
> 5. 使用
>    在需要代理的命令前增加：sudo proxychains4
>    如：
>    sudo proxychains4 ping www.docker.com
>    或
>    sudo proxychains4 -q ping www.docker.com


