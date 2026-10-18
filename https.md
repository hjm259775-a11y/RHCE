# https





https是什么

*   https 就是 http传输层中嵌入一个加密层，他比http传输额外多了通信加密功能，被称之为安全的http



ssl是什么

*   Secure Socket Layer 安全的传输加密层 专用于加密https中传输的数据





https的通信机制

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261018162937087.png" alt="image-20261018162937087" style="zoom:67%;" />



配合web服务器那边的操作

## 实现https服务



```
1.通过openssl工具创建ssl证书

1.1 生成证书用的秘钥文件
[root@server ~]# openssl genrsa -out lyh.key

1.2 通过秘钥创建证书
req 请求创建证书 -new 新的 -key 证书使用的秘钥文件 -x509 测试证书 -days 表示证书的有效期
[root@server ~]# openssl req -new -key lyh.key -x509 -days 365 -out lyh.crt

[root@server ~]# ls
anaconda-ks.cfg lyh.crt lyh.key


# 2.创建一个虚拟主机，这台虚拟主机启用https
[root@server ~]# vim /etc/nginx/conf.d/https.conf
——————————————————————————————————————————————————————————————————————————————————————————————
 1 server {
 2     listen 443 ssl http2;  # 443 ssl http2:表示启用https功能 http2版本的通信协议
 3     server_name 172.25.254.100;
 4     root /web;
 5     ssl_certificate /etc/nginx/lyh.crt; # 证书所在的绝对路径
 6     ssl_certificate_key /etc/nginx/lyh.key; # 证书的密钥文件
 7 }
——————————————————————————————————————————————————————————————————————————————————————————————

# 2.1 重新加载配置文件
[root@server ~]# nginx -s reload

# 2.3 在浏览器验证是否可访问
```





 



来一段

```
[root@xgz ~]# openssl genrsa -out xgz.key ——————————————————————————————————————————————————创建密钥
[root@xgz ~]# openssl req -new -key -x509 -days 365 -out xgz.crt ———————————————————————————通过秘钥创建证书
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Warning: Ignoring -days without -x509; not generating a certificate
Could not open file or uri for loading private key from -x509: No such file or directory
[root@xgz ~]# openssl req -new -key xgz.key -x509 -days 365 -out xgz.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:jiangsu
Locality Name (eg, city) [Default City]:nanjin
Organization Name (eg, company) [Default Company Ltd]:nailong
Organizational Unit Name (eg, section) []:https
Common Name (eg, your name or your server's hostname) []:xgz
Email Address []:242348535@qq.com
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

[root@xgz conf.d]# mv xgz.crt /etc/nginx/ ——————————————————————————————————————————————————移动到nginx文件里
[root@xgz conf.d]# mv xgz.key /etc/nginx/

[root@xgz conf.d]# vim /etc/nginx/conf.d/https.conf ————————————————————————————————————————创建虚拟主机
————————————————————————————————————————————————————————————————————————————
server{
        listen 443 ssl http2;
        server_name 192.168.44.99;
        root /root/web;
        index index.html;
        ssl_certificate /etc/nginx/xgz.crt;
        ssl_certificate_key /etc/nginx/xgz.key;
        location /ruoyizhibai/{
                root /root/web/;
        }
}
————————————————————————————————————————————————————————————————————————————
[root@xgz conf.d]# systemctl restart nginx.service

[root@xgz ruoyizhibai]# tree /root ——————————————————————————————————————————————————看一眼web什么构造
/root
├── anaconda-ks.cfg
├── web
│   ├── index.html
│   └── ruoyizhibai
│       └── haidi.html
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261018185528806.png" alt="image-20261018185528806" style="zoom:67%;" />

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261018185616439.png" alt="image-20261018185616439" style="zoom:67%;" />

























