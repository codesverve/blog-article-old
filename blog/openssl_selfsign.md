# Openssl自签名证书及HTTPS

## 步骤

### 自建CA

vim /etc/pki/tls/openssl.cnf确认并修改配置(这里文件地址根据具体的来，Ubuntu下这个地址就不是对的)

```
dir             = /etc/pki/CA # CA的工作目录
database        = $dir/index.txt # 签署证书的数据记录文件
new_certs_dir   = $dir/newcerts # 存放新签署证书的目录
serial          = $dir/serial # 新证书签署号记录文件
certificate     = $dir/ca.crt # CA的证书路径
private_key     = $dir/private/cakey.pem # CA的私钥路径
```

`su root`

`cd /etc/pki/CA` # 切换到CA的工作目录

`(umask 077; openssl genrsa -out private/cakey.pem 2048)`   # 制作CA私钥

`openssl req -new -x509 -key private/cakey.pem  -out ca.crt` # 制作自签名证书

`touch index.txt && touch serial && echo '01'> serial`  # 生成数据记录文件，生成签署号记录文件，给文件一个初始号。

## 签名服务端证书

`(umask 077; openssl genrsa -out server.key 1024)` # 制作服务器端私钥

`openssl req -new -key server.key -sha512 -out server.csr`  # 制作服务器端证书申请指定使用sha512算法签名 （默认使用sha1算法）

 `openssl ca -in server.csr -out server.crt -days 36500`  # 签署证书

## 签名客户端证书

```
# 制作客户端私钥
(umask 077; openssl genrsa -out kehuduan.key 1024)

# 制作客户端证书申请
openssl req -new -key kehuduan.key -out kehuduan.csr 

# 签署证书
openssl ca -in kehuduan.csr -out kehuduan.crt -days 3650
```

## 注意事项

1、制作证书时会提示输入密码，设置密码可选，服务器证书和客户端证书密码可以不相同。

2、服务器证书和客户端证书制作时提示输入省份、城市、域名信息等，需保持一致。

3、以下信息根证书需要和客户端证书匹配，否则可能出现签署问题。

```
countryName = match
stateOrProvinceName = match
organizationName = match
organizationalUnitName = match
```



## Nginx

```
server {
        listen       443;
        server_name  pro.server.com;
        ssi on;
        ssi_silent_errors on;
        ssi_types text/shtml;

        ssl                  on;
        ssl_certificate      /data/server/nginx/ssl/self/server.crt;
        ssl_certificate_key  /data/server/nginx/ssl/self/server.key;
        ssl_client_certificate /data/server/nginx/ssl/self/ca/ca.crt;

        ssl_verify_client on;
        ssl_protocols    TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-RC4-SHA:!ECDHE-RSA-RC4-SHA:ECDH-ECDSA-RC4-SHA:ECDH-RSA-RC4-SHA:ECDHE-RSA-AES256-SHA:!RC4-SHA:HIGH:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!CBC:!EDH:!kEDH:!PSK:!SRP:!kECDH;
        ssl_prefer_server_ciphers On;

        index index.html index.htm index.php;
        root /data/www;
        location ~ .*\.(php|php5)?$
        {
                #fastcgi_pass  unix:/tmp/php-cgi.sock;
                fastcgi_pass  127.0.0.1:9000;
                fastcgi_index index.php;
                include fastcgi.conf;
        }
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
                expires 30d;
        }
        location ~ .*\.(js|css)?$
        {
                expires 1h;
        }
###this is to use open website lianjie like on apache##
        location / {
                if (!-e $request_filename) {
                        rewrite ^(.*)$ /index.php?s=$1 last;
                        break;
                }
                 keepalive_timeout  0;
        }
        location ~ /.svn/ {
        deny all;
        }
###end##
        include /data/server/nginx/conf/rewrite/test.conf;
        access_log /log/nginx/access/access.log; 
}
```





## 签名算法种类

通过openssl dgst -help查看

## 参考
[nginx实现https双向认证](https://blog.51cto.com/tchuairen/1782945)